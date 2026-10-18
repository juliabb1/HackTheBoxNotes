---
tags: [technique, recon, windows, smb]
---
# SMB Enumeration

> SMB (Ports 139/445) = Windows-Dateifreigaben. Oft anonym („null session") durchsuchbar → Shares, Dateien, teils Credentials.

## Shares auflisten
```bash
smbclient -L //<ziel>/ -N            # anonym (kein Passwort)
smbclient -L //<ziel>/ -U user       # mit User
```

## Share betreten / Dateien
```bash
smbclient //<ziel>/share -N
# > ls / get <datei> / put <datei>
```

## Breiter enumerieren
```bash
nxc smb <ziel> -u '' -p '' --shares   # netexec: Shares + Rechte
enum4linux-ng <ziel>                  # User, Gruppen, Shares, Policy
```

## Worauf achten
- **beschreibbare** Shares → Datei ablegen (z.B. [[NTLM Hash Leak|.lnk]])
- Config-/Backup-Dateien, Skripte, Passwörter im Klartext

Tools: [[smbclient]], [[nmap]]

---
tags: [technique, recon, windows, smb]
---
# SMB Enumeration

> SMB (Ports 139/445) = Windows-Dateifreigaben. Oft anonym („null session") durchsuchbar → Shares, Dateien, teils Credentials.

## Was ist SMB?
- **Server Message Block** = Netzwerkprotokoll für den Zugriff auf Dateien, Ordner und Drucker über das Netz.
- Primär in Windows-Umgebungen → stellt die Freigaben (Shares) bereit, die per UNC-Pfad (`\\host\share`) angesprochen werden.
- Ports: **445** (direkt über TCP, heute Standard), **139** (älter, über NetBIOS).
- Auth läuft über [[NTLM Hash Leak|NTLM]] → deshalb ist SMB auch ein Vektor für Hash-Leaks und Relay.
- Ein **Share** ist ein freigegebener Ordner mit eigenen Rechten (lesbar/beschreibbar); Fehlkonfiguration erlaubt oft anonymen Zugriff.

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
- **beschreibbare** Shares → Datei ablegen (z.B. `.lnk` für NTLM Hash Leak)
- Config-/Backup-Dateien, Skripte, Passwörter im Klartext

Tools: [[smbclient]], [[nmap]]

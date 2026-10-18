---
tags: [tool, recon, smb]
---
# smbclient

> Zugriff auf Windows-Netzwerkfreigaben über das SMB-Protokoll.

## Verwendung
```bash
smbclient -L //<ziel>/ -N                       # Shares auflisten (anonym)
smbclient //<ziel>/share -N                     # Share betreten
smbclient //<ziel>/share -N -c 'put datei.lnk'  # Datei hochladen (one-shot)
```

## Flags
| Flag          | Bedeutung                        |
| ------------- | -------------------------------- |
| `-L`          | List – Shares auflisten          |
| `-N`          | No Password (anonym)             |
| `-U user`     | als User authentifizieren        |
| `-c '...'`    | Befehl direkt ausführen          |

## Im Share
`ls`, `get <datei>`, `put <datei>`, `cd <ordner>`

=> Technik: [[SMB Enumeration]]

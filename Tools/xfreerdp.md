---
tags: [tool, access, windows]
aliases: [xfreerdp3, rdp]
---
# xfreerdp

> RDP-Client für Linux → grafische Remote-Desktop-Sitzung auf einem Windows-Host.

## Verwendung
```bash
xfreerdp3 /v:<host> /u:<user> /p:<password> /cert:ignore
```

## Flags
| Flag             | Bedeutung                          |
| ---------------- | ---------------------------------- |
| `/v:<host>`      | Ziel-Host/IP                       |
| `/u:<user>`      | Benutzername                       |
| `/p:<password>`  | Passwort                           |
| `/cert:ignore`   | Zertifikatswarnung ignorieren      |
| `/d:<domain>`    | Domäne                             |
| `/dynamic-resolution` | Fenstergröße anpassen         |

> [!tip] Pass-the-Hash
> Statt `/p:` geht oft `/pth:<NT-Hash>` – Login ohne Klartext (siehe [[NTLM]]).

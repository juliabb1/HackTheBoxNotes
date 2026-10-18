---
tags: [tool, network, lateral-movement]
---
# rlogin

> Client für die **R-Services** (Paket `rsh-client`) – meldet sich per Remote-Login auf einem Host an, ähnlich SSH, aber ohne Verschlüsselung und oft ohne Passwort.

## Installation (Kali)
```bash
sudo apt install rsh-client -y
```

## Verwendung
```bash
rlogin -l sadm 10.129.234.130     # als User sadm einloggen
```

## Flags
| Flag        | Bedeutung                              |
| ----------- | -------------------------------------- |
| `-l <user>` | „log mich als diesen User ein"         |

> [!info] Kein Passwort?
> Wenn `/etc/hosts.equiv` oder `~/.rhosts` den (gleichnamigen) lokalen User als vertrauenswürdig führt, kommt gar keine Passwortabfrage.

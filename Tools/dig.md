---
tags: [tool, recon, dns]
---
# dig

> *Domain Information Groper* – befragt DNS-Server direkt.

## Verwendung
```bash
dig <name>
dig axfr <domain> @<ziel-ip>   # Zone Transfer
```

## Befehle
| Befehl                        | Was es tut                            |
| ----------------------------- | ------------------------------------- |
| `dig google.com`              | Name → IP (A-Record)                  |
| `dig -x 1.2.3.4`              | IP → Name (Reverse)                   |
| `dig google.com MX`           | Mailserver der Domain                 |
| `dig google.com NS`           | zuständige DNS-Server                 |
| `dig @8.8.8.8 google.com`     | frag gezielt *diesen* Server (`@`)    |
| `dig axfr trick.htb @ziel-ip` | **Zone Transfer**                     |

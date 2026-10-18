---
tags: [basics, linux]
aliases: [Standard-Pfade, Logpfade]
---
# Linux Standard-Pfade

> Wichtige Dateien liegen auf Linux-Systemen (je Distro/Server) **immer am selben Ort**. Man muss sie nicht vorher kennen – einfach blind ausprobieren (z.B. per LFI).

## Logs (poisonbar → RCE)
| Server / OS            | Pfad                              | Steuerbares Feld |
| ---------------------- | --------------------------------- | ---------------- |
| Apache (Ubuntu/Debian) | `/var/log/apache2/access.log`     | User-Agent       |
| Apache (Ubuntu/Debian) | `/var/log/apache2/error.log`      | –                |
| Apache (RedHat/CentOS) | `/var/log/httpd/access_log`       | User-Agent       |
| nginx                  | `/var/log/nginx/access.log`       | User-Agent       |
| SSH-Login              | `/var/log/auth.log`               | Login-Name       |
| FTP (vsftpd)           | `/var/log/vsftpd.log`             | Login-Name       |

> [!tip] Welcher Pfad stimmt?
> Server-Header (`curl -I`) verrät Apache vs. nginx; das `access.log` einmal gelesen zeigt den eigenen Request. Enthält ein Log steuerbaren Inhalt (User-Agent, Login-Name), ist es Träger für Log Poisoning.

## Configs & Systemdateien
| Datei                              | Info                                   |
| ---------------------------------- | -------------------------------------- |
| `/etc/passwd`                      | alle User (Login-User vs. Dienstkonten)|
| `/etc/shadow`                      | Passwort-Hashes (nur root)             |
| `/etc/hosts.equiv` / `~/.rhosts`   | R-Services-Trust                       |
| `/etc/nginx/sites-enabled/default` | nginx vHosts (`server_name`)           |
| `/etc/apache2/sites-enabled/`      | Apache vHosts                          |
| `/var/mail/<user>`                 | Mailbox (Mail Poisoning)               |
| `/var/www/html/`                   | Standard-Webroot                       |
| `/home/<user>/.ssh/id_rsa`         | privater SSH-Key (falls lesbar)        |
| `/proc/self/environ`               | Env-Vars des Prozesses (manchmal poisonbar) |

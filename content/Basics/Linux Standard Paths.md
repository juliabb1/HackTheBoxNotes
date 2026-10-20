---
tags: [basics, linux]
aliases: [Standard Paths, Log Paths]
---
# Linux Standard Paths

> On Linux systems important files (depending on distro/server) **always sit in the same place**. They don't need to be known in advance – just tried blindly (e.g. via LFI).

## Logs (poisonable → RCE)
| Server / OS            | Path                              | Controllable field |
| ---------------------- | --------------------------------- | ------------------ |
| Apache (Ubuntu/Debian) | `/var/log/apache2/access.log`     | User-Agent         |
| Apache (Ubuntu/Debian) | `/var/log/apache2/error.log`      | –                  |
| Apache (RedHat/CentOS) | `/var/log/httpd/access_log`       | User-Agent         |
| nginx                  | `/var/log/nginx/access.log`       | User-Agent         |
| SSH login              | `/var/log/auth.log`               | login name         |
| FTP (vsftpd)           | `/var/log/vsftpd.log`             | login name         |

> [!tip] Which path is correct?
> The Server header (`curl -I`) reveals Apache vs. nginx; reading the `access.log` once shows the request just sent. If a log contains controllable content (User-Agent, login name), it is a carrier for log poisoning.

## Configs & system files
| File                               | Info                                   |
| ---------------------------------- | -------------------------------------- |
| `/etc/passwd`                      | all users (login users vs. service accounts) |
| `/etc/shadow`                      | password hashes (root only)            |
| `/etc/hosts.equiv` / `~/.rhosts`   | R-Services trust                       |
| `/etc/nginx/sites-enabled/default` | nginx vHosts (`server_name`)           |
| `/etc/apache2/sites-enabled/`      | Apache vHosts                          |
| `/var/mail/<user>`                 | mailbox (mail poisoning)               |
| `/var/www/html/`                   | default web root                       |
| `/home/<user>/.ssh/id_rsa`         | private SSH key (if readable)          |
| `/proc/self/environ`               | env vars of the process (sometimes poisonable) |

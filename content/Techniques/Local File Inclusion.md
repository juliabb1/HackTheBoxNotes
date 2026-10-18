---
tags: [technique, exploitation, web]
aliases: [LFI]
---
# Local File Inclusion (LFI)

> Eine Seite lädt Dateien anhand eines Parameters (`?page=about.php`). Missbrauch: nicht vorgesehene lokale Dateien einbinden.

## Finden
- Reiter durchklicken, URL beobachten → Parameter, der eine Datei lädt (`page=`, `file=`, `include=`, `view=`).

## Directory Traversal
```
?page=../../../../etc/passwd
```
| Schreibweise | Zweck                                                             |
| ------------ | ----------------------------------------------------------------- |
| `../`        | ein Ordner hoch (überzählige schaden nicht)                       |
| `....//`     | Nested Traversal → Filter entfernt inneres `../`, gültiges bleibt |
| `/etc/passwd`| absoluter Pfad (falls kein Traversal nötig)                       |

## Nützliche Zieldateien
| Datei                              | Info                                    |
| ---------------------------------- | --------------------------------------- |
| `/etc/passwd`                      | alle User → [[etc-passwd]]              |
| `/etc/nginx/sites-enabled/default` | vHosts (`server_name`)                  |
| `/var/mail/<user>`                 | Mailbox → Mail/Log Poisoning            |
| `/var/log/apache2/access.log`      | Apache-Log → poisonbar (User-Agent)     |

=> Vollständige Liste + warum diese Pfade immer gleich liegen: [[Linux Standard-Pfade]].

## LFI → RCE
Wenn eine einbindbare Datei **steuerbaren Inhalt** hat (Mail, Log), kann man dort PHP platzieren → wird beim Einbinden ausgeführt (Mail/Log Poisoning).

Tool: [[curl]]

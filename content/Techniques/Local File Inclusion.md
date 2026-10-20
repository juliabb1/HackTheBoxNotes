---
tags: [technique, exploitation, web]
aliases: [LFI]
---
# Local File Inclusion (LFI)

> A page loads files based on a parameter (`?page=about.php`). Abuse: include local files that weren't intended.

## Finding
- Click through the tabs, watch the URL → a parameter that loads a file (`page=`, `file=`, `include=`, `view=`).

## Directory Traversal
```
?page=../../../../etc/passwd
```
| Notation     | Purpose                                                            |
| ------------ | ------------------------------------------------------------------ |
| `../`        | up one directory (extra ones don't hurt)                           |
| `....//`     | nested traversal → filter removes inner `../`, valid one remains   |
| `/etc/passwd`| absolute path (if no traversal is needed)                          |

## Useful target files
| File                               | Info                                    |
| ---------------------------------- | --------------------------------------- |
| `/etc/passwd`                      | all users → [[etc-passwd]]              |
| `/etc/nginx/sites-enabled/default` | vHosts (`server_name`)                  |
| `/var/mail/<user>`                 | mailbox → mail/log poisoning            |
| `/var/log/apache2/access.log`      | Apache log → poisonable (User-Agent)    |

=> Full list + why these paths always sit in the same place: [[Linux Standard Paths]].

## LFI → RCE
If an includable file has **controllable content** (mail, log), PHP can be placed there → it's executed on inclusion (mail/log poisoning).

Tool: [[curl]]

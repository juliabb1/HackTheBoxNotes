---
tags: [tool, network, lateral-movement]
---
# rlogin

> Client for the **R-Services** (package `rsh-client`) – logs into a host via remote login, similar to SSH but without encryption and often without a password.

## Installation (Kali)
```bash
sudo apt install rsh-client -y
```

## Usage
```bash
rlogin -l sadm 10.129.234.130     # log in as user sadm
```

## Flags
| Flag        | Meaning                                |
| ----------- | -------------------------------------- |
| `-l <user>` | "log me in as this user"               |

> [!info] No password?
> If `/etc/hosts.equiv` or `~/.rhosts` lists the (same-named) local user as trusted, there's no password prompt at all.

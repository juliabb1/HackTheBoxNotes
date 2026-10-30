---
tags: [tool, privesc, linux]
aliases: [getcap, setcap]
---
# getcap

> Reads which files carry Linux capabilities (`setcap` sets them) → the enumeration step for [[Linux Capabilities Abuse]].

## Commands
```bash
getcap -r / 2>/dev/null          # recurse the whole filesystem, hide errors
getcap /usr/bin/python3.8        # check a single binary
getcap -r /usr /bin /sbin        # limit the search to likely dirs (faster)
setcap -r /usr/bin/python3.8     # remove capabilities from a file (needs root)
setcap cap_setuid+ep /path/bin   # set a capability (root) — the inverse of getcap
```

Alternatives when `getcap` is missing:
```bash
find / -type f -exec getcap {} \; 2>/dev/null
```

## Flags
| Part          | Meaning                                          |
| ------------- | ------------------------------------------------ |
| `-r <path>`   | recurse through the tree starting at `<path>`    |
| `-v`          | verbose (also lists files without capabilities)  |
| `2>/dev/null` | discard the `Permission denied` errors on stderr |

## Reading the output
```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```
- left of `=`: the binary. Between `=` and `+`: the capabilities. After `+`: the capability **sets**.
- sets `e` effective, `i` inheritable, `p` permitted → `+ep` is the common "active for this binary" combination.

### Capabilities and what they mean
| Capability             | Meaning / abuse                                                      |
| ---------------------- | ------------------------------------------------------------------- |
| `cap_setuid`           | change UID to any value, incl. 0 → **direct root** (e.g. via python) |
| `cap_setgid`           | change GID to any group → access group-restricted resources         |
| `cap_net_bind_service` | bind to privileged ports < 1024 (normally root-only)                |
| `cap_net_raw`          | craft/sniff raw packets (how `ping` works without SUID root)        |
| `cap_net_admin`        | configure networking: interfaces, routes, firewall                  |
| `cap_dac_read_search`  | bypass read/search permission checks → read any file                |
| `cap_dac_override`     | bypass all read/write/execute permission checks                     |
| `cap_chown`            | change file ownership arbitrarily                                   |
| `cap_fowner`           | bypass owner checks on operations like `chmod`                      |
| `cap_sys_admin`        | broad admin powers (mount, etc.) — frequently root-equivalent       |
| `cap_sys_ptrace`       | attach to / inspect other processes → inject into a root process    |
| `cap_sys_module`       | load kernel modules → full kernel-level compromise                  |

- the dangerous ones for privesc are those that grant identity (`cap_setuid`/`cap_setgid`), bypass file permissions (`cap_dac_*`), or touch the kernel (`cap_sys_admin`/`cap_sys_module`/`cap_sys_ptrace`).
- check any finding against [GTFOBins](https://gtfobins.github.io/#+capabilities); binary + capability usually maps to a ready shell technique.
- number logic behind file modes they interact with → [[File Permissions and chmod]].

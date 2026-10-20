---
tags: [basics, linux]
aliases: [/etc/passwd, passwd, Service Accounts]
---
# /etc/passwd

> World-readable file with **all user accounts** on the system – one line per user. The first stop for user enumeration.

## Structure of a line
```
sadm:x:1001:1001:,,,:/home/sadm:/bin/bash
 │   │  │    │    │      │          └─ login shell
 │   │  │    │    │      └─ home directory
 │   │  │    │    └─ GECOS (plaintext info: name, office, …)
 │   │  │    └─ GID (primary group)
 │   │  └─ UID
 │   └─ password placeholder ("x" = real hash in /etc/shadow)
 └─ username
```

| Field    | Meaning                                                         |
| -------- | --------------------------------------------------------------- |
| username | login name                                                     |
| `x`      | password lives in `/etc/shadow` (root-readable only)           |
| UID      | user ID. `0` = root, `<1000` = system, `≥1000` = real users     |
| GID      | primary group                                                  |
| GECOS    | free-form info (often empty / `,,,`)                           |
| home     | home directory                                                 |
| shell    | what starts on login → **decisive** (see below)                |

## Login users vs. service accounts
The last field (shell) reveals whether someone *can* log in:

```
root:x:0:0:root:/root:/bin/bash            <- real user (shell)
www-data:x:33:33:...:/usr/sbin/nologin      <- service account
daemon:x:1:1:...:/bin/false                 <- service account
sadm:x:1001:1001:,,,:/home/sadm:/bin/bash   <- real user (shell)
```

> [!info] What are service accounts?
> **`/usr/sbin/nologin`** and **`/bin/false`** are "dead" shells that terminate any login immediately. Such accounts exist only so that a **program can run under its own user** (Apache = `www-data`, SSH = `sshd`, …) – not for logging in. Usually uninteresting for attacks.

## Filtering out real users
```bash
grep -vE 'nologin|false' /etc/passwd
```
`-v` discards the lines with a "dead" shell → what remains are only accounts with a real shell (`/bin/bash`), the **worthwhile targets**.

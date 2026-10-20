---
tags: [basics, linux]
aliases: [chmod, File Permissions, Permissions]
---
# File Permissions & chmod

> Every file has permissions for **owner / group / others**, three bits each: **read(4) + write(2) + execute(1)**. `chmod` sets them – numerically or symbolically.

## Permissions as numbers
Each digit = sum of 4 + 2 + 1:

| Digit | Perms  | Digit | Perms  |
| ----- | ------ | ----- | ------ |
| `7`   | `rwx`  | `3`   | `-wx`  |
| `6`   | `rw-`  | `2`   | `-w-`  |
| `5`   | `r-x`  | `1`   | `--x`  |
| `4`   | `r--`  | `0`   | `---`  |

3 digits = **owner / group / others**, e.g. `755` = `rwxr-xr-x`.

## Leading 4th digit = special bits
| Value | Bit    | Effect                                              |
| ----- | ------ | --------------------------------------------------- |
| `4`   | SUID   | runs with the **owner's** permissions               |
| `2`   | SGID   | runs with the **group's** permissions               |
| `1`   | Sticky | only the owner may delete their own files (`/tmp`)  |

Examples:
- `4755` = SUID bash (`-rwsr-xr-x`)
- `1777` = `/tmp`
- `0644` = regular file
- `0600` = SSH key
- `777` = fully open (= a commonly sought weakness)

## Reading `-rwsr-xr-x`
```
-  rws  r-x  r-x
│   │    │    └─ others: r-x
│   │    └─ group: r-x
│   └─ owner: rws  (s = SUID instead of x)
└─ file type (- file, d directory, l link)
```

> [!danger] `chmod +s` is often NOT enough
> `+s` → `-rws--S---` (SUID, but **no `x`** for group/others) → `Permission denied`.
> `chmod 4755` → `-rwsr-xr-x` → executable for everyone.

## Finding existing SUID binaries
```bash
find / -perm -4000 -type f 2>/dev/null
```
=> check against [GTFOBins](https://gtfobins.github.io/).

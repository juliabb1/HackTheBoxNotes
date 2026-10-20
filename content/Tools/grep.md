---
tags: [tool, text, enumeration]
---
# grep

> Searches text/files line by line for a pattern (regex) – filters noise out of outputs, logs and configs.

## Usage
```bash
grep "admin" file                           # lines containing "admin"
grep -i "pass" file                         # case-insensitive
grep -r "password" /var/www                 # recursive in a folder
grep -v "foo" file                          # inverted: lines WITHOUT "foo"
grep -E 'foo|bar' file                      # extended regex (OR with |)
curl http://target.htb | grep -i title      # filter the output of a pipe
```

## Flags
| Flag | Meaning                                     |
| ---- | ------------------------------------------- |
| `-i` | ignore case                                 |
| `-v` | **invert** – only lines without a match     |
| `-E` | extended regex (`\|`, `+`, `()` without `\`) |
| `-r` | recursively through directories             |
| `-n` | also show line numbers                      |
| `-o` | only the match instead of the whole line    |
| `-c` | only the count of matches                   |

> [!tip] Typical use
> User enumeration `grep -vE 'nologin|false' /etc/passwd` (discard service accounts) → explained in [[etc-passwd]].

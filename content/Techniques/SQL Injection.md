---
tags: [technique, exploitation, web, injection]
aliases: [SQLi]
---
# SQL Injection

> Injecting own SQL code through an input field because the input isn't validated properly.
## Consequences
- **Login bypass**: make the query always true → `' OR 1=1 -- `
- **Data extraction**: dump DB contents (users, hashes)
- **File read**: with the **FILE** privilege, read system files (`LOAD_FILE`)
## Types
| Type             | Principle                                      |
| ---------------- | ---------------------------------------------- |
| error-based      | DB errors reveal data                          |
| UNION-based      | append an own `SELECT` output (fast)           |
| boolean blind    | read true/false from page behavior             |
| time-based blind | read true/false from response time (`SLEEP`) – slow |
## Flow (automated with [[sqlmap]])
```bash
sqlmap -r req.txt --batch --dbs                 # DBs
sqlmap -r req.txt --batch --is-dba --privileges # privileges (FILE?)
sqlmap -r req.txt --batch --file-read=/path     # read a file
sqlmap -r req.txt --batch -D db -T tab --dump    # extract data
```

> [!tip] File read as a pivot
> FILE privilege + config files (e.g. `nginx`) = often the path to hidden vHosts or credentials, not just a DB dump.

Tool: [[sqlmap]]

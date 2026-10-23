---
tags: [tool, network]
aliases: [tcpdump]
---
# tcpdump

> A network sniffer. It watches a network interface and prints every packet coming in or going out → used to confirm an incoming callback (e.g. the LDAP lookup in [[Log4Shell]]) before committing to the full exploit.

## Usage
```bash
sudo tcpdump -i tun0 port 389
```

## Flags
| Part        | Meaning                                                      |
| ----------- | ------------------------------------------------------------ |
| `sudo`      | root required → reading network traffic is privileged        |
| `-i tun0`   | interface → only traffic on `tun0` (the HTB VPN)             |
| `port 389`  | filter → only packets on port 389 (default LDAP)            |

## Confirming a callback
```
13:37:00 IP <box>.xxxxx > <attacker-ip>.389: LDAP
```
- the target reached me on the LDAP port → the injected lookup fired → target is vulnerable

=> Used in: [[Log4Shell]]

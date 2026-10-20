---
tags: [tool, recon, dns]
---
# dig

> *Domain Information Groper* – queries DNS servers directly.

## Usage
```bash
dig <name>
dig axfr <domain> @<target-ip>   # zone transfer
```

## Commands
| Command                       | What it does                          |
| ----------------------------- | ------------------------------------- |
| `dig google.com`              | name → IP (A record)                  |
| `dig -x 1.2.3.4`              | IP → name (reverse)                   |
| `dig google.com MX`           | mail servers of the domain            |
| `dig google.com NS`           | responsible DNS servers               |
| `dig @8.8.8.8 google.com`     | query *this* server specifically (`@`) |
| `dig axfr trick.htb @target-ip` | **zone transfer**                   |

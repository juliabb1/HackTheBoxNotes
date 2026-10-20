---
tags: [technique, recon, dns]
aliases: [AXFR, DNS Zone Transfer]
---
# Zone Transfer

> A DNS server copies its entire **zone** (name list) to another server. If misconfigured, it's queryable by anyone → all hostnames + hidden subdomains in one shot.

## How to recognize it?
- DNS normally runs over **UDP/53** (small, fast).
- `53/tcp open` in the [[nmap]] scan => TCP is used, among other things, for large responses (= a whole zone) => a transfer might be allowed.

## Flow
1. Find the domain name (reverse lookup):
```bash
dig -x <target-ip> @<target-ip>     # PTR record => e.g. trick.htb
```
2. Pull the zone transfer:
```bash
dig axfr <domain> @<target-ip>
```
3. Add the found subdomains to `/etc/hosts` → reachable in the browser.

## Expectation
- **Success** → list of `A`/`NS`/`SOA` records (vHosts = the `A` entries).
- **Locked down** → `Transfer failed.`

> [!note] DNS ≠ web server
> The zone transfer only shows what the **DNS** knows. A vHost can be configured in the web server (`server_name`) **without** a DNS record → discoverable only via the web server config.

Tool: [[dig]]

---
tags: [technique, recon, dns]
aliases: [AXFR, DNS Zone Transfer]
---
# Zone Transfer

> Ein DNS-Server kopiert seine komplette **Zone** (Namensliste) an einen anderen Server. Bei Fehlkonfiguration für jeden abfragbar → alle Hostnamen + versteckte Subdomains auf einen Schlag.

## Woran erkennen?
- DNS läuft normal über **UDP/53** (klein, schnell).
- `53/tcp open` im [[nmap]]-Scan => TCP wird u.a. für große Antworten (= ganze Zone) genutzt => Transfer könnte erlaubt sein.

## Ablauf
1. Domainname finden (Reverse Lookup):
```bash
dig -x <ziel-ip> @<ziel-ip>     # PTR-Record => z.B. trick.htb
```
2. Zone Transfer ziehen:
```bash
dig axfr <domain> @<ziel-ip>
```
3. Gefundene Subdomains in `/etc/hosts` eintragen → im Browser erreichbar.

## Erwartung
- **Erfolg** → Liste aus `A`/`NS`/`SOA`-Records (vHosts = die `A`-Einträge).
- **Dicht** → `Transfer failed.`

> [!note] DNS ≠ Webserver
> Der Zone Transfer zeigt nur, was der **DNS** kennt. Ein vHost kann im Webserver (`server_name`) konfiguriert sein **ohne** DNS-Eintrag → nur über die Webserver-Config auffindbar (siehe [[Trick]]).

Tool: [[dig]]

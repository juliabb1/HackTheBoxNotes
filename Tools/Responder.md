---
tags: [tool, credential-access, windows]
aliases: [responder]
---
# Responder

> Netzwerk-Sniffing-Tool, das SMB/HTTP/LLMNR-Auth von Windows-Clients abfängt und **NTLM-Hashes** mitschneidet. Öffnet dafür lauschende Ports auf deinem Interface.

## Verwendung
```bash
responder -I tun0 -P
```

## Flags
| Flag        | Bedeutung                              |
| ----------- | -------------------------------------- |
| `-I <if>`   | Interface (z.B. `tun0` = VPN)          |
| `-P`        | Proxy-Auth erzwingen                   |
| `-w`        | WPAD-Server                            |
| `-v`        | verbose                                |

## Output
- Gefangene Hashes landen unter `/usr/share/responder/logs/` (und im Terminal).
- Format = **NetNTLMv2** → direkt in [[Hash Cracking|john/hashcat]].

=> Technik: [[NTLM Hash Leak]]

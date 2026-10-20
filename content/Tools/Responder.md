---
tags: [tool, credential-access, windows]
aliases: [responder]
---
# Responder

> Network sniffing tool that intercepts SMB/HTTP/LLMNR auth from Windows clients and captures **NTLM hashes**. It opens listening ports on the local interface for this.

## Usage
```bash
responder -I tun0 -P
```

## Flags
| Flag        | Meaning                                |
| ----------- | -------------------------------------- |
| `-I <if>`   | interface (e.g. `tun0` = VPN)          |
| `-P`        | force proxy auth                       |
| `-w`        | WPAD server                            |
| `-v`        | verbose                                |

## Output
- Captured hashes land under `/usr/share/responder/logs/` (and in the terminal).
- Format = **NetNTLMv2** → straight into john/hashcat.

=> Technique: NTLM Hash Leak

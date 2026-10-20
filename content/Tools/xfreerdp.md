---
tags: [tool, access, windows]
aliases: [xfreerdp3, rdp]
---

# xfreerdp

> RDP client for Linux → a graphical remote desktop session on a Windows host.

## Usage

```bash
xfreerdp3 /v:<host> /u:<user> /p:<password> /cert:ignore
```

## Flags

| Flag                  | Meaning                        |
| --------------------- | ------------------------------ |
| `/v:<host>`           | target host/IP                 |
| `/u:<user>`           | username                       |
| `/p:<password>`       | password                       |
| `/cert:ignore`        | ignore the certificate warning |
| `/d:<domain>`         | domain                         |
| `/dynamic-resolution` | adjust the window size         |

> [!tip] Pass-the-Hash
> Instead of `/p:<password>`, `/pth:<NT-hash>` can often be used – authentication with the hash, no plaintext needed.
>
> - It skips [[Hash Cracking]] entirely → a dumped hash is enough, even if it never cracks.
> - Only the **NT hash** works here (the stored account hash, e.g. from SAM/NTDS or `secretsdump`). A captured **NetNTLMv2** response (challenge-response, e.g. from [[Responder]]) is _not_ pass-the-hash-able → it must be cracked offline.

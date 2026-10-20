---
tags: [technique, recon, windows, smb]
---
# SMB Enumeration

> SMB (ports 139/445) = Windows file shares. Often browsable anonymously ("null session") → shares, files, sometimes credentials.

## What is SMB?
- **Server Message Block** = network protocol for accessing files, folders and printers over the network.
- Primarily in Windows environments → provides the shares addressed via a UNC path (`\\host\share`).
- Ports: **445** (directly over TCP, today's standard), **139** (older, over NetBIOS).
- Auth runs over [[NTLM Hash Leak|NTLM]] → which is why SMB is also a vector for hash leaks and relay.
- A **share** is a shared folder with its own permissions (readable/writable); misconfiguration often allows anonymous access.

## Listing shares
```bash
smbclient -L //<target>/ -N          # anonymous (no password)
smbclient -L //<target>/ -U user     # with a user
```

## Entering a share / files
```bash
smbclient //<target>/share -N
# > ls / get <file> / put <file>
```

## Broader enumeration
```bash
nxc smb <target> -u '' -p '' --shares   # netexec: shares + permissions
enum4linux-ng <target>                  # users, groups, shares, policy
```

## What to watch for
- **writable** shares → drop a file (e.g. `.lnk` for an NTLM hash leak)
- config/backup files, scripts, plaintext passwords

Tools: [[smbclient]], [[nmap]]

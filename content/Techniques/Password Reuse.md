---
tags: [technique, credential-access, lateral-movement]
aliases: [Credential Reuse, Credential Stuffing]
---
# Password Reuse

> A credential recovered in **one** place (service, config, capture, database) is tried **elsewhere** – another service, host, or account – and still works, because the same password was reused.

## When to try it
- Any time a plaintext password or crackable hash is obtained: FTP/HTTP capture, config file, database dump, backup, chat/wiki page.
- Before investing in a new exploit – reuse is the cheapest foothold / lateral move and is extremely common on easy boxes.

## Where to spray the credential
- **Other services on the same host**: SSH (22), RDP (3389), SMB (445), WinRM (5985), database logins.
- **The matching OS account**: a service/app password that equals a system user's login password.
- **Admin consoles**: an app that stores a *device* or *host* password in its settings (e.g. management panels).

## Finding → Exploiting
- Collect every username/password pair seen so far.
- Try each pair against each reachable auth surface, keeping the username constant first:
```bash
ssh nathan@<target>          # FTP creds reused on SSH
```
- For many pairs across many hosts, spray instead of guessing by hand (generic form):
```
# user:pass list against one protocol, stop on first hit
<auth-tool> <proto>://<target> -u users.txt -p passwords.txt --continue-on-success
```

> [!note] Why this works
> Not a software bug but an operational one: the same secret is set in several places for convenience. Finding it once breaks all of them. Distinguish from pass-the-hash – here the **plaintext** (or cracked) password is replayed as a normal login, not the raw hash.

> [!warning] Reuse vs. intended use
> It is only *reuse* when the credential crosses into a context it was not meant for (FTP password → SSH, app setting → host root). Cracking a user's hash and logging in as that same user on that same intended service is **not** password reuse.

## Seen on
- Cap – FTP credentials from a packet capture reused on SSH.
- Unified – host `root` password read from the UniFi settings, replayed on SSH.

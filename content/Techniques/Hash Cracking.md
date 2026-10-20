---
tags: [technique, credential-access]
---
# Hash Cracking

> Crack captured/stolen hashes offline against a wordlist → plaintext password.

## john the ripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt   # auto-detect
john --format=netntlmv2 --wordlist=rockyou.txt hash.txt     # force format
john --show hash.txt                                         # show result
```

## hashcat (GPU, faster)
```bash
hashcat -m 5600 hash.txt rockyou.txt    # 5600 = NetNTLMv2
hashcat -m 0 hash.txt rockyou.txt       # 0 = MD5
```

## Common modes / formats
| Hash          | john format | hashcat mode |
| ------------- | ----------- | ------------ |
| NetNTLMv2     | `netntlmv2` | 5600         |
| NTLM (NT)     | `nt`        | 1000         |
| MD5           | `raw-md5`   | 0            |
| bcrypt        | `bcrypt`    | 3200         |

## Wordlist
```
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt
```
=> Applied to NTLM hashes from [[Responder]].

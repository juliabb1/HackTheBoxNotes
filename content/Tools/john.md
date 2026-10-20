---
tags: [tool, credential-access]
aliases: [john the ripper, JtR]
---
# john (the ripper)

> Password cracker (CPU). Takes hashes + wordlist → plaintext.

## Usage
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt   # auto-detect
john --format=netntlmv2 --wordlist=rockyou.txt hash.txt     # force format
john --show hash.txt                                         # show cracked
```

## Flags
| Flag               | Meaning                       |
| ------------------ | ----------------------------- |
| `--wordlist=<f>`   | wordlist (usually rockyou)    |
| `--format=<fmt>`   | hash type (e.g. `netntlmv2`)  |
| `--show`           | show already-cracked hashes   |
| `--rules`          | apply mutation rules          |

=> Technique + mode table: Hash Cracking

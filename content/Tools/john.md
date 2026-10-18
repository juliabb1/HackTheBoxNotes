---
tags: [tool, credential-access]
aliases: [john the ripper, JtR]
---
# john (the ripper)

> Passwort-Cracker (CPU). Nimmt Hashes + Wordlist → Klartext.

## Verwendung
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt   # auto-detect
john --format=netntlmv2 --wordlist=rockyou.txt hash.txt     # Format erzwingen
john --show hash.txt                                         # geknackte zeigen
```

## Flags
| Flag               | Bedeutung                     |
| ------------------ | ----------------------------- |
| `--wordlist=<f>`   | Wordlist (meist rockyou)      |
| `--format=<fmt>`   | Hash-Typ (z.B. `netntlmv2`)   |
| `--show`           | bereits geknackte anzeigen    |
| `--rules`          | Mutations-Regeln anwenden     |

=> Technik + Modi-Tabelle: Hash Cracking

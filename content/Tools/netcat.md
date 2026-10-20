---
tags: [tool, network]
aliases: [nc, Netcat]
---
# netcat (nc)

> A tiny tool for raw TCP/UDP connections. Waits until the victim connects → its shell lands in the local window (reverse shell).

## Usage
```bash
nc -lvnp 4444        # listener (catch reverse shells)
nc <ip> <port>       # connect to a port
nc -zv <ip> 80       # port check
```

## Flags
| Flag | Meaning                            |
| ---- | ---------------------------------- |
| `-l` | listen – listen instead of connect |
| `-v` | verbose – connection status        |
| `-n` | numeric – no DNS resolution        |
| `-p` | port                               |
| `-z` | scan only, no data                 |
| `-u` | UDP instead of TCP                 |

> [!tip] Mnemonic
> `-lvnp` = **L**isten **V**erbose **N**umeric **P**ort → the standard listener.

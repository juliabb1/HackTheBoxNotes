---
tags: [tool, network]
aliases: [nc, Netcat]
---
# netcat (nc)

> Mini-Tool für rohe TCP/UDP-Verbindungen. Wartet, bis das Opfer sich meldet → dessen Shell landet in deinem Fenster (Reverse Shell).

## Verwendung
```bash
nc -lvnp 4444        # Listener (Reverse Shells auffangen)
nc <ip> <port>       # zu einem Port verbinden
nc -zv <ip> 80       # Port-Check
```

## Flags
| Flag | Bedeutung                          |
| ---- | ---------------------------------- |
| `-l` | listen – lauschen statt verbinden  |
| `-v` | verbose – Verbindungsstatus        |
| `-n` | numeric – keine DNS-Auflösung      |
| `-p` | Port                               |
| `-z` | nur scannen, keine Daten           |
| `-u` | UDP statt TCP                      |

> [!tip] Merkhilfe
> `-lvnp` = **L**isten **V**erbose **N**umeric **P**ort → Standard-Listener.

---
tags: [tool, network]
aliases: [tshark, Wireshark]
---
# tshark

> The command-line version of Wireshark. Reads a `.pcap` capture and prints selected packets → used to pull cleartext data (e.g. FTP credentials) out of recorded traffic.

## Usage
```bash
tshark -r 0.pcap -Y ftp
```

## Flags
| Flag        | Meaning                                              |
| ----------- | ---------------------------------------------------- |
| `-r <file>` | read a capture file instead of sniffing live         |
| `-Y <expr>` | display filter (e.g. `ftp`, `http`, `tcp.port==21`)  |
| `-V`        | full packet detail                                   |

## Wireshark vs. tshark
- **Wireshark** = GUI: open the `.pcap`, click through packets, apply filters visually.
- **tshark** = same engine, terminal only → faster when the filter is already known.
- live sniffing instead of reading a file → [[tcpdump]].

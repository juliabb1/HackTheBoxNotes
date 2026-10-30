---
tags: [technique, network]
aliases: [Packet Capture Analysis, pcap]
---
# Packet Capture Analysis

> A **pcap** is a recording of network traffic — every packet that crossed the interface during the capture. Unencrypted protocols leak their contents to anyone who reads the file.

## Why it matters
- **FTP** sends `USER` and `PASS` in cleartext → a recorded FTP login exposes the password directly.
- other cleartext protocols (HTTP, Telnet, SMTP) leak the same way.

## Reading a capture → [[tshark]]
```bash
file 0.pcap                 # confirm it is a pcap
tshark -r 0.pcap -Y ftp     # show only FTP packets
```
- GUI alternative: open the file in Wireshark and filter visually.
- live capture instead of a saved file → [[tcpdump]].

## Common extractions
- credentials from cleartext logins (FTP/HTTP basic auth).
- transferred files / objects reassembled from a stream.
- hostnames, tokens and session cookies carried in plaintext requests.

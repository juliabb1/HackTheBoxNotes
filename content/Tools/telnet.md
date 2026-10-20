---
tags: [tool, network]
---
# telnet

> Connects to a **port** and sends **raw text**. netcat does the same thing more modernly (`nc <ip> <port>`).

## Usage
```bash
telnet <ip> <port>
```
| Port | Service   | Purpose                 |
| ---- | --------- | ----------------------- |
| 25   | SMTP/mail | write mail by hand      |
| 80   | HTTP      | HTTP request by hand    |
| 110  | POP3      | retrieve mail           |

## SMTP commands (port 25)
| Command       | Meaning                              |
| ------------- | ------------------------------------ |
| `HELO`        | greeting                             |
| `MAIL FROM`   | sender (freely made up)              |
| `RCPT TO`     | recipient                            |
| `DATA`        | "here comes the mail body"           |
| `.` (alone)   | "mail finished"                      |
| `QUIT`        | close the connection                 |

=> Application: Mail/Log Poisoning

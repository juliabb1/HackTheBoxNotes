---
tags: [basics, linux]
aliases: [/etc/passwd, passwd, Dienstkonten]
---
# /etc/passwd

> Weltlesbare Datei mit **allen Benutzerkonten** des Systems – eine Zeile pro User. Erste Anlaufstelle bei der User-Enumeration.

## Aufbau einer Zeile
```
sadm:x:1001:1001:,,,:/home/sadm:/bin/bash
 │   │  │    │    │      │          └─ Login-Shell
 │   │  │    │    │      └─ Home-Verzeichnis
 │   │  │    │    └─ GECOS (Klartext-Infos: Name, Büro, …)
 │   │  │    └─ GID (primäre Gruppe)
 │   │  └─ UID
 │   └─ Passwort-Platzhalter ("x" = echter Hash in /etc/shadow)
 └─ Username
```

| Feld     | Bedeutung                                                       |
| -------- | --------------------------------------------------------------- |
| Username | Loginname                                                       |
| `x`      | Passwort liegt in `/etc/shadow` (nur root lesbar)               |
| UID      | User-ID. `0` = root, `<1000` = System, `≥1000` = echte User     |
| GID      | primäre Gruppe                                                  |
| GECOS    | freie Infos (oft leer / `,,,`)                                  |
| Home     | Heimatverzeichnis                                               |
| Shell    | was beim Login startet → **entscheidend** (s.u.)                |

## Login-User vs. Dienstkonten
Das letzte Feld (Shell) verrät, ob sich jemand einloggen *kann*:

```
root:x:0:0:root:/root:/bin/bash            <- echter User (Shell)
www-data:x:33:33:...:/usr/sbin/nologin      <- Dienstkonto
daemon:x:1:1:...:/bin/false                 <- Dienstkonto
sadm:x:1001:1001:,,,:/home/sadm:/bin/bash   <- echter User (Shell)
```

> [!info] Was sind Dienstkonten?
> **`/usr/sbin/nologin`** und **`/bin/false`** sind „tote" Shells, die jeden Login sofort beenden. Solche Konten existieren nur, damit ein **Programm unter eigenem User** laufen kann (Apache = `www-data`, SSH = `sshd`, …) – nicht zum Einloggen. Für Angriffe meist uninteressant.

## Echte User herausfiltern
```bash
grep -vE 'nologin|false' /etc/passwd
```
`-v` wirft die Zeilen mit „toter" Shell weg → übrig bleiben nur Accounts mit echter Shell (`/bin/bash`), die **lohnenden Ziele**.

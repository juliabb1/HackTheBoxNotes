---
tags: [basics, linux]
aliases: [chmod, Dateirechte, Permissions]
---
# Dateirechte & chmod

> Jede Datei hat Rechte für **Owner / Gruppe / Andere**, je drei Bits: **read(4) + write(2) + execute(1)**. `chmod` setzt sie – als Zahl oder symbolisch.

## Rechte als Zahl
Jede Ziffer = Summe aus 4 + 2 + 1:

| Ziffer | Rechte | Ziffer | Rechte |
| ------ | ------ | ------ | ------ |
| `7`    | `rwx`  | `3`    | `-wx`  |
| `6`    | `rw-`  | `2`    | `-w-`  |
| `5`    | `r-x`  | `1`    | `--x`  |
| `4`    | `r--`  | `0`    | `---`  |

3 Ziffern = **Owner / Gruppe / Andere**, z.B. `755` = `rwxr-xr-x`.

## Führende 4. Ziffer = Spezial-Bits
| Wert | Bit    | Wirkung                                           |
| ---- | ------ | ------------------------------------------------- |
| `4`  | SUID   | läuft mit Rechten des **Besitzers**               |
| `2`  | SGID   | läuft mit Rechten der **Gruppe**                  |
| `1`  | Sticky | nur Besitzer darf eigene Dateien löschen (`/tmp`) |

Beispiele:
- `4755` = SUID-Bash (`-rwsr-xr-x`)
- `1777` = `/tmp`
- `0644` = normale Datei
- `0600` = SSH-Key
- `777` = alles offen (= oft gesuchte Lücke)

## `-rwsr-xr-x` lesen
```
-  rws  r-x  r-x
│   │    │    └─ Andere: r-x
│   │    └─ Gruppe: r-x
│   └─ Owner: rws  (s = SUID statt x)
└─ Dateityp (- Datei, d Ordner, l Link)
```

> [!danger] `chmod +s` reicht oft NICHT
> `+s` → `-rws--S---` (SUID, aber **kein `x`** für Gruppe/Andere) → `Permission denied`.
> `chmod 4755` → `-rwsr-xr-x` → ausführbar für alle.

## Vorhandene SUID-Binaries finden
```bash
find / -perm -4000 -type f 2>/dev/null
```
=> gegen [GTFOBins](https://gtfobins.github.io/) prüfen.

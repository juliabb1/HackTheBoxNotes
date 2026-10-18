---
tags: [tool, text, enumeration]
---
# grep

> Durchsucht Text/Dateien zeilenweise nach einem Muster (Regex) – filtert Rauschen aus Outputs, Logs und Configs.

## Verwendung
```bash
grep "admin" datei                          # Zeilen mit "admin"
grep -i "pass" datei                        # Groß/Klein egal
grep -r "password" /var/www                 # rekursiv im Ordner
grep -v "foo" datei                         # invertiert: Zeilen OHNE "foo"
grep -E 'foo|bar' datei                     # erweiterte Regex (ODER mit |)
curl http://ziel.htb | grep -i title        # Output einer Pipe filtern
```

## Flags
| Flag | Bedeutung                                   |
| ---- | ------------------------------------------- |
| `-i` | Groß-/Kleinschreibung ignorieren            |
| `-v` | **invertieren** – nur Zeilen ohne Treffer   |
| `-E` | erweiterte Regex (`\|`, `+`, `()` ohne `\`) |
| `-r` | rekursiv durch Verzeichnisse                |
| `-n` | Zeilennummern mitanzeigen                   |
| `-o` | nur den Treffer statt der ganzen Zeile      |
| `-c` | nur Anzahl der Treffer                      |

> [!tip] Typischer Einsatz
> User-Enumeration `grep -vE 'nologin|false' /etc/passwd` (Dienstkonten rauswerfen) → erklärt in [[etc-passwd]].

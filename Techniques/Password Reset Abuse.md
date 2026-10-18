---
tags: [technique, exploitation, web]
aliases: [Insecure Password Reset]
---
# Password Reset Abuse

> Eine „Passwort vergessen"-Funktion generiert ein neues Passwort serverseitig – und gibt es versehentlich in der **HTTP-Response** zurück, statt es nur per Mail zu verschicken.

## Finden
- Reset-Formular suchen (z.B. `reset.php`, „Forgot password").
- Reset für einen bekannten/geratenen Usernamen anstoßen (`admin`).
- Response im Browser prüfen: **DevTools → Network → der POST-Request → Response**.

## Ausnutzen
```
new_password: 24bc0464
```
=> Das neue Passwort steht im Klartext in der Antwort. Damit direkt einloggen:
- Username: `admin`
- Passwort: der geleakte Wert

> [!note] Warum das klappt
> Entwickler-Bug: der Reset-Flow soll das Passwort nur setzen/mailen, rendert es aber zusätzlich in die Seite. Die Mail kommt nie an → man liest es direkt mit.


Tool: [[curl]]

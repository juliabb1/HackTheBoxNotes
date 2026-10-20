---
tags: [technique, exploitation, web]
aliases: [Insecure Password Reset]
---
# Password Reset Abuse

> A "forgot password" function generates a new password server-side – and accidentally returns it in the **HTTP response** instead of only sending it by mail.

## Finding
- Look for a reset form (e.g. `reset.php`, "Forgot password").
- Trigger a reset for a known/guessed username (`admin`).
- Check the response in the browser: **DevTools → Network → the POST request → Response**.

## Exploiting
```
new_password: 24bc0464
```
=> The new password is in the response in plaintext. Log in with it directly:
- username: `admin`
- password: the leaked value

> [!note] Why this works
> Developer bug: the reset flow should only set/mail the password, but it additionally renders it into the page. The mail never arrives → it is read directly from the response.


Tool: [[curl]]

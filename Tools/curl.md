---
tags: [tool, network, web]
---
# curl

> Kommandozeilen-Tool, das HTTP-Anfragen verschickt – „Browser ohne Fenster".

## Verwendung
```bash
curl http://ziel.htb                        # GET
curl -I http://ziel.htb                     # nur Header
curl -v http://ziel.htb                     # Request+Response komplett
curl -G http://ziel.htb/index.php --data-urlencode "page=../../etc/passwd"
curl http://ziel.htb/index.php -H "User-Agent: deinwert"  # Header-Parameter setzen

# mit Session-Cookie (z.B. nach Login) – Datei per POST-Parameter lesen:
curl http://ziel.htb/dashboard.php \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
```

## Flags
| Flag               | Bedeutung                              |
| ------------------ | -------------------------------------- |
| `-X`               | HTTP-Methode                           |
| `-d`               | Daten senden (POST-Body)               |
| `--data-urlencode` | Daten senden + automatisch kodiert     |
| `-G`               | Daten als GET-Parameter anhängen       |
| `-H`               | eigenen Header setzen                  |
| `-b` / `--cookie`  | Cookie mitschicken (`-b "PHPSESSID=…"`)|
| `-I` / `-v`        | nur Header / verbose                   |
| `-o datei`         | Antwort speichern                      |

> [!tip] `-b PHPSESSID` – authentifizierte Requests
> Nach einem Login hält der Server die Session über das Cookie `PHPSESSID`. Mit `-b "PHPSESSID=<wert>"` sprichst du geschützte Seiten (Dashboard etc.) an, als wärst du eingeloggt – den Wert aus **DevTools → Application/Storage → Cookies** oder dem Login-Response kopieren. Zentral für [[Mail Poisoning|Log Poisoning]].

> [!tip] LFI-Kombi
> `curl -G "URL" --data-urlencode "page=..." --data-urlencode "cmd=..."`
> → nie wieder `%20`, `&` oder Quotes von Hand kodieren. Siehe [[Local File Inclusion]].

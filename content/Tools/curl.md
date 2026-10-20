---
tags: [tool, network, web]
---
# curl

> Command-line tool that sends HTTP requests – "a browser without a window".

## Usage
```bash
curl http://target.htb                        # GET
curl -I http://target.htb                      # headers only
curl -v http://target.htb                      # full request+response
curl -G http://target.htb/index.php --data-urlencode "page=../../etc/passwd"
curl http://target.htb/index.php -H "User-Agent: yourvalue"  # set a header parameter

# with a session cookie (e.g. after login) – read a file via a POST parameter:
curl http://target.htb/dashboard.php \
  -b "PHPSESSID=13vsltn0mhkf744vc9hbb13q6k" \
  --data-urlencode "file=/var/log/apache2/access.log"
```

## Flags
| Flag               | Meaning                                |
| ------------------ | -------------------------------------- |
| `-X`               | HTTP method                            |
| `-d`               | send data (POST body)                  |
| `--data-urlencode` | send data + auto-encoded               |
| `-G`               | append data as GET parameters          |
| `-H`               | set a custom header                    |
| `-b` / `--cookie`  | send a cookie (`-b "PHPSESSID=…"`)     |
| `-I` / `-v`        | headers only / verbose                 |
| `-o file`          | save the response                      |

> [!tip] `-b PHPSESSID` – authenticated requests
> After a login the server keeps the session via the `PHPSESSID` cookie. With `-b "PHPSESSID=<value>"` protected pages (dashboard etc.) are addressed as if logged in – the value comes from **DevTools → Application/Storage → Cookies** or the login response. Central to log poisoning.

> [!tip] LFI combo
> `curl -G "URL" --data-urlencode "page=..." --data-urlencode "cmd=..."`
> → never encode `%20`, `&` or quotes by hand again. (LFI)

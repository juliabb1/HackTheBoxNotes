---
tags: [tool, web, proxy]
aliases: [FoxyProxy]
---
# FoxyProxy

> A browser extension for quickly switching the active proxy. It routes browser traffic through a proxy such as [[Burp Suite]] so requests can be intercepted and edited.

```
Normal:     Browser ───────────────► Website
With proxy: Browser ──► Proxy ──► Website
```
- the proxy can **see, store, modify or block** traffic before forwarding → this is why it is useful for intercepting requests

## Adding the Burp proxy
| Field                | Value             |
| -------------------- | ----------------- |
| Title / Name         | `Burp` (label)    |
| Proxy Type           | `HTTP`            |
| Hostname / IP        | `127.0.0.1`       |
| Port                 | `8080`            |
| Username / Password  | leave empty       |

- activate the `Burp` proxy in FoxyProxy → traffic now passes through [[Burp Suite]]


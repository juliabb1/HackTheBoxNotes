---
tags: [tool, web, proxy]
aliases: [Burp, BurpSuite, Burp Suite]
---
# Burp Suite

> An intercepting (man-in-the-middle) proxy. It sits between browser and server, pauses requests for inspection, and lets them be edited **before forwarding** → the way an attacker payload gets injected into a request.

## Setup
- listener runs on `127.0.0.1:8080`
- browser traffic is routed through it via [[FoxyProxy]]
- for HTTPS, Burp's CA certificate must be trusted: with the proxy active, open `http://burp` → download `cacert.der` → import it in the browser's certificate settings (otherwise HTTPS pages fail to load)

## Intercepting a request
- **Proxy → Intercept: Intercept is On** → trigger the action in the browser (e.g. a login with any credentials)
- the paused request appears in Burp, e.g. `POST /api/login` with body:
```json
{"username":"test","password":"test","remember":false,"strict":true}
```
- fields that reach the server log are the injection target (on Unified: `remember`)

## Repeater
- select the request → `Ctrl+R` → **Repeater** tab
- holds the request for inspection → edit and resend freely for payload iteration
- once in Repeater, Intercept can be turned off and set to `Forward` so the browser no longer hangs

=> Used to deliver the [[Log4Shell]] payload. Proxy switching: [[FoxyProxy]].

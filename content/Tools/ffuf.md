---
tags: [tool, web, fuzzing]
aliases: [ffuf]
---
# ffuf

> Fast web fuzzer. Replaces the `FUZZ` keyword in a URL with each value from a wordlist and reports the responses → used to enumerate hidden paths, parameters, or sequential ids.

## Usage
```bash
ffuf -u http://<box-ip>/data/FUZZ -w <(seq 0 100) -mc 200
```

## Flags
| Flag        | Meaning                                                               |
| ----------- | --------------------------------------------------------------------- |
| `-u`        | target URL; `FUZZ` marks the slot filled from the wordlist            |
| `-w`        | wordlist; `<(seq 0 100)` feeds numbers 0–100 via process substitution |
| `-mc 200`   | match only HTTP 200 (filter out 404/403 noise)                        |
| `-fs <n>`   | filter out responses of size `n` (hide a uniform "not found" page)    |

## Spotting a hit
- responses are compared by status code, size, or word count → the one that differs is interesting.
- on Cap `/data/0` and `/data/1` both return 200 of similar size → both captures exist.


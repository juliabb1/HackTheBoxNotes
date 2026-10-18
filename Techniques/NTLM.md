---
tags: [technique, windows, auth]
aliases: [NTLMv2]
---
# NTLM

> Wie Windows-Systeme **beweisen, wer du bist**, beim Zugriff auf Ressourcen: Shares (SMB), Remote Desktop, Windows-Auth-Websites.

## Challenge-Response (kein Klartext über die Leitung)
1. Client sendet **Username** an den Server.
2. Server schickt eine **zufällige Challenge** zurück.
3. Client verschlüsselt die Challenge mit seinem **Passwort-Hash**.
4. Server prüft die Antwort.

=> Übertragen wird nur ein **Hash**, nie das Passwort.

## Warum angreifbar?
- Der mitgeschnittene **NTLMv2-Response** (Challenge + Hash) lässt sich **offline cracken** → Hash Cracking.
- Windows authentifiziert sich teils **automatisch** gegen UNC-Pfade (`\\host\share`) → NTLM Hash Leak.

## Begriffe
| Begriff      | Bedeutung                                               |
| ------------ | ------------------------------------------------------- |
| NTLMv2-Hash  | abgefangener Challenge-Response → crackbar (`netntlmv2`)|
| Pass-the-Hash| NT-Hash direkt zur Auth nutzen (ohne Cracken)           |
| NTLM Relay   | abgefangene Auth an einen anderen Dienst weiterreichen  |

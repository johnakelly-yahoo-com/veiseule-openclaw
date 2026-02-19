---
summary: "CLI-referentie voor `openclaw security` (audit en het verhelpen van veelvoorkomende beveiligingsvalkuilen)"
read_when:
  - Je wilt een snelle beveiligingsaudit uitvoeren op config/status
  - Je wilt veilige “fix”-suggesties toepassen (chmod, strakkere standaardwaarden)
title: "beveiliging"
---

# `openclaw security`

Beveiligingstools (audit + optionele fixes).

Gerelateerd:

- Beveiligingsgids: [Security](/gateway/security)

## Audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

De audit waarschuwt wanneer meerdere DM-afzenders de hoofdsessie delen en raadt **beveiligde DM-modus** aan: `session.dmScope="per-channel-peer"` (of `per-account-channel-peer` voor kanalen met meerdere accounts) voor gedeelde inboxen.
Daarnaast waarschuwt hij wanneer kleine modellen (`<=300B`) worden gebruikt zonder sandboxing en met web-/browsertools ingeschakeld.
Voor webhook-ingress waarschuwt het wanneer `hooks.defaultSessionKey` niet is ingesteld, wanneer request-`sessionKey`-overschrijvingen zijn ingeschakeld en wanneer overschrijvingen zijn ingeschakeld zonder `hooks.allowedSessionKeyPrefixes`.
Het waarschuwt ook wanneer sandbox-Dockerinstellingen zijn geconfigureerd terwijl de sandboxmodus uit staat, wanneer `gateway.nodes.denyCommands` ineffectieve patroonachtige/onbekende items gebruikt, wanneer het globale `tools.profile="minimal"` wordt overschreven door agent-toolprofielen, en wanneer tools van geïnstalleerde extension-plugins mogelijk bereikbaar zijn onder een permissief toolbeleid.


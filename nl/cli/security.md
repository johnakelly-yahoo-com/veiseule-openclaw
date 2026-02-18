---
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

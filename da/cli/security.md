---
title: "sikkerhed"
---

# `openclaw security`

Sikkerhedsværktøjer (audit + valgfrie rettelser).

Relateret:

- Sikkerhedsguide: [Security](/gateway/security)

## Revision

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Revisionen advarer når flere DM-afsendere deler hovedsessionen og anbefaler **sikker DM-tilstand**: `session.dmScope="per-channel-peer"` (eller `per-account-channel-peer` for multi-account kanaler) for delte indbakker.
Det advarer også, når små modeller (`<=300B`) bruges uden sandboxing og med web/browser værktøjer aktiveret.


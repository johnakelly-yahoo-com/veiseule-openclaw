---
summary: "CLI-referens för `openclaw security` (granska och åtgärda vanliga säkerhetsfallgropar)"
read_when:
  - Du vill köra en snabb säkerhetsgranskning av konfig/tillstånd
  - Du vill tillämpa säkra ”fix”-förslag (chmod, skärpa standardinställningar)
title: "säkerhet"
---

# `openclaw security`

Säkerhetsverktyg (granskning + valfria åtgärder).

Relaterat:

- Säkerhetsguide: [Säkerhet](/gateway/security)

## Granskning

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Granskningen varnar när flera DM-avsändare delar huvudsessionen och rekommenderar **säkert DM-läge**: `session.dmScope="per-channel-peer"` (eller `per-account-peer` för multi-account kanaler) för delade inkorgar.
Den varnar också när små modeller (`<=300B`) används utan sandlåda och med webb-/webbläsarverktyg aktiverade.
För webhook-ingress varnas det när `hooks.defaultSessionKey` inte är satt, när åsidosättningar av request-`sessionKey` är aktiverade och när åsidosättningar är aktiverade utan `hooks.allowedSessionKeyPrefixes`.
Det varnas också när sandbox-Docker-inställningar är konfigurerade medan sandbox-läget är avstängt, när `gateway.nodes.denyCommands` använder ineffektiva mönsterliknande/okända poster, när global `tools.profile="minimal"` åsidosätts av agenters verktygsprofiler och när installerade extension-pluginverktyg kan vara åtkomliga under en tillåtande verktygspolicy.


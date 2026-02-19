---
summary: "Sanggunian ng CLI para sa `openclaw security` (pag-audit at pag-ayos ng mga karaniwang security footgun)"
read_when:
  - Gusto mong magpatakbo ng mabilisang security audit sa config/state
  - Gusto mong ilapat ang mga ligtas na mungkahing “fix” (chmod, paghihigpit ng mga default)
title: "seguridad"
---

# `openclaw security`

Mga security tool (audit + opsyonal na mga fix).

Kaugnay:

- Gabay sa seguridad: [Security](/gateway/security)

## Pag-audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Nagbabala ang audit kapag maraming DM sender ang nagbabahagi ng pangunahing session at inirerekomenda ang **secure DM mode**: `session.dmScope="per-channel-peer"` (o `per-account-channel-peer` para sa mga multi-account channel) para sa mga shared inbox.
Nagbibigay din ito ng babala kapag ang maliliit na modelo (`<=300B`) ay ginagamit nang walang sandboxing at may naka-enable na web/browser tools.
Para sa webhook ingress, nagbibigay ito ng babala kapag hindi nakatakda ang `hooks.defaultSessionKey`, kapag naka-enable ang mga override ng request na `sessionKey`, at kapag naka-enable ang mga override nang walang `hooks.allowedSessionKeyPrefixes`.
Nagbibigay din ito ng babala kapag naka-configure ang mga setting ng sandbox Docker habang naka-off ang sandbox mode, kapag ang `gateway.nodes.denyCommands` ay gumagamit ng mga hindi epektibong pattern-like/hindi kilalang entry, kapag ang global `tools.profile="minimal"` ay na-o-override ng mga profile ng tool ng agent, at kapag ang mga naka-install na extension plugin tool ay maaaring ma-access sa ilalim ng permissive na tool policy.


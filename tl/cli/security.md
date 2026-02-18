---
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


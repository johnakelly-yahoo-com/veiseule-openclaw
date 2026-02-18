---
title: "seguridad"
---

# `openclaw security`

Herramientas de seguridad (auditoría + correcciones opcionales).

Relacionado:

- Guía de seguridad: [Security](/gateway/security)

## Auditoría

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

La auditoría advierte cuando varios remitentes de mensajes directos comparten la sesión principal y recomienda el **modo seguro de mensajes directos**: `session.dmScope="per-channel-peer"` (o `per-account-channel-peer` para canales de múltiples cuentas) para bandejas de entrada compartidas.
También advierte cuando se usan modelos pequeños (`<=300B`) sin sandboxing y con herramientas web/navegador habilitadas.



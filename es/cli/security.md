---
summary: "Referencia de la CLI para `openclaw security` (auditar y corregir errores comunes de seguridad)"
read_when:
  - Quiere ejecutar una auditoría rápida de seguridad sobre la configuración/el estado
  - Quiere aplicar sugerencias seguras de “corrección” (chmod, endurecer valores predeterminados)
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
Para el ingreso de webhooks, muestra advertencias cuando `hooks.defaultSessionKey` no está configurado, cuando están habilitadas las anulaciones de `sessionKey` en la solicitud y cuando las anulaciones están habilitadas sin `hooks.allowedSessionKeyPrefixes`.
También muestra advertencias cuando la configuración de sandbox Docker está definida mientras el modo sandbox está desactivado, cuando `gateway.nodes.denyCommands` usa entradas ineficaces o de tipo patrón/desconocidas, cuando el `tools.profile="minimal"` global es sobrescrito por perfiles de herramientas del agente y cuando las herramientas de plugins de extensión instalados pueden ser accesibles bajo una política de herramientas permisiva.


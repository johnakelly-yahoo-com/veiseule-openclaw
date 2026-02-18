---
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` es un comando proporcionado por un plugin. Solo aparece si el plugin de llamada de voz está instalado y habilitado.

Documento principal:

- Plugin de llamada de voz: [Voice Call](/plugins/voice-call)

## Comandos comunes

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Exposición de webhooks (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Nota de seguridad: exponga el endpoint del webhook únicamente a redes en las que confíe. Prefiera Tailscale Serve sobre Funnel cuando sea posible.

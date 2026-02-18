---
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` est une commande fournie par un plugin. Elle n’apparait que si le plugin voice-call est installe et active.

Documentation principale :

- Plugin voice-call : [Voice Call](/plugins/voice-call)

## Commandes courantes

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Exposition des webhooks (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Note de securite : n’exposez le point de terminaison du webhook qu’aux reseaux auxquels vous faites confiance. Preferez Tailscale Serve a Funnel lorsque c’est possible.


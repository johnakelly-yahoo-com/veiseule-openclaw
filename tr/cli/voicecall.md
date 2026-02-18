---
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` eklenti tarafından sağlanan bir komuttur. Yalnızca voice-call eklentisi yüklü ve etkinse görünür.

Birincil belge:

- Voice-call eklentisi: [Voice Call](/plugins/voice-call)

## Yaygın komutlar

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Webhook'ların dışa açılması (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Güvenlik notu: webhook uç noktasını yalnızca güvendiğiniz ağlara açın. Mümkün olduğunda Funnel yerine Tailscale Serve'ü tercih edin.


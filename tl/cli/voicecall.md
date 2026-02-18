---
title: "voicecall"
---

# `openclaw voicecall`

Ang `voicecall` ay isang command na ibinibigay ng plugin. Lumalabas lamang ito kung ang voice-call plugin ay naka-install at naka-enable.

Pangunahing doc:

- Plugin para sa Tawag sa Boses: [Tawag sa Boses](/plugins/voice-call)

## Mga karaniwang command

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Pag-expose ng mga webhook (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

Paalalang pangseguridad: ilantad lamang ang webhook endpoint sa mga network na pinagkakatiwalaan mo. Mas piliin ang Tailscale Serve kaysa Funnel kung maaari.


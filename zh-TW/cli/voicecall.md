---
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` 是由外掛提供的指令。只有在已安裝並啟用語音通話外掛時才會顯示。

主要文件：

- 語音通話外掛：[Voice Call](/plugins/voice-call)

## 常用指令

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## 公開 Webhook（Tailscale）

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

安全注意事項：僅將 webhook 端點公開給您信任的網路。若可行，請優先使用 Tailscale Serve 而非 Funnel。


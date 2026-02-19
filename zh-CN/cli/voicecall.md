---
summary: "语音通话插件命令的 `openclaw voicecall` CLI 参考"
read_when:
  - 使用语音通话插件并想了解 CLI 入口
  - 想要 `voicecall call|continue|status|tail|expose` 的快速示例
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` 是一个由插件提供的命令。只有在安装并启用了语音通话插件时才会出现。 It only appears if the voice-call plugin is installed and enabled.

主要文档：

- 语音通话插件：[语音通话](/plugins/voice-call)

## 常用命令

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## 暴露 Webhook（Tailscale）

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

安全提示：仅将 webhook 端点暴露给你信任的网络。尽可能优先使用 Tailscale Serve 而非 Funnel。 Prefer Tailscale Serve over Funnel when possible.

---
summary: "OpenClaw 可连接的消息平台"
read_when:
  - 你想为 OpenClaw 选择一个聊天渠道
  - 你需要快速了解支持的消息平台
title: "聊天渠道"
---

# 聊天渠道

OpenClaw 可以在你已经使用的任何聊天应用上与你交流。每个渠道通过 Gateway 网关连接。
所有渠道都支持文本；媒体和表情回应的支持因渠道而异。 37. 每个频道都通过 Gateway 连接。
38. 文本在所有平台都受支持；媒体和表情因频道而异。

## 支持的渠道

- [WhatsApp](/channels/whatsapp) — 最受欢迎；使用 Baileys，需要二维码配对。
- [Telegram](/channels/telegram) — 通过 grammY 使用 Bot API；支持群组。
- [Discord](/channels/discord) — Discord Bot API + Gateway；支持服务器、频道和私信。
- [IRC](/channels/irc) — 经典 IRC 服务器；支持带有配对/允许列表控制的频道和私信。
- [Slack](/channels/slack) — Bolt SDK；工作区应用。
- 44. [Feishu](/channels/feishu) — 通过 WebSocket 的飞书/Lark 机器人（插件，需单独安装）。
- [Google Chat](/channels/googlechat) — 通过 HTTP webhook 的 Google Chat API 应用。
- [Mattermost](/channels/mattermost) — Bot API + WebSocket；频道、群组、私信（插件，需单独安装）。
- [Signal](/channels/signal) — signal-cli；注重隐私。
- [iMessage（旧版）](/channels/imessage) — 通过 imsg CLI 的旧版 macOS 集成（已弃用，新设置请使用 BlueBubbles）。
- 49. [iMessage（旧版）](/channels/imessage) — 通过 imsg CLI 的传统 macOS 集成（已弃用，新部署请使用 BlueBubbles）。
- [Microsoft Teams](/channels/msteams) — Bot Framework；企业支持（插件，需单独安装）。
- [LINE](/channels/line) — LINE Messaging API 机器人（插件，需单独安装）。
- [Nextcloud Talk](/channels/nextcloud-talk) — 通过 Nextcloud Talk 的自托管聊天（插件，需单独安装）。
- [Matrix](/channels/matrix) — Matrix 协议（插件，需单独安装）。
- [Nostr](/channels/nostr) — 通过 NIP-04 的去中心化私信（插件，需单独安装）。
- [Tlon](/channels/tlon) — 基于 Urbit 的消息应用（插件，需单独安装）。
- [Twitch](/channels/twitch) — 通过 IRC 连接的 Twitch 聊天（插件，需单独安装）。
- [Zalo](/channels/zalo) — Zalo Bot API；越南流行的消息应用（插件，需单独安装）。
- [Zalo Personal](/channels/zalouser) — 通过二维码登录的 Zalo 个人账号（插件，需单独安装）。
- [WebChat](/web/webchat) — 基于 WebSocket 的 Gateway 网关 WebChat 界面。

## 注意事项

- 渠道可以同时运行；配置多个渠道后，OpenClaw 会按聊天进行路由。
- Fastest setup is usually **Telegram** (simple bot token). 13. WhatsApp 需要通过二维码配对，并且
  会在磁盘上存储更多状态。
- 群组行为因渠道而异；参见[群组](/channels/groups)。
- 为安全起见，私信配对和允许列表会被强制执行；参见[安全](/gateway/security)。
- Telegram 内部机制：[grammY 说明](/channels/grammy)。
- 故障排除：[渠道故障排除](/channels/troubleshooting)。
- 模型提供商单独记录；参见[模型提供商](/providers/models)。


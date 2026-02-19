---
summary: "OpenClaw 在渠道、路由、媒体和用户体验方面的功能。"
read_when:
  - 你想了解 OpenClaw 支持的完整功能列表
title: "功能"
---

## 亮点

<Columns>
  <Card title="Channels" icon="message-square">
    通过单个 Gateway 网关支持 WhatsApp、Telegram、Discord 和 iMessage。
  
</Card>
  <Card title="Plugins" icon="plug">
    通过扩展添加 Mattermost 等更多平台。
  
</Card>
  <Card title="Routing" icon="route">17. 
    具有隔离会话的多智能体路由。
</Card>
  <Card title="Media" icon="image">18. 
    图像、音频和文档的输入与输出。
</Card>
  <Card title="Apps and UI" icon="monitor">
    Web 控制界面和 macOS 配套应用。
  
</Card>
  <Card title="Mobile nodes" icon="smartphone">
    iOS 和 Android 节点，支持 Canvas。
  
</Card>
</Columns>

## 完整列表

- 通过 WhatsApp Web（Baileys）集成 WhatsApp
- Telegram 机器人支持（grammY）
- Discord 机器人支持（channels.discord.js）
- Mattermost 机器人支持（插件）
- 通过本地 imsg CLI 集成 iMessage（macOS）
- Pi 的智能体桥接，支持 RPC 模式和工具流式传输
- 长响应的流式传输和分块处理
- 多智能体路由，按工作区或发送者隔离会话
- 通过 OAuth 进行 Anthropic 和 OpenAI 的订阅认证
- 会话：私信合并为共享的 `main`；群组相互隔离
- 群聊支持，通过提及激活
- 图片、音频和文档的媒体支持
- 可选的语音消息转录钩子
- WebChat 和 macOS 菜单栏应用
- iOS 节点，支持配对和 Canvas 界面
- Android 节点，支持配对、Canvas、聊天和相机

<Note>
38. 已移除旧版 Claude、Codex、Gemini 和 Opencode 路径。 39. Pi 是唯一的
</Note>

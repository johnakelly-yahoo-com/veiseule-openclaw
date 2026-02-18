---
title: "功能"
---

## 重點特色

<Columns>
  <Card title="Channels" icon="message-square">
    使用單一 Gateway 閘道器即可支援 WhatsApp、Telegram、Discord 與 iMessage。
  </Card>
  <Card title="Plugins" icon="plug">
透過擴充功能新增 Mattermost 及更多功能。
  </Card>
  <Card title="Routing" icon="route">
    具備隔離工作階段的多代理程式路由。
  </Card>
  <Card title="Media" icon="image">
支援影像、音訊與文件的輸入與輸出。
  </Card>
  <Card title="Apps and UI" icon="monitor">
    Web 控制 UI 與 macOS 配套應用程式。
  </Card>
  <Card title="Mobile nodes" icon="smartphone">
    支援 Canvas 的 iOS 與 Android 節點。
  </Card>
</Columns>

## 完整清單

- 透過 WhatsApp Web（Baileys）的 WhatsApp 整合
- Telegram 機器人支援（grammY）
- Discord 機器人支援（channels.discord.js）
- 支援 Mattermost 機器人（外掛）
- 透過本機 imsg CLI（macOS）的 iMessage 整合
- 以 RPC 模式為 Pi 提供代理程式橋接，支援工具串流
- 長回應的串流與分段處理
- 依工作區或寄件者進行隔離工作階段的多代理程式路由
- 透過 OAuth 為 Anthropic 與 OpenAI 提供訂閱式身分驗證
- 工作階段：私聊會合併為共享的 `main`；群組為隔離
- 支援群組聊天並以提及方式啟用
- 影像、音訊與文件的媒體支援
- 可選的語音備忘錄轉錄掛鉤
- WebChat 與 macOS 選單列應用程式
- 具備配對與 Canvas 介面的 iOS 節點
- 具備配對、Canvas、聊天與相機的 Android 節點

<Note>
已移除舊版 Claude、Codex、Gemini 與 Opencode 路徑。Pi 是唯一的
coding agent path.
</Note>

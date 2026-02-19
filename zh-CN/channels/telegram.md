---
summary: "Telegram 机器人支持状态、功能和配置"
read_when:
  - 开发 Telegram 功能或 webhook
title: "Telegram"
---

# Telegram（Bot API）

Status: production-ready for bot DMs + groups via grammY. Long polling 是默认模式；webhook 模式为可选。

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Telegram 的默认私信策略是配对。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    跨频道诊断与修复操作手册。
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    完整的频道配置模式和示例。
  
</Card>
</CardGroup>

## 快速设置（入门）

<Steps>
  <Step title="Create the bot token in BotFather">打开 Telegram 并与 **@BotFather**（[直达链接](https://t.me/BotFather)）对话。确认用户名确实是 `@BotFather`。

    ```
    运行 `/newbot`，然后按照提示操作（名称 + 以 `bot` 结尾的用户名）。
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    环境变量回退：`TELEGRAM_BOT_TOKEN=...`（仅适用于默认账户）。
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    配对码将在 1 小时后过期。
    ```

  
</Step>

  <Step title="Add the bot to a group">
    将机器人添加到你的群组，然后设置 `channels.telegram.groups` 和 `groupPolicy` 以匹配你的访问模型。
  
</Step>
</Steps>

<Note>
Token 解析顺序支持账户感知。 在实际情况下，配置值优先生效而不是环境变量回退，且 `TELEGRAM_BOT_TOKEN` 仅适用于默认账户。
</Note>

## Telegram 端设置

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Telegram 机器人默认启用**隐私模式**，这会限制它们接收哪些群组消息。
如果你的机器人必须看到*所有*群组消息，有两个选项：

    ```
    将群组中的任何消息转发给 Telegram 上的 `@userinfobot` 或 `@getidsbot` 以查看聊天 ID（负数，如 `-1001234567890`）。
    ```

  
</Accordion>

  <Accordion title="Group permissions">管理员状态在群组内设置（Telegram UI）。管理员机器人始终接收所有群组消息，因此如果需要完全可见性，请使用管理员身份。

    ```
    管理员机器人会接收所有群组消息，这对于始终在线的群组行为非常有用。
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setprivacy` — 控制机器人是否可以看到所有群组消息。
    ```

  
</Accordion>
</AccordionGroup>

## 访问控制与激活

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` 控制私聊访问：

    ```
    - `pairing`（默认）
    - `allowlist`
    - `open`（要求 `allowFrom` 包含 `"*"`）
    - `disabled`
    
    `channels.telegram.allowFrom` 接受 Telegram 数字用户 ID。支持并会规范化 `telegram:` / `tg:` 前缀。
    引导向导支持输入 `@username`，并会将其解析为数字 ID。
    如果你已升级且配置中包含 `@username` 形式的 allowlist 条目，请运行 `openclaw doctor --fix` 进行解析（尽力而为；需要 Telegram bot token）。
    
    ### 查找你的 Telegram 用户 ID
    
    更安全的方法（无需第三方机器人）：
    
    1. 私聊你的机器人。
    2. 运行 `openclaw logs --follow`。
    3. 查看 `from.id`。
    
    官方 Bot API 方法：
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    私信 `@userinfobot` 或 `@getidsbot` 并使用返回的用户 id。
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">两个独立的控制：

    ```
    `channels.telegram.groupAllowFrom`：群组发送者允许列表（id/用户名）。
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // 在此群组中始终响应
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">
    群组回复默认需要提及机器人。

    ```
    提及可以来自：
    
    - 原生 `@botusername` 提及，或
    - 在以下位置配置的提及模式：
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    会话级命令开关：
    
    - `/activation always`
    - `/activation mention`
    
    这些仅更新会话状态。如需持久化，请使用配置。
    
    持久化配置示例：
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // 或完全省略 groups
      },
    },
  },
}
```

    ```
    **隐私注意：** `@userinfobot` 是第三方机器人。如果你更倾向于其他方式，将机器人添加到群组，发送一条消息，然后使用 `openclaw logs --follow` 读取 `chat.id`，或使用 Bot API `getUpdates`。
    ```

  
</Tab>
</Tabs>

## 运行时行为

- Telegram 由 gateway 进程管理。
- 确定性路由：回复返回到 Telegram；模型不会选择渠道。
- 入站消息被规范化为共享渠道信封，包含回复上下文和媒体占位符。
- 获取群组聊天 ID .topics.<topicId> \` 下添加每话题覆盖。
- 私聊在某些边缘情况下可能包含 `message_thread_id`。OpenClaw 保持私信会话键不变，但在存在线程 id 时仍将其用于回复/草稿流式传输。
- 长轮询使用 grammY runner，每个聊天按顺序处理；总体并发受 `agents.defaults.maxConcurrent` 限制。 整体 runner sink 并发由 `agents.defaults.maxConcurrent` 控制。
- Telegram Bot API 不支持已读回执；没有 `sendReadReceipts` 选项。

## 功能参考

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw 可以在 Telegram 私信中使用 `sendMessageDraft` 流式传输部分回复。

    ```
    要求：
    
    - `channels.telegram.streamMode` 不是 `"off"`（默认：`"partial"`）
    
    模式：
    
    - `off`：不进行实时预览
    - `partial`：基于部分文本进行高频预览更新
    - `block`：使用 `channels.telegram.draftChunk` 进行分块预览更新
    
    `streamMode: "block"` 时的 `draftChunk` 默认值：
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` 会受到 `channels.telegram.textChunkLimit` 的限制。
    
    这在私聊和群组/话题中均可用。
    
    对于纯文本回复，OpenClaw 会保持同一条预览消息，并在最后原地编辑完成（不会发送第二条消息）。
    
    对于复杂回复（例如媒体负载），OpenClaw 会回退为正常的最终发送方式，然后清理预览消息。
    
    `streamMode` 与分块流式传输是分开的。当为 Telegram 显式启用分块流式传输时，OpenClaw 会跳过预览流以避免双重流式输出。
    
    仅限 Telegram 的推理流：
    
    - `/reasoning stream` 会在生成过程中将推理内容发送到实时预览
    - 最终答案发送时不包含推理文本
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">出站 Telegram 文本使用 `parse_mode: "HTML"`（Telegram 支持的标签子集）。

    ```
    来自模型的原始 HTML 会被转义，以避免 Telegram 解析错误。
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Telegram 命令菜单注册在启动时通过 `setMyCommands` 处理。

    ```
    原生命令默认值：
    
    - `commands.native: "auto"` 为 Telegram 启用原生命令
    
    添加自定义命令菜单项：
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git 备份" },
        { command: "generate", description: "创建图片" },
      ],
    },
  },
}
```

    ```
    规则：
    
    - 名称会被规范化（去除前导 `/`，转为小写）
    - 有效模式：`a-z`、`0-9`、`_`，长度 `1..32`
    - 自定义命令不能覆盖原生命令
    - 冲突/重复项会被跳过并记录日志
    
    说明：
    
    - 自定义命令仅为菜单项；不会自动实现其行为
    - 即使未显示在 Telegram 菜单中，插件/skill 命令在手动输入时仍可生效
    
    如果禁用原生命令，内置命令会被移除。若已配置，自定义/插件命令仍可注册。
    
    常见设置失败：
    
    - `setMyCommands failed` 通常表示无法访问 `api.telegram.org`（出站 DNS/HTTPS 被阻止）。
    
    ### 设备配对命令（`device-pair` 插件）
    
    安装 `device-pair` 插件后：
    
    1. `/pair` 生成设置代码
    2. 在 iOS 应用中粘贴代码
    3. `/pair approve` 批准最新的待处理请求
    
    更多详情： [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios)。
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    配置内联键盘作用域：

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    对于每账户配置：
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    默认：`allowlist`。
    旧版：`capabilities: ["inlineButtons"]` = `inlineButtons: "all"`。
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "选择一个选项：",
  buttons: [
    [
      { text: "是", callback_data: "yes" },
      { text: "否", callback_data: "no" },
    ],
    [{ text: "取消", callback_data: "cancel" }],
  ],
}
```

    ```
    回调点击会作为文本传递给 agent：
    `callback_data: <value>`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram 工具操作包括：

    ```
    工具：`telegram`，使用 `react` 动作（`chatId`、`messageId`、`emoji`）。
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram 通过标签支持可选的线程回复：

    ```
    通用话题（线程 id `1`）是特殊的：消息发送省略 `message_thread_id`（Telegram 会拒绝），但输入指示器仍然包含它。
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">话题（论坛超级群组）

    ```
    **论坛群组：** 论坛群组中的反应包含 `message_thread_id`，使用类似 `agent:main:telegram:group:{chatId}:topic:{threadId}` 的会话键。这确保同一话题中的反应和消息保持在一起。
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### 语音消息

    ```
    `[[audio_as_voice]]` — 将音频作为语音备忘录而不是文件发送。
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    
        ### 视频消息
    
    Telegram 区分视频文件和视频笔记。
    
    消息操作示例：
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    视频笔记不支持标题；提供的消息文本将单独发送。
    
    ### 贴纸
    
    入站贴纸处理：
    
    - 静态 WEBP：下载并处理（占位符 `<media:sticker>`）
    - 动画 TGS：跳过
    - 视频 WEBM：跳过
    
    贴纸上下文字段：
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    贴纸缓存文件：
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    贴纸会在可能的情况下描述一次并缓存，以减少重复的视觉调用。
    
    启用贴纸操作：
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    发送贴纸
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    贴纸缓存
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "猫 挥手",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">从 Telegram API 接收 `message_reaction` 更新

    ```
    启用后，OpenClaw 会将系统事件加入队列，例如：
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    配置：
    
    - `channels.telegram.reactionNotifications`: `off | own | all`（默认：`own`）
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive`（默认：`minimal`）
    
    说明：
    
    - `own` 表示仅针对用户对机器人发送消息的反应（基于已发送消息缓存，尽力实现）。
    - Telegram 在反应更新中不提供线程 ID。
      - 非论坛群组路由到群组会话
      - 论坛群组路由到群组的通用话题会话（`:topic:1`），而不是原始话题
    
    轮询/webhook 的 `allowed_updates` 会自动包含 `message_reaction`。
    ```

  
</Accordion>

  <Accordion title="Ack reactions">Telegram 可以在智能体生成响应时流式传输**草稿气泡**。
OpenClaw 使用 Bot API `sendMessageDraft`（不是真实消息），然后将最终回复作为普通消息发送。

    ```
    解析顺序：
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agent 身份 emoji 回退（`agents.list[].identity.emoji`，否则为 "👀"）
    
    说明：
    
    - Telegram 需要使用 Unicode emoji（例如 "👀"）。
    - 使用 `""` 可为某个 channel 或 account 禁用该反应。
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    默认启用 channel 配置写入（`configWrites !== false`）。

    ```
    群组升级为超级群组，Telegram 发出 `migrate_to_chat_id`（聊天 ID 更改）。OpenClaw 可以自动迁移 `channels.telegram.groups`。
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    默认：长轮询。

    ```
    如果你的公共 URL 不同，使用反向代理并将 `channels.telegram.webhookUrl` 指向公共端点。
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    出站文本按 `channels.telegram.textChunkLimit` 分块（默认 4000）。
    可选的换行分块：设置 `channels.telegram.chunkMode="newline"` 在长度分块之前按空行（段落边界）分割。
    媒体下载/上传受 `channels.telegram.mediaMaxMb` 限制（默认 5）。
    Telegram Bot API 请求在 `channels.telegram.timeoutSeconds` 后超时（通过 grammY 默认 500）。设置较低的值以避免长时间挂起。
    群组历史上下文使用 `channels.telegram.historyLimit`（或 `channels.telegram.accounts.*.historyLimit`），回退到 `messages.groupChat.historyLimit`。设置 `0` 禁用（默认 50）。
    - 私聊历史控制：
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - 出站 Telegram API 重试可通过 `channels.telegram.retry` 配置。

    ```
    CLI 发送目标可以是数字 chat ID 或用户名：
    ```

```bash
示例：`openclaw message send --channel telegram --target 123456789 --message "hi"`。
```

  
</Accordion>
</AccordionGroup>

## 故障排除

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    如果你设置了 `channels.telegram.groups.*.requireMention=false`，Telegram 的 Bot API **隐私模式**必须禁用。
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - 当存在 `channels.telegram.groups` 时，群组必须在列表中（或包含 `"*"`）
    - 验证机器人是否为该群组成员
    - 查看日志：`openclaw logs --follow` 以获取跳过原因
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    日志中出现 `setMyCommands failed` 通常意味着到 `api.telegram.org` 的出站 HTTPS/DNS 被阻止。
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + 自定义 fetch/proxy 在 AbortSignal 类型不匹配时可能会触发立即中止行为。
    - 某些主机会优先将 `api.telegram.org` 解析为 IPv6；若 IPv6 出站异常，可能导致 Telegram API 间歇性失败。
    - 验证 DNS 解析结果：
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

更多帮助：[渠道故障排除](/channels/troubleshooting)。

## 配置参考（Telegram）

主要参考：

- `channels.telegram.enabled`：启用/禁用渠道启动。

- `channels.telegram.botToken`：机器人 token（BotFather）。

- `channels.telegram.tokenFile`：从文件路径读取 token。

- `channels.telegram.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）。

- `channels.telegram.allowFrom`：私信允许列表（id/用户名）。`open` 需要 `"*"`。 `open` requires `"*"`. `openclaw doctor --fix` 可以将旧版 `@username` 条目解析为 ID。

- `channels.telegram.groupPolicy`：`open | allowlist | disabled`（默认：allowlist）。

- 允许哪些群组\*\*（通过 `channels.telegram.groups` 的群组允许列表）： `openclaw doctor --fix` 可以将旧版 `@username` 条目解析为 ID。

- `channels.telegram.groups`：每群组默认值 + 允许列表（使用 `"*"` 作为全局默认值）。
  - `channels.telegram.accounts.<account>`"disabled"`= 不接受任何群组消息
      默认是`groupPolicy: "allowlist"`（除非添加 `groupAllowFrom\` 否则被阻止）。
  - `channels.telegram.groups.<id>.requireMention`：提及门控默认值。
  - `channels.telegram.groups.<id>.skills`：skill 过滤器（省略 = 所有 skills，空 = 无）。
  - `channels.telegram.groups.<id>.allowFrom`：每群组发送者允许列表覆盖。
  - `channels.telegram.groups.<id>.systemPrompt`：群组的额外系统提示。
  - `channels.telegram.groups.<id>.enabled`：为 `false` 时禁用群组。
  - .topics.<threadId>`channels.telegram.groups.<id>.*`：每话题覆盖（与群组相同的字段）。
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`：每话题提及门控覆盖。
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic mention gating override.

- `channels.telegram.capabilities.inlineButtons`：`off | dm | group | all | allowlist`（默认：allowlist）。

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`：每账户覆盖。

- `channels.telegram.replyToMode`：`off | first | all`（默认：`off`）。

- `channels.telegram.textChunkLimit`：出站分块大小（字符）。

- `channels.telegram.chunkMode`：`length`（默认）或 `newline` 在长度分块之前按空行（段落边界）分割。

- `channels.telegram.linkPreview`：切换出站消息的链接预览（默认：true）。

- `channels.telegram.streamMode: "off" | "partial" | "block"`（默认：`partial`）

- `channels.telegram.mediaMaxMb`：入站/出站媒体上限（MB）。

- `channels.telegram.retry`：出站 Telegram API 调用的重试策略（attempts、minDelayMs、maxDelayMs、jitter）。

- `channels.telegram.network.autoSelectFamily`：覆盖 Node autoSelectFamily（true=启用，false=禁用）。在 Node 22 上默认禁用以避免 Happy Eyeballs 超时。 Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`：Bot API 调用的代理 URL（SOCKS/HTTP）。

- `channels.telegram.webhookUrl`：启用 webhook 模式（需要 `channels.telegram.webhookSecret`）。

- `channels.telegram.webhookSecret`：webhook 密钥（设置 webhookUrl 时必需）。

- `channels.telegram.webhookPath`：本地 webhook 路径（默认 `/telegram-webhook`）。

- 本地监听器绑定到 `0.0.0.0:8787`，默认服务于 `POST /telegram-webhook`。

- `channels.telegram.actions.reactions`：门控 Telegram 工具反应。

- `channels.telegram.actions.sendMessage`：门控 Telegram 工具消息发送。

- `channels.telegram.actions.deleteMessage`：门控 Telegram 工具消息删除。

- `channels.telegram.actions.sticker`：门控 Telegram 贴纸动作 — 发送和搜索（默认：false）。

- `channels.telegram.reactionNotifications`：`off | own | all` — 控制哪些反应触发系统事件（未设置时默认：`own`）。

- `channels.telegram.reactionLevel`：`off | ack | minimal | extensive` — 控制智能体的反应能力（未设置时默认：`minimal`）。

- 完整配置：[配置](/gateway/configuration)

Telegram 特有的高信号字段：

- startup/auth：`enabled`、`botToken`、`tokenFile`、`accounts.*`
- 即使在 `groupPolicy: "open"` 的群组中，命令也需要授权
- command/menu：`commands.native`、`customCommands`
- threading/replies：`replyToMode`
- 可选（仅用于 `streamMode: "block"`）：
- formatting/delivery：`textChunkLimit`、`chunkMode`、`linkPreview`、`responsePrefix`
- media/network：`mediaMaxMb`、`timeoutSeconds`、`retry`、`network.autoSelectFamily`、`proxy`
- Webhook 模式：设置 `channels.telegram.webhookUrl` 和 `channels.telegram.webhookSecret`（可选 `channels.telegram.webhookPath`）。
- actions/capabilities：`capabilities.inlineButtons`、`actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- 反应通知
- writes/history：`configWrites`、`historyLimit`、`dmHistoryLimit`、`dms.*.historyLimit`

## 相关

- channels/telegram.md
- [Channel routing](/channels/channel-routing)
- `channels.telegram.streamMode`：`off | partial | block`（草稿流式传输）。


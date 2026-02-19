---
summary: "Discord 机器人支持状态、功能和配置"
read_when:
  - 开发 Discord 渠道功能时
title: "Discord"
---

# Discord（Bot API）

状态：已通过官方 Discord gateway 支持 DM 和服务器频道。

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DM 默认进入配对模式。
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    原生命令行为和命令目录。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    跨频道诊断与修复流程。
  
</Card>
</CardGroup>

## 快速设置（新手）

<Steps>
  <Step title="Create a Discord bot and enable intents">在 Discord Developer Portal 中创建一个应用，添加一个 bot，然后启用：

    ```
    **Server Members Intent**（推荐；服务器中的某些成员/用户查找和允许列表匹配需要）
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    默认账户的环境变量回退：
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">使用所需权限邀请机器人到你的服务器，以便在你想使用的地方读取/发送消息。

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    配对码将在 1 小时后过期。
    ```

  
</Step>
</Steps>

<Note>
Token 解析与账户相关。 配置中的 token 值优先生效，其次才是环境变量回退。 `DISCORD_BOT_TOKEN` 仅用于默认账户。
</Note>

## 运行时模型

- Gateway 负责维护 Discord 连接。
- 回复路由是确定性的：来自 Discord 的入站消息将回复到 Discord。
- 直接聊天会合并到智能体的主会话（默认 `agent:main:main`）；服务器频道保持隔离为 `agent:<agentId>:discord:channel:<channelId>`（显示名称使用 `discord:<guildSlug>#<channelSlug>`）。
- 服务器频道使用独立的会话键（`agent:<agentId>:discord:channel:<channelId>`）。
- 群组私信默认被忽略；通过 `channels.discord.dm.groupEnabled` 启用，并可选择通过 `channels.discord.dm.groupChannels` 进行限制。
- 原生命令使用隔离的会话键（`agent:<agentId>:discord:slash:<userId>`）而不是共享的 `main` 会话。

## 访问控制与路由

<Tabs>
  <Tab title="DM policy">要保持旧的"对任何人开放"行为：设置 `channels.discord.dm.policy="open"` 和 `channels.discord.dm.allowFrom=["*"]`。

    ```
    要使用硬编码允许列表：设置 `channels.discord.dm.policy="allowlist"` 并在 `channels.discord.dm.allowFrom` 中列出发送者。
    ```

  
</Tab>

  <Tab title="Guild policy">行为由 `channels.discord.replyToMode` 控制：

    ```
    原生命令遵循与私信/服务器消息相同的允许列表（`channels.discord.dm.allowFrom`、`channels.discord.guilds`、每频道规则）。
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    如果你只设置了 `DISCORD_BOT_TOKEN` 而从未创建 `channels.discord` 部分，运行时会将 `groupPolicy` 默认为 `open`。添加 `channels.discord.groupPolicy`、`channels.defaults.groupPolicy` 或服务器/频道允许列表来锁定它。
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    默认情况下，服务器消息需要提及才会触发。


    ```
    提及检测包括：
    
    - 明确提及 bot
    - 已配置的提及模式（`agents.list[].groupChat.mentionPatterns`，回退为 `messages.groupChat.mentionPatterns`）
    - 在支持的情况下，对 bot 的隐式回复行为
    
    `requireMention` 可按服务器/频道配置（`channels.discord.guilds...`）。
    
    群组 DM：
    
    - 默认：忽略（`dm.groupEnabled=false`）
    - 可选允许列表，通过 `dm.groupChannels`（频道 ID 或 slug）配置
    ```

  
</Tab>
</Tabs>

### 基于角色的 agent 路由

使用 `bindings[].match.roles` 根据角色 ID 将 Discord 服务器成员路由到不同的 agent。 基于角色的绑定仅接受角色 ID，并在 peer 或父级 peer 绑定之后、仅 guild 绑定之前进行评估。 如果一个绑定还设置了其他匹配字段（例如 `peer` + `guildId` + `roles`），则所有已配置的字段都必须匹配。

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Developer Portal 设置

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    创建 Discord 应用程序 → Bot，启用你需要的意图（私信 + 服务器消息 + 消息内容），并获取机器人令牌。
    ```

  
</Accordion>

  <Accordion title="Privileged intents">在 **Bot** → **Privileged Gateway Intents** 中启用：

    ```
    - Message Content Intent
    - Server Members Intent（推荐）
    
    Presence intent 为可选，仅当你希望接收在线状态更新时才需要启用。设置 bot 在线状态（`setPresence`）不需要为成员启用在线状态更新。
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">在你的应用中：**OAuth2** → **URL Generator**

    ```
    - scopes：`bot`、`applications.commands`
    
    典型的基础权限：
    
    - 查看频道
    - 发送消息
    - 读取消息历史
    - 嵌入链接
    - 附加文件
    - 添加反应（可选）
    
    除非明确需要，否则避免使用 `Administrator`。
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    启用 Discord Developer Mode，然后复制：

    ```
    - 服务器 ID
    - 频道 ID
    - 用户 ID
    
    在 OpenClaw 配置中优先使用数字 ID，以获得更可靠的审计和探测结果。
    ```

  
</Accordion>
</AccordionGroup>

## 原生命令与命令授权

- `commands.native` 默认为 `"auto"`，并在 Discord 上启用。
- 你的配置中有 `channels.discord.execApprovals.enabled: true`。
- `commands.native=false` 会显式清除之前已注册的 Discord 原生命令。
- 原生命令的授权使用与普通消息处理相同的 Discord 允许列表/策略。
- 斜杠命令可能在 Discord UI 中对未在允许列表中的用户仍然可见；OpenClaw 在执行时强制执行允许列表并回复"未授权"。

请参阅 [Slash commands](/tools/slash-commands) 了解命令目录和行为。

## 功能详情

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord 支持在代理输出中使用回复标签：

    ```
    原生回复线程**默认关闭**；使用 `channels.discord.replyToMode` 和回复标签启用。
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    服务器历史上下文：

    ```
    可选服务器上下文历史：设置 `channels.discord.historyLimit`（默认 20，回退到 `messages.groupChat.historyLimit`）以在回复提及时包含最近 N 条服务器消息作为上下文。设置 `0` 禁用。
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`：

    ```
    - `off`
    - `own`（默认）
    - `all`
    - `allowlist`（使用 `guilds.<id>.users`）
    
    反应事件会被转换为系统事件，并附加到已路由的 Discord 会话中。
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` 会在 OpenClaw 处理传入消息时发送一个确认表情符号。

    ```
    解析顺序：
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - 代理身份表情符号回退（`agents.list[].identity.emoji`，否则为 "👀"）
    
    说明：
    
    - Discord 接受 Unicode 表情符号或自定义表情名称。
    - 使用 `""` 可在频道或账户级别禁用该反应。
    ```

  
</Accordion>

  <Accordion title="Config writes">
    默认启用由频道发起的配置写入。

    ```
    这会影响 `/config set|unset` 流程（在启用命令功能时）。
    
    禁用：
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    通过 `channels.discord.proxy` 将 Discord gateway WebSocket 流量路由到 HTTP(S) 代理。

```json5
服务器频道：发送时使用 `channel:<channelId>`。默认需要提及，可以按服务器或按频道设置。
```

    ```
    按账户覆盖：
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    启用 PluralKit 解析，将代理消息映射到系统成员身份：

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // 可选；私有系统需要
      },
    },
  },
}
```

    ```
    说明：
    
    - allowlists 可使用 `pk:<memberId>`
    - 成员显示名称按名称/slug 匹配
    - 查找使用原始消息 ID，并受时间窗口限制
    - 如果查找失败，代理消息将被视为机器人消息，并在未设置 `allowBots=true` 时被丢弃
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    仅在设置了状态或活动字段时才会应用在线状态更新。

    ```
    仅状态示例：
    ```

```json5
或配置：`channels.discord.token: "..."`。
```

    ```
    活动示例（自定义状态为默认活动类型）：
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    直播示例：
    ```

```json5
直接聊天：默认通过 `channels.discord.dm.policy` 进行安全保护（默认：`"pairing"`）。未知发送者会收到配对码（1 小时后过期）；通过 `openclaw pairing approve discord <code>` 批准。
```

    ```
    活动类型映射：
    
    - 0：Playing
    - 1：Streaming（需要 `activityUrl`）
    - 2：Listening
    - 3：Watching
    - 4：Custom（使用活动文本作为状态内容；可选表情）
    - 5：Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord 支持在私信中基于按钮的 exec 审批，并可选择在发起频道中发布审批提示。

    ```
    配置路径：
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target`（`dm` | `channel` | `both`，默认：`dm`）
    - `agentFilter`、`sessionFilter`、`cleanupAfterResolve`
    
    当 `target` 为 `channel` 或 `both` 时，审批提示将在频道中可见。只有已配置的 approvers 可以使用按钮；其他用户将收到仅自己可见的拒绝提示。审批提示包含命令文本，因此仅应在受信任的频道中启用频道投递。如果无法从会话键派生频道 ID，OpenClaw 将回退为通过私信投递。
    
    如果审批因未知的 approval ID 失败，请检查 approver 列表和功能是否已启用。
    
    相关文档：[Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## 工具操作

Discord 消息操作包括消息发送、频道管理、内容审核、在线状态以及元数据相关操作。

核心示例：

- `readMessages`、`sendMessage`、`editMessage`、`deleteMessage`
- `react` / `reactions`（添加或列出表情反应）
- `timeout`、`kick`、`ban`
- presence：`setPresence`

操作开关位于 `channels.discord.actions.*`。

默认开关行为：

| 操作组                                                                                                                                                       | 默认 |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | -- |
| reactions、messages、threads、pins、polls、search、memberInfo、roleInfo、channelInfo、channels、voiceStatus、events、stickers、emojiUploads、stickerUploads、permissions | 启用 |
| roles                                                                                                                                                     | 禁用 |
| moderation                                                                                                                                                | 启用 |
| presence                                                                                                                                                  | 禁用 |

## Components v2 UI

OpenClaw 使用 Discord components v2 实现 exec 审批和跨上下文标记。 Discord 消息操作也可以接受 `components` 以实现自定义 UI（高级用法；需要 Carbon 组件实例），同时仍支持旧版 `embeds`，但不推荐使用。

- `channels.discord.ui.components.accentColor` 用于设置 Discord 组件容器使用的强调色（十六进制）。
- 可按账户通过 `channels.discord.accounts.<id> .ui.components.accentColor` 进行设置。
- 当存在 components v2 时，将忽略 `embeds`。

示例：

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## messages

Discord 语音消息会显示波形预览，并且需要 OGG/Opus 音频及相关元数据。 OpenClaw 会自动生成波形，但需要在 gateway 主机上提供 `ffmpeg` 和 `ffprobe` 以检查和转换音频文件。

要求和限制：

- 请提供**本地文件路径**（不接受 URL）。
- 省略文本内容（Discord 不允许在同一负载中同时发送文本 + 语音消息）。
- 接受任意音频格式；OpenClaw 会在需要时转换为 OGG/Opus。

示例：

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## 故障排除

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    **"Used disallowed intents"**：在开发者门户中启用 **Message Content Intent**（可能还需要 **Server Members Intent**），然后重启 Gateway 网关。
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `groupPolicy`：控制服务器频道处理（`open|disabled|allowlist`）；`allowlist` 需要频道允许列表。
    ```

```bash
首先：运行 `openclaw doctor` 和 `openclaw channels status --probe`（可操作的警告 + 快速审计）。
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    常见原因：

    ```
    要**不允许任何频道**，设置 `channels.discord.groupPolicy: "disabled"`（或保持空允许列表）。
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` 权限检查仅适用于数字频道 ID。

    ```
    如果使用 slug 键，运行时匹配仍然可以生效，但 probe 无法完全验证权限。
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **私信不工作**：`channels.discord.dm.enabled=false`、`channels.discord.dm.policy="disabled"`，或者你尚未被批准（`channels.discord.dm.policy="pairing"`）。
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">机器人发送的消息默认被忽略；设置 `channels.discord.allowBots=true` 允许它们（自己的消息仍被过滤）。

    ```
    如果设置 `channels.discord.allowBots=true`，请使用严格的提及和允许列表规则以避免循环行为。
    ```

  
</Accordion>
</AccordionGroup>

## 配置参考指引

主要参考：

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

高频 Discord 字段：

- startup/auth：`enabled`、`token`、`accounts.*`、`allowBots`
- `guilds.<id> .channels.<channel> .allow`：当 `groupPolicy="allowlist"` 时允许/拒绝频道。
- command：`commands.native`、`commands.useAccessGroups`、`configWrites`
- `dmHistoryLimit`：私信历史限制（用户轮次）。每用户覆盖：`dms["<user_id>"].historyLimit`。
- delivery：`textChunkLimit`、`chunkMode`、`maxLinesPerMessage`
- media/retry：`mediaMaxMb`、`retry`
- actions：`actions.*`
- presence：`activity`、`status`、`activityType`、`activityUrl`
- UI：`ui.components.accentColor`
- features：`pluralkit`、`execApprovals`、`intents`、`agentComponents`、`heartbeat`、`responsePrefix`

## 安全与运维

- 将 bot token 视为机密（在受监管环境中优先使用 `DISCORD_BOT_TOKEN`）。
- 授予最小权限的 Discord 权限。
- 如果命令部署/状态已过期，请重启 gateway，并使用 `openclaw channels status --probe` 重新检查。

## 相关

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- 完整命令列表 + 配置：[斜杠命令](/tools/slash-commands)


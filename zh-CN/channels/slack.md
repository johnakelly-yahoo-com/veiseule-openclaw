---
summary: "Slack 的 socket 或 HTTP webhook 模式设置"
read_when:
  - Setting up Slack or debugging Slack socket/HTTP mode
title: "Slack"
---

# Slack

状态：已可用于生产环境的 DMs + 通过 Slack 应用集成的 channels。 HTTP 模式（Events API）

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DMs 默认处于配对模式。
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    原生命令行为和命令目录。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    跨渠道诊断和修复操作手册。
  
</Card>
</CardGroup>

## 快速设置（新手）

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">在 https://api.slack.com/apps 创建一个 Slack 应用（从头开始）。

        ```
        **Socket Mode** → 开启。然后前往 **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes**，添加 `connections:write` 权限范围。复制 **App Token**（`xapp-...`）。
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Env 回退（仅默认账户）：
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Subscribe app events">
          订阅以下 bot 事件：
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          同时为 DMs 启用 App Home **Messages Tab**。
        
</Step>
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        **Event Subscriptions** → 启用事件并将 **Request URL** 设置为你的 Gateway 网关 webhook 路径（默认 `/slack/events`）。
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      多账户 HTTP 模式：设置 `channels.slack.accounts.<id> .mode = "http"` 并为每个账户提供唯一的 `webhookPath`，以便每个 Slack 应用可以指向自己的 URL。

  
</Tab>
</Tabs>

## 令牌模型

- `botToken` + `appToken` 是 Socket Mode 所必需的。
- HTTP 模式需要 `botToken` + `signingSecret`。
- 配置中的令牌会覆盖环境变量回退值。
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` 环境变量回退仅适用于默认账户。
- 明确设置 userTokenReadOnly 的示例（允许用户令牌写入）：
- 可选：如果希望发送的消息使用当前活动代理身份（自定义 `username` 和图标），请添加 `chat:write.customize`。 `icon_emoji` 使用 `:emoji_name:` 语法。

<Tip>
对于操作/目录读取，在已配置时可以优先使用用户令牌。 设置 `userTokenReadOnly: false` 允许在 bot 令牌不可用时使用用户令牌进行写入操作，这意味着操作以安装用户的访问权限运行。将用户令牌视为高权限，并保持操作门控和白名单严格。
</Tip>

## 访问控制与路由

<Tabs>
  <Tab title="DM policy">要允许任何人：设置 `channels.slack.dm.policy="open"` 和 `channels.slack.dm.allowFrom=["*"]`。

    ```
    Slash Commands → 如果你使用 `channels.slack.slashCommand`，创建 `/openclaw`。如果启用原生命令，为每个内置命令添加一个斜杠命令（名称与 `/help` 相同）。除非你设置 `channels.slack.commands.native: true`，否则 Slack 默认关闭原生命令（全局 `commands.native` 是 `"auto"`，对 Slack 保持关闭）。
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` 控制频道处理（`open|disabled|allowlist`）。

    ```
    如果你只设置了 `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` 而从未创建 `channels.slack` 部分，运行时默认将 `groupPolicy` 设为 `open`。添加 `channels.slack.groupPolicy`、`channels.defaults.groupPolicy` 或频道白名单来锁定它。
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    频道消息默认需要被提及才会触发。

    ```
    提及门控通过 `channels.slack.channels` 控制（将 `requireMention` 设置为 `true`）；`agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）也算作提及。
    ```

  
</Tab>
</Tabs>

## 命令与斜杠命令行为

- 原生命令注册使用 `commands.native`（全局默认 `"auto"` → Slack 关闭），可以使用 `channels.slack.commands.native` 按工作空间覆盖。文本命令需要独立的 `/...` 消息，可以使用 `commands.text: false` 禁用。Slack 斜杠命令在 Slack 应用中管理，不会自动移除。使用 `commands.useAccessGroups: false` 绕过命令的访问组检查。
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- 启用原生命令时，请在 Slack 中注册匹配的斜杠命令（`/<command>` 名称）。
- 如果未启用原生命令，可以通过 `channels.slack.slashCommand` 运行单个已配置的斜杠命令。

默认斜杠命令设置：

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

斜杠命令会话使用隔离的键：

- 斜杠命令使用 `agent:<agentId>:slack:slash:<userId>` 会话（前缀可通过 `channels.slack.slashCommand.sessionPrefix` 配置）。

并且仍会将命令执行路由到目标会话（`CommandTargetSessionKey`）。

## 线程、会话与回复标签

- 私信路由为 `direct`；频道为 `channel`；MPIM 为 `group`。
- 在默认 `session.dmScope=main` 下，Slack 私信会合并到代理的主会话。
- 频道映射到 `agent:<agentId>:slack:channel:<channelId>` 会话。
- 在线程回复时，在适用情况下可创建线程会话后缀（`:thread:<threadTs>`）。
- `channels.slack.thread.historyScope` 默认值为 `thread`；`thread.inheritParent` 默认值为 `false`。
- `channels.slack.historyLimit`（或 `channels.slack.accounts.*.historyLimit`）控制将多少条最近的频道/群组消息包含到提示中。

回复线程控制：

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- 当未设置聊天类型覆盖时，旧版 `channels.slack.dm.replyToMode` 仍可作为 `direct` 的回退。

支持手动回复标签：

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — 回复特定的消息 id。

注意：`replyToMode="off"` 会禁用隐式回复线程。 显式的 `[[reply_to_*]]` 标签仍会生效。

## 媒体、分块与投递

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack 文件附件会通过 Slack 托管的私有 URL 下载（基于令牌认证的请求流程），在获取成功且满足大小限制时写入媒体存储。


    ```
    媒体上传受 `channels.slack.mediaMaxMb` 限制（默认 20）。
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">    - 文本分块使用 `channels.slack.textChunkLimit`（默认 4000）
    - `channels.slack.chunkMode="newline"` 启用按段落优先拆分
    - 文件发送使用 Slack 上传 API，并可包含线程回复（`thread_ts`）
    - 若已配置，出站媒体大小上限遵循 `channels.slack.mediaMaxMb`；否则频道发送使用媒体管道中的 MIME 类型默认值
  
</Accordion>

  <Accordion title="Delivery targets">
    首选显式目标：

    ```
    - `user:<id>` 用于私信
    - `channel:<id>` 用于频道
    
    向用户目标发送消息时，将通过 Slack 会话 API 打开私信。
    ```

  
</Accordion>
</AccordionGroup>

## 操作与门控

Slack 工具操作可以通过 `channels.slack.actions.*` 进行门控：

当前 Slack 工具中可用的操作分组：

| 操作组        | 默认  |
| ---------- | --- |
| messages   | 已启用 |
| reactions  | 已启用 |
| pins       | 已启用 |
| memberInfo | 已启用 |
| emojiList  | 已启用 |

## 事件与运行行为

- 消息编辑/删除/线程广播会映射为系统事件。
- 添加/移除反应事件会映射为系统事件。
- 成员加入/离开、频道创建/重命名以及固定消息添加/移除事件会被映射为系统事件。
- 当启用 `configWrites` 时，`channel_id_changed` 可以迁移频道配置键。
- 频道主题/用途元数据被视为不受信任的上下文，并可能被注入到路由上下文中。

## Ack 反应

`ackReaction` 会在 OpenClaw 处理入站消息时发送一个确认表情。

解析顺序：

- `或`channels.slack.channels.<name>.ackReaction\`
- `channels.slack.ackReaction`
- `replyToMode`
- agent 身份表情回退（`agents.list[].identity.emoji`，否则为 "👀"）

注意：

- Slack 需要使用短代码（例如 `"eyes"`）。
- 使用 `""` 可为某个频道或账户禁用该反应。

## Manifest 和 scope 检查清单

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">如果你配置了 `channels.slack.userToken`，在 **User Token Scopes** 下添加这些。

    ```
    `channels:history`、`groups:history`、`im:history`、`mpim:history`
    https://docs.slack.dev/reference/methods/conversations.history
    ```

  
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="No replies in channels">
    按顺序检查：

    ```
    `allowlist` 要求频道列在 `channels.slack.channels` 中。
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    检查：

    ```
    要**不允许任何频道**，设置 `channels.slack.groupPolicy: "disabled"`（或保留空白名单）。
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">创建一个 Slack 应用并启用 **Socket Mode**。
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    验证：

    ```
    当你的 Gateway 网关可以通过 HTTPS 被 Slack 访问时（服务器部署的典型情况），使用 HTTP webhook 模式。
    HTTP 模式使用 Events API + Interactivity + Slash Commands，共享一个请求 URL。
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    确认你是否有意：

    ```
    如果启用原生命令，为每个要公开的命令添加一个 `slash_commands` 条目（与 `/help` 列表匹配）。使用 `channels.slack.commands.native` 覆盖。
    ```

  
</Accordion>
</AccordionGroup>

## 配置参考指引

优先级：

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  高信号 Slack 字段：

  - 模式/认证：`mode`、`botToken`、`appToken`、`signingSecret`、`webhookPath`、`accounts.*`
  - 私信访问：`dm.enabled`、`dmPolicy`、`allowFrom`（旧版：`dm.policy`、`dm.allowFrom`）、`dm.groupEnabled`、`dm.groupChannels`
  - `allow`：当 `groupPolicy="allowlist"` 时允许/拒绝频道。
  - 回退到 `messages.groupChat.historyLimit`。设置为 `0` 以禁用（默认 50）。
  - 消息传递：`textChunkLimit`、`chunkMode`、`mediaMaxMb`
  - 运维/功能：`configWrites`、`commands.native`、`slashCommand.*`、`actions.*`、`userToken`、`userTokenReadOnly`

## 相关

- channels/slack.md
- `channel`：标准频道（公开/私有）
- [Troubleshooting](/channels/troubleshooting)
- [Configuration](/gateway/configuration)
- 完整命令列表 + 配置：[斜杠命令](/tools/slash-commands)


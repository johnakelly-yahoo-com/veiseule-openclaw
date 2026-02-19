---
title: "配置参考"
description: "~/.openclaw/openclaw.json 的完整逐字段参考"
---

# 配置参考

`~/.openclaw/openclaw.json` 中可用的所有字段。 有关面向任务的概览，请参见 [Configuration](/gateway/configuration)。

配置格式为 **JSON5**（允许注释和尾随逗号）。 所有字段均为可选——在省略时，OpenClaw 会使用安全的默认值。

---

## Channels

当配置中存在对应部分时，每个 channel 会自动启动（除非设置了 `enabled: false`）。

### 私信和群组访问

所有 channel 都支持私信策略和群组策略：

| 私信策略          | 行为                                 |
| ------------- | ---------------------------------- |
| `pairing`（默认） | 未知发送者会收到一次性配对码；需由所有者批准             |
| `allowlist`   | 仅允许在 `allowFrom`（或已配对的允许列表存储）中的发送者 |
| `open`        | 允许所有传入私信（需要设置 `allowFrom: ["*"]`）  |
| `disabled`    | 忽略所有传入私信                           |

| 群组策略            | 行为                 |
| --------------- | ------------------ |
| `allowlist`（默认） | 仅允许匹配已配置允许列表的群组    |
| `open`          | 绕过群组允许列表（仍然适用提及限制） |
| `disabled`      | 阻止所有群组/房间消息        |

<Note>
`channels.defaults.groupPolicy` 在 provider 未设置 `groupPolicy` 时作为默认值。
配对码将在 1 小时后过期。 待处理的私信配对请求每个 channel 最多 **3 个**。
Slack/Discord 有一个特殊回退机制：如果其 provider 配置部分完全缺失，运行时群组策略可能解析为 `open`（启动时会发出警告）。
</Note>

### WhatsApp

WhatsApp 通过 gateway 的 web channel（Baileys Web）运行。 当存在已链接的会话时会自动启动。

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // 蓝色已读标记（自聊模式下为 false）
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- 出站命令默认使用 `default` 账户（如果存在）；否则使用第一个已配置的账户 id（按排序顺序）。
- 旧版单账户 Baileys 认证目录会由 `openclaw doctor` 迁移到 `whatsapp/default`。
- 按账户覆盖：`channels.whatsapp.accounts.<id>``.sendReadReceipts`，`channels.whatsapp.accounts.<id>``.dmPolicy`，`channels.whatsapp.accounts.<id>``.allowFrom`。

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "回答保持简洁。",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "保持主题相关。",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git 备份" },
        { command: "generate", description: "创建图像" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Bot token：`channels.telegram.botToken` 或 `channels.telegram.tokenFile`，默认账户可使用 `TELEGRAM_BOT_TOKEN` 作为回退。
- `configWrites: false` 会阻止由 Telegram 发起的配置写入（超级群组 ID 迁移、`/config set|unset`）。
- Telegram 流式预览使用 `sendMessage` + `editMessageText`（在私聊和群聊中均可使用）。
- 重试策略：参见 [Retry policy](/concepts/retry)。

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
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
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
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

- Token：`channels.discord.token`，默认账户可使用 `DISCORD_BOT_TOKEN` 作为回退。
- 投递目标请使用 `user:<id>`（私信）或 `channel:<id>`（服务器频道）；不接受纯数字 ID。
- 服务器 slug 使用小写字母并将空格替换为 `-`；频道键使用 slug 化名称（不含 `#`）。 优先使用服务器 ID。
- 默认会忽略机器人发送的消息。 `allowBots: true` 可启用这些消息（仍会过滤机器人自身的消息）。
- `maxLinesPerMessage`（默认 17）会在消息行数过多时进行拆分，即使未超过 2000 字符。
- `channels.discord.ui.components.accentColor` 设置 Discord components v2 容器的强调色。

**反应通知模式：** `off`（无），`own`（仅机器人消息，默认），`all`（所有消息），`allowlist`（来自 `guilds.<id>` `.users` 上的所有消息）。

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- Service account JSON：可内联（`serviceAccount`）或基于文件（`serviceAccountFile`）。
- 环境变量回退：`GOOGLE_CHAT_SERVICE_ACCOUNT` 或 `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
- 投递目标请使用 `spaces/<spaceId>` 或 `users/<userId|email>`。

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **Socket mode** 需要同时提供 `botToken` 和 `appToken`（默认账户可通过 `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` 作为环境变量回退）。
- **HTTP mode** 需要 `botToken` 以及 `signingSecret`（可在根级别或按账户配置）。
- `configWrites: false` 会阻止由 Slack 发起的配置写入。
- 投递目标请使用 `user:<id>`（私信）或 `channel:<id>`。

**反应通知模式：** `off`、`own`（默认）、`all`、`allowlist`（来自 `reactionAllowlist`）。

**线程会话隔离：** `thread.historyScope` 可设置为每个线程独立（默认）或在整个频道内共享。 `thread.inheritParent` 会将父频道的对话记录复制到新线程中。

| 操作组        | 默认  | 说明                |
| ---------- | --- | ----------------- |
| reactions  | 启用  | 添加反应 + 列出反应       |
| messages   | 启用  | 读取 / 发送 / 编辑 / 删除 |
| pins       | 启用  | 置顶/取消置顶/列表        |
| memberInfo | 已启用 | 成员信息              |
| emojiList  | 已启用 | 自定义表情列表           |

### Mattermost

Mattermost 以插件形式发布：`openclaw plugins install @openclaw/mattermost`。

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

聊天模式：`oncall`（在被 @ 提及时回复，默认）、`onmessage`（每条消息都回复）、`onchar`（以触发前缀开头的消息）。

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**表情反应通知模式：** `off`、`own`（默认）、`all`、`allowlist`（来自 `reactionAllowlist`）。

### iMessage

OpenClaw 会启动 `imsg rpc`（通过 stdio 的 JSON-RPC）。 无需守护进程或端口。

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- 需要对 Messages 数据库授予“完全磁盘访问”权限。
- 优先使用 `chat_id:<id>` 作为目标。 使用 `imsg chats --limit 20` 列出聊天。
- `cliPath` 可以指向一个 SSH 包装脚本；设置 `remoteHost` 以通过 SCP 获取附件。

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### 多账号（所有渠道）

每个渠道可运行多个账号（每个账号都有自己的 `accountId`）：

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- 当省略 `accountId` 时，将使用 `default`（CLI + 路由）。
- 环境变量中的 token 仅适用于 **default** 账号。
- 基础渠道设置将应用于所有账号，除非在各账号中单独覆盖。
- 使用 `bindings[].match.accountId` 将不同账号路由到不同的 agent。

### 群聊提及门控

群消息默认 **需要被提及**（元数据提及或正则模式）。 适用于 WhatsApp、Telegram、Discord、Google Chat 和 iMessage 群聊。

**提及类型：**

- **元数据提及**：平台原生的 @ 提及。 在 WhatsApp 自聊模式下会被忽略。
- **文本模式**：位于 `agents.list[].groupChat.mentionPatterns` 中的正则模式。 始终会进行检查。
- 仅当可以检测到提及时（原生提及或至少一个模式）才会强制执行提及门控。

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` 设置全局默认值。 频道可以通过 `channels.<channel>` 覆盖`.historyLimit`（或按账户单独设置）。 设置为 `0` 可禁用。

#### 私聊历史记录限制

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

优先级：单个 DM 覆盖 → 提供方默认值 → 不限制（全部保留）。

支持：`telegram`、`whatsapp`、`discord`、`slack`、`signal`、`imessage`、`msteams`。

#### 自聊模式

在 `allowFrom` 中包含你自己的号码以启用自聊模式（忽略原生 @ 提及，仅响应文本模式）：

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### 命令（聊天命令处理）

```json5
{
  commands: {
    native: "auto", // 在支持时注册原生命令
    text: true, // 在聊天消息中解析 /commands
    bash: false, // 允许 !（别名：/bash）
    bashForegroundMs: 2000,
    config: false, // 允许 /config
    debug: false, // 允许 /debug
    restart: false, // 允许 /restart + gateway 重启工具
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- 文本命令必须是以 `/` 开头的**独立**消息。
- `native: "auto"` 会为 Discord/Telegram 启用原生命令，Slack 保持关闭。
- 可按频道覆盖：`channels.discord.commands.native`（布尔值或 `"auto"`）。 `false` 会清除之前已注册的命令。
- `channels.telegram.customCommands` 添加额外的 Telegram 机器人菜单项。
- `bash: true` 启用 `! <cmd>` 用于主机 shell。 需要 `tools.elevated.enabled`，且发送者必须在 `tools.elevated.allowFrom.<channel>` 中。\`.
- `config: true` 启用 `/config`（读取/写入 `openclaw.json`）。
- `channels.<provider>``.configWrites` 控制每个频道的配置修改权限（默认：true）。
- `allowFrom` 按提供方分别设置。 设置后，它将成为**唯一**的授权来源（频道白名单/配对以及 `useAccessGroups` 将被忽略）。
- `useAccessGroups: false` 允许在未设置 `allowFrom` 时使命令绕过访问组策略。

</Accordion>

---

## Agent 默认值

### `agents.defaults.workspace`

默认值：`~/.openclaw/workspace`。

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

可选的仓库根目录，会显示在系统提示的 Runtime 行中。 如果未设置，OpenClaw 会从 workspace 向上遍历自动检测。

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

禁用自动创建 workspace 引导文件（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md`）。

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

每个工作区 bootstrap 文件在被截断前的最大字符数。 默认值：`20000`。

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

在所有工作区 bootstrap 文件中注入的最大总字符数。 默认值：`24000`。

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

系统提示上下文使用的时区（不影响消息时间戳）。 回退为主机时区。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

系统提示中的时间格式。 默认值：`auto`（操作系统偏好）。

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`：格式为 `provider/model`（例如：`anthropic/claude-opus-4-6`）。 如果省略 provider，OpenClaw 将默认使用 `anthropic`（已弃用）。
- `models`：为 `/model` 配置的模型目录和允许列表。 每个条目可以包含 `alias`（快捷名称）和 `params`（特定于 provider 的参数：`temperature`、`maxTokens`）。
- `imageModel`：仅当主模型不支持图像输入时使用。
- `maxConcurrent`：跨会话的最大并行 agent 运行数（每个会话内仍为串行）。 默认值：1。

**内置 alias 简写**（仅当模型存在于 `agents.defaults.models` 中时适用）：

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

你配置的 alias 始终优先于默认值。

Z.AI GLM-4.x 模型会自动启用 thinking 模式，除非你设置 `--thinking off` 或自行定义 `agents.defaults.models["zai/<model>"].params.thinking`。

### `agents.defaults.cliBackends`

用于纯文本回退运行（不调用工具）的可选 CLI 后端。 当 API 提供商发生故障时可作为备用方案。

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- CLI 后端以文本为主；工具始终被禁用。
- 当设置了 `sessionArg` 时支持会话。
- 当 `imageArg` 接受文件路径时支持图片透传。

### `agents.defaults.heartbeat`

周期性心跳运行。

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`：时长字符串（ms/s/m/h）。 默认值：`30m`。
- 针对单个 agent：设置 `agents.list[].heartbeat`。 当任一 agent 定义了 `heartbeat` 时，**仅这些 agent** 会运行心跳。
- 心跳会运行完整的 agent 轮次——间隔越短，消耗的 token 越多。

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`：`default` 或 `safeguard`（针对长历史记录的分块摘要）。 参见 [Compaction](/concepts/compaction)。
- `memoryFlush`：在自动压缩前执行一次静默的 agent 轮次，用于存储持久记忆。 当工作区为只读时将跳过。

### `agents.defaults.contextPruning`

在发送给 LLM 之前，从内存上下文中裁剪**旧的工具结果**。 **不会**修改磁盘上的会话历史。

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` 启用裁剪流程。
- `ttl` 控制下次可再次执行裁剪的时间（自上次缓存触达后计算）。
- 裁剪会先对过大的工具结果进行软裁剪，如有需要再对更旧的工具结果进行硬清除。

**软裁剪（Soft-trim）** 保留开头和结尾，并在中间插入 `...`。

**硬清除（Hard-clear）** 使用占位符替换整个工具结果。

注意：

- 图片块永远不会被裁剪或清除。
- 比例基于字符数（近似值），而非精确的 token 数量。
- 如果 assistant 消息少于 `keepLastAssistants` 条，则跳过裁剪。

</Accordion>

有关行为细节，请参见 [Session Pruning](/concepts/session-pruning)。

### 分块流式输出

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- 非 Telegram 渠道需要显式设置 `*.blockStreaming: true` 才能启用分块回复。
- 渠道覆盖：`channels.<channel>`.blockStreamingCoalesce`（以及按账户区分的变体）。 Signal/Slack/Discord/Google Chat 默认 `minChars: 1500\`。
- `humanDelay`：块回复之间的随机暂停。 `natural` = 800–2500ms。 按代理覆盖：`agents.list[].humanDelay`。

行为和分块详情请参见 [Streaming](/concepts/streaming)。

### 正在输入指示器

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- 默认值：私聊/被提及时为 `instant`，未被提及的群聊为 `message`。
- 按会话覆盖：`session.typingMode`、`session.typingIntervalSeconds`。

参见 [Typing Indicators](/concepts/typing-indicators)。

### `agents.defaults.sandbox`

为嵌入式代理提供可选的 **Docker 沙箱隔离**。 完整指南请参见 [Sandboxing](/gateway/sandboxing)。

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

<Accordion title="Sandbox details">

**工作区访问：**

- `none`：在 `~/.openclaw/sandboxes` 下为每个作用域创建独立的沙箱工作区
- `ro`：沙箱工作区位于 `/workspace`，代理工作区以只读方式挂载到 `/agent`
- `rw`：代理工作区以读写方式挂载到 `/workspace`

**作用域：**

- `session`：每个会话一个容器 + 工作区
- `agent`：每个代理一个容器 + 工作区（默认）
- `shared`：共享容器和工作区（无跨会话隔离）

**`setupCommand`** 在容器创建后运行一次（通过 `sh -lc`）。 需要网络出站访问、可写根文件系统以及 root 用户。

**容器默认使用 `network: "none"`** —— 如果代理需要出站访问，请设置为 `"bridge"`。

**入站附件** 会被暂存到当前工作区的 `media/inbound/*` 中。

**`docker.binds`** 用于挂载额外的主机目录；全局和按代理的绑定会合并。

**沙箱浏览器**（`sandbox.browser.enabled`）：在容器中运行的 Chromium + CDP。 noVNC URL 会注入到系统提示中。 不需要在主配置中启用 `browser.enabled`。

- `allowHostControl: false`（默认）会阻止沙箱会话访问主机浏览器。
- `sandbox.browser.binds` 仅将额外的主机目录挂载到沙箱浏览器容器中。 设置后（包括 `[]`），它将替换浏览器容器的 `docker.binds`。

</Accordion>

构建镜像：

```bash
scripts/sandbox-setup.sh           # 主沙箱镜像
scripts/sandbox-browser-setup.sh   # 可选浏览器镜像
```

### `agents.list`（按代理覆盖）

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // 或 { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`：稳定的代理 id（必填）。
- `default`：当设置多个时，以第一个为准（会记录警告日志）。 如果未设置，则默认使用列表中的第一项。
- `model`：字符串形式仅覆盖 `primary`；对象形式 `{ primary, fallbacks }` 会同时覆盖两者（`[]` 表示禁用全局 fallbacks）。
- `identity.avatar`：相对于 workspace 的路径、`http(s)` URL，或 `data:` URI。
- `identity` 派生默认值：`ackReaction` 来自 `emoji`，`mentionPatterns` 来自 `name`/`emoji`。
- `subagents.allowAgents`：用于 `sessions_spawn` 的 agent id 允许列表（`["*"]` = 任意；默认：仅限相同 agent）。

---

## 多 Agent 路由

在一个 Gateway 内运行多个相互隔离的 agent。 参见 [Multi-Agent](/concepts/multi-agent)。

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

### 绑定匹配字段

- `match.channel`（必填）
- `match.accountId`（可选；`*` = 任意账户；省略 = 默认账户）
- `match.peer`（可选；`{ kind: direct|group|channel, id }`）
- `match.guildId` / `match.teamId`（可选；特定于 channel）

**确定性的匹配顺序：**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`（精确匹配，无 peer/guild/team）
5. `match.accountId: "*"`（整个 channel 范围）
6. 默认 agent

在每一层级内，第一个匹配的 `bindings` 条目生效。

### 按 agent 配置的访问策略

<Accordion title="Full access (no sandbox)">

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

</Accordion>

有关优先级的详细说明，请参见 [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools)。

---

## 会话

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // legacy (runtime always uses "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**：DM 的分组方式。
  - `main`：所有 DM 共享主会话。
  - `per-peer`：按发送者 id 在各 channel 间隔离。
  - `per-channel-peer`：按 channel + 发送者 隔离（推荐用于多用户收件箱）。
  - `per-account-channel-peer`：按 账户 + channel + 发送者 隔离（推荐用于多账户）。
- **`identityLinks`**：将规范 id 映射到带 provider 前缀的 peer，用于跨 channel 会话共享。
- **`reset`**：主要的重置策略。 `daily` 在本地时间 `atHour` 重置；`idle` 在 `idleMinutes` 后重置。 当两者同时配置时，以先到期者为准。
- **`resetByType`**：按类型覆盖（`direct`、`group`、`thread`）。 旧版 `dm` 可作为 `direct` 的别名。
- **`mainKey`**：遗留字段。 运行时现在始终使用 `"main"` 作为主私聊 bucket。
- **`sendPolicy`**：按 `channel`、`chatType`（`direct|group|channel`，兼容旧别名 `dm`）、`keyPrefix` 或 `rawKeyPrefix` 进行匹配。 优先匹配到的拒绝规则生效。
- **`maintenance`**：`warn` 在会话被清理时向当前活动会话发出警告；`enforce` 执行修剪和轮换。

</Accordion>

---

## 消息

```json5
{
  messages: {
    responsePrefix: "🦞", // 或 "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 表示禁用
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### 响应前缀

按 channel/account 覆盖：`channels.<channel>.responsePrefix`、`channels.<channel>.accounts.<id>.responsePrefix`。

解析顺序（越具体优先级越高）：account → channel → global。 `""` 表示禁用并停止向上级联。 `"auto"` 会派生为 `[{identity.name}]`。

**模板变量：**

| 变量                | 说明      | 示例                          |
| ----------------- | ------- | --------------------------- |
| `{model}`         | 模型简称    | `claude-opus-4-6`           |
| `{modelFull}`     | 完整模型标识符 | `anthropic/claude-opus-4-6` |
| `{provider}`      | 提供方名称   | `anthropic`                 |
| `{thinkingLevel}` | 当前思考级别  | `high`、`low`、`off`          |
| `{identity.name}` | 代理身份名称  | （与 `"auto"` 相同）             |

变量不区分大小写。 `{think}` 是 `{thinkingLevel}` 的别名。

### Ack 表情反应

- 默认使用当前活动代理的 `identity.emoji`，否则为 `"👀"`。 设置为 `""` 可禁用。
- 按 channel 覆盖：`channels.<channel>`.ackReaction`, `channels.<channel> .accounts.<id> .ackReaction\`.
- 解析顺序：account → channel → `messages.ackReaction` → identity 回退。
- 范围：`group-mentions`（默认）、`group-all`、`direct`、`all`。
- `removeAckAfterReply`：回复后移除 ack（仅适用于 Slack/Discord/Telegram/Google Chat）。

### 入站防抖

将同一发送者的快速纯文本消息合并为一次 agent 轮次。 媒体/附件会立即刷新。 控制命令绕过防抖。

### TTS（文本转语音）

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

- `auto` 控制自动 TTS。 `/tts off|always|inbound|tagged` 可在每个会话中覆盖设置。
- `summaryModel` 会覆盖 `agents.defaults.model.primary` 用于自动摘要。
- API keys 会回退到 `ELEVENLABS_API_KEY`/`XI_API_KEY` 和 `OPENAI_API_KEY`。

---

## Talk

Talk 模式的默认设置（macOS/iOS/Android）。

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Voice ID 会回退到 `ELEVENLABS_VOICE_ID` 或 `SAG_VOICE_ID`。
- `apiKey` 会回退到 `ELEVENLABS_API_KEY`。
- `voiceAliases` 允许 Talk 指令使用友好的名称。

---

## 工具

### 工具配置文件

`tools.profile` 在 `tools.allow`/`tools.deny` 之前设置基础允许列表：

| 配置文件        | 包含                                                                                    |
| ----------- | ------------------------------------------------------------------------------------- |
| `minimal`   | 仅 `session_status`                                                                    |
| `coding`    | `group:fs`、`group:runtime`、`group:sessions`、`group:memory`、`image`                    |
| `messaging` | `group:messaging`、`sessions_list`、`sessions_history`、`sessions_send`、`session_status` |
| `full`      | 无限制（等同于未设置）                                                                           |

### 工具组

| 组                  | 工具                                                                                   |
| ------------------ | ------------------------------------------------------------------------------------ |
| `group:runtime`    | `exec`、`process`（`bash` 可作为 `exec` 的别名）                                              |
| `group:fs`         | `read`、`write`、`edit`、`apply_patch`                                                  |
| `group:sessions`   | `sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status` |
| `group:memory`     | `memory_search`、`memory_get`                                                         |
| `group:web`        | `web_search`、`web_fetch`                                                             |
| `group:ui`         | `browser`、`canvas`                                                                   |
| `group:automation` | `cron`、`gateway`                                                                     |
| `group:messaging`  | `message`                                                                            |
| `group:nodes`      | `nodes`                                                                              |
| `group:openclaw`   | 所有内置工具（不包括 provider 插件）                                                              |

### `tools.allow` / `tools.deny`

全局工具允许/拒绝策略（拒绝优先生效）。 不区分大小写，支持 `*` 通配符。 即使 Docker 沙箱关闭时也会生效。

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

为特定 provider 或模型进一步限制工具。 顺序：基础 profile → provider profile → allow/deny。

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

控制提升（主机）exec 访问权限：

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- 按代理覆盖（`agents.list[].tools.elevated`）只能进一步收紧权限。
- `/elevated on|off|ask|full` 会为每个会话存储状态；内联指令仅作用于单条消息。
- 提升权限的 `exec` 在主机上运行，绕过沙箱机制。

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // 或 BRAVE_API_KEY 环境变量
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

配置入站媒体内容理解（图片/音频/视频）：

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Media model entry fields">

**Provider 条目**（`type: "provider"` 或省略）：

- `provider`：API 提供商 id（`openai`、`anthropic`、`google`/`gemini`、`groq` 等）
- `model`：模型 id 覆盖
- `profile` / `preferredProfile`：认证配置选择

**CLI 条目**（`type: "cli"`）：

- `command`：要运行的可执行文件
- `args`：模板化参数（支持 `{{MediaPath}}`、`{{Prompt}}`、`{{MaxChars}}` 等）

**通用字段：**

- `capabilities`：可选列表（`image`、`audio`、`video`）。 默认值：`openai`/`anthropic`/`minimax` → image，`google` → image+audio+video，`groq` → audio。
- `prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`：针对每个条目的覆盖设置。
- 失败时会回退到下一个条目。

Provider 认证遵循标准顺序：auth profiles → 环境变量 → `models.providers.*.apiKey`。

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`：用于新生成子代理的默认模型。 如果省略，子代理将继承调用者的模型。
- 每个子代理的工具策略：`tools.subagents.tools.allow` / `tools.subagents.tools.deny`。

---

## 自定义 Provider 和基础 URL

OpenClaw 使用 pi-coding-agent 模型目录。 可通过配置中的 `models.providers` 或 `~/.openclaw/agents/<agentId>/agent/models.json` 添加自定义 Provider。

```json5
{
  models: {
    mode: "merge", // merge（默认） | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- 使用 `authHeader: true` + `headers` 以满足自定义认证需求。
- 使用 `OPENCLAW_AGENT_DIR`（或 `PI_CODING_AGENT_DIR`）覆盖代理配置根目录。

### Provider 示例

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

Cerebras 使用 `cerebras/zai-glm-4.7`；Z.AI 直连使用 `zai/glm-4.7`。

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

设置 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）。 快捷方式：`openclaw onboard --auth-choice opencode-zen`。

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

设置 `ZAI_API_KEY`。 `z.ai/*` 和 `z-ai/*` 是可接受的别名。 快捷方式：`openclaw onboard --auth-choice zai-api-key`。

- 通用端点：`https://api.z.ai/api/paas/v4`
- 编码端点（默认）：`https://api.z.ai/api/coding/paas/v4`
- 对于通用端点，请通过覆盖 base URL 来定义自定义 Provider。

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

对于中国端点：`baseUrl: "https://api.moonshot.cn/v1"` 或 `openclaw onboard --auth-choice moonshot-api-key-cn`。

</Accordion>

<Accordion title="Kimi Coding">

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

兼容 Anthropic，内置提供商。 快捷方式：`openclaw onboard --auth-choice kimi-code-api-key`。

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Base URL 不应包含 `/v1`（Anthropic 客户端会自动追加）。 快捷方式：`openclaw onboard --auth-choice synthetic-api-key`。

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

设置 `MINIMAX_API_KEY`。 快捷方式：`openclaw onboard --auth-choice minimax-api`。

</Accordion>

<Accordion title="Local models (LM Studio)">

参见 [Local Models](/gateway/local-models)。 简而言之：在高性能硬件上通过 LM Studio Responses API 运行 MiniMax M2.1；同时合并托管模型作为备用。

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`：仅对内置 skills 生效的可选允许列表（managed/workspace skills 不受影响）。
- `entries.<skillKey>.enabled: false`：即使已内置/已安装，也会禁用该 skill。
- `entries.<skillKey>.apiKey`：为声明了主环境变量的 skills 提供的便捷配置。

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- 从 `~/.openclaw/extensions`、`<workspace>/.openclaw/extensions` 以及 `plugins.load.paths` 加载。
- **配置更改需要重启 gateway。**
- `allow`：可选允许列表（仅加载列出的 plugins）。 `deny` 优先生效。

参见 [Plugins](/tools/plugin)。

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` 会禁用 `act:evaluate` 和 `wait --fn`。
- 远程 profiles 为仅附加模式（无法启动/停止/重置）。
- 自动检测顺序：若默认浏览器为 Chromium 内核 → Chrome → Brave → Edge → Chromium → Chrome Canary。
- 控制服务：仅监听回环地址（端口由 `gateway.port` 派生，默认 `18791`）。

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`：原生应用 UI 外观的强调色（如 Talk 模式气泡的色调等）。
- `assistant`：覆盖 Control UI 的身份标识。 默认回退为当前激活的 agent 身份。

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`：`local`（运行 gateway）或 `remote`（连接到远程 gateway）。 除非设置为 `local`，否则 Gateway 会拒绝启动。
- `port`：用于 WS + HTTP 的单一复用端口。 优先级：`--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`。
- `bind`：`auto`、`loopback`（默认）、`lan`（`0.0.0.0`）、`tailnet`（仅 Tailscale IP）或 `custom`。
- **Auth**：默认必需。 非 loopback 绑定需要共享的 token/password。 Onboarding 向导默认会生成一个 token。
- `auth.mode: "trusted-proxy"`：将身份验证委托给具备身份感知能力的反向代理，并信任来自 `gateway.trustedProxies` 的身份头（参见 [Trusted Proxy Auth](/gateway/trusted-proxy-auth)）。
- `auth.allowTailscale`：当为 `true` 时，Tailscale Serve 的身份头可满足身份验证（通过 `tailscale whois` 验证）。 当 `tailscale.mode = "serve"` 时，默认值为 `true`。
- `auth.rateLimit`：可选的身份验证失败限流器。 按客户端 IP 和身份验证范围分别应用（shared-secret 和 device-token 独立跟踪）。 被阻止的请求返回 `429` + `Retry-After`。
  - `auth.rateLimit.exemptLoopback` 默认值为 `true`；如果你希望对 localhost 流量也进行限流（用于测试环境或严格的代理部署），请设置为 `false`。
- `tailscale.mode`：`serve`（仅 tailnet，loopback 绑定）或 `funnel`（公网访问，需要身份验证）。
- `remote.transport`：`ssh`（默认）或 `direct`（ws/wss）。 对于 `direct`，`remote.url` 必须是 `ws://` 或 `wss://`。
- `gateway.remote.token` 仅用于远程 CLI 调用；不会启用本地 gateway 身份验证。
- `trustedProxies`：终止 TLS 的反向代理 IP。 仅列出你控制的代理。
- `gateway.tools.deny`：为 HTTP `POST /tools/invoke` 额外阻止的工具名称（扩展默认拒绝列表）。
- `gateway.tools.allow`：从默认 HTTP 拒绝列表中移除工具名称。

</Accordion>

### 兼容 OpenAI 的端点

- Chat Completions：默认禁用。 使用 `gateway.http.endpoints.chatCompletions.enabled: true` 启用。
- Responses API：`gateway.http.endpoints.responses.enabled`。
- Responses URL 输入加固：
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### 多实例隔离

在同一主机上运行多个 gateway，使用不同的端口和状态目录：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

便捷参数：`--dev`（使用 `~/.openclaw-dev` + 端口 `19001`），`--profile <name>`（使用 `~/.openclaw-<name>`）。

参见 [Multiple Gateways](/gateway/multiple-gateways)。

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

身份验证：`Authorization: Bearer <token>` 或 `x-openclaw-token: <token>`。

**端点：**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - 仅当 `hooks.allowRequestSessionKey=true`（默认：`false`）时，才会接受请求负载中的 `sessionKey`。
- `POST /hooks/<name>` → 通过 `hooks.mappings` 解析

<Accordion title="Mapping details">

- `match.path` 匹配 `/hooks` 之后的子路径（例如 `/hooks/gmail` → `gmail`）。
- `match.source` 匹配通用路径中的某个负载字段。
- 类似 `{{messages[0].subject}}` 的模板会从负载中读取数据。
- `transform` 可以指向一个返回 hook action 的 JS/TS 模块。
  - `transform.module` 必须是相对路径，并且必须位于 `hooks.transformsDir` 内（绝对路径和路径穿越将被拒绝）。
- `agentId` 将路由到特定 agent；未知 ID 将回退到默认值。
- `allowedAgentIds`：限制显式路由（`*` 或省略 = 允许全部，`[]` = 全部拒绝）。
- `defaultSessionKey`：可选的固定 session key，用于在未显式提供 `sessionKey` 时运行 hook agent。
- `allowRequestSessionKey`：允许 `/hooks/agent` 调用方设置 `sessionKey`（默认：`false`）。
- `allowedSessionKeyPrefixes`：可选的显式 `sessionKey` 前缀白名单（请求 + 映射），例如 `[
  "hook:"
  ]`。
- `deliver: true` 会将最终回复发送到某个 channel；`channel` 默认为 `last`。
- `model` 会覆盖本次 hook 运行所使用的 LLM（如果设置了模型目录，则必须在允许范围内）。

</Accordion>

### Gmail 集成

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- 配置后，Gateway 会在启动时自动运行 `gog gmail watch serve`。 设置 `OPENCLAW_SKIP_GMAIL_WATCHER=1` 可禁用。
- 不要在 Gateway 运行的同时单独运行 `gog gmail watch serve`。

---

## Canvas 主机

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // 或 OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- 在 Gateway 端口下通过 HTTP 提供可由 agent 编辑的 HTML/CSS/JS 和 A2UI：
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- 仅限本地：保持 `gateway.bind: "loopback"`（默认）。
- 在非 loopback 绑定下：canvas 路由需要 Gateway 认证（token/password/trusted-proxy），与其他 Gateway HTTP 接口相同。
- Node WebViews 通常不会发送认证头；在节点完成配对并连接后，Gateway 会允许基于私有 IP 的回退方式，使节点在不将敏感信息暴露到 URL 中的情况下加载 canvas/A2UI。
- 向已提供的 HTML 注入 live-reload 客户端。
- 当目录为空时自动创建初始 `index.html`。
- 同时在 `/__openclaw__/a2ui/` 提供 A2UI。
- 更改需要重启 gateway。
- 对于大型目录或出现 `EMFILE` 错误时，请禁用 live reload。

---

## 发现机制

### mDNS（Bonjour）

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal`（默认）：在 TXT 记录中省略 `cliPath` 和 `sshPort`。
- `full`：包含 `cliPath` 和 `sshPort`。
- 主机名默认为 `openclaw`。 使用 `OPENCLAW_MDNS_HOSTNAME` 进行覆盖。

### 广域（DNS-SD）

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

在 `~/.openclaw/dns/` 下写入一个单播 DNS-SD 区域。 如需跨网络发现，请搭配 DNS 服务器（推荐 CoreDNS）+ Tailscale 分离 DNS。

设置：`openclaw dns setup --apply`。

---

## 环境

### `env`（内联环境变量）

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- 仅当进程环境中缺少对应键时，才会应用内联环境变量。
- `.env` 文件：当前工作目录 `.env` + `~/.openclaw/.env`（均不会覆盖已有变量）。
- `shellEnv`：从你的登录 shell 配置中导入缺失的预期键。
- 完整优先级请参阅 [Environment](/help/environment)。

### 环境变量替换

在任意配置字符串中使用 `${VAR_NAME}` 引用环境变量：

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- 仅匹配大写名称：`[A-Z_][A-Z0-9_]*`。
- 缺失或为空的变量会在加载配置时抛出错误。
- 使用 `$${VAR}` 转义以表示字面量 `${VAR}`。
- 支持与 `$include` 一起使用。

---

## 认证存储

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- 每个 agent 的认证配置文件存储在 `<agentDir>/auth-profiles.json`。
- 旧版 OAuth 会从 `~/.openclaw/credentials/oauth.json` 导入。
- 请参阅 [OAuth](/concepts/oauth)。

---

## 日志

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- 默认日志文件：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`。
- 设置 `logging.file` 以使用固定路径。
- 使用 `--verbose` 时，`consoleLevel` 会提升为 `debug`。

---

## 向导

由 CLI 向导（`onboard`、`configure`、`doctor`）写入的元数据：

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## 身份

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

由 macOS 入门引导助手写入。 派生默认值：

- `messages.ackReaction` 来自 `identity.emoji`（回退为 👀）
- `mentionPatterns` 来自 `identity.name`/`identity.emoji`
- `avatar` 支持：工作区相对路径、`http(s)` URL 或 `data:` URI

---

## Bridge（旧版，已移除）

当前版本不再包含 TCP bridge。 节点通过 Gateway WebSocket 连接。 `bridge.*` 键已不再属于配置 schema 的一部分（在移除前会导致校验失败；`openclaw doctor --fix` 可以清除未知键）。

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // 持续时间字符串或 false
  },
}
```

- `sessionRetention`：在清理之前保留已完成 cron 会话的时长。 默认值：`24h`。

参见 [Cron Jobs](/automation/cron-jobs)。

---

## 媒体模型模板变量

在 `tools.media.*.models[].args` 中展开的模板占位符：

| 变量                 | 说明                                 |
| ------------------ | ---------------------------------- |
| `{{Body}}`         | 完整的入站消息正文                          |
| `{{RawBody}}`      | 原始正文（不包含历史记录/发送者包装信息）              |
| `{{BodyStripped}}` | 移除群组提及后的正文                         |
| `{{From}}`         | 发送方标识符                             |
| `{{To}}`           | 目标标识符                              |
| `{{MessageSid}}`   | 渠道消息 ID                            |
| `{{SessionId}}`    | 当前会话 UUID                          |
| `{{IsNewSession}}` | 当创建新会话时为 `"true"`                  |
| `{{MediaUrl}}`     | 入站媒体伪 URL                          |
| `{{MediaPath}}`    | 本地媒体路径                             |
| `{{MediaType}}`    | 媒体类型（image/audio/document/…）       |
| `{{Transcript}}`   | 音频转录文本                             |
| `{{Prompt}}`       | 用于 CLI 条目的已解析媒体提示词                 |
| `{{MaxChars}}`     | 已解析 CLI 条目的最大输出字符数                 |
| `{{ChatType}}`     | `"direct"` 或 `"group"`             |
| `{{GroupSubject}}` | 群组主题（尽力获取）                         |
| `{{GroupMembers}}` | 群成员预览（尽力获取）                        |
| `{{SenderName}}`   | 发送者显示名称（尽力获取）                      |
| `{{SenderE164}}`   | 发送者电话号码（尽力获取）                      |
| `{{Provider}}`     | 提供商提示（whatsapp、telegram、discord 等） |

---

## 配置支持包含（`$include`）

将配置拆分为多个文件：

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**合并行为：**

- 单个文件：替换其所在的对象。
- 文件数组：按顺序进行深度合并（后面的覆盖前面的）。
- 同级键：在包含内容之后合并（覆盖被包含的值）。
- 嵌套包含：最多支持 10 层嵌套。
- 路径：支持相对路径（相对于包含它的文件）、绝对路径或 `../` 上级引用。
- 错误：对于缺失文件、解析错误和循环包含，会提供清晰的错误信息。

---

_相关： [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

---
title: "設定參考"
description: "`~/.openclaw/openclaw.json` 的完整逐欄位說明"
---

# 設定參考

`~/.openclaw/openclaw.json` 中所有可用欄位。 如需以任務為導向的概覽，請參閱 [Configuration](/gateway/configuration)。

設定格式為 **JSON5**（允許註解與尾隨逗號）。 所有欄位皆為選填 — 若省略，OpenClaw 會使用安全的預設值。

---

## 通道

當通道的設定區段存在時，會自動啟動（除非設定 `enabled: false`）。

### 私訊與群組存取

所有通道皆支援私訊政策與群組政策：

| 私訊政策          | 行為                               |
| ------------- | -------------------------------- |
| `pairing`（預設） | 未知寄件者會收到一次性配對碼；需由擁有者核准           |
| `allowlist`   | 僅允許在 `allowFrom`（或已配對的允許清單）中的寄件者 |
| `open`        | 允許所有傳入私訊（需設定 `allowFrom: ["*"]`） |
| `disabled`    | 忽略所有傳入私訊                         |

| 群組政策            | 行為                   |
| --------------- | -------------------- |
| `allowlist`（預設） | 僅允許符合已設定允許清單的群組      |
| `open`          | 繞過群組允許清單（仍會套用提及門檻規則） |
| `disabled`      | 封鎖所有群組／聊天室訊息         |

<Note>
`channels.defaults.groupPolicy` 會在提供者的 `groupPolicy` 未設定時作為預設值。
配對碼會在 1 小時後過期。 待處理的私訊配對請求每個頻道最多 **3 個**。
Slack/Discord 有特殊的備援機制：如果其 provider 區段完全缺失，執行時的群組政策可能會解析為 `open`（並在啟動時顯示警告）。
</Note>

### WhatsApp

WhatsApp 透過 gateway 的 Web 通道（Baileys Web）運作。 當存在已連結的工作階段時會自動啟動。

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // 藍色勾勾（self-chat 模式為 false）
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

- 對外指令預設使用帳號 `default`（若存在）；否則使用第一個已設定的帳號 ID（排序後）。
- 舊版單一帳號的 Baileys 認證目錄會由 `openclaw doctor` 遷移至 `whatsapp/default`。
- 每個帳號的覆寫設定：`channels.whatsapp.accounts.<id>`.sendReadReceipts`、`channels.whatsapp.accounts.<id>`.dmPolicy`、`channels.whatsapp.accounts.<id>`.allowFrom\`。

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
          systemPrompt: "請保持簡潔回答。",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "請保持在主題範圍內。",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git 備份" },
        { command: "generate", description: "建立圖片" },
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

- Bot token：`channels.telegram.botToken` 或 `channels.telegram.tokenFile`，預設帳號可使用 `TELEGRAM_BOT_TOKEN` 作為備援。
- `configWrites: false` 會阻擋由 Telegram 發起的設定寫入（例如超級群組 ID 遷移、`/config set|unset`）。
- Telegram 串流預覽使用 `sendMessage` + `editMessageText`（可於私聊與群組聊天中運作）。
- 重試策略：請參閱 [Retry policy](/concepts/retry)。

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
              systemPrompt: "僅提供簡短回答。",
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

- Token：`channels.discord.token`，預設帳號可使用 `DISCORD_BOT_TOKEN` 作為備援。
- 請使用 `user:<id>`（私訊）或 `channel:<id>`（guild 頻道）作為傳送目標；不接受純數字 ID。
- Guild slug 為小寫並以 `-` 取代空格；頻道鍵名使用 slug 後的名稱（不含 `#`）。 建議優先使用 guild ID。
- 預設會忽略由 Bot 發送的訊息。 `allowBots: true` 會啟用處理這些訊息（仍會過濾自身訊息）。
- `maxLinesPerMessage`（預設 17）即使在少於 2000 個字元時，也會分割過長訊息。
- `channels.discord.ui.components.accentColor` 用於設定 Discord components v2 容器的強調色。

**反應通知模式：** `off`（無）、`own`（僅 Bot 的訊息，預設）、`all`（所有訊息）、`allowlist`（來自 `guilds.<id>`.users\` 的所有訊息）。

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

- Service account JSON：可使用內嵌（`serviceAccount`）或檔案方式（`serviceAccountFile`）。
- 環境變數備援：`GOOGLE_CHAT_SERVICE_ACCOUNT` 或 `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
- 傳送目標請使用 `spaces/<spaceId>` 或 `users/<userId|email>`。

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

- **Socket 模式** 需要同時設定 `botToken` 與 `appToken`（預設帳號可使用 `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` 作為環境變數備援）。
- **HTTP 模式** 需要 `botToken` 以及 `signingSecret`（可設定於 root 或各帳號層級）。
- `configWrites: false` 會阻擋來自 Slack 的設定寫入操作。
- 傳送目標請使用 `user:<id>`（DM）或 `channel:<id>`。

**Reaction 通知模式：** `off`、`own`（預設）、`all`、`allowlist`（來自 `reactionAllowlist`）。

**執行緒工作階段隔離：** `thread.historyScope` 可設為每個執行緒（預設）或在整個頻道中共用。 `thread.inheritParent` 會將父頻道的對話紀錄複製到新的執行緒中。

| 動作群組       | 預設 | 說明          |
| ---------- | -- | ----------- |
| reactions  | 啟用 | 新增反應 + 列出反應 |
| messages   | 啟用 | 讀取／傳送／編輯／刪除 |
| pins       | 啟用 | 釘選／取消釘選／列出  |
| memberInfo | 啟用 | 成員資訊        |
| emojiList  | 啟用 | 自訂 emoji 清單 |

### Mattermost

Mattermost 以外掛形式提供：`openclaw plugins install @openclaw/mattermost`。

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

聊天模式：`oncall`（在 @ 提及時回應，預設）、`onmessage`（每則訊息都回應）、`onchar`（以觸發前綴開頭的訊息）。

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

**Reaction 通知模式：** `off`、`own`（預設）、`all`、`allowlist`（來自 `reactionAllowlist`）。

### iMessage

OpenClaw 會啟動 `imsg rpc`（透過 stdio 進行 JSON-RPC）。 不需要 daemon 或連接埠。

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

- 需要對 Messages DB 的完整磁碟存取權。
- 優先使用 `chat_id:<id>` 目標。 使用 `imsg chats --limit 20` 列出聊天。
- `cliPath` 可以指向 SSH 包裝器；設定 `remoteHost` 以透過 SCP 取得附件。

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### 多帳號（所有頻道）

每個頻道可執行多個帳號（各自擁有自己的 `accountId`）：

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

- 當省略 `accountId` 時，會使用 `default`（CLI + 路由）。
- 環境變數中的 token 僅套用於 **default** 帳號。
- 除非在個別帳號中覆寫，否則基礎頻道設定會套用至所有帳號。
- 使用 `bindings[].match.accountId` 將各帳號路由到不同的 agent。

### 群組聊天提及門檻

群組訊息預設為 **必須被提及**（中繼資料提及或 regex 模式）。 適用於 WhatsApp、Telegram、Discord、Google Chat 與 iMessage 群組聊天。

**提及類型：**

- **中繼資料提及**：平台原生的 @ 提及。 在 WhatsApp 自聊模式中會被忽略。
- **文字模式**：定義於 `agents.list[].groupChat.mentionPatterns` 的 Regex 模式。 一律會檢查。
- 僅在可偵測提及時（原生提及或至少一個模式）才會強制執行提及門檻。

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

`messages.groupChat.historyLimit` 設定全域預設值。 各頻道可使用 `channels.<channel> 進行覆寫.historyLimit`（或針對個別帳號）。 設為 `0` 以停用。

#### 私訊歷史紀錄限制

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

解析順序：每個私訊的覆寫設定 → 供應商預設值 → 無限制（全部保留）。

支援：`telegram`、`whatsapp`、`discord`、`slack`、`signal`、`imessage`、`msteams`。

#### 自聊模式

在 `allowFrom` 中加入你自己的號碼以啟用自聊模式（忽略原生 @ 提及，僅回應文字模式）：

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

### 指令（聊天指令處理）

```json5
{
  commands: {
    native: "auto", // 在支援時註冊原生指令
    text: true, // 解析聊天訊息中的 /commands
    bash: false, // 允許 !（別名：/bash）
    bashForegroundMs: 2000,
    config: false, // 允許 /config
    debug: false, // 允許 /debug
    restart: false, // 允許 /restart + gateway restart 工具
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- 文字指令必須是**獨立**訊息，且以 `/` 開頭。
- `native: "auto"` 會為 Discord/Telegram 啟用原生指令，並將 Slack 保持為關閉。
- 可針對每個頻道覆寫：`channels.discord.commands.native`（布林值或 `"auto"`）。 `false` 會清除先前已註冊的指令。
- `channels.telegram.customCommands` 可新增額外的 Telegram 機器人選單項目。
- `bash: true` 啟用 `! <cmd>` 以使用主機 shell。 需要 `tools.elevated.enabled` 且發送者位於 `tools.elevated.allowFrom.<channel>`。
- `config: true` 啟用 `/config`（讀取/寫入 `openclaw.json`）。
- `channels.<provider>`.configWrites\` 控制每個 channel 是否允許修改設定（預設：true）。
- `allowFrom` 為各 provider 獨立設定。 設定後，它將成為**唯一**的授權來源（channel allowlists／配對及 `useAccessGroups` 將被忽略）。
- 當未設定 `allowFrom` 時，`useAccessGroups: false` 允許指令繞過 access-group 政策。

</Accordion>

---

## Agent 預設值

### `agents.defaults.workspace`

預設：`~/.openclaw/workspace`。

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

選用的 repository 根目錄，會顯示於系統提示的 Runtime 行。 若未設定，OpenClaw 會從 workspace 向上遍歷以自動偵測。

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

停用自動建立 workspace 啟動檔（`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`）。

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

每個 workspace 啟動檔在截斷前的最大字元數。 預設：`20000`。

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

所有 workspace 啟動檔合計可注入的最大總字元數。 預設：`24000`。

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

系統提示內容所使用的時區（不影響訊息時間戳）。 若未設定，則使用主機時區。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

系統提示中的時間格式。 預設：`auto`（依作業系統偏好）。

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

- `model.primary`：格式為 `provider/model`（例如：`anthropic/claude-opus-4-6`）。 如果省略 provider，OpenClaw 會預設為 `anthropic`（已棄用）。
- `models`：為 `/model` 設定的模型目錄與允許清單。 每個項目可包含 `alias`（捷徑）與 `params`（依 provider 而定，例如：`temperature`、`maxTokens`）。
- `imageModel`：僅在主要模型不支援影像輸入時使用。
- `maxConcurrent`：跨多個 sessions 可同時執行的最大 agent 數量（每個 session 仍為序列化執行）。 預設值：1。

**內建 alias 簡寫**（僅在模型存在於 `agents.defaults.models` 時套用）：

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

您自訂的 aliases 會優先於預設值生效。

Z.AI GLM-4.x 模型會自動啟用 thinking 模式，除非您設定 `--thinking off`，或自行定義 `agents.defaults.models["zai/<model>"].params.thinking`。

### `agents.defaults.cliBackends`

選用的 CLI 後端，用於純文字備援執行（不呼叫工具）。 當 API providers 發生故障時，可作為備援方案。

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

- CLI 後端以文字為主；工具功能一律停用。
- 當設定 `sessionArg` 時支援 sessions。
- 當 `imageArg` 可接受檔案路徑時，支援影像傳遞。

### `agents.defaults.heartbeat`

定期執行的 heartbeat。

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

- `every`：時間長度字串（ms/s/m/h）。 預設值：`30m`。
- 針對個別 agent：設定 `agents.list[].heartbeat`。 當任何 agent 定義了 `heartbeat` 時，**僅有這些 agents** 會執行 heartbeat。
- Heartbeat 會執行完整的 agent 回合——間隔越短會消耗越多 tokens。

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

- `mode`：`default` 或 `safeguard`（針對長歷史記錄的分段摘要）。 請參閱 [Compaction](/concepts/compaction)。
- `memoryFlush`：在自動壓縮前執行的靜默代理回合，用於儲存持久記憶。 當工作區為唯讀時會跳過。

### `agents.defaults.contextPruning`

在將內容傳送至 LLM 之前，從記憶體中的上下文移除**舊的工具結果**。 **不會**修改磁碟上的會話歷史記錄。

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

- `mode: "cache-ttl"` 會啟用清理流程。
- `ttl` 控制下次可再次執行清理的時間（自上次快取存取後起算）。
- 清理會先對過大的工具結果進行軟性裁剪，必要時再對較舊的工具結果進行硬性清除。

**Soft-trim** 會保留開頭與結尾，並在中間插入 `...`。

**Hard-clear** 會以預留字串取代整個工具結果。

注意事項：

- 圖片區塊永遠不會被裁剪或清除。
- 比例以字元數為基準（約略值），並非精確的 token 數量。
- 若 assistant 訊息少於 `keepLastAssistants`，則會跳過清理。

</Accordion>

行為細節請參閱 [Session Pruning](/concepts/session-pruning)。

### 區塊串流

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

- 非 Telegram 通道需要明確設定 `*.blockStreaming: true` 才能啟用區塊回覆。
- 通道覆寫：`channels.<channel>.blockStreamingCoalesce`（以及每個帳戶的變體設定）。 Signal/Slack/Discord/Google Chat 預設 `minChars: 1500`。
- `humanDelay`：區塊回覆之間的隨機暫停。 `natural` = 800–2500ms。 每個代理覆寫：`agents.list[].humanDelay`。

行為與分段細節請參閱 [Streaming](/concepts/streaming)。

### 輸入指示器

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

- 預設值：在私聊／被提及時為 `instant`，在未被提及的群組聊天中為 `message`。
- 每個會話的覆寫：`session.typingMode`、`session.typingIntervalSeconds`。

請參閱 [Typing Indicators](/concepts/typing-indicators)。

### `agents.defaults.sandbox`

為內嵌代理提供可選的 **Docker 沙箱化**。 完整指南請參閱 [Sandboxing](/gateway/sandboxing)。

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

**工作區存取：**

- `none`：在 `~/.openclaw/sandboxes` 下為每個範圍建立沙箱工作區
- `ro`：沙箱工作區位於 `/workspace`，代理工作區以唯讀方式掛載於 `/agent`
- `rw`：以讀寫模式掛載代理工作區至 `/workspace`

**範圍：**

- `session`：每個 session 對應一個容器 + 工作區
- `agent`：每個 agent 對應一個容器 + 工作區（預設）
- `shared`：共用容器與工作區（無跨 session 隔離）

**`setupCommand`** 會在容器建立後執行一次（透過 `sh -lc`）。 需要對外網路連線（network egress）、可寫入的 root，以及 root 使用者權限。

**容器預設為 `network: "none"`** —— 若代理需要對外存取，請設為 `"bridge"`。

**傳入附件** 會暫存於目前工作區的 `media/inbound/*`。

**`docker.binds`** 用於掛載額外的主機目錄；全域與每個 agent 的 binds 會合併。

**沙箱瀏覽器**（`sandbox.browser.enabled`）：在容器中執行 Chromium + CDP。 noVNC URL 會注入至系統提示（system prompt）。 不需要在主設定中啟用 `browser.enabled`。

- `allowHostControl: false`（預設）會阻止沙箱 session 存取主機瀏覽器。
- `sandbox.browser.binds` 僅將額外的主機目錄掛載到沙箱瀏覽器容器中。 設定後（包含 `[]`），會取代瀏覽器容器的 `docker.binds`。

</Accordion>

建置映像檔：

```bash
scripts/sandbox-setup.sh           # 主沙箱映像檔
scripts/sandbox-browser-setup.sh   # 可選的瀏覽器映像檔
```

### `agents.list`（每個 agent 的覆寫設定）

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

- `id`：穩定的 agent 識別碼（必填）。
- `default`：若設定多個，則以第一個為準（會記錄警告）。 若未設定，則清單中的第一個為預設。
- `model`：字串形式僅覆寫 `primary`；物件形式 `{ primary, fallbacks }` 會同時覆寫兩者（`[]` 會停用全域 fallbacks）。
- `identity.avatar`：相對於工作區的路徑、`http(s)` URL，或 `data:` URI。
- `identity` 會推導預設值：`ackReaction` 來自 `emoji`，`mentionPatterns` 來自 `name`/`emoji`。
- `subagents.allowAgents`：`sessions_spawn` 可允許的 agent id 白名單（`["*"]` = 任意；預設：僅相同 agent）。

---

## 多 agent 路由

在單一 Gateway 中執行多個彼此隔離的 agent。 請參閱 [Multi-Agent](/concepts/multi-agent)。

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

### Binding 比對欄位

- `match.channel`（必填）
- `match.accountId`（選填；`*` = 任意帳號；未填 = 預設帳號）
- `match.peer`（選填；`{ kind: direct|group|channel, id }`）
- `match.guildId` / `match.teamId`（選填；依頻道而定）

**確定性的比對順序：**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`（精確匹配，不包含 peer/guild/team）
5. `match.accountId: "*"`（整個頻道）
6. 預設代理

在每個層級中，第一個符合的 `bindings` 項目優先生效。

### 每個代理的存取設定檔

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

請參閱 [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) 了解優先順序的詳細說明。

---

## Session

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

- **`dmScope`**：DM 的分組方式。
  - `main`：所有 DM 共用主 Session。
  - `per-peer`：依發送者 ID（跨頻道）隔離。
  - `per-channel-peer`：依頻道 + 發送者隔離（建議用於多使用者收件匣）。
  - `per-account-channel-peer`：依帳號 + 頻道 + 發送者隔離（建議用於多帳號情境）。
- **`identityLinks`**：將標準 ID 對應至帶有 provider 前綴的 peer，以實現跨頻道 Session 共用。
- **`reset`**：主要的重置策略。 `daily` 會在本地時間 `atHour` 重置；`idle` 則在閒置 `idleMinutes` 後重置。 若同時設定兩者，則以先到期者為準。
- **`resetByType`**：依類型覆寫設定（`direct`、`group`、`thread`）。 舊版 `dm` 仍可作為 `direct` 的別名使用。
- **`mainKey`**：舊版欄位。 執行時現在一律使用 `"main"` 作為主要的直接聊天 bucket。
- **`sendPolicy`**：可依 `channel`、`chatType`（`direct|group|channel`，舊版 `dm` 為別名）、`keyPrefix` 或 `rawKeyPrefix` 進行匹配。 第一個符合的 deny 規則優先生效。
- **`maintenance`**：`warn` 在清除時會警告目前的 Session；`enforce` 則會套用清理與輪替。

</Accordion>

---

## 訊息

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
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
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### 回應前綴

各頻道／帳號覆寫：`channels.<channel>.responsePrefix`、`channels.<channel>.accounts.<id>.responsePrefix`。

解析順序（越具體優先）：account → channel → global。 `""` 會停用並中止向上繼承。 `"auto"` 會自動產生 `[{identity.name}]`。

**範本變數：**

| 變數                | 說明       | 範例                          |
| ----------------- | -------- | --------------------------- |
| `{model}`         | 簡短模型名稱   | `claude-opus-4-6`           |
| `{modelFull}`     | 完整模型識別名稱 | `anthropic/claude-opus-4-6` |
| `{provider}`      | 供應商名稱    | `anthropic`                 |
| `{thinkingLevel}` | 目前的思考等級  | `high`, `low`, `off`        |
| `{identity.name}` | 代理身分名稱   | （與 `"auto"` 相同）             |

變數不區分大小寫。 `{think}` 是 `{thinkingLevel}` 的別名。

### Ack 反應

- 預設為啟用中的代理 `identity.emoji`，否則為 `"👀"`。 設為 `""` 以停用。
- 每個頻道覆寫：`channels.<channel>`
  .ackReaction`, `channels.<channel>`
  .accounts.<id>
  .ackReaction`。
- 解析順序：account → channel → `messages.ackReaction` → identity 預設值。
- 範圍：`group-mentions`（預設）、`group-all`、`direct`、`all`。
- `removeAckAfterReply`：在回覆後移除 ack（僅限 Slack/Discord/Telegram/Google Chat）。

### 輸入防抖（Inbound debounce）

將同一位發送者的快速純文字訊息合併為單一代理回合。 媒體／附件會立即送出。 控制指令會略過防抖機制。

### TTS（文字轉語音）

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

- `auto` 用於控制自動 TTS。 `/tts off|always|inbound|tagged` 可在每個工作階段中覆寫。
- `summaryModel` 會覆寫 `agents.defaults.model.primary` 以用於自動摘要。
- API keys 會回退使用 `ELEVENLABS_API_KEY`/`XI_API_KEY` 與 `OPENAI_API_KEY`。

---

## Talk

Talk 模式的預設值（macOS/iOS/Android）。

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

- Voice ID 會回退至 `ELEVENLABS_VOICE_ID` 或 `SAG_VOICE_ID`。
- `apiKey` 會回退至 `ELEVENLABS_API_KEY`。
- `voiceAliases` 讓 Talk 指令可使用易讀的名稱。

---

## 工具

### 工具設定檔

`tools.profile` 會在 `tools.allow`/`tools.deny` 之前設定基礎允許清單：

| 設定檔         | 包含                                                                                    |
| ----------- | ------------------------------------------------------------------------------------- |
| `minimal`   | 僅 `session_status`                                                                    |
| `coding`    | `group:fs`、`group:runtime`、`group:sessions`、`group:memory`、`image`                    |
| `messaging` | `group:messaging`、`sessions_list`、`sessions_history`、`sessions_send`、`session_status` |
| `full`      | 無限制（等同於未設定）                                                                           |

### 工具群組

| 群組                 | 工具                                                                                   |
| ------------------ | ------------------------------------------------------------------------------------ |
| `group:runtime`    | `exec`、`process`（`bash` 可作為 `exec` 的別名）                                              |
| `group:fs`         | `read`、`write`、`edit`、`apply_patch`                                                  |
| `group:sessions`   | `sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status` |
| `group:memory`     | `memory_search`、`memory_get`                                                         |
| `group:web`        | `web_search`、`web_fetch`                                                             |
| `group:ui`         | `browser`、`canvas`                                                                   |
| `group:automation` | `cron`、`gateway`                                                                     |
| `group:messaging`  | `message`                                                                            |
| `group:nodes`      | `nodes`                                                                              |
| `group:openclaw`   | 所有內建工具（不包含 provider 外掛）                                                              |

### `tools.allow` / `tools.deny`

全域工具允許／拒絕策略（拒絕優先生效）。 不區分大小寫，支援 `*` 萬用字元。 即使關閉 Docker 沙箱時仍會套用。

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

針對特定提供者或模型進一步限制工具。 順序：基礎設定檔 → 提供者設定檔 → allow/deny。

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

控制提升權限（主機）exec 存取：

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

- 每個代理的覆寫設定（`agents.list[].tools.elevated`）只能進一步限制權限。
- `/elevated on|off|ask|full` 會為每個工作階段儲存狀態；內嵌指令僅套用於單一訊息。
- 提升權限的 `exec` 會在主機上執行，並繞過沙箱機制。

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
        apiKey: "brave_api_key", // 或使用 BRAVE_API_KEY 環境變數
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

設定傳入媒體理解（影像／音訊／影片）：

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

**提供者項目**（`type: "provider"` 或省略）：

- `provider`：API 提供者 ID（`openai`、`anthropic`、`google`/`gemini`、`groq` 等）
- `model`：模型 ID 覆寫
- `profile` / `preferredProfile`：驗證設定檔選擇

**CLI 項目**（`type: "cli"`）：

- `command`：要執行的可執行檔
- `args`：樣板化參數（支援 `{{MediaPath}}`、`{{Prompt}}`、`{{MaxChars}}` 等）

**共通欄位：**

- `capabilities`：選用清單（`image`、`audio`、`video`）。 預設值：`openai`/`anthropic`/`minimax` → image，`google` → image+audio+video，`groq` → audio。
- `prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`：每個項目的覆寫設定。
- 發生錯誤時會回退至下一個項目。

提供者驗證遵循標準順序：auth profiles → 環境變數 → `models.providers.*.apiKey`。

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

- `model`：已啟動子代理的預設模型。 如果省略，子代理將繼承呼叫者的模型。
- 每個子代理的工具策略：`tools.subagents.tools.allow` / `tools.subagents.tools.deny`。

---

## 自訂供應商與 base URL

OpenClaw 使用 pi-coding-agent 模型目錄。 可透過設定中的 `models.providers` 或 `~/.openclaw/agents/<agentId>/agent/models.json` 新增自訂供應商。

```json5
{
  models: {
    mode: "merge", // merge (default) | replace
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

- 使用 `authHeader: true` + `headers` 以滿足自訂驗證需求。
- 使用 `OPENCLAW_AGENT_DIR`（或 `PI_CODING_AGENT_DIR`）覆寫代理設定根目錄。

### 供應商範例

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

Cerebras 使用 `cerebras/zai-glm-4.7`；Z.AI 直連使用 `zai/glm-4.7`。

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

設定 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）。 快速指令：`openclaw onboard --auth-choice opencode-zen`。

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

設定 `ZAI_API_KEY`。 `z.ai/*` 與 `z-ai/*` 都是可接受的別名。 快速指令：`openclaw onboard --auth-choice zai-api-key`。

- 通用端點：`https://api.z.ai/api/paas/v4`
- Coding 端點（預設）：`https://api.z.ai/api/coding/paas/v4`
- 若使用通用端點，請透過覆寫 base URL 來定義自訂供應商。

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

中國端點請使用：`baseUrl: "https://api.moonshot.cn/v1"` 或 `openclaw onboard --auth-choice moonshot-api-key-cn`。

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

相容 Anthropic，內建供應商。 快速指令：`openclaw onboard --auth-choice kimi-code-api-key`。

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

Base URL 不應包含 `/v1`（Anthropic 用戶端會自動附加）。 快速指令：`openclaw onboard --auth-choice synthetic-api-key`。

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

設定 `MINIMAX_API_KEY`。 快速指令：`openclaw onboard --auth-choice minimax-api`。

</Accordion>

<Accordion title="Local models (LM Studio)">

請參閱 [Local Models](/gateway/local-models)。 重點摘要：在高效能硬體上透過 LM Studio Responses API 執行 MiniMax M2.1；同時保留合併的託管模型作為備援。

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

- `allowBundled`：僅適用於內建 skills 的選用允許清單（不影響 managed/workspace skills）。
- `entries.<skillKey>`.enabled: false\` 可在 skill 為內建或已安裝時停用它。
- `entries.<skillKey>`.apiKey\`：供 Skills 宣告主要環境變數時使用的便利屬性。

---

## 外掛（Plugins）

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

- 從 `~/.openclaw/extensions`、`<workspace>/.openclaw/extensions` 以及 `plugins.load.paths` 載入。
- **設定變更需要重新啟動 gateway。**
- `allow`：選用的允許清單（僅載入清單中列出的外掛）。 `deny` 具有優先權。

請參閱 [Plugins](/tools/plugin)。

---

## 瀏覽器（Browser）

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

- `evaluateEnabled: false` 會停用 `act:evaluate` 與 `wait --fn`。
- 遠端設定檔僅支援附加（attach-only）（無法啟動／停止／重設）。
- 自動偵測順序：若為 Chromium 核心瀏覽器則使用預設瀏覽器 → Chrome → Brave → Edge → Chromium → Chrome Canary。
- 控制服務：僅限 loopback（連接埠從 `gateway.port` 推導，預設為 `18791`）。

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji、簡短文字、圖片 URL 或 data URI
    },
  },
}
```

- `seamColor`：原生應用程式 UI 外框的強調色（例如 Talk Mode 氣泡色調等）。
- `assistant`：覆寫控制 UI 的身分識別設定。 若未設定，則回退為目前啟用的 agent 身分。

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
      // password: "your-password", // 或 OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // 用於 mode=trusted-proxy；請參閱 /gateway/trusted-proxy-auth
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
      // 額外的 /tools/invoke HTTP 拒絕清單
      deny: ["browser"],
      // 從預設 HTTP 拒絕清單中移除工具
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`：`local`（執行 gateway）或 `remote`（連線至遠端 gateway）。 除非為 `local`，否則 Gateway 將拒絕啟動。
- `port`：單一多工連接埠，同時用於 WS 與 HTTP。 優先順序：`--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`。
- `bind`：`auto`、`loopback`（預設）、`lan`（`0.0.0.0`）、`tailnet`（僅限 Tailscale IP）或 `custom`。
- **Auth**：預設為必填。 非 loopback 綁定需要共用的 token／password。 初始設定精靈會預設產生一個 token。
- `auth.mode: "trusted-proxy"`：將驗證委派給具備身分識別能力的反向代理，並信任來自 `gateway.trustedProxies` 的身分標頭（請參閱 [Trusted Proxy Auth](/gateway/trusted-proxy-auth)）。
- `auth.allowTailscale`：當為 `true` 時，Tailscale Serve 的身分標頭可通過驗證（透過 `tailscale whois` 驗證）。 當 `tailscale.mode = "serve"` 時，預設為 `true`。
- `auth.rateLimit`：選用的驗證失敗次數限制器。 依用戶端 IP 與驗證範圍分別套用（shared-secret 與 device-token 會獨立追蹤）。 被封鎖的嘗試會回傳 `429` + `Retry-After`。
  - `auth.rateLimit.exemptLoopback` 預設為 `true`；若有意讓 localhost 流量也受到速率限制（例如測試環境或嚴格的代理部署），請設為 `false`。
- `tailscale.mode`：`serve`（僅限 tailnet，綁定 loopback）或 `funnel`（公開存取，需要驗證）。
- `remote.transport`：`ssh`（預設）或 `direct`（ws/wss）。 若使用 `direct`，`remote.url` 必須為 `ws://` 或 `wss://`。
- `gateway.remote.token` 僅用於遠端 CLI 呼叫；不會啟用本機 gateway 驗證。
- `trustedProxies`：終止 TLS 的反向代理 IP。 僅列出你所控制的代理。
- `gateway.tools.deny`：針對 HTTP `POST /tools/invoke` 額外封鎖的工具名稱（擴充預設的 deny 清單）。
- `gateway.tools.allow`：從預設的 HTTP deny 清單中移除工具名稱。

</Accordion>

### 與 OpenAI 相容的端點

- Chat Completions：預設為停用。 使用 `gateway.http.endpoints.chatCompletions.enabled: true` 啟用。
- Responses API：`gateway.http.endpoints.responses.enabled`。
- Responses URL 輸入強化：
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### 多實例隔離

在同一主機上以不同的連接埠與狀態目錄執行多個 gateway：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

便利旗標：`--dev`（使用 `~/.openclaw-dev` + 連接埠 `19001`）、`--profile <name>`（使用 `~/.openclaw-<name>`）。

請參閱 [Multiple Gateways](/gateway/multiple-gateways)。

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

Auth：`Authorization: Bearer <token>` 或 `x-openclaw-token: <token>`。

**端點：**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` }\`
  - 僅當 `hooks.allowRequestSessionKey=true`（預設：`false`）時，才會接受來自請求負載的 `sessionKey`。
- `POST /hooks/<name>` → 透過 `hooks.mappings` 解析

<Accordion title="Mapping details">

- `match.path` 會比對 `/hooks` 之後的子路徑（例如 `/hooks/gmail` → `gmail`）。
- `match.source` 會比對通用路徑中的某個負載欄位。
- 如 `{{messages[0].subject}}` 之類的樣板會從負載中讀取資料。
- `transform` 可以指向一個回傳 hook 動作的 JS/TS 模組。
  - `transform.module` 必須為相對路徑，且僅能位於 `hooks.transformsDir` 之內（不接受絕對路徑與路徑穿越）。
- `agentId` 會路由至指定的 agent；未知的 ID 會回退至預設值。
- `allowedAgentIds`：限制明確路由（`*` 或省略 = 允許全部，`[]` = 全部拒絕）。
- `defaultSessionKey`：選用的固定 session key，供未明確提供 `sessionKey` 的 hook agent 執行使用。
- `allowRequestSessionKey`：允許 `/hooks/agent` 呼叫者設定 `sessionKey`（預設：`false`）。
- `allowedSessionKeyPrefixes`：選用的 `sessionKey` 前綴允許清單（請求 + mapping），例如 `[
  "hook:"
  ]`。
- `deliver: true` 會將最終回覆傳送至某個 channel；`channel` 預設為 `last`。
- `model` 會覆寫此 hook 執行所使用的 LLM（若已設定模型目錄，則必須為允許的模型）。

</Accordion>

### Gmail 整合

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

- Gateway 設定完成後，會在開機時自動啟動 `gog gmail watch serve`。 設定 `OPENCLAW_SKIP_GMAIL_WATCHER=1` 以停用。
- 請勿在 Gateway 執行期間另外啟動 `gog gmail watch serve`。

---

## Canvas 主機

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- 在 Gateway 連接埠下透過 HTTP 提供可由代理編輯的 HTML/CSS/JS 與 A2UI：
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- 僅限本機：請保持 `gateway.bind: "loopback"`（預設值）。
- 非 loopback 綁定：canvas 路由需要 Gateway 驗證（token/password/trusted-proxy），與其他 Gateway HTTP 介面相同。
- Node WebViews 通常不會傳送驗證標頭；當節點完成配對並連線後，Gateway 會允許私有 IP 回退機制，讓節點在不將機密暴露於 URL 的情況下載入 canvas/A2UI。
- 將即時重載（live-reload）用戶端注入到所提供的 HTML 中。
- 在目錄為空時自動建立初始 `index.html`。
- 同時在 `/__openclaw__/a2ui/` 提供 A2UI。
- 變更後需要重新啟動 gateway。
- 若目錄過大或出現 `EMFILE` 錯誤，請停用即時重載。

---

## 探索

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

- `minimal`（預設）：在 TXT 記錄中省略 `cliPath` + `sshPort`。
- `full`：包含 `cliPath` + `sshPort`。
- 主機名稱預設為 `openclaw`。 可使用 `OPENCLAW_MDNS_HOSTNAME` 覆寫。

### 廣域（DNS-SD）

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

在 `~/.openclaw/dns/` 下寫入單播 DNS-SD 區域。 若需跨網路探索，請搭配 DNS 伺服器（建議使用 CoreDNS）與 Tailscale 分割 DNS。

設定：`openclaw dns setup --apply`。

---

## 環境

### `env`（內嵌環境變數）

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

- 僅在程序環境變數中缺少該鍵時，才會套用內嵌環境變數。
- `.env` 檔案：目前工作目錄的 `.env` 與 `~/.openclaw/.env`（兩者皆不會覆寫現有變數）。
- `shellEnv`：從你的登入 shell 設定檔匯入缺少的預期鍵值。
- 完整優先順序請參閱 [Environment](/help/environment)。

### 環境變數替換

可在任何設定字串中使用 `${VAR_NAME}` 參考環境變數：

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- 僅比對全大寫名稱：`[A-Z_][A-Z0-9_]*`。
- 若變數缺失或為空，將在載入設定時拋出錯誤。
- 使用 `$${VAR}` 來表示字面值 `${VAR}`。
- 可搭配 `$include` 使用。

---

## Auth 儲存

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

- 每個 agent 的 Auth 設定檔儲存在 `<agentDir>/auth-profiles.json`。
- 從 `~/.openclaw/credentials/oauth.json` 匯入舊版 OAuth 設定。
- 請參閱 [OAuth](/concepts/oauth)。

---

## 日誌

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

- 預設日誌檔案：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`。
- 設定 `logging.file` 以使用固定路徑。
- 當使用 `--verbose` 時，`consoleLevel` 會提升為 `debug`。

---

## 精靈

由 CLI 精靈（`onboard`、`configure`、`doctor`）寫入的中繼資料：

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

## 身分識別

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

由 macOS 入門精靈寫入。 衍生預設值：

- `messages.ackReaction` 取自 `identity.emoji`（預設為 👀）
- `mentionPatterns` 取自 `identity.name`／`identity.emoji`
- `avatar` 支援：工作區相對路徑、`http(s)` URL，或 `data:` URI

---

## Bridge（舊版，已移除）

目前版本已不再包含 TCP bridge。 節點透過 Gateway WebSocket 連線。 `bridge.*` 鍵已不再屬於設定結構的一部分（在移除前驗證會失敗；可使用 `openclaw doctor --fix` 移除未知鍵）。

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
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`：在清除前保留已完成 cron 工作階段的時間長度。 預設：`24h`。

請參閱 [Cron Jobs](/automation/cron-jobs)。

---

## 媒體模型範本變數

在 `tools.media.*.models[].args` 中展開的範本預留位置：

| 變數                 | 說明                                   |
| ------------------ | ------------------------------------ |
| `{{Body}}`         | 完整的傳入訊息內容                            |
| `{{RawBody}}`      | 原始內容（不包含歷史／傳送者包裝）                    |
| `{{BodyStripped}}` | 已移除群組提及的訊息內容                         |
| `{{From}}`         | 發送者識別碼                               |
| `{{To}}`           | 目的地識別碼                               |
| `{{MessageSid}}`   | 頻道訊息 ID                              |
| `{{SessionId}}`    | 目前工作階段 UUID                          |
| `{{IsNewSession}}` | 建立新工作階段時為 `"true"`                   |
| `{{MediaUrl}}`     | 傳入媒體的偽 URL                           |
| `{{MediaPath}}`    | 本機媒體路徑                               |
| `{{MediaType}}`    | 媒體類型（image/audio/document/…）         |
| `{{Transcript}}`   | 音訊逐字稿                                |
| `{{Prompt}}`       | 用於 CLI 項目的已解析媒體提示詞                   |
| `{{MaxChars}}`     | 用於 CLI 項目的已解析最大輸出字元數                 |
| `{{ChatType}}`     | `"direct"` 或 `"group"`               |
| `{{GroupSubject}}` | 群組主題（盡力取得）                           |
| `{{GroupMembers}}` | 群組成員預覽（盡力取得）                         |
| `{{SenderName}}`   | 發送者顯示名稱（盡力取得）                        |
| `{{SenderE164}}`   | 發送者電話號碼（盡力取得）                        |
| `{{Provider}}`     | 服務提供者提示（whatsapp、telegram、discord 等） |

---

## 設定包含（`$include`）

將設定拆分為多個檔案：

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

**合併行為：**

- 單一檔案：取代其所屬的整個物件。
- 檔案陣列：依序進行深度合併（後者覆寫前者）。
- 同層鍵值：在 includes 之後合併（覆寫已包含的值）。
- 巢狀 includes：最多支援 10 層。
- 路徑：相對路徑（相對於包含該檔案的檔案）、絕對路徑，或 `../` 上層參照。
- 錯誤：針對遺失檔案、解析錯誤與循環包含提供清楚的錯誤訊息。

---

_相關： [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

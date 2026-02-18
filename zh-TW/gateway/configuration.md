---
summary: "All configuration options for ~/.openclaw/openclaw.json with examples"
read_when:
  - Adding or modifying config fields
title: "設定"
---

# Configuration 🔧

OpenClaw reads an optional **JSON5** config from `~/.openclaw/openclaw.json` (comments + trailing commas allowed).

If the file is missing, OpenClaw uses safe-ish defaults (embedded Pi agent + per-sender sessions + workspace `~/.openclaw/workspace`). You usually only need a config to:

- restrict who can trigger the bot (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, etc.)
- control group allowlists + mention behavior (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- customize message prefixes (`messages`)
- set the agent's workspace (`agents.defaults.workspace` or `agents.list[].workspace`)
- tune the embedded agent defaults (`agents.defaults`) and session behavior (`session`)
- set per-agent identity (`agents.list[].identity`)

> **New to configuration?** Check out the [Configuration Examples](/gateway/configuration-examples) guide for complete examples with detailed explanations!

## Strict config validation

OpenClaw only accepts configurations that fully match the schema.
Unknown keys, malformed types, or invalid values cause the Gateway to **refuse to start** for safety.

When validation fails:

- The Gateway does not boot.
- Only diagnostic commands are allowed (for example: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Run `openclaw doctor` to see the exact issues.
- Run `openclaw doctor --fix` (or `--yes`) to apply migrations/repairs.

Doctor never writes changes unless you explicitly opt into `--fix`/`--yes`.

## Schema + UI hints

The Gateway exposes a JSON Schema representation of the config via `config.schema` for UI editors.
The Control UI renders a form from this schema, with a **Raw JSON** editor as an escape hatch.

Channel plugins and extensions can register schema + UI hints for their config, so channel settings
stay schema-driven across apps without hard-coded forms.

Hints (labels, grouping, sensitive fields) ship alongside the schema so clients can render
better forms without hard-coding config knowledge.

## Apply + restart (RPC)

Use `config.apply` to validate + write the full config and restart the Gateway in one step.
It writes a restart sentinel and pings the last active session after the Gateway comes back.

Warning: `config.apply` replaces the **entire config**. If you want to change only a few keys,
use `config.patch` or `openclaw config set`. Keep a backup of `~/.openclaw/openclaw.json`.

Params:

- `raw` (string) — JSON5 payload for the entire config
- `baseHash` (optional) — config hash from `config.get` (required when a config already exists)
- `sessionKey` (optional) — last active session key for the wake-up ping
- `note` (optional) — note to include in the restart sentinel
- `restartDelayMs` (optional) — delay before restart (default 2000)

Example (via `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Partial updates (RPC)

Use `config.patch` to merge a partial update into the existing config without clobbering
unrelated keys. It applies JSON merge patch semantics:

- objects merge recursively
- `null` deletes a key
- arrays replace
  Like `config.apply`, it validates, writes the config, stores a restart sentinel, and schedules
  the Gateway restart (with an optional wake when `sessionKey` is provided).

Params:

- `raw` (string) — JSON5 payload containing just the keys to change
- `baseHash` (required) — config hash from `config.get`
- `sessionKey` (optional) — last active session key for the wake-up ping
- `note` (optional) — note to include in the restart sentinel
- `restartDelayMs` (optional) — delay before restart (default 2000)

Example:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Minimal config (recommended starting point)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Build the default image once with:

```bash
scripts/sandbox-setup.sh
```

## Self-chat mode (recommended for group control)

To prevent the bot from responding to WhatsApp @-mentions in groups (only respond to specific text triggers):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
      },
    ],
  },
  channels: {
    whatsapp: {
      // Allowlist is DMs only; including your own number enables self-chat mode.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## Config Includes (`$include`)

Split your config into multiple files using the `$include` directive. This is useful for:

- Organizing large configs (e.g., per-client agent definitions)
- Sharing common settings across environments
- Keeping sensitive configs separate

### Basic usage

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },

  // Include a single file (replaces the key's value)
  agents: { $include: "./agents.json5" },

  // Include multiple files (deep-merged in order)
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### Merge behavior

- **Single file**: Replaces the object containing `$include`
- **Array of files**: Deep-merges files in order (later files override earlier ones)
- **With sibling keys**: Sibling keys are merged after includes (override included values)
- **Sibling keys + arrays/primitives**: Not supported (included content must be an object)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### Nested includes

Included files can themselves contain `$include` directives (up to 10 levels deep):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### Path resolution

- **Relative paths**: Resolved relative to the including file
- **Absolute paths**: Used as-is
- **Parent directories**: `../` references work as expected

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### Error handling

- **Missing file**: Clear error with resolved path
- **Parse error**: Shows which included file failed
- **Circular includes**: Detected and reported with include chain

### Example: Multi-client legal setup

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## Common options

### Env vars + `.env`

OpenClaw reads env vars from the parent process (shell, launchd/systemd, CI, etc.).

Additionally, it loads:

- `.env` from the current working directory (if present)
- a global fallback `.env` from `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`)

Neither `.env` file overrides existing env vars.

You can also provide inline env vars in config. These are only applied if the
process env is missing the key (same non-overriding rule):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

See [/environment](/help/environment) for full precedence and sources.

### `env.shellEnv` (optional)

Opt-in convenience: if enabled and none of the expected keys are set yet, OpenClaw runs your login shell and imports only the missing expected keys (never overrides).
This effectively sources your shell profile.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Env var equivalent:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### Env var substitution in config

You can reference environment variables directly in any config string value using
`${VAR_NAME}` syntax. Variables are substituted at config load time, before validation.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**Rules:**

- Only uppercase env var names are matched: `[A-Z_][A-Z0-9_]*`
- Missing or empty env vars throw an error at config load
- Escape with `$${VAR}` to output a literal `${VAR}`
- Works with `$include` (included files also get substitution)

**Inline substitution:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### Auth storage (OAuth + API keys)

OpenClaw stores **per-agent** auth profiles (OAuth + API keys) in:

- `<agentDir>/auth-profiles.json` (default: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

另請參閱：[/concepts/oauth](/concepts/oauth)

舊版 OAuth 匯入：

- `~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）

內嵌的 Pi agent 會在以下位置維護執行期快取：

- `<agentDir>/auth.json`（自動管理；請勿手動編輯）

舊版 agent 目錄（多 agent 之前）：

- `~/.openclaw/agent/*`（由 `openclaw doctor` 遷移至 `~/.openclaw/agents/<defaultAgentId>/agent/*`）

覆寫設定：

- OAuth 目錄（僅供舊版匯入）：`OPENCLAW_OAUTH_DIR`
- Agent 目錄（預設 agent 根目錄覆寫）：`OPENCLAW_AGENT_DIR`（建議使用），`PI_CODING_AGENT_DIR`（舊版）

首次使用時，OpenClaw 會將 `oauth.json` 項目匯入 `auth-profiles.json`。

### `auth`

Auth profiles 的選用中繼資料。 這**不會**儲存祕密；它會將
profile ID 對應到提供者 + 模式（以及選用的電子郵件），並定義用於容錯移轉的提供者輪替順序。

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

### `agents.list[].identity`

每個 agent 可選用的身分識別，用於預設值與使用者體驗。 此項由 macOS 上線引導助理寫入。

若已設定，OpenClaw 會推導預設值（僅在你尚未明確設定時）：

- `messages.ackReaction` 取自**作用中 agent** 的 `identity.emoji`（回退為 👀）
- `agents.list[].groupChat.mentionPatterns` 取自 agent 的 `identity.name`/`identity.emoji`（因此在 Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp 的群組中可使用「@Samantha」）
- `identity.avatar` 可接受工作區相對的圖片路徑或遠端 URL/data URL。 本機檔案必須位於 agent 工作區內。

`identity.avatar` 可接受：

- 工作區相對路徑（必須位於 agent 工作區內）
- `http(s)` URL
- `data:` URI

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

### `wizard`

由 CLI 精靈（`onboard`、`configure`、`doctor`）寫入的中繼資料。

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

### `記錄`

- 預設日誌檔案：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- 如果你想要固定路徑，請將 `logging.file` 設為 `/tmp/openclaw/openclaw.log`。
- 主控台輸出可透過以下方式分別調整：
  - `logging.consoleLevel`（預設為 `info`，使用 `--verbose` 時提升為 `debug`）
  - `logging.consoleStyle`（`pretty` | `compact` | `json`）
- 工具摘要可被遮蔽以避免洩漏祕密：
  - `logging.redactSensitive`（`off` | `tools`，預設：`tools`）
  - `logging.redactPatterns`（正則字串陣列；會覆寫預設值）

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

控制 WhatsApp 私訊（DM）的處理方式：

- `"pairing"`（預設）：未知的傳送者會收到配對碼；擁有者必須核准
- `"allowlist"`: only allow senders in `channels.whatsapp.allowFrom` (or paired allow store)
- `"open"`: allow all inbound DMs (**requires** `channels.whatsapp.allowFrom` to include `"*"`)
- `"disabled"`: ignore all inbound DMs

配對碼在 1 小時後過期；機器人只會在建立新的請求時傳送配對碼。 15. 待處理的私訊配對請求預設每個頻道最多 **3 個**。

Pairing approvals:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

可觸發 WhatsApp 自動回覆的 E.164 電話號碼允許清單（**僅限 DM**）。
If empty and `channels.whatsapp.dmPolicy="pairing"`, unknown senders will receive a pairing code.
群組請使用 `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`。

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50, // optional inbound media cap (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

控制是否將傳入的 WhatsApp 訊息標示為已讀（藍勾）。 24. 預設：`true`。

Self-chat mode always skips read receipts, even when enabled.

每個帳號的覆寫設定：`channels.whatsapp.accounts.<id>27. `.sendReadReceipts\`。

```json5
28. {
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (multi-account)

在同一個閘道中執行多個 WhatsApp 帳號：

```json5
31. {
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

注意事項：

- Outbound commands default to account `default` if present; otherwise the first configured account id (sorted).
- The legacy single-account Baileys auth dir is migrated by `openclaw doctor` into `whatsapp/default`.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

Run multiple accounts per channel (each account has its own `accountId` and optional `name`):

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

注意事項：

- `default` is used when `accountId` is omitted (CLI + routing).
- 8. 環境變數中的 token 僅適用於 **default** 帳戶。
- Base channel settings (group policy, mention gating, etc.) apply to all accounts unless overridden per account.
- Use `bindings[].match.accountId` to route each account to a different agents.defaults.

### Group chat mention gating (`agents.list[].groupChat` + `messages.groupChat`)

Group messages default to **require mention** (either metadata mention or regex patterns). Applies to WhatsApp, Telegram, Discord, Google Chat, and iMessage group chats.

**Mention types:**

- **Metadata mentions**: Native platform @-mentions (e.g., WhatsApp tap-to-mention). 在 WhatsApp 自聊模式中會被忽略（請參閱 `channels.whatsapp.allowFrom`）。
- **Text patterns**: Regex patterns defined in `agents.list[].groupChat.mentionPatterns`. 不論是否為自聊模式，皆會檢查。
- 50. 只有在能進行提及偵測時才會強制執行提及門檻（原生提及或至少一個 `mentionPattern`）。

```json5
1. {
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

2. `messages.groupChat.historyLimit` 設定群組歷史上下文的全域預設值。 3. 頻道可以透過 `channels.<channel>` 覆寫.historyLimit`（或 `channels.<channel>4. `.accounts.*.historyLimit`（適用於多帳號）。 5. 設定為 `0` 以停用歷史包裝。

#### 6. 私訊（DM）歷史限制

7. 私訊對話使用由代理管理的、以工作階段為基礎的歷史。 8. 你可以限制每個 DM 工作階段保留的使用者回合數：

```json5
9. {
  channels: {
    telegram: {
      dmHistoryLimit: 30, // 將 DM 工作階段限制為 30 個使用者回合
      dms: {
        "123456789": { historyLimit: 50 }, // 依使用者覆寫（使用者 ID）
      },
    },
  },
}
```

10. 解析順序：

1. 11. 單一 DM 覆寫：`channels.<provider>`12. `.dms[userId].historyLimit`
2. 13. 供應商預設：`channels.<provider>`14. `.dmHistoryLimit`
3. 15. 無限制（保留所有歷史）

16) 支援的供應商：`telegram`、`whatsapp`、`discord`、`slack`、`signal`、`imessage`、`msteams`。

17. 依代理覆寫（一旦設定即具有優先權，即使是 `[]`）：

```json5
18. {
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

19. 提及閘控（mention gating）預設值依各頻道設定（`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.imessage.groups`、`channels.discord.guilds`）。 20. 當設定了 `*.groups` 時，它同時也會作為群組允許清單；包含 `"*"` 以允許所有群組。

21. 僅回應**特定文字觸發詞**（忽略原生 @ 提及）：

```json5
22. {
  channels: {
    whatsapp: {
      // 包含你自己的號碼以啟用自聊模式（忽略原生 @ 提及）。
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // 只有這些文字模式會觸發回應
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### 23. 群組政策（依頻道）

24. 使用 `channels.*.groupPolicy` 來控制是否接受群組／房間訊息：

```json5
25. {
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"],
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: {
          channels: { help: { allow: true } },
        },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
  },
}
```

注意事項：

- 26. `"open"`：群組會略過允許清單；提及閘控仍然適用。
- 27. `"disabled"`：封鎖所有群組／房間訊息。
- 28. `"allowlist"`：僅允許符合設定之允許清單的群組／房間。
- 29. `channels.defaults.groupPolicy` 會在供應商的 `groupPolicy` 未設定時設定預設值。
- 30. WhatsApp／Telegram／Signal／iMessage／Microsoft Teams 使用 `groupAllowFrom`（備援：明確的 `allowFrom`）。
- 31. Discord／Slack 使用頻道允許清單（`channels.discord.guilds.*.channels`、`channels.slack.channels`）。
- 32. 群組 DM（Discord／Slack）仍受 `dm.groupEnabled` + `dm.groupChannels` 控制。
- 33. 預設為 `groupPolicy: "allowlist"`（除非被 `channels.defaults.groupPolicy` 覆寫）；若未設定任何允許清單，群組訊息會被封鎖。

### 34. 多代理路由（`agents.list` + `bindings`）

35. 在單一 Gateway 內執行多個彼此隔離的代理（獨立的工作區、`agentDir`、工作階段）。
36. 傳入訊息會透過繫結（bindings）路由至代理。

- 37. `agents.list[]`：每個代理的覆寫設定。
  - 38. `id`：穩定的代理 ID（必填）。
  - 39. `default`：選用；若設定了多個，第一個生效並記錄警告。
        40. 若皆未設定，清單中的**第一個項目**即為預設代理。
  - `name`：代理的顯示名稱。
  - `workspace`：預設為 `~/.openclaw/workspace-<agentId>`（對於 `main`，回退至 `agents.defaults.workspace`）。
  - `agentDir`：預設為 `~/.openclaw/agents/<agentId>/agent`。
  - `model`：每個代理的預設模型，會覆蓋該代理的 `agents.defaults.model`。
    - 字串形式：`"provider/model"`，僅覆蓋 `agents.defaults.model.primary`。
    - 物件形式：`{ primary, fallbacks }`（`fallbacks` 會覆蓋 `agents.defaults.model.fallbacks`；`[]` 會為該代理停用全域後備）。
  - `identity`：每個代理的名稱／主題／表情符號（用於提及模式 + 回應反應）。
  - `groupChat`：每個代理的提及門控（`mentionPatterns`）。
  - `sandbox`：每個代理的沙箱設定（覆蓋 `agents.defaults.sandbox`）。
    - `mode`：`"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`：`"none"` | `"ro"` | `"rw"`
    - `scope`：`"session"` | `"agent"` | `"shared"`
    - `workspaceRoot`：自訂沙箱工作區根目錄
    - `docker`：每個代理的 Docker 覆寫（例如 `image`、`network`、`env`、`setupCommand`、限制；當 `scope: "shared"` 時會被忽略）
    - `browser`：每個代理的沙箱化瀏覽器覆寫（當 `scope: "shared"` 時會被忽略）
    - `prune`：每個代理的沙箱修剪覆寫（當 `scope: "shared"` 時會被忽略）
  - `subagents`：每個代理的子代理預設值。
    - `allowAgents`：允許從此代理進行 `sessions_spawn` 的代理 ID 白名單（`["*"]` = 允許任何；預設：僅相同代理）
  - `tools`：每個代理的工具限制（在沙箱工具策略之前套用）。
    - `profile`：基礎工具設定檔（在 allow/deny 之前套用）
    - `allow`：允許的工具名稱陣列
    - `deny`：禁止的工具名稱陣列（禁止優先）
- `agents.defaults`：共用的代理預設值（模型、工作區、沙箱等）。
- `bindings[]`：將入站訊息路由到某個 `agentId`。
  - `match.channel`（必填）
  - `match.accountId`（選填；`*` = 任一帳號；省略 = 預設帳號）
  - `match.peer`（選填；`{ kind: direct|group|channel, id }`）
  - `match.guildId` / `match.teamId`（選填；特定於頻道）

確定性的匹配順序：

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`（精確匹配，無 peer/guild/team）
5. `match.accountId: "*"`（整個頻道，無 peer/guild/team）
6. 預設代理（`agents.list[].default`，否則為清單第一個項目，否則為 `"main"`）

在每個匹配層級中，`bindings` 內第一個匹配的項目勝出。

#### 每個代理的存取設定檔（多代理）

每個代理都可以攜帶自己的沙箱 + 工具策略。 使用此功能在同一個閘道中混合存取
層級：

- **完整存取**（個人代理）
- **Read-only** tools + workspace
- **No filesystem access** (messaging/session tools only)

See [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) for precedence and
additional examples.

Full access (no sandbox):

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

Read-only tools + read-only workspace:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
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

37. 無檔案系統存取（啟用訊息/工作階段工具）：

```json5
38. {
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
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

Example: two WhatsApp accounts → two agents:

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
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

### 41. `tools.agentToAgent`（選用）

Agent-to-agent messaging is opt-in:

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

### `messages.queue`

Controls how inbound messages behave when an agent run is already active.

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

Debounce rapid inbound messages from the **same sender** so multiple back-to-back
messages become a single agent turn. 49. 去彈跳的作用範圍為每個頻道 + 對話，並使用最新訊息作為回覆串接／ID。

```json5
50. {
  messages: {
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

注意事項：

- Debounce batches **text-only** messages; media/attachments flush immediately.
- Control commands (e.g. `/queue`, `/new`) bypass debouncing so they stay standalone.

### `commands` (chat command handling)

Controls how chat commands are enabled across connectors.

```json5
{
  commands: {
    native: "auto", // register native commands when supported (auto)
    text: true, // parse slash commands in chat messages
    bash: false, // allow ! (alias: /bash) (host-only; requires tools.elevated allowlists)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false, // allow /config (writes to disk)
    debug: false, // allow /debug (runtime-only overrides)
    restart: false, // allow /restart + gateway restart tool
    useAccessGroups: true, // enforce access-group allowlists/policies for commands
  },
}
```

注意事項：

- Text commands must be sent as a **standalone** message and use the leading `/` (no plain-text aliases).
- `commands.text: false` disables parsing chat messages for commands.
- `commands.native: "auto"` (default) turns on native commands for Discord/Telegram and leaves Slack off; unsupported channels stay text-only.
- Set `commands.native: true|false` to force all, or override per channel with `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool or `"auto"`). `false` clears previously registered commands on Discord/Telegram at startup; Slack commands are managed in the Slack app.
- `channels.telegram.customCommands` adds extra Telegram bot menu entries. Names are normalized; conflicts with native commands are ignored.
- `commands.bash: true` enables `! <cmd>` to run host shell commands (`/bash <cmd>` also works as an alias). Requires `tools.elevated.enabled` and allowlisting the sender in `tools.elevated.allowFrom.<channel>` 下。
- `commands.bashForegroundMs` controls how long bash waits before backgrounding. While a bash job is running, new `! <cmd>` requests are rejected (one at a time).
- `commands.config: true` enables `/config` (reads/writes `openclaw.json`).
- `channels.<provider>.configWrites` gates config mutations initiated by that channel (default: true). 這適用於 `/config set|unset` 以及供應商特定的自動遷移（Telegram 超級群組 ID 變更、Slack 頻道 ID 變更）。
- `commands.debug: true` 會啟用 `/debug`（僅執行期覆寫）。
- `commands.restart: true` enables `/restart` and the gateway tool restart action.
- `commands.useAccessGroups: false` 允許指令繞過存取群組的允許清單／政策。
- Slash commands and directives are only honored for **authorized senders**. Authorization is derived from
  channel allowlists/pairing plus `commands.useAccessGroups`.

### `web`（WhatsApp web 頻道執行階段）

WhatsApp runs through the gateway’s web channel (Baileys Web). It starts automatically when a linked session exists.
Set `web.enabled: false` to keep it off by default.

```json5
{
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

### `channels.telegram` (bot transport)

OpenClaw starts Telegram only when a `channels.telegram` config section exists. The bot token is resolved from `channels.telegram.botToken` (or `channels.telegram.tokenFile`), with `TELEGRAM_BOT_TOKEN` as a fallback for the default account.
Set `channels.telegram.enabled: false` to disable automatic startup.
Multi-account support lives under `channels.telegram.accounts` (see the multi-account section above). Env tokens only apply to the default account.
Set `channels.telegram.configWrites: false` to block Telegram-initiated config writes (including supergroup ID migrations and `/config set|unset`).

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50, // include last N group messages as context (0 disables)
      replyToMode: "first", // off | first | all
      linkPreview: true, // toggle outbound link previews
      streamMode: "partial", // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {
        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        // transport overrides
        autoSelectFamily: false,
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook", // requires webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

Draft streaming notes:

- Uses Telegram `sendMessageDraft` (draft bubble, not a real message).
- Requires **private chat topics** (message_thread_id in DMs; bot has topics enabled).
- `/reasoning stream` streams reasoning into the draft, then sends the final answer.
  Retry policy defaults and behavior are documented in [Retry policy](/concepts/retry).

### `channels.discord` (bot transport)

Configure the Discord bot by setting the bot token and optional gating:
Multi-account support lives under `channels.discord.accounts` (see the multi-account section above). Env tokens only apply to the default account.

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8, // clamp inbound media size
      allowBots: false, // allow bot-authored messages
      actions: {
        // tool action gates (false disables)
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
      dm: {
        enabled: true, // disable all DMs when false
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false, // enable group DMs
        groupChannels: ["openclaw-dm"], // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {
          // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false, // per-guild default
          reactionNotifications: "own", // off | own | all | allowlist
          users: ["987654321098765432"], // optional per-guild user allowlist
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
      historyLimit: 20, // include last N guild messages as context
      textChunkLimit: 2000, // optional outbound text chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      maxLinesPerMessage: 17, // soft max lines per message (Discord UI clipping)
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

OpenClaw starts Discord only when a `channels.discord` config section exists. The token is resolved from `channels.discord.token`, with `DISCORD_BOT_TOKEN` as a fallback for the default account (unless `channels.discord.enabled` is `false`). Use `user:<id>` (DM) or `channel:<id>` (guild channel) when specifying delivery targets for cron/CLI commands; bare numeric IDs are ambiguous and rejected.
Guild slugs are lowercase with spaces replaced by `-`; channel keys use the slugged channel name (no leading `#`). Prefer guild ids as keys to avoid rename ambiguity.
Bot-authored messages are ignored by default. Enable with `channels.discord.allowBots` (own messages are still filtered to prevent self-reply loops).
Reaction notification modes:

- `off`：無反應事件。
- `own`: reactions on the bot's own messages (default).
- `all`：所有訊息上的所有反應。
- `allowlist`：來自 `guilds.<id>.users` 的反應（套用於所有訊息；空清單表示停用）。
  Outbound text is chunked by `channels.discord.textChunkLimit` (default 2000). 將 `channels.discord.chunkMode="newline"` 設為在長度分段前，先依空白行（段落邊界）切分。 Discord clients can clip very tall messages, so `channels.discord.maxLinesPerMessage` (default 17) splits long multi-line replies even when under 2000 chars.
  Retry policy defaults and behavior are documented in [Retry policy](/concepts/retry).

### `channels.googlechat` (Chat API webhook)

Google Chat runs over HTTP webhooks with app-level auth (service account).
Multi-account support lives under `channels.googlechat.accounts` (see the multi-account section above). Env vars only apply to the default account.

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; improves mention detection
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"], // optional; "open" requires ["*"]
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

注意事項：

- Service account JSON can be inline (`serviceAccount`) or file-based (`serviceAccountFile`).
- Env fallbacks for the default account: `GOOGLE_CHAT_SERVICE_ACCOUNT` or `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- `audienceType` + `audience` must match the Chat app’s webhook auth config.
- Use `spaces/<spaceId>` or `users/<userId|email>` when setting delivery targets.

### `channels.slack` (socket mode)

Slack runs in Socket Mode and requires both a bot token and app token:

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"],
      },
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
      historyLimit: 50, // include last N channel/group messages as context (0 disables)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
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

Multi-account support lives under `channels.slack.accounts` (see the multi-account section above). Env tokens only apply to the default account.

當提供者啟用且兩個 token 都已設定（透過設定檔或 `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`）時，OpenClaw 會啟動 Slack。 為 cron/CLI 指令指定投遞目標時，使用 `user:<id>`（私訊）或 `channel:<id>`。
Set `channels.slack.configWrites: false` to block Slack-initiated config writes (including channel ID migrations and `/config set|unset`).

Bot-authored messages are ignored by default. 可透過 `channels.slack.allowBots` 或 `channels.slack.channels.<id>` 啟用。.allowBots\` 啟用。

Reaction notification modes:

- `off`：無反應事件。
- `own`: reactions on the bot's own messages (default).
- `all`：所有訊息上的所有反應。
- `allowlist`: reactions from `channels.slack.reactionAllowlist` on all messages (empty list disables).

Thread session isolation:

- `channels.slack.thread.historyScope` controls whether thread history is per-thread (`thread`, default) or shared across the channel (`channel`).
- `channels.slack.thread.inheritParent` controls whether new thread sessions inherit the parent channel transcript (default: false).

Slack action groups (gate `slack` tool actions):

| 動作群組       | Default | Notes       |
| ---------- | ------- | ----------- |
| reactions  | enabled | 新增反應＋列出反應   |
| messages   | enabled | 讀取／傳送／編輯／刪除 |
| pins       | enabled | 釘選／取消釘選／列出  |
| memberInfo | enabled | 成員資訊        |
| emojiList  | enabled | 自訂表情符號清單    |

### `channels.mattermost`（bot token）

Mattermost 以外掛形式提供，未隨核心安裝一併提供。
Install it first: `openclaw plugins install @openclaw/mattermost` (or `./extensions/mattermost` from a git checkout).

Mattermost 需要 bot token 以及伺服器的 base URL：

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

OpenClaw starts Mattermost when the account is configured (bot token + base URL) and enabled. 對於預設帳號（除非 `channels.mattermost.enabled` 為 `false`），token + base URL 會從 `channels.mattermost.botToken` + `channels.mattermost.baseUrl`，或 `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` 解析取得。

Chat modes:

- `oncall`（預設）：僅在被 @ 提及時回應頻道訊息。
- `onmessage`：回覆每一則頻道訊息。
- `onchar`: respond when a message starts with a trigger prefix (`channels.mattermost.oncharPrefixes`, default `[">", "!"]`).

Access control:

- 預設私訊：`channels.mattermost.dmPolicy="pairing"`（未知寄件者會取得配對碼）。
- 公開私訊：`channels.mattermost.dmPolicy="open"` 加上 `channels.mattermost.allowFrom=["*"]`。
- Groups: `channels.mattermost.groupPolicy="allowlist"` by default (mention-gated). 使用 `channels.mattermost.groupAllowFrom` 來限制寄件者。

多帳號支援位於 `channels.mattermost.accounts`（見上方的多帳號章節）。 Env vars only apply to the default account.
指定投遞目標時，使用 `channel:<id>` 或 `user:<id>`（或 `@username`）；未加前綴的 ID 會被視為頻道 ID。

### 1. `channels.signal`（signal-cli）

Signal reactions can emit system events (shared reaction tooling):

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // include last N group messages as context (0 disables)
    },
  },
}
```

2. 回應通知模式：

- `off`：無反應事件。
- `own`: reactions on the bot's own messages (default).
- `all`：所有訊息上的所有反應。
- `allowlist`: reactions from `channels.signal.reactionAllowlist` on all messages (empty list disables).

### `channels.imessage` (imsg CLI)

OpenClaw spawns `imsg rpc` (JSON-RPC over stdio). No daemon or port required.

```json5
4. {
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // 使用 SSH 包裝器時，透過 SCP 傳送遠端附件
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50, // 將最近 N 則群組訊息作為上下文（0 代表停用）
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

Multi-account support lives under `channels.imessage.accounts` (see the multi-account section above).

注意事項：

- Requires Full Disk Access to the Messages DB.
- The first send will prompt for Messages automation permission.
- Prefer `chat_id:<id>` targets. Use `imsg chats --limit 20` to list chats.
- `channels.imessage.cliPath` can point to a wrapper script (e.g. `ssh` to another Mac that runs `imsg rpc`); use SSH keys to avoid password prompts.
- For remote SSH wrappers, set `channels.imessage.remoteHost` to fetch attachments via SCP when `includeAttachments` is enabled.

範例包裝器：

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

Sets the **single global workspace directory** used by the agent for file operations.

預設：`~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

If `agents.defaults.sandbox` is enabled, non-main sessions can override this with their
own per-scope workspaces under `agents.defaults.sandbox.workspaceRoot`.

### `agents.defaults.repoRoot`

Optional repository root to show in the system prompt’s Runtime line. If unset, OpenClaw
tries to detect a `.git` directory by walking upward from the workspace (and current
working directory). The path must exist to be used.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

5. 停用自動建立工作區啟動檔案（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 以及 `BOOTSTRAP.md`）。

Use this for pre-seeded deployments where your workspace files come from a repo.

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Max characters of each workspace bootstrap file injected into the system prompt
before truncation. 預設：`20000`.

When a file exceeds this limit, OpenClaw logs a warning and injects a truncated
head/tail with a marker.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

Sets the user’s timezone for **system prompt context** (not for timestamps in
message envelopes). If unset, OpenClaw uses the host timezone at runtime.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Controls the **time format** shown in the system prompt’s Current Date & Time section.
Default: `auto` (OS preference).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `訊息`

Controls inbound/outbound prefixes and optional ack reactions.
See [Messages](/concepts/messages) for queueing, sessions, and streaming context.

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

`responsePrefix` is applied to **all outbound replies** (tool summaries, block
streaming, final replies) across channels unless already present.

Overrides can be configured per channel and per account:

- `channels.<channel>.responsePrefix`
- `channels.<channel>.accounts.<id>.responsePrefix`

Resolution order (most specific wins):

1. `channels.<channel>.accounts.<id>.responsePrefix`
2. `channels.<channel>.responsePrefix`
3. `messages.responsePrefix`

Semantics:

- `undefined` falls through to the next level.
- `""` explicitly disables the prefix and stops the cascade.
- `"auto"` derives `[{identity.name}]` for the routed agent.

Overrides apply to all channels, including extensions, and to every outbound reply kind.

If `messages.responsePrefix` is unset, no prefix is applied by default. WhatsApp self-chat
replies are the exception: they default to `[{identity.name}]` when set, otherwise
`[openclaw]`, so same-phone conversations stay legible.
Set it to `"auto"` to derive `[{identity.name}]` for the routed agent (when set).

#### Template variables

The `responsePrefix` string can include template variables that resolve dynamically:

| 變數                | Description            | 範例                                         |
| ----------------- | ---------------------- | ------------------------------------------ |
| `{model}`         | Short model name       | `claude-opus-4-6`, `gpt-4o`                |
| `{modelFull}`     | Full model identifier  | `anthropic/claude-opus-4-6`                |
| `{provider}`      | Provider name          | `anthropic`, `openai`                      |
| `{thinkingLevel}` | Current thinking level | `high`, `low`, `off`                       |
| `{identity.name}` | Agent identity name    | (same as `"auto"` mode) |

Variables are case-insensitive (`{MODEL}` = `{model}`). `{think}` is an alias for `{thinkingLevel}`.
6. 尚未解析的變數會以字面文字保留。

```json5
2. {
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]",
  },
}
```

3. 範例輸出：`[claude-opus-4-6 | think:high] 這是我的回應...`

4. WhatsApp 的入站前綴是透過 `channels.whatsapp.messagePrefix` 設定（已淘汰：
   `messages.messagePrefix`）。 5. 預設保持 **不變**：當 `channels.whatsapp.allowFrom` 為空時為 `"[openclaw]"`，否則為 `""`（沒有前綴）。 7. 當使用
   `"[openclaw]"` 時，若被路由的代理已設定 `identity.name`，OpenClaw 會改用 `[{identity.name}]`。

`ackReaction` sends a best-effort emoji reaction to acknowledge inbound messages
on channels that support reactions (Slack/Discord/Telegram/Google Chat). 8. 預設為
已設定的作用中代理之 `identity.emoji`，否則為 `"👀"`。 9. 將其設為 `""` 以停用。

10. `ackReactionScope` 控制反應觸發的時機：

- 9. `group-mentions`（預設）：僅在群組／房間需要提及 **且** 機器人被提及時
- 10. `group-all`：所有群組／房間訊息
- 11. `direct`：僅限私訊
- 12. `all`：所有訊息

`removeAckAfterReply` removes the bot’s ack reaction after a reply is sent
(Slack/Discord/Telegram/Google Chat only). 13. 預設值：`false`。

#### 17. `messages.tts`

18. 為外送回覆啟用文字轉語音。 14. 啟用時，OpenClaw 會使用 ElevenLabs 或 OpenAI 產生音訊，並將其附加到回應中。 15. Telegram 使用 Opus 語音備忘；其他頻道傳送 MP3 音訊。

```json5
21. {
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
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

注意事項：

- `messages.tts.auto` controls auto‑TTS (`off`, `always`, `inbound`, `tagged`).
- `/tts off|always|inbound|tagged` sets the per‑session auto mode (overrides config).
- 16. `messages.tts.enabled` 為舊版；doctor 會將其遷移至 `messages.tts.auto`。
- 25. `prefsPath` 儲存本地覆寫設定（提供者/限制/摘要）。
- 26. `maxTextLength` 是 TTS 輸入的硬性上限；摘要會被截斷以符合限制。
- 27. `summaryModel` 會覆寫自動摘要所使用的 `agents.defaults.model.primary`。
  - Accepts `provider/model` or an alias from `agents.defaults.models`.
- `modelOverrides` enables model-driven overrides like `[[tts:...]]` tags (on by default).
- `/tts limit` and `/tts summary` control per-user summarization settings.
- `apiKey` values fall back to `ELEVENLABS_API_KEY`/`XI_API_KEY` and `OPENAI_API_KEY`.
- `elevenlabs.baseUrl` overrides the ElevenLabs API base URL.
- 33. `elevenlabs.voiceSettings` 支援 `stability`/`similarityBoost`/`style`（0..1）、
      `useSpeakerBoost`，以及 `speed`（0.5..2.0）。

### 34. `talk`

35. Talk 模式的預設值（macOS/iOS/Android）。 Voice IDs fall back to `ELEVENLABS_VOICE_ID` or `SAG_VOICE_ID` when unset.
    `apiKey` falls back to `ELEVENLABS_API_KEY` (or the gateway’s shell profile) when unset.
36. `voiceAliases` 讓 Talk 指令可使用友善名稱（例如 `"voice":"Clawd"`）。

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

### `agents.defaults`

Controls the embedded agent runtime (model/thinking/verbose/timeouts).
18. `agents.defaults.models` 定義已設定的模型目錄（同時作為 `/model` 的允許清單）。
`agents.defaults.model.primary` sets the default model; `agents.defaults.model.fallbacks` are global failovers.
`agents.defaults.imageModel` is optional and is **only used if the primary model lacks image input**.
19. 每個 `agents.defaults.models` 項目可以包含：

- `alias` (optional model shortcut, e.g. `/opus`).
- `params` (optional provider-specific API params passed through to the model request).

`params` is also applied to streaming runs (embedded agent + compaction). Supported keys today: `temperature`, `maxTokens`. These merge with call-time options; caller-supplied values win. `temperature` is an advanced knob—leave unset unless you know the model’s defaults and need a change.

Example:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

Z.AI GLM-4.x models automatically enable thinking mode unless you:

- 20. 設定 `--thinking off`，或
- define `agents.defaults.models["zai/<model>"].params.thinking` yourself.

OpenClaw also ships a few built-in alias shorthands. 17. 預設值僅在模型
已存在於 `agents.defaults.models` 時才會套用：

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- 21. `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

If you configure the same alias name (case-insensitive) yourself, your value wins (defaults never override).

Example: Opus 4.6 primary with MiniMax M2.1 fallback (hosted MiniMax):

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
    },
  },
}
```

MiniMax auth: set `MINIMAX_API_KEY` (env) or configure `models.providers.minimax`.

#### `agents.defaults.cliBackends` (CLI fallback)

21. 可選的 CLI 後端，用於僅文字的備援執行（不呼叫工具）。 These are useful as a
    backup path when API providers fail. 當你設定了可接受檔案路徑的 `imageArg` 時，支援影像直通。

注意事項：

- CLI backends are **text-first**; tools are always disabled.
- Sessions are supported when `sessionArg` is set; session ids are persisted per backend.
- 23. 對於 `claude-cli`，已內建預設值。 Override the command path if PATH is minimal
      (launchd/systemd).

Example:

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

```json5
24. {
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false,
            },
          },
        },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free",
        ],
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
      heartbeat: {
        every: "30m",
        target: "last",
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000,
      },
      contextTokens: 200000,
    },
  },
}
```

#### 38. `agents.defaults.contextPruning`（工具結果修剪）

25. `agents.defaults.contextPruning` 會在請求送至 LLM 前，從記憶體中的上下文修剪 **舊的工具結果**。
    此功能旨在降低隨時間累積大量工具輸出的健談型代理之 token 使用量。

高階說明：

High level:

- 保護最後 `keepLastAssistants` 則助理訊息（該點之後的工具結果不會被修剪）。
- 保護啟動前綴（第一則使用者訊息之前的內容都不會被修剪）。
- Protects the bootstrap prefix (nothing before the first user message is pruned).
- Modes:
  - `adaptive`: soft-trims oversized tool results (keep head/tail) when the estimated context ratio crosses `softTrimRatio`.
    `aggressive`：一律在截止點之前，將符合條件的工具結果以 `hardClear.placeholder` 取代（不做比例檢查）。
  - 軟性 vs 硬性修剪（送往 LLM 的上下文中有何改變）：

**軟性修剪**：僅適用於**過大**的工具結果。

- **Soft-trim**: only for _oversized_ tool results. 26. 保留開頭與結尾，並在中間插入 `...`。
  - Before: `toolResult("…very long output…")`
  - After: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
- **Hard-clear**: replaces the entire tool result with the placeholder.
  - Before: `toolResult("…very long output…")`
  - 27. 之後：`toolResult("[Old tool result content cleared]")`

包含**影像區塊**的工具結果目前會被略過（永遠不會被修剪／清除）。

- 估算的「上下文比例」是以**字元數**（近似）為基礎，而非精確的 token。
- 若工作階段尚未包含至少 `keepLastAssistants` 則助理訊息，則會跳過修剪。
- 1. 如果工作階段尚未包含至少 `keepLastAssistants` 則助理訊息，則會跳過修剪。
- 預設（adaptive）：

28. 預設（自適應）：

```json5
29. {
  agents: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

To disable:

```json5
30. {
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

Defaults (when `mode` is `"adaptive"` or `"aggressive"`):

- `keepLastAssistants`：`3`
- 31. `softTrimRatio`：`0.3`（僅適用於自適應）
- 32. `hardClearRatio`：`0.5`（僅適用於自適應）
- 33. `minPrunableToolChars`：`50000`（僅適用於自適應）
- 範例（aggressive，最小設定）：
- `hardClear`：`{ enabled: true, placeholder: "[Old tool result content cleared]" }`

34. 範例（激進、最小化）：

```json5
35. {
  agents: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

{
agents: {
defaults: {
contextPruning: {
mode: "adaptive",
keepLastAssistants: 3,
softTrimRatio: 0.3,
hardClearRatio: 0.5,
minPrunableToolChars: 50000,
softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
// 選用：限制僅對特定工具進行修剪（deny 優先；支援 "\*" 萬用字元）
tools: { deny: ["browser", "canvas"] },
},
},
},
}

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Optional: restrict pruning to specific tools (deny wins; supports "*" wildcards)
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

See [/concepts/session-pruning](/concepts/session-pruning) for behavior details.

#### `agents.defaults.compaction`（預留空間 + 記憶體清理）

37. `agents.defaults.compaction.mode` 用於選擇壓縮摘要策略。 Defaults to `default`; set `safeguard` to enable chunked summarization for very long histories. See [/concepts/compaction](/concepts/compaction).

21. `agents.defaults.compaction.reserveTokensFloor` 會為 Pi 壓縮強制設定最小的 `reserveTokens`
    值（預設：`20000`）。 22. 將其設為 `0` 以停用此下限。

當工作階段的 token 預估值跨過低於壓縮限制的
軟性門檻時觸發。 38. 當工作階段的權杖數量估計值跨越低於壓縮上限的軟性門檻時觸發。

`memoryFlush.enabled`：`true`

- 39. `memoryFlush.enabled`：`true`
- 27. `memoryFlush.softThresholdTokens`: `4000`
- 注意：當工作階段的工作區為唯讀時，會略過 memory flush
  （`agents.defaults.sandbox.workspaceAccess: "ro"` 或 `"none"`）。
- 40. 注意：當工作階段工作區為唯讀時，會略過記憶體清空
      （`agents.defaults.sandbox.workspaceAccess: "ro"` 或 `"none"`）。

30. 範例（調校）：

```json5
封鎖串流：
```

32. 封鎖串流：

- `agents.defaults.blockStreamingDefault`：`"on"`/`"off"`（預設關閉）。

- 41. 頻道覆寫：`*.blockStreaming`（以及各帳戶變體）以強制開啟或關閉區塊串流。
  42. 非 Telegram 頻道需要明確設定 `*.blockStreaming: true` 才能啟用區塊回覆。

- 35. `agents.defaults.blockStreamingBreak`: `"text_end"` 或 `"message_end"`（預設：text_end）。

- `agents.defaults.blockStreamingChunk`: soft chunking for streamed blocks. {
  agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  Example:

  ```json5
  38. {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: merge streamed blocks before sending.
  Signal／Slack／Discord／Google Chat 預設
  為 `minChars: 1500`，除非另行覆寫。 41. Signal/Slack/Discord/Google Chat 預設
  為 `minChars: 1500`，除非另行覆寫。
  42. 頻道覆寫：`channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  （以及每帳號變體）。

- `agents.defaults.humanDelay`: randomized pause between **block replies** after the first.
  Modes: `off` (default), `natural` (800–2500ms), `custom` (use `minMs`/`maxMs`).
  45. 每個代理的覆寫：`agents.list[].humanDelay`。
  Example:

  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } },
  }
  ```

  See [/concepts/streaming](/concepts/streaming) for behavior + chunking details.

Typing indicators:

- `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`. `session.typingMode`：每個工作階段覆寫此模式。
- `session.typingMode`：每個 session 的模式覆寫。
- `agents.defaults.typingIntervalSeconds`：輸入中訊號的刷新頻率（預設：6 秒）。
- `session.typingIntervalSeconds`：每個 session 的刷新間隔覆寫。
  行為細節請參閱 [/concepts/typing-indicators](/concepts/typing-indicators)。

`agents.defaults.model.primary` should be set as `provider/model` (e.g. `anthropic/claude-opus-4-6`).
別名來自 `agents.defaults.models.*.alias`（例如 `Opus`）。
若省略 provider，OpenClaw 目前會暫時假設為 `anthropic`，作為棄用過渡的回退機制。
`agents.defaults.heartbeat` 用於設定週期性的心跳執行：

`agents.defaults.heartbeat` configures periodic heartbeat runs:

- `every`：時間長度字串（`ms`、`s`、`m`、`h`）；預設單位為分鐘。 Default:
  `30m`. 設定為 `0m` 以停用。
- `model`：心跳執行時可選的模型覆寫（`provider/model`）。
- `includeReasoning`: when `true`, heartbeats will also deliver the separate `Reasoning:` message when available (same shape as `/reasoning on`). 預設值：`false`。
- `session`：可選的 session 鍵，用於控制心跳在哪個 session 中執行。 Default: `main`.
- `to`：可選的收件者覆寫（頻道特定 ID，例如 WhatsApp 的 E.164、Telegram 的 chat id）。
- 42. `target`：可選的傳送頻道（`last`、`whatsapp`、`telegram`、`discord`、`slack`、`msteams`、`signal`、`imessage`、`none`）。 預設值：`last`。
- `prompt`: optional override for the heartbeat body (default: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）。心跳訊息中的內嵌指令會照常套用（但請避免由心跳變更工作階段預設）。 Overrides are sent verbatim; include a `Read HEARTBEAT.md` line if you still want the file read.
- `ackMaxChars`：在 `HEARTBEAT_OK` 之後、實際傳遞前允許的最大字元數（預設：300）。

每代理程式 Heartbeat:

- Set `agents.list[].heartbeat` to enable or override heartbeat settings for a specific agent.
- 43. 若任何代理項目定義了 `heartbeat`，則 **只有那些代理** 會執行心跳；預設值
      會成為這些代理共用的基準。

Heartbeats run full agent turns. Shorter intervals burn more tokens; be mindful
of `every`, keep `HEARTBEAT.md` tiny, and/or choose a cheaper `model`.

44. `tools.exec` 用於設定背景執行的預設值：

- 2. `backgroundMs`：自動轉為背景前的時間（毫秒，預設 10000）
- `timeoutSec`: auto-kill after this runtime (seconds, default 1800)
- `cleanupMs`: how long to keep finished sessions in memory (ms, default 1800000)
- `notifyOnExit`：背景化的執行結束時，佇列一個系統事件並請求心跳（預設 true）。
- 6. `applyPatch.enabled`：啟用實驗性的 `apply_patch`（僅限 OpenAI/OpenAI Codex；預設 false）
- `applyPatch.allowModels`: optional allowlist of model ids (e.g. `gpt-5.2` or `openai/gpt-5.2`)
  Note: `applyPatch` is only under `tools.exec`.

`tools.web` 用於設定網頁搜尋與擷取工具：

- `tools.web.search.enabled`（預設：當金鑰存在時為 true）。
- `tools.web.search.apiKey` (recommended: set via `openclaw configure --section web`, or use `BRAVE_API_KEY` env var)
- `tools.web.search.maxResults`（1–10，預設 5）。
- `tools.web.search.timeoutSeconds`（預設 30）
- `tools.web.search.cacheTtlMinutes`（預設 15）
- `tools.web.fetch.enabled`（預設 true）。
- `tools.web.fetch.maxChars`（預設 50000）
- `tools.web.fetch.maxCharsCap`（預設 50000；會限制來自設定或工具呼叫的 maxChars）。
- `tools.web.fetch.timeoutSeconds`（預設 30）
- `tools.web.fetch.cacheTtlMinutes`（預設 15）
- `tools.web.fetch.userAgent`（選用覆寫）
- `tools.web.fetch.readability` (default true; disable to use basic HTML cleanup only)
- `tools.web.fetch.firecrawl.enabled`（當設定 API key 時預設為 true）。
- `tools.web.fetch.firecrawl.apiKey` (optional; defaults to `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl`（預設 [https://api.firecrawl.dev](https://api.firecrawl.dev)）。
- `tools.web.fetch.firecrawl.onlyMainContent` (default true)
- `tools.web.fetch.firecrawl.maxAgeMs` (optional)
- `tools.web.fetch.firecrawl.timeoutSeconds` (optional)

45. `tools.media` 用於設定入站媒體理解（影像／音訊／影片）：

- `tools.media.models`: shared model list (capability-tagged; used after per-cap lists).
- 46. `tools.media.concurrency`：最大同時能力執行數（預設 2）。
- `tools.media.image` / `tools.media.audio` / `tools.media.video`：
  - `enabled`: opt-out switch (default true when models are configured).
  - `prompt`：可選的提示覆寫（圖片／影片會自動附加 `maxChars` 提示）。
  - `maxChars`: max output characters (default 500 for image/video; unset for audio).
  - 25. `maxBytes`：要傳送的最大媒體大小（預設：圖片 10MB、音訊 20MB、影片 50MB）。
  - `timeoutSeconds`：請求逾時時間（預設：影像 60 秒、音訊 60 秒、影片 120 秒）。
  - `language`: optional audio hint.
  - 47. `attachments`：附件政策（`mode`、`maxAttachments`、`prefer`）。
  - `scope`：選用的門檻控制（先符合者生效），可使用 `match.channel`、`match.chatType` 或 `match.keyPrefix`。
  - 48. `models`：模型項目的有序清單；失敗或媒體尺寸過大時，會回退至下一個項目。
- 31. 每個 `models[]` 項目：
  - 49. 提供者項目（`type: "provider"` 或省略）：
    - 33. `provider`：API 提供者 ID（`openai`、`anthropic`、`google`/`gemini`、`groq` 等）。
    - 50. `model`：模型 ID 覆寫（影像必填；音訊提供者預設為 `gpt-4o-mini-transcribe`／`whisper-large-v3-turbo`，影片則為 `gemini-3-flash-preview`）。
    - `profile` / `preferredProfile`：驗證設定檔選擇。
  - CLI 入口（`type: "cli"`）：
    - `command`：要執行的可執行檔。
    - `args`：樣板化參數（支援 `{{MediaPath}}`、`{{Prompt}}`、`{{MaxChars}}` 等）。
  - `capabilities`：可選清單（`image`、`audio`、`video`），用於限制共享入口。 省略時的預設：`openai`/`anthropic`/`minimax` → 影像，`google` → 影像+音訊+影片，`groq` → 音訊。
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` can be overridden per entry.

If no models are configured (or `enabled: false`), understanding is skipped; the model still receives the original attachments.

供應商驗證遵循標準模型驗證順序（驗證設定檔、如 `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY` 等環境變數，或 `models.providers.*.apiKey`）。

Example:

```json5
{
  tools: {
    media: {
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

45. `agents.defaults.subagents` 設定子代理的預設值：

- `model`：產生的子代理所使用的預設模型（字串或 `{ primary, fallbacks }`）。 若省略，子代理會繼承呼叫者的模型，除非在代理或呼叫層級另行覆寫。
- 48. `maxConcurrent`：子代理同時執行的最大數量（預設 1）
- `archiveAfterMinutes`：在 N 分鐘後自動封存子代理工作階段（預設 60；設為 `0` 以停用）。
- 每個子代理的工具政策：`tools.subagents.tools.allow` / `tools.subagents.tools.deny`（deny 優先）。

`tools.profile` sets a **base tool allowlist** before `tools.allow`/`tools.deny`:

- `minimal`：僅 `session_status`
- `coding`：`group:fs`、`group:runtime`、`group:sessions`、`group:memory`、`image`
- `messaging`：`group:messaging`、`sessions_list`、`sessions_history`、`sessions_send`、`session_status`
- `full`：不限制（與未設定相同）

每個代理的覆寫：`agents.list[].tools.profile`。

範例（預設僅訊息傳遞，另允許 Slack + Discord 工具）：

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

範例（程式開發設定檔，但在所有地方拒絕 exec/process）：

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` lets you **further restrict** tools for specific providers (or a single `provider/model`).
Per-agent override: `agents.list[].tools.byProvider`.

順序：基礎設定檔 → 供應商設定檔 → 允許/拒絕策略。
供應商鍵可接受 `provider`（例如 `google-antigravity`）或 `provider/model`
（例如 `openai/gpt-5.2`）。

範例（保留全域程式開發設定檔，但為 Google Antigravity 使用最小工具集）：

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Example (provider/model-specific allowlist):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

`tools.allow` / `tools.deny` 設定全域工具允許/拒絕策略（deny 優先）。
Matching is case-insensitive and supports `*` wildcards (`"*"` means all tools).
This is applied even when the Docker sandbox is **off**.

Example (disable browser/canvas everywhere):

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

Tool groups (shorthands) work in **global** and **per-agent** tool policies:

- `group:runtime`：`exec`、`bash`、`process`
- `group:fs`：`read`、`write`、`edit`、`apply_patch`
- `group:sessions`：`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status`
- `group:memory`：`memory_search`、`memory_get`
- `group:web`：`web_search`、`web_fetch`
- `group:ui`：`browser`、`canvas`
- `group:automation`：`cron`、`gateway`
- `group:messaging`：`message`
- `group:nodes`：`nodes`
- `group:openclaw`：所有內建的 OpenClaw 工具（不包含提供者外掛）

`tools.elevated` controls elevated (host) exec access:

- `enabled`：允許提升模式（預設 true）。
- `allowFrom`: per-channel allowlists (empty = disabled)
  - `whatsapp`: E.164 numbers
  - `telegram`：聊天 id 或使用者名稱。
  - `discord`: user ids or usernames (falls back to `channels.discord.dm.allowFrom` if omitted)
  - `signal`：E.164 號碼。
  - `imessage`: handles/chat ids
  - `webchat`: session ids or usernames

Example:

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

Per-agent override (further restrict):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false },
        },
      },
    ],
  },
}
```

注意事項：

- `tools.elevated` is the global baseline. `agents.list[].tools.elevated` can only further restrict (both must allow).
- `/elevated on|off|ask|full` stores state per session key; inline directives apply to a single message.
- Elevated `exec` runs on the host and bypasses sandboxing.
- Tool policy still applies; if `exec` is denied, elevated cannot be used.

`agents.defaults.maxConcurrent` sets the maximum number of embedded agent runs that can
execute in parallel across sessions. Each session is still serialized (one run
per session key at a time). Default: 1.

### `agents.defaults.sandbox`

Optional **Docker sandboxing** for the embedded agent. Intended for non-main
sessions so they cannot access your host system.

Details: [Sandboxing](/gateway/sandboxing)

Defaults (if enabled):

- scope: `"agent"` (one container + workspace per agent)
- Debian bookworm-slim based image
- agent workspace access: `workspaceAccess: "none"` (default)
  - `"none"`: use a per-scope sandbox workspace under `~/.openclaw/sandboxes`
- `"ro"`: keep the sandbox workspace at `/workspace`, and mount the agent workspace read-only at `/agent` (disables `write`/`edit`/`apply_patch`)
  - `"rw"`: mount the agent workspace read/write at `/workspace`
- 自動修剪：閒置 > 24 小時 或 存在時間 > 7 天
- tool policy: allow only `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (deny wins)
  - configure via `tools.sandbox.tools`, override per-agent via `agents.list[].tools.sandbox.tools`
  - tool group shorthands supported in sandbox policy: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (see [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- optional sandboxed browser (Chromium + CDP, noVNC observer)
- hardening knobs: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Warning: `scope: "shared"` means a shared container and shared workspace. No
cross-session isolation. Use `scope: "session"` for per-session isolation.

Legacy: `perSession` is still supported (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` 會在容器建立後 **只執行一次**（在容器內透過 `sh -lc`）。
For package installs, ensure network egress, a writable root FS, and a root user.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
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
          // Per-agent override (multi-agent): agents.list[].sandbox.docker.*
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
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7, // 0 disables max-age pruning
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

使用以下指令建置預設的 sandbox 映像一次：

```bash
scripts/sandbox-setup.sh
```

Note: sandbox containers default to `network: "none"`; set `agents.defaults.sandbox.docker.network`
to `"bridge"` (or your custom network) if the agent needs outbound access.

注意：傳入的附件會被放置到目前工作區的 `media/inbound/*`。 當 `workspaceAccess: "rw"` 時，表示檔案會寫入代理的工作區。

Note: `docker.binds` mounts additional host directories; global and per-agent binds are merged.

使用以下方式建置可選的瀏覽器映像：

```bash
scripts/sandbox-browser-setup.sh
```

當 `agents.defaults.sandbox.browser.enabled=true` 時，瀏覽器工具會使用沙箱化的 Chromium 實例（CDP）。 12. 若啟用 noVNC（在 headless=false 時為預設），
noVNC URL 會被注入到系統提示中，讓代理可以引用。
This does not require `browser.enabled` in the main config; the sandbox control
URL is injected per session.

`agents.defaults.sandbox.browser.allowHostControl` (default: false) allows
sandboxed sessions to explicitly target the **host** browser control server
via the browser tool (`target: "host"`). 如果你想要嚴格的 sandbox 隔離，請保持此選項關閉。

Allowlists for remote control:

- `allowedControlUrls`：允許用於 `target: "custom"` 的精確控制 URL。
- `allowedControlHosts`: hostnames permitted (hostname only, no port).
- `allowedControlPorts`: ports permitted (defaults: http=80, https=443).
  預設值：所有允許清單皆未設定（無限制）。 `allowHostControl` 預設為 false。

### `models`（自訂提供者 + 基礎 URL）

OpenClaw 使用 **pi-coding-agent** 模型目錄。 你可以新增自訂提供者
（LiteLLM、本地 OpenAI 相容伺服器、Anthropic 代理等） by writing
`~/.openclaw/agents/<agentId>/agent/models.json` or by defining the same schema inside your
OpenClaw config under `models.providers`.
各提供者總覽 + 範例：[/concepts/model-providers](/concepts/model-providers)。

When `models.providers` is present, OpenClaw writes/merges a `models.json` into
`~/.openclaw/agents/<agentId>/agent/` on startup:

- default behavior: **merge** (keeps existing providers, overrides on name)
- set `models.mode: "replace"` to overwrite the file contents

透過 `agents.defaults.model.primary`（提供者/模型）選擇模型。

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {},
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
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

### OpenCode Zen（多模型代理）

OpenCode Zen is a multi-model gateway with per-model endpoints. OpenClaw 使用
來自 pi-ai 的內建 `opencode` 提供者；請從
[https://opencode.ai/auth](https://opencode.ai/auth) 設定 `OPENCODE_API_KEY`（或
`OPENCODE_ZEN_API_KEY`）。

注意事項：

- 模型參考使用 `opencode/<modelId>`（例如：`opencode/claude-opus-4-6`）。
- If you enable an allowlist via `agents.defaults.models`, add each model you plan to use.
- 捷徑：`openclaw onboard --auth-choice opencode-zen`。

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

### Z.AI（GLM-4.7）— 提供者別名支援

Z.AI 模型可透過內建的 `zai` 提供者使用。 在你的環境中設定 `ZAI_API_KEY`
，並以 provider/model 的方式引用模型。

捷徑：`openclaw onboard --auth-choice zai-api-key`。

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

注意事項：

- `z.ai/*` 與 `z-ai/*` 為可接受的別名，並會正規化為 `zai/*`。
- 如果缺少 `ZAI_API_KEY`，對 `zai/*` 的請求將在執行時因驗證錯誤而失敗。
- 範例錯誤：`No API key found for provider "zai".`
- Z.AI 的通用 API 端點為 `https://api.z.ai/api/paas/v4`。 GLM 的程式碼請求
  使用專用的 Coding 端點 `https://api.z.ai/api/coding/paas/v4`。
  內建的 `zai` 提供者使用 Coding 端點。 If you need the general
  endpoint, define a custom provider in `models.providers` with the base URL
  override (see the custom providers section above).
- 10. 在文件/設定中使用假的佔位符；切勿提交真實的 API 金鑰。

### Moonshot AI（Kimi）

11. 使用 Moonshot 的 OpenAI 相容端點：

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

注意事項：

- 在環境中設定 `MOONSHOT_API_KEY`，或使用 `openclaw onboard --auth-choice moonshot-api-key`。
- 模型參照：`moonshot/kimi-k2.5`。
- For the China endpoint, either:
  - 執行 `openclaw onboard --auth-choice moonshot-api-key-cn`（精靈會設定 `https://api.moonshot.cn/v1`），或
  - 17. 在 `models.providers.moonshot` 中手動設定 `baseUrl: "https://api.moonshot.cn/v1"`。

### Kimi Coding

18. 使用 Moonshot AI 的 Kimi Coding 端點（Anthropic 相容，內建提供者）：

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

注意事項：

- Set `KIMI_API_KEY` in the environment or use `openclaw onboard --auth-choice kimi-code-api-key`.
- 模型參照：`kimi-coding/k2p5`。

### 22. Synthetic（Anthropic 相容）

使用 Synthetic 的 Anthropic 相容端點：

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

注意事項：

- 設定 `SYNTHETIC_API_KEY`，或使用 `openclaw onboard --auth-choice synthetic-api-key`。
- 25. 模型參考：`synthetic/hf:MiniMaxAI/MiniMax-M2.1`。
- 26. Base URL 應省略 `/v1`，因為 Anthropic 用戶端會自動附加。

### 27. 本地模型（LM Studio）— 建議設定

請參閱 [/gateway/local-models](/gateway/local-models) 以取得目前的本地使用指南。 29. TL;DR：在高階硬體上透過 LM Studio Responses API 執行 MiniMax M2.1；保留託管模型合併以作為備援。

### MiniMax M2.1

30. 不使用 LM Studio 直接使用 MiniMax M2.1：

```json5
31. {
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" },
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
            // Pricing: update in models.json if you need exact cost tracking.
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

注意事項：

- 設定 `MINIMAX_API_KEY` 環境變數，或使用 `openclaw onboard --auth-choice minimax-api`。
- 可用模型：`MiniMax-M2.1`（預設）。
- 34. 若需要精確的成本追蹤，請在 `models.json` 中更新定價。

### 35. Cerebras（GLM 4.6 / 4.7）

Use Cerebras via their OpenAI-compatible endpoint:

```json5
37. {
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

注意事項：

- 38. Cerebras 請使用 `cerebras/zai-glm-4.7`；Z.AI 直連請使用 `zai/glm-4.7`。
- Set `CEREBRAS_API_KEY` in the environment or config.

注意事項：

- 40. 支援的 API：`openai-completions`、`openai-responses`、`anthropic-messages`、
      `google-generative-ai`
- Use `authHeader: true` + `headers` for custom auth needs.
- 若希望將 `models.json` 儲存在其他位置，可使用 `OPENCLAW_AGENT_DIR`（或 `PI_CODING_AGENT_DIR`）覆寫代理設定根目錄（預設：`~/.openclaw/agents/main/agent`）。

### `session`

Controls session scoping, reset policy, reset triggers, and where the session store is written.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    // Default is already per-agent under ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // You can override with {agentId} templating:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

欄位：

- `mainKey`：直接聊天的分桶鍵（預設：`"main"`）。 Useful when you want to “rename” the primary DM thread without changing `agentId`.
  - Sandbox note: `agents.defaults.sandbox.mode: "non-main"` uses this key to detect the main session. Any session key that does not match `mainKey` (groups/channels) is sandboxed.
- `dmScope`: how DM sessions are grouped (default: `"main"`).
  - `main`: all DMs share the main session for continuity.
  - `per-peer`：依據跨頻道的發送者 ID 隔離 DM。
  - `per-channel-peer`: isolate DMs per channel + sender (recommended for multi-user inboxes).
  - `per-account-channel-peer`：依帳號 + 頻道 + 發送者隔離 DM（建議用於多帳號收件匣）。
  - Secure DM mode (recommended): set `session.dmScope: "per-channel-peer"` when multiple people can DM the bot (shared inboxes, multi-person allowlists, or `dmPolicy: "open"`).
- `identityLinks`: map canonical ids to provider-prefixed peers so the same person shares a DM session across channels when using `per-peer`, `per-channel-peer`, or `per-account-channel-peer`.
  - Example: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
- `reset`：主要重設政策。 Defaults to daily resets at 4:00 AM local time on the gateway host.
  - `mode`: `daily` or `idle` (default: `daily` when `reset` is present).
  - `atHour`: local hour (0-23) for the daily reset boundary.
  - `idleMinutes`: sliding idle window in minutes. When daily + idle are both configured, whichever expires first wins.
- `resetByType`: per-session overrides for `direct`, `group`, and `thread`. Legacy `dm` key is accepted as an alias for `direct`.
  - 若只設定舊版的 `session.idleMinutes`，且未設定任何 `reset`／`resetByType`，OpenClaw 會為了向後相容而維持僅閒置模式。
- `heartbeatIdleMinutes`：用於心跳檢查的選用閒置覆寫（啟用時仍會套用每日重置）。
- `agentToAgent.maxPingPongTurns`：請求者／目標之間允許的最大來回回覆次數（0–5，預設 5）。
- `sendPolicy.default`：當沒有規則符合時的回退行為（`allow` 或 `deny`）。
- `sendPolicy.rules[]`：可依 `channel`、`chatType`（`direct|group|room`）或 `keyPrefix`（例如 `cron:`）進行比對。 First deny wins; otherwise allow.

### `skills`（技能設定）

Controls bundled allowlist, install preferences, extra skill folders, and per-skill
overrides. 適用於**內建**技能與 `~/.openclaw/skills`（若名稱衝突，工作區技能仍優先）。

欄位：

- `allowBundled`：僅適用於**內建** Skills 的可選允許清單。若設定，只有清單中的內建 Skills 符合資格（不影響受管／工作區 Skills）。 若設定此項，僅這些內建技能可用（不影響受管理／工作區技能）。
- `load.extraDirs`：要掃描的額外 Skills 目錄（最低優先順序）。
- `install.preferBrew`：可用時偏好使用 brew 安裝器（預設：true）。
- `install.nodeManager`：Node 安裝器偏好（`npm` | `pnpm` | `yarn`，預設：npm）。
- `entries.<skillKey>`:\`：各技能的設定覆寫。

逐一 Skill 欄位：

- `enabled`：將 `false` 設為關閉，以停用該 Skill，即使它是隨附／已安裝。
- `env`：為代理程式執行時注入的環境變數（僅在尚未設定時）。
- `apiKey`：為宣告主要環境變數的技能提供的選用便利設定（例如 `nano-banana-pro` → `GEMINI_API_KEY`）。

Example:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### `plugins`（擴充套件）

控制外掛探索、允許／拒絕，以及各外掛的設定。 Plugins are loaded
from `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, plus any
`plugins.load.paths` entries. **Config changes require a gateway restart.**
See [/plugin](/tools/plugin) for full usage.

欄位：

- `enabled`: master toggle for plugin loading (default: true).
- `allow`: optional allowlist of plugin ids; when set, only listed plugins load.
- `deny`：可選的外掛 ID 拒絕清單（deny 優先）。
- `load.paths`: extra plugin files or directories to load (absolute or `~`).
- `entries.<pluginId>`：每個外掛的覆寫。
  - `enabled`: set `false` to disable.
  - `config`: plugin-specific config object (validated by the plugin if provided).

Example:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio",
        },
      },
    },
  },
}
```

### `browser` (openclaw-managed browser)

OpenClaw can start a **dedicated, isolated** Chrome/Brave/Edge/Chromium instance for openclaw and expose a small loopback control service.
Profiles can point at a **remote** Chromium-based browser via `profiles.<name>.cdpUrl`. Remote
profiles are attach-only (start/stop/reset are disabled).

`browser.cdpUrl` remains for legacy single-profile configs and as the base
scheme/host for profiles that only set `cdpPort`.

預設值：

- enabled: `true`
- evaluateEnabled: `true` (set `false` to disable `act:evaluate` and `wait --fn`)
- control service: loopback only (port derived from `gateway.port`, default `18791`)
- CDP URL: `http://127.0.0.1:18792` (control service + 1, legacy single-profile)
- profile color: `#FF4500` (lobster-orange)
- Note: the control server is started by the running gateway (OpenClaw.app menubar, or `openclaw gateway`).
- Auto-detect order: default browser if Chromium-based; otherwise Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // set true when tunneling a remote CDP to localhost
  },
}
```

### `ui` (Appearance)

Optional accent color used by the native apps for UI chrome (e.g. Talk Mode bubble tint).

If unset, clients fall back to a muted light-blue.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Optional: Control UI assistant identity override.
    // If unset, the Control UI uses the active agent identity (config or IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, or image URL/data URI
    },
  },
}
```

### `gateway` (Gateway server mode + bind)

Use `gateway.mode` to explicitly declare whether this machine should run the Gateway.

Defaults:

- mode: **unset** (treated as “do not auto-start”)
- bind: `loopback`
- port: `18789` (single port for WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // token gates WS + Control UI access
    // tailscale: { mode: "off" | "serve" | "funnel" }
  },
}
```

Control UI base path:

- `gateway.controlUi.basePath` sets the URL prefix where the Control UI is served.
- Examples: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
- Default: root (`/`) (unchanged).
- `gateway.controlUi.root` sets the filesystem root for Control UI assets (default: `dist/control-ui`).
- `gateway.controlUi.allowInsecureAuth` allows token-only auth for the Control UI when
  device identity is omitted (typically over HTTP). Default: `false`. Prefer HTTPS
  (Tailscale Serve) or `127.0.0.1`.
- `gateway.controlUi.dangerouslyDisableDeviceAuth` disables device identity checks for the
  Control UI (token/password only). Default: `false`. Break-glass only.

Related docs:

- [Control UI](/web/control-ui)
- [Web overview](/web)
- [Tailscale](/gateway/tailscale)
- [Remote access](/gateway/remote)

受信任的代理：

- `gateway.trustedProxies`: list of reverse proxy IPs that terminate TLS in front of the Gateway.
- 2. 當連線來自這些 IP 之一時，OpenClaw 會使用 `x-forwarded-for`（或 `x-real-ip`）來判定用戶端 IP，以進行本地配對檢查與 HTTP 驗證／本地檢查。
- 3. 只列出你完全掌控的代理，並確保它們會**覆寫**傳入的 `x-forwarded-for`。

注意事項：

- `openclaw gateway` refuses to start unless `gateway.mode` is set to `local` (or you pass the override flag).
- 5. `gateway.port` 控制用於 WebSocket + HTTP（控制 UI、hooks、A2UI）的單一多工連接埠。
- OpenAI Chat Completions endpoint: **disabled by default**; enable with `gateway.http.endpoints.chatCompletions.enabled: true`.
- 7. 優先順序：`--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > 預設 `18789`。
- Gateway auth is required by default (token/password or Tailscale Serve identity). Non-loopback binds require a shared token/password.
- The onboarding wizard generates a gateway token by default (even on loopback).
- `gateway.remote.token` is **only** for remote CLI calls; it does not enable local gateway auth. `gateway.token` is ignored.

13. 驗證與 Tailscale：

- `gateway.auth.mode` sets the handshake requirements (`token` or `password`). 15. 未設定時，假定使用 token 驗證。
- 16. `gateway.auth.token` 儲存用於 token 驗證的共用 token（供同一台機器上的 CLI 使用）。
- When `gateway.auth.mode` is set, only that method is accepted (plus optional Tailscale headers).
- `gateway.auth.password` 可以在此設定，或透過 `OPENCLAW_GATEWAY_PASSWORD`（建議）。
- 19. `gateway.auth.allowTailscale` 允許 Tailscale Serve 身分標頭
      （`tailscale-user-login`）在請求經由 loopback 抵達，且包含 `x-forwarded-for`、`x-forwarded-proto` 與 `x-forwarded-host` 時滿足驗證需求。 OpenClaw
      verifies the identity by resolving the `x-forwarded-for` address via
      `tailscale whois` before accepting it. When `true`, Serve requests do not need
      a token/password; set `false` to require explicit credentials. Defaults to
      `true` when `tailscale.mode = "serve"` and auth mode is not `password`.
- `gateway.tailscale.mode: "serve"` uses Tailscale Serve (tailnet only, loopback bind).
- `gateway.tailscale.mode: "funnel"` exposes the dashboard publicly; requires auth.
- 25. `gateway.tailscale.resetOnExit` 在關閉時重設 Serve／Funnel 設定。

Remote client defaults (CLI):

- 27. `gateway.remote.url` 在 `gateway.mode = "remote"` 時，設定 CLI 呼叫的預設 Gateway WebSocket URL。
- 28. `gateway.remote.transport` 選擇 macOS 的遠端傳輸方式（預設 `ssh`，`direct` 用於 ws／wss）。 29. 當使用 `direct` 時，`gateway.remote.url` 必須是 `ws://` 或 `wss://`。 30. `ws://host` 預設使用連接埠 `18789`。
- 31. `gateway.remote.token` 提供遠端呼叫所需的 token（未設定則無驗證）。
- `gateway.remote.password` supplies the password for remote calls (leave unset for no auth).

33. macOS App 行為：

- OpenClaw.app watches `~/.openclaw/openclaw.json` and switches modes live when `gateway.mode` or `gateway.remote.url` changes.
- 5. 若未設定 `gateway.mode` 但已設定 `gateway.remote.url`，macOS 應用程式會將其視為遠端模式。
- When you change connection mode in the macOS app, it writes `gateway.mode` (and `gateway.remote.url` + `gateway.remote.transport` in remote mode) back to the config file.

```json5
7. {
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

38. 直接傳輸範例（macOS App）：

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### 40. `gateway.reload`（設定熱重載）

Gateway 會監看 `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`），並自動套用變更。

模式:

- `hybrid`（預設）：即時套用安全的變更；關鍵變更時重新啟動 Gateway。
- `hot`：只套用可熱更新的變更；當需要重新啟動時記錄日誌。
- `restart`：任何設定變更都會重新啟動 Gateway。
- `off`：停用熱重載。

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

#### 熱重載矩陣（檔案 + 影響）

監看的檔案：

- `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）

熱套用（不需完整重新啟動 Gateway）：

- `hooks`（Webhook 驗證／路徑／對應）+ `hooks.gmail`（Gmail 監看器重新啟動）
- `browser`（瀏覽器控制伺服器重新啟動）
- `cron`（cron 服務重新啟動 + 併發更新）
- `agents.defaults.heartbeat`（心跳執行器重新啟動）
- `web`（WhatsApp Web 頻道重新啟動）
- `telegram`、`discord`、`signal`、`imessage`（頻道重新啟動）
- `agent`、`models`、`routing`、`messages`、`session`、`whatsapp`、`logging`、`skills`、`ui`、`talk`、`identity`、`wizard`（動態讀取）

需要完整重新啟動 Gateway：

- `gateway`（連接埠／綁定／驗證／控制 UI／tailscale）
- `bridge`（舊版）
- `探索`
- `canvasHost`
- `plugins`
- 任何未知／不支援的設定路徑（為了安全預設為重新啟動）

### 多實例隔離

要在同一台主機上執行多個 Gateway（用於備援或救援機器人），請隔離每個實例的狀態與設定，並使用唯一的連接埠：

- `OPENCLAW_CONFIG_PATH`（每個實例的設定）
- `OPENCLAW_STATE_DIR`（工作階段／憑證）
- `agents.defaults.workspace`（記憶）
- `gateway.port`（每個實例唯一）

便利旗標（CLI）：

- `openclaw --dev …` → 使用 `~/.openclaw-dev` + 從基準 `19001` 起位移連接埠
- `openclaw --profile <name> …` → 使用 `~/.openclaw-<name>`（連接埠由設定／環境變數／旗標指定）

請參閱 [Gateway runbook](/gateway) 以了解推導出的連接埠對應（gateway/browser/canvas）。
請參閱 [Multiple gateways](/gateway/multiple-gateways) 以了解瀏覽器／CDP 連接埠隔離細節。

Example:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

### `hooks`（Gateway Webhook）

在 Gateway HTTP 伺服器上啟用一個簡單的 HTTP Webhook 端點。

預設值：

- enabled: `false`
- path: `/hooks`
- maxBodyBytes: `262144`（256 KB）

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
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

請求必須包含 Hook Token：

- `Authorization: Bearer <token>` **或**
- `x-openclaw-token: <token>`

端點：

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` }\`
- `POST /hooks/<name>` → 透過 `hooks.mappings` 解析

`/hooks/agent` 會一律將摘要發佈到主工作階段（並可透過 `wakeMode: "now"` 選擇性地立即觸發心跳）。

對應說明：

- `match.path` 會比對 `/hooks` 之後的子路徑（例如 `/hooks/gmail` → `gmail`）。
- `match.source` 會比對負載中的欄位（例如 `{ source: "gmail" }`），因此你可以使用通用的 `/hooks/ingest` 路徑。
- Templates like `{{messages[0].subject}}` read from the payload.
- `transform` 可指向回傳 hook 動作的 JS/TS 模組。
- `deliver: true` sends the final reply to a channel; `channel` defaults to `last` (falls back to WhatsApp).
- 如果沒有先前的傳送路由，請明確設定 `channel` + `to`（Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams 為必填）。
- `model` overrides the LLM for this hook run (`provider/model` or alias; must be allowed if `agents.defaults.models` is set).

Gmail 輔助設定（由 `openclaw webhooks gmail setup` / `run` 使用）：

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

      // 選用：為 Gmail hook 處理使用較便宜的模型
      // 在驗證/速率限制/逾時時，會先回退到 agents.defaults.model.fallbacks，然後是 primary
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // 選用：Gmail hooks 的預設 thinking 等級
      thinking: "off",
    },
  },
}
```

Model override for Gmail hooks:

- `hooks.gmail.model` specifies a model to use for Gmail hook processing (defaults to session primary).
- Accepts `provider/model` refs or aliases from `agents.defaults.models`.
- Falls back to `agents.defaults.model.fallbacks`, then `agents.defaults.model.primary`, on auth/rate-limit/timeouts.
- 如果設定了 `agents.defaults.models`，請將 hooks 模型加入允許清單。
- At startup, warns if the configured model is not in the model catalog or allowlist.
- `hooks.gmail.thinking` sets the default thinking level for Gmail hooks and is overridden by per-hook `thinking`.

Gateway auto-start:

- If `hooks.enabled=true` and `hooks.gmail.account` is set, the Gateway starts
  `gog gmail watch serve` on boot and auto-renews the watch.
- Set `OPENCLAW_SKIP_GMAIL_WATCHER=1` to disable the auto-start (for manual runs).
- Avoid running a separate `gog gmail watch serve` alongside the Gateway; it will
  fail with `listen tcp 127.0.0.1:8788: bind: address already in use`.

注意：當 `tailscale.mode` 開啟時，OpenClaw 會將 `serve.path` 預設為 `/`，
以便 Tailscale 能正確代理 `/gmail-pubsub`（它會移除設定的路徑前綴）。
If you need the backend to receive the prefixed path, set
`hooks.gmail.tailscale.target` to a full URL (and align `serve.path`).

### `canvasHost` (LAN/tailnet Canvas file server + live reload)

The Gateway serves a directory of HTML/CSS/JS over HTTP so iOS/Android nodes can simply `canvas.navigate` to it.

預設根目錄：`~/.openclaw/workspace/canvas`  
預設連接埠：`18793`（為避免與 openclaw 瀏覽器 CDP 連接埠 `18792` 衝突而選擇）  
伺服器會監聽 **gateway 綁定位址**（LAN 或 Tailnet），以便節點可連線。

The server:

- serves files under `canvasHost.root`
- injects a tiny live-reload client into served HTML
- watches the directory and broadcasts reloads over a WebSocket endpoint at `/__openclaw__/ws`
- auto-creates a starter `index.html` when the directory is empty (so you see something immediately)
- also serves A2UI at `/__openclaw__/a2ui/` and is advertised to nodes as `canvasHostUrl`
  (always used by nodes for Canvas/A2UI)

如果目錄很大或遇到 `EMFILE`，請停用即時重新載入（與檔案監看）：

- config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

`canvasHost.*` 的變更需要重新啟動 gateway（設定重新載入會觸發重啟）。

停用方式：

- config: `canvasHost: { enabled: false }`
- 環境變數：`OPENCLAW_SKIP_CANVAS_HOST=1`

### `bridge`（舊版 TCP bridge，已移除）

Current builds no longer include the TCP bridge listener; `bridge.*` config keys are ignored.
Nodes connect over the Gateway WebSocket. 此章節保留作為歷史參考。

Legacy behavior:

- Gateway 可為節點（iOS/Android）暴露一個簡單的 TCP bridge，通常在連接埠 `18790`。

預設值：

- enabled: `true`
- port: `18790`
- bind: `lan` (binds to `0.0.0.0`)

綁定模式：

- `lan`: `0.0.0.0` (reachable on any interface, including LAN/Wi‑Fi and Tailscale)
- `tailnet`: bind only to the machine’s Tailscale IP (recommended for Vienna ⇄ London)
- `loopback`: `127.0.0.1` (local only)
- `auto`: prefer tailnet IP if present, else `lan`

TLS:

- `bridge.tls.enabled`: enable TLS for bridge connections (TLS-only when enabled).
- `bridge.tls.autoGenerate`: generate a self-signed cert when no cert/key are present (default: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: PEM paths for the bridge certificate + private key.
- `bridge.tls.caPath`: optional PEM CA bundle (custom roots or future mTLS).

When TLS is enabled, the Gateway advertises `bridgeTls=1` and `bridgeTlsSha256` in discovery TXT
records so nodes can pin the certificate. Manual connections use trust-on-first-use if no
fingerprint is stored yet.
Auto-generated certs require `openssl` on PATH; if generation fails, the bridge will not start.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Uses ~/.openclaw/bridge/tls/bridge-{cert,key}.pem when omitted.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS broadcast mode)

控制 LAN mDNS 探索廣播（`_openclaw-gw._tcp`）。

- `minimal` (default): omit `cliPath` + `sshPort` from TXT records
- `full`: include `cliPath` + `sshPort` in TXT records
- `off`: disable mDNS broadcasts entirely
- Hostname: defaults to `openclaw` (advertises `openclaw.local`). Override with `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { mode: "minimal" } },
}
```

### `discovery.wideArea` (Wide-Area Bonjour / unicast DNS‑SD)

When enabled, the Gateway writes a unicast DNS-SD zone for `_openclaw-gw._tcp` under `~/.openclaw/dns/` using the configured discovery domain (example: `openclaw.internal.`).

若要讓 iOS/Android 跨網路探索（維也納 ⇄ 倫敦），請搭配：

- 在 Gateway 主機上提供你所選網域的 DNS 伺服器（建議使用 CoreDNS）
- Tailscale **分割 DNS**，讓用戶端透過 Gateway DNS 伺服器解析該網域

One-time setup helper (gateway host):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## Media model template variables

Template placeholders are expanded in `tools.media.*.models[].args` and `tools.media.models[].args` (and any future templated argument fields).

\| 變數               | 說明                                                                            |
\| ------------------ | ------------------------------------------------------------------------------- | -------- | ------- | ---------- | ----- | ------ | -------- | ------- | ------- | --- |
\| `{{Body}}`         | 完整的傳入訊息內容                                                               |
\| `{{RawBody}}`      | 原始傳入訊息內容（無歷史/寄件者包裝；最適合指令解析）                            |
\| `{{BodyStripped}}` | 移除群組提及的訊息內容（代理的最佳預設）                                        |
\| `{{From}}`         | 寄件者識別碼（WhatsApp 為 E.164；依通道可能不同）                               |
\| `{{To}}`           | 目的地識別碼                                                                     |
\| `{{MessageSid}}`   | 通道訊息 ID（若可用）                                                           |
\| `{{SessionId}}`    | 目前工作階段 UUID                                                               |
\| `{{IsNewSession}}` | 建立新工作階段時為 `"true"`                                                   |
\| `{{MediaUrl}}`     | 傳入媒體的擬 URL（若存在）                                                      |
\| `{{MediaPath}}`    | 本地媒體路徑（若已下載）                                                        |
\| `{{MediaType}}`    | 媒體類型（image/audio/document/…）                                             |
\| `{{Transcript}}`   | Audio transcript (when enabled)                                                 |
\| `{{Prompt}}`       | Resolved media prompt for CLI entries                                           |
\| `{{MaxChars}}`     | Resolved max output chars for CLI entries                                       |
\| `{{ChatType}}`     | `"direct"` or `"group"`                                                         |
\| `{{GroupSubject}}` | Group subject (best effort)                                                     |
\| `{{GroupMembers}}` | Group members preview (best effort)                                             |
\| `{{SenderName}}`   | Sender display name (best effort)                                               |
\| `{{SenderE164}}`   | Sender phone number (best effort)                                               |
\| `{{Provider}}`     | Provider hint (whatsapp                                                         | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## Cron（Gateway 排程器）

Cron is a Gateway-owned scheduler for wakeups and scheduled jobs. 功能概覽與 CLI 範例請參閱 [Cron jobs](/automation/cron-jobs)。

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_下一步：[Agent Runtime](/concepts/agent)_ 🦞

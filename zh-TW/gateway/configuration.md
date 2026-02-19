---
summary: "Configuration 概覽：常見任務、快速設定與完整參考文件連結"
read_when:
  - 首次設定 OpenClaw
  - 尋找常見的設定模式
  - 導覽至特定設定區段
title: "設定"
---

# Configuration 🔧

OpenClaw reads an optional **JSON5** config from `~/.openclaw/openclaw.json` (comments + trailing commas allowed).

If the file is missing, OpenClaw uses safe-ish defaults (embedded Pi agent + per-sender sessions + workspace `~/.openclaw/workspace`). You usually only need a config to:

- 連接各種管道並控制誰可以向機器人傳送訊息
- 設定模型、工具、沙箱機制或自動化（cron、hooks）
- 調整工作階段、媒體、網路或 UI

請參閱 [完整參考](/gateway/configuration-reference) 以查看所有可用欄位。

<Tip>
**New to configuration?** Check out the [Configuration Examples](/gateway/configuration-examples) guide for complete examples with detailed explanations!
</Tip>

## Minimal config (recommended starting point)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Configuration 🔧

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # 完整設定精靈
    openclaw configure     # 設定精靈
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    The Gateway exposes a JSON Schema representation of the config via `config.schema` for UI editors.
    The Control UI renders a form from this schema, with a **Raw JSON** editor as an escape hatch.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`） Gateway 會監看該檔案並自動套用變更（請參閱 [hot reload](#config-hot-reload)）。
  
</Tab>
</Tabs>

## Strict config validation

<Warning>
OpenClaw only accepts configurations that fully match the schema. Unknown keys, malformed types, or invalid values cause the Gateway to **refuse to start** for safety. 唯一允許的根層級例外是 `$schema`（字串），讓編輯器可附加 JSON Schema 中繼資料。
</Warning>

When validation fails:

- The Gateway does not boot.
- Only diagnostic commands are allowed (for example: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Run `openclaw doctor` to see the exact issues.
- Run `openclaw doctor --fix` (or `--yes`) to apply migrations/repairs.

## 常見任務

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    每個 channel 都在 `channels. 下擁有各自的設定區段。<provider>` 下。 請參閱各 channel 的專屬頁面以取得設定步驟：

    ```
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

  
</Accordion>

  <Accordion title="Choose and configure models">
    設定主要模型及可選的備援模型：

    ```
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

  
</Accordion>

  <Accordion title="Control who can message the bot">
    透過 `dmPolicy` 於各 channel 中控制 DM 存取：

```json5
{
  session: {
    dmScope: "per-channel-peer",  // 建議用於多使用者
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

- `dmScope`：`main`（共用） | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
- 有關範圍設定、身分連結與傳送政策，請參閱 [Session Management](/concepts/session)。
- 所有欄位請參閱 [完整參考](/gateway/configuration-reference#session)。

    ```
    預設為 `groupPolicy: "allowlist"`（除非被 `channels.defaults.groupPolicy` 覆寫）；若未設定任何允許清單，群組訊息會被封鎖。
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Group messages default to **require mention** (either metadata mention or regex patterns). `agents.defaults.subagents` 設定子代理的預設值：

    ```
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

  
</Accordion>

  <Accordion title="Configure sessions and resets">Thread session isolation:

    ```
    
        在隔離的 Docker 容器中執行 agent 工作階段：
    ```

  
</Accordion>

  <Accordion title="Enable sandboxing">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```


    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">- `every`：時間長度字串（`30m`、`2h`）。設為 `0m` 可停用。
- `target`：`last` | `whatsapp` | `telegram` | `discord` | `none`
- 完整指南請參閱 [Heartbeat](/gateway/heartbeat)。

    ```
    
        執行多個彼此隔離、擁有獨立工作區與工作階段的 agents：
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    功能概覽與 CLI 範例請參閱 [Cron jobs](/automation/cron-jobs)。
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">在 Gateway HTTP 伺服器上啟用一個簡單的 HTTP Webhook 端點。

    ```
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

  
</Accordion>

  <Accordion title="Configure multi-agent routing">  
</Accordion>


    ```
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

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Split your config into multiple files using the `$include` directive. This is useful for:

    ```
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

  
</Accordion>
</AccordionGroup>

## `gateway.reload`（設定熱重載）

重新載入模式

### **`hybrid`**（預設）

| Modes:                   | Merge behavior                                                          |
| ---------------------------------------- | ----------------------------------------------------------------------- |
| 即時套用安全的變更。                               | 對於關鍵變更會自動重新啟動。 **`hot`**                                                |
| 僅即時套用安全的變更。                              | 當需要重新啟動時會記錄警告——由你自行處理。 **`restart`**                                    |
| 任何設定變更（無論是否安全）都會重新啟動 Gateway。            | Restarts the Gateway on any config change, safe or not. |
| **Inline substitution:** | 停用檔案監看。 變更將在下次手動重新啟動時生效。                                                |

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

### 哪些可熱套用，哪些需要重新啟動

大多數欄位可在不中斷服務的情況下熱套用。 在 `hybrid` 模式下，需要重新啟動的變更會自動處理。

| 類別                                   | 欄位                                             | 需要重新啟動？                            |
| ------------------------------------ | ---------------------------------------------- | ---------------------------------- |
| \`channels.<channel> | `channels.*`, `web`（WhatsApp）— 所有內建與擴充通道       | 否                                  |
| 代理與模型                                | `agent`, `agents`, `models`, `routing`         | 否                                  |
| 自動化                                  | `hooks`, `cron`, `agent.heartbeat`             | 否                                  |
| messages                             | `messages.queue`                               | 否                                  |
| 工具與媒體                                | `tools`, `browser`, `skills`, `audio`, `talk`  | 否                                  |
| UI 與其他                               | `ui`, `logging`, `identity`, `bindings`        | 否                                  |
| Gateway 伺服器                          | `gateway.*`（port、bind、auth、tailscale、TLS、HTTP） | **Rules:**         |
| 基礎設施                                 | `discovery`, `canvasHost`, `plugins`           | **Mention types:** |

<Note>
`gateway.reload` 和 `gateway.remote` 為例外——變更它們**不會**觸發重新啟動。
</Note>

## Partial updates (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Use `config.apply` to validate + write the full config and restart the Gateway in one step.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">Use `config.patch` to merge a partial update into the existing config without clobbering
unrelated keys. It applies JSON merge patch semantics:

    ````
    - 物件會遞迴合併
    - `null` 會刪除鍵
    - 陣列會直接取代
    
    參數：
    
    - `raw`（string）— 僅包含要變更鍵的 JSON5
    - `baseHash`（必填）— 來自 `config.get` 的設定雜湊
    - `sessionKey`、`note`、`restartDelayMs` — 與 `config.apply` 相同
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    
    ````

  
</Accordion>
</AccordionGroup>

## 環境變數

OpenClaw reads env vars from the parent process (shell, launchd/systemd, CI, etc.).

- `.env` from the current working directory (if present)
- a global fallback `.env` from `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`)

Neither `.env` file overrides existing env vars. You can also provide inline env vars in config.

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

<Accordion title="Shell env import (optional)">Opt-in convenience: if enabled and none of the expected keys are set yet, OpenClaw runs your login shell and imports only the missing expected keys (never overrides).

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

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">You can reference environment variables directly in any config string value using
`${VAR_NAME}` syntax.

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

Defaults:

- Only uppercase env var names are matched: `[A-Z_][A-Z0-9_]*`
- Missing or empty env vars throw an error at config load
- Escape with `$${VAR}` to output a literal `${VAR}`
- Works with `$include` (included files also get substitution)
- 內嵌替換：`"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

See [/environment](/help/environment) for full precedence and sources.

## 完整參考

Config Includes (`$include`)

---

Example (provider/model-specific allowlist):

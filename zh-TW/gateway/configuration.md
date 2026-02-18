---
title: "設定"
---

# Configuration

OpenClaw 會從 `~/.openclaw/openclaw.json` 讀取一個可選的 <Tooltip tip="JSON5 支援註解與尾隨逗號">**JSON5**</Tooltip> 設定檔。

如果檔案不存在，OpenClaw 會使用安全的預設值。常見需要新增設定檔的原因：

- 連接各種通道並控制誰可以傳訊給機器人
- 設定模型、工具、沙箱或自動化（cron、hooks）
- 調整工作階段、媒體、網路或 UI 行為

請參閱 [完整參考](/gateway/configuration-reference) 以查看所有可用欄位。

<Tip>
**第一次設定？** 建議從 `openclaw onboard` 的互動式精靈開始，或查看 [Configuration Examples](/gateway/configuration-examples) 指南以取得可直接複製貼上的完整範例設定。
</Tip>

## Minimal config

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Editing config

<Tabs>
  <Tab title="互動式精靈">
    ```bash
    openclaw onboard       # 完整設定精靈
    openclaw configure     # 設定精靈
    ```
  
</Tab>
  <Tab title="CLI（單行指令）">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    開啟 [http://127.0.0.1:18789](http://127.0.0.1:18789) 並使用 **Config** 分頁。
    Control UI 會根據設定的 schema 產生表單，並提供 **Raw JSON** 編輯器作為進階選項。
  
</Tab>
  <Tab title="直接編輯">
    直接編輯 `~/.openclaw/openclaw.json`。Gateway 會監看檔案並自動套用變更（請參閱 [hot reload](#config-hot-reload)）。
  
</Tab>
</Tabs>

## Strict validation

<Warning>
OpenClaw 僅接受完全符合 schema 的設定。未知的鍵、型別錯誤或無效值都會導致 Gateway **拒絕啟動**。唯一允許的根層例外是 `$schema`（字串），方便編輯器附加 JSON Schema 中繼資料。
</Warning>

當驗證失敗時：

- Gateway 不會啟動
- 僅能使用診斷指令（`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`）
- 執行 `openclaw doctor` 查看具體問題
- 執行 `openclaw doctor --fix`（或 `--yes`）自動修復

## Common tasks

<AccordionGroup>
  <Accordion title="設定通道（WhatsApp、Telegram、Discord 等）">
    每個通道都有自己在 `channels.<provider>` 底下的設定區塊。請參閱各通道頁面以了解詳細設定步驟：

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    所有通道都共用相同的 DM 政策模式：

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```
  
</Accordion>

  <Accordion title="選擇並設定模型">
    設定主要模型與可選備援：

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` 定義模型目錄，並同時作為 `/model` 的允許清單。
    - 模型參考格式為 `provider/model`（例如 `anthropic/claude-opus-4-6`）。
    - 請參閱 [Models CLI](/concepts/models) 了解聊天中切換模型，及 [Model Failover](/concepts/model-failover) 了解備援與驗證輪替。
    - 自訂／自架提供者請參閱參考文件中的 [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls)。

  
</Accordion>

  <Accordion title="控制誰可以傳訊給機器人">
    每個通道透過 `dmPolicy` 控制私訊存取：

    - `"pairing"`（預設）：未知寄件者會收到一次性配對碼
    - `"allowlist"`：僅允許 `allowFrom`（或已配對儲存）中的寄件者
    - `"open"`：允許所有私訊（需要 `allowFrom: ["*"]`）
    - `"disabled"`：忽略所有私訊

    群組請使用 `groupPolicy` + `groupAllowFrom` 或各通道特定的允許清單。

    詳細請參閱 [完整參考](/gateway/configuration-reference#dm-and-group-access)。

  
</Accordion>

  <Accordion title="設定群組提及門控">
    群組訊息預設為 **需要提及**。可為每個 agent 設定：

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Metadata mentions**：平台原生 @ 提及
    - **Text patterns**：`mentionPatterns` 中的正則模式
    - 請參閱 [完整參考](/gateway/configuration-reference#group-chat-mention-gating)

  
</Accordion>

  <Accordion title="設定工作階段與重置">
    工作階段控制對話連續性與隔離：

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - 請參閱 [Session Management](/concepts/session)
    - 完整欄位請參閱 [完整參考](/gateway/configuration-reference#session)

  
</Accordion>

  <Accordion title="啟用沙箱">
    在隔離的 Docker 容器中執行 agent：

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",
            scope: "agent",
          },
        },
      },
    }
    ```

    先建置映像：`scripts/sandbox-setup.sh`

    請參閱 [Sandboxing](/gateway/sandboxing)。

  
</Accordion>

  <Accordion title="設定心跳（定期檢查）">
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

    - `every`: 時間字串（`30m`、`2h`），`0m` 代表停用
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - 請參閱 [Heartbeat](/gateway/heartbeat)

  
</Accordion>

  <Accordion title="設定 cron 任務">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    請參閱 [Cron jobs](/automation/cron-jobs)。

  
</Accordion>

  <Accordion title="設定 Webhooks（hooks）">
    啟用 Gateway 上的 HTTP webhook：

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    請參閱 [完整參考](/gateway/configuration-reference#hooks)。

  
</Accordion>

  <Accordion title="多代理路由設定">
    在同一 Gateway 中執行多個隔離的 agent：

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

    請參閱 [Multi-Agent](/concepts/multi-agent)。

  
</Accordion>

  <Accordion title="將設定拆分為多個檔案（$include）">
    使用 `$include` 組織大型設定：

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **單一檔案**：取代該物件
    - **檔案陣列**：依序深度合併（後者覆蓋前者）
    - **同層鍵**：在 include 之後合併（可覆蓋）
    - **巢狀 include**：最多 10 層
    - **相對路徑**：相對於包含它的檔案
    - **錯誤處理**：缺檔、解析錯誤、循環 include 都會明確報錯

  
</Accordion>
</AccordionGroup>

## Config hot reload

Gateway 會監看 `~/.openclaw/openclaw.json` 並自動套用變更——大多數設定不需要手動重新啟動。

### Reload modes

| Mode                   | Behavior                                                                                |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (default) | 即時套用安全變更；關鍵變更時自動重新啟動。                                             |
| **`hot`**              | 僅熱套用安全變更；需要重啟時會記錄警告。                                               |
| **`restart`**          | 任何設定變更都會重新啟動 Gateway。                                                     |
| **`off`**              | 停用檔案監看；需手動重新啟動才會生效。                                                 |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

## Environment variables

OpenClaw 會讀取父行程的環境變數，另外還會讀取：

- 目前工作目錄下的 `.env`（若存在）
- `~/.openclaw/.env`（全域備援）

兩者都不會覆蓋已存在的環境變數。也可在設定檔中內嵌：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="在設定值中使用環境變數替換">
  可在任何字串設定中使用 `${VAR_NAME}`：

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

規則：

- 僅匹配大寫名稱 `[A-Z_][A-Z0-9_]*`
- 缺失或空值會在載入時報錯
- 使用 `$${VAR}` 輸出字面值
- 適用於 `$include` 檔案
- 例如 `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

完整優先順序請參閱 [Environment](/help/environment)。

## Full reference

完整逐欄位說明請參閱 **[Configuration Reference](/gateway/configuration-reference)**。

---

_相關： [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_


---
title: "子代理程式"
---

# 子代理程式

40. 子代理可讓你在不阻塞主對話的情況下執行背景任務。 41. 當你產生一個子代理時，它會在自己獨立的工作階段中執行、完成工作，並在結束時將結果回報到聊天中。

42. **使用案例：**

- 43. 在主代理持續回答問題的同時研究一個主題
- 44. 並行執行多個長時間任務（網頁擷取、程式碼分析、檔案處理）
- 45. 在多代理架構中將任務委派給專門化的代理

## 快速開始

46. 使用子代理最簡單的方式是自然地向你的代理提出要求：

> 47. "產生一個子代理來研究最新的 Node.js 發行說明"

48. 代理會在幕後呼叫 `sessions_spawn` 工具。 49. 當子代理完成時，會將其發現回報到你的聊天中。

50. 你也可以明確指定選項：

> "Spawn a sub-agent to analyze the server logs from today. Use gpt-5.2 and set a 5-minute timeout."

## How It Works

<Steps>
  <Step title="Main agent spawns">
    The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
  </Step>
  <Step title="Sub-agent runs in the background">
    A new isolated session is created (`agent:<agentId>:subagent:<uuid>`) on the dedicated `subagent` queue lane.
  </Step>
  <Step title="Result is announced">
    When the sub-agent finishes, it announces its findings back to the requester chat. The main agent posts a natural-language summary.
  </Step>
  <Step title="Session is archived">
    The sub-agent session is auto-archived after 60 minutes (configurable). Transcripts are preserved.
  </Step>
</Steps>

<Tip>
Each sub-agent has its **own** context and token usage. Set a cheaper model for sub-agents to save costs — see [Setting a Default Model](#setting-a-default-model) below.
</Tip>

## 設定

Sub-agents work out of the box with no configuration. 預設值：

- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- Max concurrent: 8
- Auto-archive: after 60 minutes

### Setting a Default Model

Use a cheaper model for sub-agents to save on token costs:

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### Setting a Default Thinking Level

```json5
{
  agents: {
    defaults: {
      subagents: {
        thinking: "low",
      },
    },
  },
}
```

### Per-Agent Overrides

In a multi-agent setup, you can set sub-agent defaults per agent:

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### 併發

Control how many sub-agents can run at the same time:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

Sub-agents use a dedicated queue lane (`subagent`) separate from the main agent queue, so sub-agent runs don't block inbound replies.

### Auto-Archive

Sub-agent sessions are automatically archived after a configurable period:

```json5
{
  agents: {
    defaults: {
      subagents: {
        archiveAfterMinutes: 120, // default: 60
      },
    },
  },
}
```

<Note>
Archive renames the transcript to `*.deleted.<timestamp>` (same folder) — transcripts are preserved, not deleted. Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
</Note>

## The `sessions_spawn` Tool

This is the tool the agent calls to create sub-agents.

### 參數

| Parameter           | 類型                       | Default                               | Description                                                                                       |
| ------------------- | ------------------------ | ------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `task`              | string                   | _(required)_       | What the sub-agent should do                                                                      |
| `label`             | string                   | —                                     | Short label for identification                                                                    |
| `agentId`           | string                   | _(caller's agent)_ | Spawn under a different agent id (must be allowed)                             |
| `模型`                | string                   | _(optional)_       | Override the model for this sub-agent                                                             |
| `thinking`          | string                   | _(optional)_       | Override thinking level (`off`, `low`, `medium`, `high`, etc.) |
| `runTimeoutSeconds` | number                   | `0` (no limit)     | Abort the sub-agent after N seconds                                                               |
| `清理`                | `"delete"` \\| `"keep"` | `"keep"`                              | `"delete"` archives immediately after announce                                                    |

### Model Resolution Order

The sub-agent model is resolved in this order (first match wins):

1. Explicit `model` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.model`
3. Global default: `agents.defaults.subagents.model`
4. Target agent’s normal model resolution for that new session

Thinking level is resolved in this order:

1. Explicit `thinking` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.thinking`
3. Global default: `agents.defaults.subagents.thinking`
4. Otherwise no sub-agent-specific thinking override is applied

<Note>
Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
</Note>

### Cross-Agent Spawning

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

```json5
{
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // or ["*"] to allow any
        },
      },
    ],
  },
}
```

<Tip>
Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.
</Tip>

## Managing Sub-Agents (`/subagents`)

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

| 指令                                         | Description                                                       |
| ------------------------------------------ | ----------------------------------------------------------------- |
| `/subagents list`                          | List all sub-agent runs (active and completed) |
| `/subagents stop <id\\|#\\|all>`         | Stop a running sub-agent                                          |
| `/subagents log <id\\|#> [limit] [tools]` | View sub-agent transcript                                         |
| `/subagents info <id\\|#>`                | Show detailed run metadata                                        |
| `/subagents send <id\\|#> <message>`      | Send a message to a running sub-agent                             |

You can reference sub-agents by list index (`1`, `2`), run id prefix, full session key, or `last`.

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">
    ```
    /subagents list
    ```

    ````
    ```
    🧭 Subagents (current session)
    Active: 1 · Done: 2
    1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
    2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
    3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
    ```
    
    ```
    /subagents stop 3
    ```
    
    ```
    ⚙️ Stop requested for deploy staging.
    ```
    ````

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">
    ```
    /subagents info 1
    ```

    ````
    ```
    ℹ️ Subagent info
    Status: ✅
    Label: research logs
    Task: Research the latest server error logs and summarize findings
    Run: a1b2c3d4-...
    Session: agent:main:subagent:...
    Runtime: 2m31s
    Cleanup: keep
    Outcome: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">
    ```
    /subagents log 1 10
    ```

    ````
    Shows the last 10 messages from the sub-agent's transcript. Add `tools` to include tool call messages:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">
    ```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    Sends a message into the running sub-agent's session and waits up to 30 seconds for a reply.
    ```

  </Accordion>
</AccordionGroup>

## Announce (How Results Come Back)

When a sub-agent finishes, it goes through an **announce** step:

1. The sub-agent's final reply is captured
2. A summary message is sent to the main agent's session with the result, status, and stats
3. The main agent posts a natural-language summary to your chat

在可用時，公告回覆會保留執行緒／主題路由（Slack 執行緒、Telegram 主題、Matrix 執行緒）。

### Announce Stats

Each announce includes a stats line with:

- Runtime duration
- Token 使用量（輸入／輸出／總計）
- 1. 預估成本（當模型定價透過 `models.providers.*.models[].cost` 設定時）
- 2. 工作階段金鑰、工作階段 ID，以及逐字稿路徑

### 3. 公告狀態

4. 公告訊息包含一個從執行階段結果（而非模型輸出）推導出的狀態：

- 5. **成功完成**（`ok`）— 任務正常完成
- 6. **錯誤** — 任務失敗（錯誤細節在備註中）
- 7. **逾時** — 任務超過 `runTimeoutSeconds`
- 8. **未知** — 無法判定狀態

<Tip>
9. 若不需要對使用者公告，主代理的摘要步驟可以回傳 `NO_REPLY`，且不會張貼任何內容。
10. 這與 `ANNOUNCE_SKIP` 不同，後者用於代理對代理的公告流程（`sessions_send`）。
</Tip>

## 11. 工具政策

12. 預設情況下，子代理可使用 **除了一組被拒絕的工具之外的所有工具**，這些工具對背景任務而言不安全或不必要：

<AccordionGroup>
  <Accordion title="Default denied tools">13. 
    | 被拒絕的工具 | 原因 |
    |-------------|--------|
    | `sessions_list` | 工作階段管理 — 由主代理協調 |
    | `sessions_history` | 工作階段管理 — 由主代理協調 |
    | `sessions_send` | 工作階段管理 — 由主代理協調 |
    | `sessions_spawn` | 不允許巢狀擴散（子代理不能再產生子代理） |
    | `gateway` | 系統管理 — 子代理使用具危險性 |
    | `agents_list` | 系統管理 |
    | `whatsapp_login` | 互動式設定 — 非任務 |
    | `session_status` | 狀態／排程 — 由主代理協調 |
    | `cron` | 狀態／排程 — 由主代理協調 |
    | `memory_search` | 改為在 spawn 提示中傳遞相關資訊 |
    | `memory_get` | 改為在 spawn 提示中傳遞相關資訊 |</Accordion>
</AccordionGroup>

### 14. 自訂子代理工具

15. 你可以進一步限制子代理工具：

```json5
16. {
  tools: {
    subagents: {
      tools: {
        // deny 永遠優先於 allow
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

17. 若要將子代理限制為 **僅能** 使用特定工具：

```json5
18. {
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // 若設定 deny，仍然以 deny 為優先
      },
    },
  },
}
```

<Note>
19. 自訂的 deny 項目會 **加入到** 預設的拒絕清單中。 20. 若設定了 `allow`，則僅有這些工具可用（仍會套用預設的拒絕清單）。
</Note>

## 驗證

子代理的驗證是依據 **agent id** 進行，而非依據工作階段類型：

- 21. 驗證存放區會從目標代理的 `agentDir` 載入
- 22. 主代理的驗證設定檔會作為 **備援** 合併進來（發生衝突時以代理本身的設定為準）
- 23. 合併是累加式的 — 主代理的設定檔永遠可作為備援使用

<Note>24. 
目前尚未支援每個子代理完全隔離的驗證。</Note>

## 25. 情境與系統提示

26. 與主代理相比，子代理會收到較精簡的系統提示：

- 27. **包含：** Tooling、Workspace、Runtime 區段，以及 `AGENTS.md` 與 `TOOLS.md`
- 28. **不包含：** `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md`

29. 子代理也會收到一個以任務為導向的系統提示，指示其專注於被指派的任務、完成任務，且不要充當主代理。

## 30. 停止子代理

| 31. 方法                     | 32. 效果                              |
| ------------------------------------------------- | ---------------------------------------------------------- |
| 33. 在聊天中輸入 `/stop`         | 34. 中止主工作階段 **以及** 從其產生的所有進行中的子代理執行 |
| 35. `/subagents stop <id>` | 36. 停止特定子代理而不影響主工作階段                |
| 37. `runTimeoutSeconds`    | 38. 在指定時間後自動中止子代理執行                 |

<Note>
39. `runTimeoutSeconds` **不會** 自動封存工作階段。 40. 工作階段會保留，直到一般的封存計時器觸發。
</Note>

## 41. 完整設定範例

<Accordion title="Complete sub-agent configuration">42. 
```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4" },
      subagents: {
        model: "minimax/MiniMax-M2.1",
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
      },
      {
        id: "ops",
        name: "Ops Agent",
        subagents: {
          model: "anthropic/claude-sonnet-4",
          allowAgents: ["main"], // ops 可以在 "main" 之下產生子代理
        },
      },
    ],
  },
  tools: {
    subagents: {
      tools: {
        deny: ["browser"], // 子代理不能使用瀏覽器
      },
    },
  },
}
```</Accordion>

## 限制

<Warning>
43. - **盡力而為的公告：** 若 gateway 重新啟動，尚未完成的公告工作將會遺失。
44. - **不允許巢狀產生：** 子代理不能產生自己的子代理。
45. - **共享資源：** 子代理共用 gateway 行程；請使用 `maxConcurrent` 作為安全閥。
46. - **自動封存為盡力而為：** gateway 重新啟動時，尚未觸發的封存計時器將會遺失。
</Warning>

## 19. 另請參閱

- 47. [Session Tools](/concepts/session-tool) — 關於 `sessions_spawn` 與其他工作階段工具的詳細說明
- 48. [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — 每個代理的工具限制與沙箱機制
- 49. [Configuration](/gateway/configuration) — `agents.defaults.subagents` 參考
- 50. [Queue](/concepts/queue) — `subagent` 佇列的運作方式

---
title: "子代理程式"
---

# 子代理程式

子代理是在既有代理執行期間所產生的背景代理執行。它們在自己的工作階段（`agent:<agentId>:subagent:<uuid>`）中運行，完成後會將結果**公告**回請求者的聊天頻道。

## Slash command

使用 `/subagents` 來檢視或控制**目前工作階段**的子代理執行：

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` 會顯示執行中繼資料（狀態、時間戳記、工作階段 id、逐字稿路徑、cleanup）。

主要目標：

- 在不阻塞主執行的情況下，平行化「研究／長時間任務／慢速工具」工作。
- 預設讓子代理彼此隔離（工作階段分離 + 可選沙箱）。
- 讓工具介面難以被誤用：子代理**預設不會**取得工作階段工具。
- 支援可設定的巢狀深度，以實現 orchestrator 模式。

成本注意：每個子代理都有其**自己的**上下文與 token 使用量。對於繁重或重複性任務，請為子代理設定較便宜的模型，並將主代理維持在較高品質的模型上。  
你可以透過 `agents.defaults.subagents.model` 或每個代理的覆寫設定來配置。

## Tool

使用 `sessions_spawn`：

- 啟動子代理執行（`deliver: false`，全域 lane：`subagent`）
- 接著執行公告步驟，並將公告回覆張貼到請求者聊天頻道
- 預設模型：繼承呼叫者，除非你設定 `agents.defaults.subagents.model`（或每個代理的 `agents.list[].subagents.model`）；若明確提供 `sessions_spawn.model`，則以其為準。
- 預設 thinking：繼承呼叫者，除非你設定 `agents.defaults.subagents.thinking`（或每個代理的 `agents.list[].subagents.thinking`）；若明確提供 `sessions_spawn.thinking`，則以其為準。

工具參數：

- `task`（必填）
- `label?`（選填）
- `agentId?`（選填；若允許，可在其他 agent id 之下產生）
- `model?`（選填；覆寫子代理模型；若為無效值將被跳過，子代理會以預設模型執行，並在工具結果中顯示警告）
- `thinking?`（選填；覆寫子代理的 thinking 等級）
- `runTimeoutSeconds?`（預設 `0`；設定後，子代理執行會在 N 秒後中止）
- `cleanup?`（`delete|keep`，預設 `keep`）

允許清單：

- `agents.list[].subagents.allowAgents`：可透過 `agentId` 指定的 agent id 清單（`["*"]` 代表允許任何）。預設：僅限請求者代理。

探索：

- 使用 `agents_list` 來查看目前哪些 agent id 允許 `sessions_spawn`。

自動封存：

- 子代理工作階段會在 `agents.defaults.subagents.archiveAfterMinutes`（預設：60）後自動封存。
- 封存使用 `sessions.delete`，並將逐字稿重新命名為 `*.deleted.<timestamp>`（相同資料夾）。
- `cleanup: "delete"` 會在公告後立即封存（仍會透過重新命名保留逐字稿）。
- 自動封存為盡力而為；若 gateway 重新啟動，尚未觸發的計時器將會遺失。
- `runTimeoutSeconds` **不會** 自動封存；它只會停止執行。工作階段會保留，直到自動封存。
- 自動封存同樣適用於 depth-1 與 depth-2 工作階段。

## Nested Sub-Agents

預設情況下，子代理不能再產生自己的子代理（`maxSpawnDepth: 1`）。你可以將 `maxSpawnDepth` 設為 `2` 來啟用一層巢狀，允許 **orchestrator 模式**：主代理 → orchestrator 子代理 → worker 子子代理。

### How to enable

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // allow sub-agents to spawn children (default: 1)
        maxChildrenPerAgent: 5, // max active children per agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
      },
    },
  },
}
```

### Depth levels

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | 主代理                                       | Always                       |
| 1     | `agent:<id>:subagent:<uuid>`                 | 子代理（當允許 depth 2 時為 orchestrator）    | Only if `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | 子子代理（leaf worker）                      | Never                        |

### Announce chain

結果會沿著鏈向上回傳：

1. Depth-2 worker 完成 → 向其父層（depth-1 orchestrator）公告
2. Depth-1 orchestrator 接收公告、整合結果、完成 → 向主代理公告
3. 主代理接收公告並回覆給使用者

每一層只會看到其直屬子代理的公告。

### Tool policy by depth

- **Depth 1（orchestrator，當 `maxSpawnDepth >= 2`）**：取得 `sessions_spawn`、`subagents`、`sessions_list`、`sessions_history` 以管理其子代理。其他工作階段／系統工具仍被拒絕。
- **Depth 1（leaf，當 `maxSpawnDepth == 1`）**：沒有工作階段工具（目前預設行為）。
- **Depth 2（leaf worker）**：沒有工作階段工具 — 在 depth 2 永遠拒絕 `sessions_spawn`，不能再產生子代理。

### Per-agent spawn limit

每個代理工作階段（任何深度）同時間最多只能有 `maxChildrenPerAgent`（預設：5）個活躍子代理。這可防止單一 orchestrator 失控式擴散。

### Cascade stop

停止 depth-1 orchestrator 會自動停止其所有 depth-2 子代理：

- 在主聊天中輸入 `/stop` 會停止所有 depth-1 代理，並級聯停止其 depth-2 子代理。
- `/subagents kill <id>` 會停止特定子代理，並級聯停止其子代理。
- `/subagents kill all` 會停止請求者的所有子代理，並級聯停止。

## Authentication

子代理的驗證是依據 **agent id** 解析，而非依據工作階段類型：

- 子代理的工作階段金鑰為 `agent:<agentId>:subagent:<uuid>`。
- 驗證存放區會從該代理的 `agentDir` 載入。
- 主代理的驗證設定檔會作為**備援**合併進來；若發生衝突，以子代理所屬代理的設定為準。

注意：此合併為累加式，因此主代理的設定檔永遠可作為備援。目前尚未支援每個代理完全隔離的驗證。

## Announce

子代理透過公告步驟回報結果：

- 公告步驟在子代理工作階段內執行（不是在請求者工作階段）。
- 若子代理精確回覆 `ANNOUNCE_SKIP`，則不會張貼任何內容。
- 否則，公告回覆會透過後續的 `agent` 呼叫（`deliver=true`）張貼到請求者聊天頻道。
- 在可用時，公告回覆會保留執行緒／主題路由（Slack 執行緒、Telegram 主題、Matrix 執行緒）。
- 公告訊息會被標準化為穩定範本：
  - `Status:` 來自執行結果（`success`、`error`、`timeout` 或 `unknown`）。
  - `Result:` 公告步驟中的摘要內容（若缺少則為 `(not available)`）。
  - `Notes:` 錯誤細節與其他有用資訊。
- `Status` 並非從模型輸出推斷，而是來自執行階段結果訊號。

公告內容在結尾會包含統計資訊（即使包裝後仍會保留）：

- 執行時間（例如 `runtime 5m12s`）
- Token 使用量（input/output/total）
- 當模型定價透過 `models.providers.*.models[].cost` 設定時的預估成本
- `sessionKey`、`sessionId` 與逐字稿路徑（主代理可透過 `sessions_history` 取得歷史或直接檢視檔案）

## Tool Policy (sub-agent tools)

預設情況下，子代理可使用**所有工具，除了工作階段工具與系統工具**：

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

當 `maxSpawnDepth >= 2` 時，depth-1 的 orchestrator 子代理還會額外取得 `sessions_spawn`、`subagents`、`sessions_list` 與 `sessions_history` 以管理其子代理。

可透過設定覆寫：

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

子代理使用專用的 in-process 佇列 lane：

- Lane 名稱：`subagent`
- 併發數：`agents.defaults.subagents.maxConcurrent`（預設 `8`）

## Stopping

- 在請求者聊天中輸入 `/stop` 會中止請求者工作階段，並停止其所產生的所有活躍子代理執行（包含巢狀子代理）。
- `/subagents kill <id>` 會停止特定子代理，並級聯停止其子代理。

## Limitations

- 子代理公告為**盡力而為**。若 gateway 重新啟動，尚未完成的「回傳公告」工作將會遺失。
- 子代理仍與主代理共用同一個 gateway 行程資源；請將 `maxConcurrent` 視為安全閥。
- `sessions_spawn` 永遠為非阻塞：它會立即回傳 `{ status: "accepted", runId, childSessionKey }`。
- 子代理的上下文只會注入 `AGENTS.md` 與 `TOOLS.md`（不包含 `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 或 `BOOTSTRAP.md`）。
- 最大巢狀深度為 5（`maxSpawnDepth` 範圍：1–5）。多數使用情境建議使用 depth 2。
- `maxChildrenPerAgent` 會限制每個工作階段的活躍子代理數（預設：5，範圍：1–20）。
---
summary: "子代理程式：產生隔離的代理程式執行，並將結果回報給請求者聊天"
read_when:
  - 你需要透過代理程式進行背景／平行工作
  - 你正在變更 sessions_spawn 或 子代理程式 工具政策
title: "子代理程式"
---

# Per-Agent Overrides

子代理是從現有代理執行中啟動的背景代理執行個體。 它們在各自的工作階段中執行（`agent:<agentId>:subagent:<uuid>`），完成後會將結果**公告**回請求者的聊天通道。

## 指令

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` 會顯示執行中繼資料（狀態、時間戳記、session id、逐字稿路徑、清理資訊）。

主要目標：

- 在不阻塞主要執行流程的情況下，將「研究／長時間任務／緩慢工具」工作平行化。
- 預設保持子代理隔離（工作階段分離 + 可選沙箱化）。
- 維持工具介面難以被誤用：子代理預設**不會**取得 session 工具。
- 支援可設定的巢狀深度，以實現協調器（orchestrator）模式。

Each sub-agent has its **own** context and token usage. 對於繁重或重複性
任務，請為子代理設定較低成本的模型，並讓主要代理維持在較高品質的模型上。
In a multi-agent setup, you can set sub-agent defaults per agent:

## 工具

Explicit `model` parameter in the `sessions_spawn` call

- Global default: `agents.defaults.subagents.thinking`
- 接著執行公告步驟，並將公告回覆發佈到請求者的聊天通道
- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Per-agent config: `agents.list[].subagents.thinking`

工具參數：

- `task`
- `label`
- Spawn under a different agent id (must be allowed)
- Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- Abort the sub-agent after N seconds
- `"delete"` \\

Allowlist：

- Per-agent config: `agents.list[].subagents.model` 預設：僅限請求者代理。

探索（Discovery）：

- Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.

Auto-Archive

- Sub-agent sessions are automatically archived after a configurable period:
- Archive renames the transcript to `*.deleted.` （相同資料夾）。
- `"delete"` archives immediately after announce
- Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
- `runTimeoutSeconds` **不會** 自動封存工作階段。 40. 工作階段會保留直到自動封存。
- 自動封存同樣適用於深度 1 與深度 2 的工作階段。

## 子代理程式

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

### 如何啟用

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

### 深度層級

| 深度 | Session key 形狀                                                                                                                                                                                                                                                                                                                                                                                                               | 角色                                                    | 可以啟動子代理嗎？               |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | ----------------------- |
| 0  | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                    | _(caller's agent)_                 | 永遠                      |
| 1  | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                             | 子代理（當允許深度 2 時為 orchestrator）                          | 僅當 `maxSpawnDepth >= 2` |
| 2  | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // or ["\*"] to allow any&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Managing Sub-Agents (`/subagents`) | 永不                      |

### 公告鏈

結果會沿著鏈向上回傳：

1. 深度 2 工作代理完成 → 向其父層（深度 1 orchestrator）發出公告
2. 深度 1 orchestrator 接收公告，彙整結果並完成 → 向主代理發出公告
3. 主代理接收公告並將結果交付給使用者

每一層只能看到來自其直接子代理的公告。

### 工具政策

- **深度 1（orchestrator，當 `maxSpawnDepth >= 2` 時）**：取得 `sessions_spawn`、`subagents`、`sessions_list`、`sessions_history` 以便管理其子代理。 其他 session/system 工具仍然被拒絕。
- **深度 1（leaf，當 `maxSpawnDepth == 1` 時）**：無 session 工具（目前預設行為）。
- **深度 2（leaf worker）**：無 session 工具——在深度 2 時 `sessions_spawn` 一律被拒絕。 不可再啟動子代理。

### Cross-Agent Spawning

每個代理 session（任何深度）同一時間最多可擁有 `maxChildrenPerAgent`（預設：5）個啟用中的子代理。 這可防止單一 orchestrator 發生失控式扇出。

### 級聯停止

停止深度 1 orchestrator 會自動停止其所有深度 2 子代理：

- 在主聊天中執行 `/stop` 會停止所有深度 1 代理，並級聯停止其深度 2 子代理。
- 子代理也會收到一個以任務為導向的系統提示，指示其專注於被指派的任務、完成任務，且不要充當主代理。
- `/subagents kill all` 會停止請求者的所有子代理，並進行級聯。

## 驗證

Sub-agent auth is resolved by **agent id**, not by session type:

- 子代理的 session key 為 `agent:<agentId>:subagent:<uuid>`。
- auth store 會從該代理的 `agentDir` 載入。
- 主代理的 auth profiles 會合併作為**後備（fallback）**；若發生衝突，代理的 profiles 會覆蓋主 profiles。

注意：此合併為累加式，因此主 profiles 始終可作為後備使用。 目前尚未支援每個子代理完全隔離的驗證。
</Note>

## 公告狀態

子代理透過公告步驟回報：

- 公告步驟在子代理 session 內執行（而非請求者 session）。
- 若子代理回覆完全為 `ANNOUNCE_SKIP`，則不會發佈任何內容。
- When the sub-agent finishes, it announces its findings back to the requester chat.
- 在可用時，公告回覆會保留執行緒／主題路由（Slack 執行緒、Telegram 主題、Matrix 執行緒）。
- 公告訊息會標準化為穩定的範本：
  - `Status:` 依執行結果而定（`success`、`error`、`timeout` 或 `unknown`）。
  - `Result:` 公告步驟中的摘要內容（若缺失則為 `(not available)`）。
  - `Notes:` 錯誤細節及其他有用的背景資訊。
- `Status` 並非從模型輸出推斷；而是來自執行階段的結果訊號。

Each announce includes a stats line with:

- 執行時間（例如：`runtime 5m12s`）
- Token 使用量（輸入／輸出／總計）
- 預估成本（當模型定價透過 `models.providers.*.models[].cost` 設定時）
- `sessionKey`、`sessionId` 以及對話紀錄路徑（因此主代理可透過 `sessions_history` 取得歷史記錄，或在磁碟上檢視該檔案）

## 工具政策（子代理工具）

預設情況下，子代理可使用**除 session 工具與系統工具之外的所有工具**：

- Explicit `thinking` parameter in the `sessions_spawn` call
- `runTimeoutSeconds`
- `sessions_send`
- The `sessions_spawn` Tool

當 `maxSpawnDepth >= 2` 時，深度為 1 的協調型子代理會額外取得 `sessions_spawn`、`subagents`、`sessions_list` 與 `sessions_history`，以便管理其子代理。

可透過設定覆寫：

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

## 併發

子代理使用專用的程序內佇列通道：

- :subagent:
- Global default: `agents.defaults.subagents.model`

## 停止子代理

- 在請求者聊天中傳送 `/stop` 會中止請求者的 session，並停止由其啟動的所有作用中子代理執行，並級聯至巢狀子代理。
- `) on the dedicated `subagent\` queue lane.

## 限制

- 子代理公告為\*\*盡力而為（best-effort）\*\*機制。 若 gateway 重新啟動，尚未完成的「announce back」工作將會遺失。
- 子代理仍與同一個 gateway 行程共享資源；請將 `maxConcurrent` 視為安全閥。
- The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
- 子代理的內容上下文僅注入 `AGENTS.md` + `TOOLS.md`（不包含 `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 或 `BOOTSTRAP.md`）。
- 最大巢狀深度為 5（`maxSpawnDepth` 範圍：1–5）。 多數使用情境建議使用深度 2。
- `maxChildrenPerAgent` 限制每個 session 的作用中子代理數量（預設：5，範圍：1–20）。

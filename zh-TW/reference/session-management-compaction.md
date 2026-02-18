---
title: "工作階段管理深入解析"
---

# 工作階段管理與壓縮（深入解析）

本文件說明 OpenClaw 如何端到端管理工作階段：

- **工作階段路由**（傳入訊息如何對應到一個 `sessionKey`）
- **工作階段儲存庫**（`sessions.json`）及其追蹤內容
- **對話記錄持久化**（`*.jsonl`）及其結構
- **逐字稿衛生**（在執行前的提供者特定修正）
- **上下文限制**（上下文視窗 vs 追蹤的權杖）
- **壓縮機制**（手動 + 自動壓縮）以及預壓縮流程的掛鉤位置
- **無聲例行整理**（例如不應產生使用者可見輸出的記憶體寫入）

如果你想先看高層概覽，請從以下開始：

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## Source of truth: the Gateway

OpenClaw 的設計以單一 **Gateway process** 為核心，負責擁有工作階段狀態。

- UI（macOS app、Web Control UI、TUI）應向 Gateway 閘道器查詢工作階段清單與權杖計數。
- 在遠端模式下，工作階段檔案位於遠端主機；「檢查你本機 Mac 的檔案」不會反映 Gateway 閘道器實際使用的內容。

---

## 兩層持久化

OpenClaw 以兩層方式持久化工作階段：

1. **工作階段儲存庫（`sessions.json`）**
   - 鍵／值對映：`sessionKey -> SessionEntry`
   - 小型、可變更、可安全編輯（或刪除項目）
   - Tracks session metadata (current session id, last activity, toggles, token counters, etc.)

2. **逐字稿（`<sessionId>.jsonl`）**
   - 具有樹狀結構的附加寫入逐字稿（項目具有 `id` + `parentId`）
   - 儲存實際對話 + 工具呼叫 + 壓縮摘要
   - 用於重建未來回合的模型上下文

---

## 磁碟上的位置

在 Gateway 閘道器主機上，依代理程式區分：

- 儲存庫：`~/.openclaw/agents/<agentId>/sessions/sessions.json`
- 逐字稿：`~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Telegram 主題工作階段：`.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw 透過 `src/config/sessions.ts` 解析這些路徑。

---

## 工作階段金鑰（`sessionKey`）

`sessionKey` 用來識別你所在的「對話桶」（路由 + 隔離）。

常見模式：

- 主要／直接聊天（每個代理程式）：`agent:<agentId>:<mainKey>`（預設 `main`）
- 群組：`agent:<agentId>:<channel>:group:<id>`
- 房間／頻道（Discord／Slack）：`agent:<agentId>:<channel>:channel:<id>` 或 `...:room:<id>`
- Cron：`cron:<job.id>`
- Webhook：`hook:<uuid>`（除非被覆寫）

正式規則記載於 [/concepts/session](/concepts/session)。

---

## 工作階段 ID（`sessionId`）

每個 `sessionKey` 都指向一個目前的 `sessionId`（持續該對話的逐字稿檔案）。

Rules of thumb:

- **重設**（`/new`、`/reset`）會為該 `sessionKey` 建立新的 `sessionId`。
- **每日重設**（預設為 Gateway 閘道器主機的當地時間凌晨 4:00）會在重設邊界後的下一則訊息建立新的 `sessionId`。
- **閒置到期**（`session.reset.idleMinutes` 或舊版 `session.idleMinutes`）在超過閒置視窗後收到訊息時建立新的 `sessionId`。若同時設定每日 + 閒置，以先到期者為準。 When daily + idle are both configured, whichever expires first wins.

實作細節：判斷發生於 `src/auto-reply/reply/session.ts` 中的 `initSessionState()`。

---

## Session store schema (`sessions.json`)

儲存庫的值型別為 `src/config/sessions.ts` 中的 `SessionEntry`。

主要欄位（非完整）：

- `sessionId`：目前逐字稿 ID（除非設定 `sessionFile`，否則檔名由此衍生）
- `updatedAt`：最後活動時間戳
- `sessionFile`：可選的逐字稿路徑覆寫
- `chatType`：`direct | group | room`（協助 UI 與傳送政策）
- `provider`、`subject`、`room`、`space`、`displayName`：群組／頻道標示的中繼資料
- 切換開關：
  - `thinkingLevel`、`verboseLevel`、`reasoningLevel`、`elevatedLevel`
  - `sendPolicy`（每個工作階段的覆寫）
- 模型選擇：
  - `providerOverride`、`modelOverride`、`authProfileOverride`
- 權杖計數器（盡力而為／依提供者而異）：
  - `inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`
- `compactionCount`：此工作階段金鑰完成自動壓縮的次數
- `memoryFlushAt`：上一次壓縮前記憶體寫入的時間戳
- `memoryFlushCompactionCount`：上一次寫入執行時的壓縮計數

儲存區可以安全編輯，但 Gateway 才是權威：它可能在工作階段執行時重寫或重新補水項目。

---

## Transcript structure (`*.jsonl`)

逐字稿由 `@mariozechner/pi-coding-agent` 的 `SessionManager` 管理。

檔案為 JSONL：

- 第一行：工作階段標頭（`type: "session"`，包含 `id`、`cwd`、`timestamp`，以及可選的 `parentSession`）
- 接著：具有 `id` + `parentId`（樹狀）的工作階段項目

Notable entry types:

- `message`：使用者／助理／toolResult 訊息
- `custom_message`：由擴充注入、**會**進入模型上下文的訊息（可在 UI 中隱藏）
- `custom`：不會進入模型上下文的擴充狀態
- `compaction`：具有 `firstKeptEntryId` 與 `tokensBefore` 的持久化壓縮摘要
- `branch_summary`：在樹分支導覽時的持久化摘要

OpenClaw 有意 **不**「修補」逐字稿；Gateway 閘道器使用 `SessionManager` 來讀寫它們。

---

## 上下文視窗 vs 追蹤的權杖

有兩個不同概念需要注意：

1. **模型上下文視窗**：每個模型的硬性上限（模型可見的權杖）
2. **工作階段儲存庫計數器**：寫入 `sessions.json` 的滾動統計（用於 /status 與儀表板）

若你在調整限制：

- 上下文視窗來自模型目錄（並且可透過設定覆寫）。
- 儲存庫中的 `contextTokens` 是執行時估計／回報值；不要把它當成嚴格保證。

更多資訊請見 [/token-use](/reference/token-use)。

---

## 壓縮：它是什麼

Compaction summarizes older conversation into a persisted `compaction` entry in the transcript and keeps recent messages intact.

壓縮後，後續回合會看到：

- 壓縮摘要
- `firstKeptEntryId` 之後的訊息

壓縮是**持久性的**（不同於工作階段修剪）。 請參閱 [/concepts/session-pruning](/concepts/session-pruning)。

---

## When auto-compaction happens (Pi runtime)

In the embedded Pi agent, auto-compaction triggers in two cases:

1. **溢位復原**：模型回傳上下文溢位錯誤 → 壓縮 → 重試。
2. **閾值維護**：成功完成一個回合後，當：

`contextTokens > contextWindow - reserveTokens`

Where:

- `contextWindow` 是模型的上下文視窗
- `reserveTokens` 是為提示詞 + 下一次模型輸出保留的餘裕

這些是 Pi 執行階段的語意（OpenClaw 會消費事件，但是否壓縮由 Pi 決定）。

---

## 壓縮設定（`reserveTokens`、`keepRecentTokens`）

Pi 的壓縮設定位於 Pi 設定中：

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw 也會為內嵌執行強制一個安全下限：

- 若 `compaction.reserveTokens < reserveTokensFloor`，OpenClaw 會提高它。
- Default floor is `20000` tokens.
- 設定 `agents.defaults.compaction.reserveTokensFloor: 0` 可停用下限。
- 若原本已更高，OpenClaw 會維持不變。

Why: leave enough headroom for multi-turn “housekeeping” (like memory writes) before compaction becomes unavoidable.

實作：`src/agents/pi-settings.ts` 中的 `ensurePiCompactionReserveTokens()`
（由 `src/agents/pi-embedded-runner.ts` 呼叫）。

---

## 使用者可見的介面

You can observe compaction and session state via:

- `/status`（任何聊天工作階段中）
- `openclaw status`（CLI）
- `openclaw sessions` / `sessions --json`
- 詳細模式：`🧹 Auto-compaction complete` + 壓縮次數

---

## Silent housekeeping (`NO_REPLY`)

OpenClaw 支援用於背景工作的「無聲」回合，使用者不應看到中間輸出。

慣例：

- 助理以 `NO_REPLY` 開頭輸出，表示「不要將回覆傳遞給使用者」。
- OpenClaw strips/suppresses this in the delivery layer.

自 `2026.1.10` 起，當部分串流區塊以 `NO_REPLY` 開頭時，OpenClaw 也會抑制 **草稿／輸入中串流**，避免無聲操作在回合中途洩漏部分輸出。

---

## Pre-compaction “memory flush” (implemented)

目標：在自動壓縮發生之前，執行一次無聲的代理程式回合，將可持久化的
狀態寫入磁碟（例如代理程式工作區中的 `memory/YYYY-MM-DD.md`），以確保壓縮不會
抹除關鍵上下文。

OpenClaw 採用 **壓縮前閾值寫入** 的方式：

1. 監控工作階段上下文使用量。
2. 當超過「軟性閾值」（低於 Pi 的壓縮閾值）時，對代理程式執行一次無聲的
   「立即寫入記憶體」指令。
3. 使用 `NO_REPLY`，讓使用者看不到任何輸出。

設定（`agents.defaults.compaction.memoryFlush`）：

- `enabled`（預設：`true`）
- `softThresholdTokens`（預設：`4000`）
- `prompt`（寫入回合的使用者訊息）
- `systemPrompt`（附加於寫入回合的額外系統提示）

注意事項：

- 預設的提示／系統提示包含 `NO_REPLY` 提示以抑制傳遞。
- 清空操作在每個壓縮週期中只會執行一次（記錄於 `sessions.json`）。
- 僅對內嵌 Pi 工作階段執行（CLI 後端會略過）。
- The flush is skipped when the session workspace is read-only (`workspaceAccess: "ro"` or `"none"`).
- See [Memory](/concepts/memory) for the workspace file layout and write patterns.

Pi 也在擴充 API 中提供 `session_before_compact` 掛鉤，但 OpenClaw 的
寫入邏輯目前位於 Gateway 閘道器端。

---

## 疑難排解檢查清單

- 11. 工作階段金鑰錯誤？ 工作階段金鑰錯誤？先從 [/concepts/session](/concepts/session) 開始，並確認 `/status` 中的 `sessionKey`。
- 儲存區與逐字稿不一致？ Confirm the Gateway host and the store path from `openclaw status`.
- Compaction spam? 15. 檢查：
  - 模型上下文視窗（是否過小）
  - 壓縮設定（`reserveTokens` 對模型視窗而言過高，可能導致提早壓縮）
  - 工具結果膨脹：啟用／調整工作階段修剪
- Silent turns leaking? 確認回覆以 `NO_REPLY`（完全相同的權杖）開頭，且你正在使用包含串流抑制修復的版本。

---
title: "WhatsApp"
---

# WhatsApp（網頁頻道）

Status: WhatsApp Web via Baileys only. Gateway owns the session(s).

## 快速開始（新手）

1. 盡可能使用**獨立的電話號碼**（建議）。
2. 在 `~/.openclaw/openclaw.json` 中設定 WhatsApp。
3. 執行 `openclaw channels login` 掃描 QR Code（已連結裝置）。
4. 啟動 Gateway 閘道器.

最小設定：

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

## 目標

- 在單一 Gateway 閘道器 程序中支援多個 WhatsApp 帳號（multi-account）。
- 可預測的路由：回覆會回到 WhatsApp，不進行模型路由。
- 模型能看到足夠的上下文以理解引用回覆。

## 設定寫入

預設允許 WhatsApp 寫入由 `/config set|unset` 觸發的設定更新（需要 `commands.config: true`）。

停用方式：

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## Architecture (who owns what)

- **Gateway 閘道器**：擁有 Baileys socket 與收件匣迴圈。
- **CLI／macOS 應用程式**：與 Gateway 閘道器 通訊；不直接使用 Baileys。
- **主動監聽器**：外送訊息需要；否則送出會快速失敗。

## 取得電話號碼（兩種模式）

WhatsApp 需要真實的行動電話號碼進行驗證。VoIP 與虛擬號碼通常會被封鎖。以下是兩種支援在 WhatsApp 上執行 OpenClaw 的方式： VoIP and virtual numbers are usually blocked. There are two supported ways to run OpenClaw on WhatsApp:

### 專用號碼（建議）

Use a **separate phone number** for OpenClaw. Best UX, clean routing, no self-chat quirks. Ideal setup: **spare/old Android phone + eSIM**. Leave it on Wi‑Fi and power, and link it via QR.

**WhatsApp Business:** You can use WhatsApp Business on the same device with a different number. **WhatsApp Business：** 可在同一裝置上使用不同號碼的 WhatsApp Business。非常適合將個人 WhatsApp 與 OpenClaw 分開——安裝 WhatsApp Business，並在其中註冊 OpenClaw 的號碼。

**範例設定（專用號碼、單一使用者允許清單）：**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

**Pairing mode (optional):**
If you want pairing instead of allowlist, set `channels.whatsapp.dmPolicy` to `pairing`. **配對模式（選用）：**  
若你想使用配對而非允許清單，將 `channels.whatsapp.dmPolicy` 設為 `pairing`。未知寄件者會收到配對碼；使用下列指令核准：
`openclaw pairing approve whatsapp <code>`

### 個人號碼（備援）

Quick fallback: run OpenClaw on **your own number**. Message yourself (WhatsApp “Message yourself”) for testing so you don’t spam contacts. Expect to read verification codes on your main phone during setup and experiments. **Must enable self-chat mode.**
When the wizard asks for your personal WhatsApp number, enter the phone you will message from (the owner/sender), not the assistant number.

**範例設定（個人號碼、自我聊天）：**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

自我聊天回覆在設定 `[{identity.name}]` 時會成為預設（否則為 `[openclaw]`），
前提是 `messages.responsePrefix` 未設定。可明確設定以自訂或停用
該前綴（使用 `""` 以移除）。 Set it explicitly to customize or disable
the prefix (use `""` to remove it).

### Number sourcing tips

- **當地電信商的 eSIM**（最可靠）
  - 奧地利：[hot.at](https://www.hot.at)
  - 英國：[giffgaff](https://www.giffgaff.com) — 免費 SIM，無合約
- **預付 SIM** — 便宜，只需接收一次驗證 SMS

**避免：** TextNow、Google Voice、多數「免費 SMS」服務 — WhatsApp 對這些封鎖非常嚴格。

**Tip:** The number only needs to receive one verification SMS. After that, WhatsApp Web sessions persist via `creds.json`.

## 為什麼不用 Twilio？

- 早期的 OpenClaw 版本支援 Twilio 的 WhatsApp Business 整合。
- WhatsApp Business 號碼不適合個人助理。
- Meta 強制 24 小時回覆視窗；若過去 24 小時未回覆，商業號碼無法主動發送新訊息。
- 高頻或「聊天式」使用會觸發嚴格封鎖，因為商業帳號不適合發送大量個人助理訊息。
- 結果：傳遞不穩定且常被封鎖，因此已移除支援。

## Login + credentials

- 登入指令：`openclaw channels login`（透過已連結裝置的 QR）。
- 多帳號登入：`openclaw channels login --account <id>`（`<id>` = `accountId`）。
- 預設帳號（省略 `--account` 時）：若存在 `default` 則使用之，否則使用第一個已設定的帳號 id（排序後）。
- 憑證儲存在 `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`。
- 備份副本位於 `creds.json.bak`（損毀時復原）。
- 相容舊版：較舊的安裝會將 Baileys 檔案直接存於 `~/.openclaw/credentials/`。
- 登出：`openclaw channels logout`（或 `--account <id>`）會刪除 WhatsApp 驗證狀態（但保留共用的 `oauth.json`）。
- 已登出 socket ⇒ 錯誤會指示重新連結。

## Inbound flow (DM + group)

- WhatsApp 事件來自 `messages.upsert`（Baileys）。
- Inbox listeners are detached on shutdown to avoid accumulating event handlers in tests/restarts.
- 狀態／廣播聊天會被忽略。
- 私聊使用 E.164；群組使用 group JID。
- **私訊政策**：`channels.whatsapp.dmPolicy` 控制私聊存取（預設：`pairing`）。
  - 配對：未知寄件者會收到配對碼（透過 `openclaw pairing approve whatsapp <code>` 核准；代碼 1 小時後過期）。
  - 開放：需要 `channels.whatsapp.allowFrom` 包含 `"*"`。
  - 你已連結的 WhatsApp 號碼會被隱含信任，因此自我訊息會略過 `channels.whatsapp.dmPolicy` 與 `channels.whatsapp.allowFrom` 檢查。

### 個人號碼模式（備援）

若你在**個人 WhatsApp 號碼**上執行 OpenClaw，請啟用 `channels.whatsapp.selfChatMode`（見上方範例）。

行為：

- Outbound DMs never trigger pairing replies (prevents spamming contacts).
- 進站未知寄件者仍遵循 `channels.whatsapp.dmPolicy`。
- 自我聊天模式（allowFrom 包含你的號碼）會避免自動已讀回條，並忽略提及 JID。
- Read receipts sent for non-self-chat DMs.

## Read receipts

By default, the gateway marks inbound WhatsApp messages as read (blue ticks) once they are accepted.

全域停用：

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

依帳號停用：

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

注意事項：

- Self-chat mode always skips read receipts.

## WhatsApp FAQ：發送訊息 + 配對

**連結 WhatsApp 後，OpenClaw 會不會傳訊給隨機聯絡人？**  
不會。預設私訊政策是**配對**，因此未知寄件者只會收到配對碼，且其訊息**不會被處理**。OpenClaw 只會回覆它收到的聊天，或你明確觸發的送出（代理程式／CLI）。 Default DM policy is **pairing**, so unknown senders only get a pairing code and their message is **not processed**. OpenClaw only replies to chats it receives, or to sends you explicitly trigger (agent/CLI).

**WhatsApp 的配對如何運作？**  
配對是針對未知寄件者的私訊閘門：

- First DM from a new sender returns a short code (message is not processed).
- 使用：`openclaw pairing approve whatsapp <code>` 核准（清單使用 `openclaw pairing list whatsapp`）。
- Codes expire after 1 hour; pending requests are capped at 3 per channel.

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Replies still come from the **same WhatsApp account**, and direct chats collapse to each agent's main session, so use **one agent per person**. DM access control (`dmPolicy`/`allowFrom`) is global per WhatsApp account. See [Multi-Agent Routing](/concepts/multi-agent).

**Why do you ask for my phone number in the wizard?**  
The wizard uses it to set your **allowlist/owner** so your own DMs are permitted. It’s not used for auto-sending. If you run on your personal WhatsApp number, use that same number and enable `channels.whatsapp.selfChatMode`.

## 訊息正規化（模型所見內容）

- `Body` 是目前訊息本文與信封。

- 引用回覆的上下文**一定會附加**：

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- 同時設定回覆中繼資料：
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = 引用的本文或媒體佔位符
  - `ReplyToSender` = 已知時的 E.164

- 僅媒體的進站訊息使用佔位符：
  - `<media:image|video|audio|document|sticker>`

## 群組

- 群組對應至 `agent:<agentId>:whatsapp:group:<jid>` 工作階段。
- 群組政策：`channels.whatsapp.groupPolicy = open|disabled|allowlist`（預設 `allowlist`）。
- 啟用模式：
  - `mention`（預設）：需要 @提及或正則符合。
  - `always`：一律觸發。
- `/activation mention|always` 僅限擁有者，且必須作為單獨訊息送出。
- 擁有者 = `channels.whatsapp.allowFrom`（或未設定時為自身 E.164）。
- **歷史注入**（僅待處理）：
  - 最近的_未處理_訊息（預設 50）插入於：
    `[Chat messages since your last reply - for context]`（已在工作階段中的訊息不會重新注入）
  - 目前訊息位於：
    `[Current message - respond to this]`
  - 會附加寄件者後綴：`[from: Name (+E164)]`
- Group metadata cached 5 min (subject + participants).

## 回覆傳遞（串接）

- WhatsApp Web 會送出標準訊息（目前 Gateway 閘道器 不支援引用回覆串接）。
- Reply tags are ignored on this channel.

## 確認反應（收件即自動反應）

WhatsApp 可在收到訊息後立即自動送出表情符號反應，於機器人產生回覆之前，提供即時回饋給使用者，表示訊息已收到。 This provides instant feedback to users that their message was received.

**設定：**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**選項：**

- `emoji`（字串）：用於確認的表情符號（例如「👀」、「✅」、「📨」）。空白或省略 = 停用功能。 Empty or omitted = feature disabled.
- `direct`（布林，預設：`true`）：在私聊／DM 中送出反應。
- `group`（字串，預設：`"mentions"`）：群組聊天行為：
  - `"always"`：對所有群組訊息反應（即使沒有 @提及）
  - `"mentions"`: React only when bot is @mentioned
  - `"never"`：群組中永不反應

**Per-account override:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**行為說明：**

- 反應會在收到訊息後**立即**送出，早於輸入中指示或機器人回覆。
- 在啟用 `requireMention: false`（啟用：一律） 的群組中，`group: "mentions"` 會對所有訊息反應（不僅限 @提及）。
- Fire-and-forget：反應失敗會被記錄，但不會阻止機器人回覆。
- 群組反應會自動包含參與者 JID。
- WhatsApp 會忽略 `messages.ackReaction`；請改用 `channels.whatsapp.ackReaction`。

## 代理程式工具（反應）

- 工具：`whatsapp`，含 `react` 動作（`chatJid`、`messageId`、`emoji`，選用 `remove`）。
- 選用：`participant`（群組寄件者）、`fromMe`（對自己訊息反應）、`accountId`（多帳號）。
- 反應移除語意：請見 [/tools/reactions](/tools/reactions)。
- 工具門控：`channels.whatsapp.actions.reactions`（預設：啟用）。

## 限制

- 外送文字會分段至 `channels.whatsapp.textChunkLimit`（預設 4000）。
- 選用換行分段：設定 `channels.whatsapp.chunkMode="newline"`，在長度分段前依空白行（段落邊界）切分。
- 進站媒體儲存上限由 `channels.whatsapp.mediaMaxMb` 控制（預設 50 MB）。
- 外送媒體項目上限為 `agents.defaults.mediaMaxMb`（預設 5 MB）。

## 外送（文字 + 媒體）

- 使用啟用中的網頁監聽器；若 Gateway 閘道器 未執行則報錯。
- 文字分段：每則最多 4k（可透過 `channels.whatsapp.textChunkLimit` 設定，選用 `channels.whatsapp.chunkMode`）。
- 媒體：
  - 支援圖片／影片／音訊／文件。
  - 音訊以 PTT 送出；`audio/ogg` ⇒ `audio/ogg; codecs=opus`。
  - 只有第一個媒體項目可加上說明文字。
  - Media fetch supports HTTP(S) and local paths.
  - 動畫 GIF：WhatsApp 期望使用具 `gifPlayback: true` 的 MP4 以內嵌循環。
    - CLI：`openclaw message send --media <mp4> --gif-playback`
    - Gateway 閘道器：`send` 參數包含 `gifPlayback: true`

## 語音備忘（PTT 音訊）

WhatsApp 會將音訊以**語音備忘**（PTT 氣泡）送出。

- Best results: OGG/Opus. 最佳結果：OGG/Opus。OpenClaw 會將 `audio/ogg` 重寫為 `audio/ogg; codecs=opus`。
- `[[audio_as_voice]]` 在 WhatsApp 上會被忽略（音訊已以語音備忘形式送出）。

## 媒體限制與最佳化

- Default outbound cap: 5 MB (per media item).
- 覆寫：`agents.defaults.mediaMaxMb`。
- 圖片會自動最佳化為上限內的 JPEG（調整尺寸 + 品質掃描）。
- Oversize media => error; media reply falls back to text warning.

## 心跳

- **Gateway 閘道器 心跳**：記錄連線健康狀態（`web.heartbeatSeconds`，預設 60 秒）。
- **代理程式心跳**：可依代理程式設定（`agents.list[].heartbeat`），或全域
  透過 `agents.defaults.heartbeat`（當未設定每代理程式項目時的後備）。
  - 使用已設定的心跳提示（預設：`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）+ `HEARTBEAT_OK` 略過行為。
  - 傳遞預設送往最後使用的頻道（或設定的目標）。

## 重新連線行為

- 退避策略：`web.reconnect`：
  - `initialMs`、`maxMs`、`factor`、`jitter`、`maxAttempts`。
- 若達到 maxAttempts，網頁監控會停止（降級）。
- 已登出 ⇒ 停止並需要重新連結。

## 設定速查

- `channels.whatsapp.dmPolicy`（私訊政策：配對／允許清單／開放／停用）。
- `channels.whatsapp.selfChatMode`（同機設定；機器人使用你的個人 WhatsApp 號碼）。
- `channels.whatsapp.allowFrom` (DM allowlist). `channels.whatsapp.allowFrom`（私訊允許清單）。WhatsApp 使用 E.164 電話號碼（無使用者名稱）。
- `channels.whatsapp.mediaMaxMb`（進站媒體儲存上限）。
- `channels.whatsapp.ackReaction`（收件即自動反應：`{emoji, direct, group}`）。
- `channels.whatsapp.accounts.<accountId>.*`（每帳號設定 + 選用 `authDir`）。
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb`（每帳號進站媒體上限）。
- `channels.whatsapp.accounts.<accountId>.ackReaction`（每帳號確認反應覆寫）。
- `channels.whatsapp.groupAllowFrom`（群組寄件者允許清單）。
- `channels.whatsapp.groupPolicy`（群組政策）。
- `channels.whatsapp.historyLimit`／`channels.whatsapp.accounts.<accountId>.historyLimit`（群組歷史上下文；`0` 停用）。
- `channels.whatsapp.dmHistoryLimit`（私訊歷史上限（以使用者輪次計））。每使用者覆寫：`channels.whatsapp.dms["<phone>"].historyLimit`。 私訊歷史可用 `channels.msteams.dmHistoryLimit`（使用者輪次）限制。每位使用者可用 `channels.whatsapp.dms["<phone>"].historyLimit` 覆寫。
- `channels.whatsapp.groups`（群組允許清單 + 提及門控預設；使用 `"*"` 以允許全部）
- `channels.whatsapp.actions.reactions`（WhatsApp 工具反應門控）。
- `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix`（進站前綴；每帳號：`channels.whatsapp.accounts.<accountId>.messagePrefix`；已棄用：`messages.messagePrefix`）
- `messages.responsePrefix`（外送前綴）
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model`（選用覆寫）
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*`（每代理程式覆寫）
- `session.*`（scope、idle、store、mainKey）
- `web.enabled`（為 false 時停用頻道啟動）
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Logs + troubleshooting

- 子系統：`whatsapp/inbound`、`whatsapp/outbound`、`web-heartbeat`、`web-reconnect`。
- 記錄檔：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`（可設定）。
- 疑難排解指南：[Gateway troubleshooting](/gateway/troubleshooting)。

## 疑難排解（快速）

**未連結／需要 QR 登入**

- 症狀：`channels status` 顯示 `linked: false` 或警告「Not linked」。
- 修復：在 閘道器主機 上執行 `openclaw channels login` 並掃描 QR（WhatsApp → 設定 → 已連結裝置）。

**已連結但已中斷／重新連線迴圈**

- 症狀：`channels status` 顯示 `running, disconnected` 或警告「Linked but disconnected」。
- Fix: `openclaw doctor` (or restart the gateway). If it persists, relink via `channels login` and inspect `openclaw logs --follow`.

**Bun 執行環境**

- Bun is **not recommended**. **不建議** 使用 Bun。WhatsApp（Baileys）與 Telegram 在 Bun 上不穩定。
  請以 **Node** 執行 Gateway 閘道器。（請見 入門指南 的執行環境說明。）
  Run the gateway with **Node**. (See Getting Started runtime note.)


---
summary: "Telegram bot support status, capabilities, and configuration"
read_when:
  - 進行 Telegram 功能或 webhook 相關工作時
title: "Telegram"
---

# Telegram（Bot API）

Status: production-ready for bot DMs + groups via grammY. Long-polling by default; webhook optional.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Pairing is the default token exchange used for Telegram DMs.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    跨通道診斷與修復操作手冊。
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    完整的通道設定模式與範例。
  
</Card>
</CardGroup>

## 快速設定（新手）

<Steps>
  <Step title="Create the bot token in BotFather">開啟 Telegram 並與 **@BotFather** 對話（[直接連結](https://t.me/BotFather)）。確認帳號名稱完全為 `@BotFather`。 Confirm the handle is exactly `@BotFather`.

    ```
    執行 `/newbot`，依照提示操作並儲存權杖。
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Env option: `TELEGRAM_BOT_TOKEN=...` (works for the default account).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    配對碼會在 1 小時後過期。
    ```

  
</Step>

  <Step title="Add the bot to a group">
    將機器人加入您的群組，然後設定 `channels.telegram.groups` 與 `groupPolicy` 以符合您的存取模型。
  
</Step>
</Steps>

<Note>
權杖解析順序具帳號感知能力。 在實務上，設定檔中的值會優先於環境變數的備援值，而 `TELEGRAM_BOT_TOKEN` 僅適用於預設帳號。
</Note>

## Telegram 端設定

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Telegram 機器人預設啟用 **隱私模式**，限制其可接收的群組訊息。
若機器人需要看到「所有」群組訊息，有兩種方式：
If your bot must see _all_ group messages, you have two options:

    ```
    管理員狀態在群組內（Telegram UI）設定。管理員機器人一律會接收所有群組訊息；若需要完整可見性，請使用管理員。 Admin bots always receive all
    group messages, so use admin if you need full visibility.
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    管理員身分需在 Telegram 群組設定中控制。
  

    ```
    Add the bot as a group **admin** (admin bots receive all messages).
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - `/setjoingroups` 允許／拒絕加入群組
    - `/setprivacy` 設定群組可見性行為
    ```

  
</Accordion>
</AccordionGroup>

## 存取控制與啟用

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` 控制私訊存取：

    ```
    - `pairing`（預設）
    - `allowlist`
    - `open`（需要在 `allowFrom` 中包含 `"*"`）
    - `disabled`
    
    `channels.telegram.allowFrom` 接受數字型 Telegram 使用者 ID。支援並會正規化 `telegram:` / `tg:` 前綴。
    引導精靈支援輸入 `@username`，並會將其解析為數字 ID。
    如果您升級後的設定中包含 `@username` 的 allowlist 項目，請執行 `openclaw doctor --fix` 進行解析（盡力而為；需要 Telegram bot token）。
    
    ### 查找您的 Telegram 使用者 ID
    
    較安全的方法（無需第三方機器人）：
    
    1. 私訊您的機器人。
    2. 執行 `openclaw logs --follow`。
    3. 讀取 `from.id`。
    
    官方 Bot API 方法：
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    私訊 `@userinfobot` 或 `@getidsbot`，使用回傳的使用者 ID。
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">兩個彼此獨立的控制：

    ```
    `channels.telegram.groupAllowFrom`：群組寄件者允許清單（ID／使用者名稱）。
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">
    預設情況下，群組回覆需要被提及。
  

    ```
    提及可以來自：
    
    - 原生 `@botusername` 提及，或
    - 以下位置中設定的提及模式：
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    工作階段層級的指令切換：
    
    - `/activation always`
    - `/activation mention`
    
    這些僅會更新工作階段狀態。若需持久化，請使用設定檔。
    
    持久化設定範例：
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }, // all groups, always respond
      },
    },
  },
}
```

    ```
    將群組中的任一訊息轉傳給 Telegram 上的 `@userinfobot` 或 `@getidsbot`，即可看到聊天 ID（負數，例如 `-1001234567890`）。
    ```

  
</Tab>
</Tabs>

## 執行階段行為

- Telegram 由 gateway 行程負責管理。
- 決定性路由：回覆會送回 Telegram；模型不會選擇頻道。
- Inbound messages are normalized into the shared channel envelope with reply context and media placeholders.
- 群組工作階段依群組 ID 隔離。 .topics.<threadId>
- 私人聊天在某些邊緣情況下也可能包含 `message_thread_id`。OpenClaw 會保持 DM 工作階段金鑰不變，但若存在，回覆／草稿串流仍會使用該 thread id。
- 長輪詢使用 grammY runner，並針對每個聊天／每個執行緒進行排序處理。 整體 runner sink 的並行度使用 `agents.defaults.maxConcurrent`。
- Telegram Bot API 不支援已讀回條；不存在 `sendReadReceipts` 選項。

## 功能參考

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw 可在 Telegram 私訊（DM）中使用 `sendMessageDraft` 串流部分回覆。

    ```
    Requirement:
    
    - `channels.telegram.streamMode` 不是 `"off"`（預設：`"partial"`）
    
    Modes:
    
    - `off`：不顯示即時預覽
    - `partial`：根據部分文字頻繁更新預覽
    - `block`：使用 `channels.telegram.draftChunk` 進行分塊預覽更新
    
    `draftChunk` 在 `streamMode: "block"` 時的預設值：
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` 會受 `channels.telegram.textChunkLimit` 限制。
    
    此功能適用於私訊與群組／主題。
    
    對於純文字回覆，OpenClaw 會保留同一則預覽訊息，並在原處進行最終編輯（不會發送第二則訊息）。
    
    對於較複雜的回覆（例如媒體內容），OpenClaw 會回退為一般的最終傳送方式，然後清除預覽訊息。
    
    `streamMode` 與區塊串流（block streaming）是分開的。當 Telegram 明確啟用區塊串流時，OpenClaw 會跳過預覽串流以避免雙重串流。
    
    僅限 Telegram 的推理串流：
    
    - `/reasoning stream` 會在產生內容時將推理過程傳送到即時預覽
    - 最終答案會在不包含推理文字的情況下送出
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">對外送出的 Telegram 文字使用 `parse_mode: "HTML"`（Telegram 支援的標籤子集）。

    ```
    來自模型的原始 HTML 會被跳脫，以避免 Telegram 解析錯誤。
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Some commands can be handled by plugins/skills without being registered in Telegram’s command menu.

    ```
    Native command defaults:
    
    - `commands.native: "auto"` 會為 Telegram 啟用原生命令
    
    新增自訂命令選單項目：
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Rules:
    
    - 名稱會正規化（移除前導 `/`，轉為小寫）
    - 有效格式：`a-z`、`0-9`、`_`，長度 `1..32`
    - 自訂命令不得覆寫原生命令
    - 衝突或重複的命令會被跳過並記錄日誌
    
    Notes:
    
    - 自訂命令僅為選單項目；不會自動實作行為
    - 即使未顯示在 Telegram 選單中，plugin/skill 命令在手動輸入時仍可運作
    
    若停用原生命令，內建命令會被移除。若已設定，自訂／plugin 命令仍可註冊。
    
    常見設定失敗：
    
    - `setMyCommands failed` 通常表示無法對 `api.telegram.org` 進行對外 DNS/HTTPS 連線。
    
    ### Device pairing commands (`device-pair` plugin)
    
    安裝 `device-pair` plugin 後：
    
    1. `/pair` 會產生設定代碼
    2. 在 iOS app 中貼上代碼
    3. `/pair approve` 會核准最新的待處理請求
    
    更多細節：[Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios)。
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    設定 inline 鍵盤範圍：

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    每帳號設定：
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    Default: `allowlist`. 預設：`allowlist`。
    舊版：`capabilities: ["inlineButtons"]` = `inlineButtons: "all"`。
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    當使用者點擊按鈕時，回呼資料會以以下格式作為訊息傳回代理程式：
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram 工具動作包含：

    ```
    工具：`telegram`，包含 `react` 動作（`chatId`、`messageId`、`emoji`）。
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram 支援透過標籤進行選擇性的回覆串接：

    ```
    `[[reply_to:<id>]]` —— 回覆指定的訊息 ID。
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">主題（論壇超級群組）

    ```
    **論壇群組：** 論壇群組中的反應會包含 `message_thread_id`，並使用如 `agent:main:telegram:group:{chatId}:topic:{threadId}` 的工作階段金鑰，確保同一主題內的反應與訊息保持一致。 This ensures reactions and messages in the same topic stay together.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">### Audio messages

    ```
    Telegram 區分語音訊息與音訊檔案。
    
    - 預設：音訊檔案行為
    - 在 agent 回覆中加入標籤 `[[audio_as_voice]]` 可強制以語音訊息方式傳送
    
    訊息動作範例：
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    Video messages (video vs video note)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    影片訊息（Video notes）不支援說明文字；提供的訊息文字會另外傳送。
    
    ### Stickers
    
    接收貼圖處理：
    
    - 靜態 WEBP：下載並處理（佔位符 `<media:sticker>`）
    - 動畫 TGS：略過
    - 影片 WEBM：略過
    
    貼圖上下文字段：
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    貼圖快取檔案：
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    貼圖會在可能時描述一次並快取，以減少重複的視覺呼叫。
    
    啟用貼圖動作：
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Sending stickers
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Sticker cache
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">從 Telegram API 接收 `message_reaction` 更新

    ```
    Telegram 論壇主題在每則訊息中包含 `message_thread_id`。OpenClaw： OpenClaw:
    ```

  
</Accordion>

  <Accordion title="Ack reactions">**反應的運作方式：**
Telegram 的反應會以 **獨立的 `message_reaction` 事件** 到達，而非訊息負載中的屬性。當使用者加入反應時，OpenClaw 會： When a user adds a reaction, OpenClaw:

    ```
    Resolution order:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agent 身分 emoji 備援（`agents.list[].identity.emoji`，否則為 "👀"）
    
    Notes:
    
    - Telegram 需要使用 unicode emoji（例如 "👀"）。
    - 使用 `""` 可在特定 channel 或 account 停用反應。
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    預設啟用 Channel 設定寫入（`configWrites !== false`）。

    ```
    預設允許 Telegram 寫入由頻道事件或 `/config set|unset` 觸發的設定更新。
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    預設：long polling。

    ```
    Webhook 模式：設定 `channels.telegram.webhookUrl` 與 `channels.telegram.webhookSecret`（可選 `channels.telegram.webhookPath`）。
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    對外文字會分塊至 `channels.telegram.textChunkLimit`（預設 4000）。
    可選的換行分塊：設定 `channels.telegram.chunkMode="newline"`，在長度分塊前先依空白行（段落邊界）分割。
    媒體下載／上傳上限為 `channels.telegram.mediaMaxMb`（預設 5）。
    - `channels.telegram.timeoutSeconds` 會覆寫 Telegram API client 的 timeout（若未設定，則使用 grammY 預設值）。
    群組歷史脈絡使用 `channels.telegram.historyLimit`（或 `channels.telegram.accounts.*.historyLimit`），後備為 `messages.groupChat.historyLimit`。設定 `0` 可停用（預設 50）。 Set `0` to disable (default 50).
    - DM 歷史記錄控制：
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>私訊（DM）歷史可用 `channels.telegram.dmHistoryLimit`（使用者回合）限制；每位使用者可用 `channels.telegram.dms["<user_id>"].historyLimit` 覆寫。 私訊歷史可用 `channels.msteams.dmHistoryLimit`（使用者輪次）限制。每位使用者可用 `channels.telegram.dms["<user_id>"].historyLimit` 覆寫。

    ```
    CLI 傳送目標可以是數字 chat ID 或 username：
    ```

```bash
範例：`openclaw message send --channel telegram --target 123456789 --message "hi"`。
```

  
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    若你設定了 `channels.telegram.groups.*.requireMention=false`，必須停用 Telegram Bot API 的 **隱私模式**。
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - 當存在 `channels.telegram.groups` 時，群組必須列出（或包含 `"*"`）
    - 驗證 bot 是否為該群組成員
    - 檢查日誌：`openclaw logs --follow` 以查看被跳過的原因
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    日誌中的 `setMyCommands failed` 通常表示對 `api.telegram.org` 的對外 HTTPS／DNS 被阻擋。
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + 自訂 fetch/proxy 在 AbortSignal 類型不相符時，可能會觸發立即中止行為。
    - 某些主機會優先將 `api.telegram.org` 解析為 IPv6；若 IPv6 對外連線異常，可能導致 Telegram API 間歇性失敗。
    - 驗證 DNS 回應：
    ```

```bash
快速檢查：`dig +short api.telegram.org A` 與 `dig +short api.telegram.org AAAA`，確認 DNS 回傳內容。
```

  
</Accordion>
</AccordionGroup>

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## Telegram 設定參考指引

主要參考：

- `channels.telegram.enabled`：啟用／停用頻道啟動。

- `channels.telegram.botToken`：機器人權杖（BotFather）。

- `channels.telegram.tokenFile`：從檔案路徑讀取權杖。

- `channels.telegram.dmPolicy`：`pairing | allowlist | open | disabled`（預設：配對）。

- `channels.telegram.allowFrom`：DM 允許清單（數字型 Telegram 使用者 ID）。 `open` 需要 `"*"`。 `openclaw doctor --fix` 可將舊版 `@username` 項目解析為 ID。

- `channels.telegram.groupPolicy`：`open | allowlist | disabled`（預設：允許清單）。

- `channels.telegram.allowFrom` 接受數字使用者 ID（建議）或 `@username` 項目。**不是** 機器人使用者名稱；請使用人類寄件者的 ID。精靈可接受 `@username`，並在可能時解析為數字 ID。 It is **not** the bot username; use the human sender’s ID. The wizard accepts `@username` and resolves it to the numeric ID when possible.

- 多數使用者想要：`groupPolicy: "allowlist"` ＋ `groupAllowFrom` ＋ 在 `channels.telegram.groups` 中列出特定群組
  - `channels.telegram.groups.<id>.groupPolicy`：群組層級覆寫 groupPolicy（`open | allowlist | disabled`）。
  - `channels.telegram.groups.<id>.requireMention`：提及閘控預設。
  - `channels.telegram.groups.<id>.skills`：skill 篩選（省略＝所有 skills，空白＝無）。
  - `channels.telegram.groups.<id>.allowFrom`：每群組寄件者允許清單覆寫。
  - `channels.telegram.groups.<id>.systemPrompt`：群組的額外系統提示。
  - `channels.telegram.groups.<id>.enabled`：當 `false` 時停用群組。
  - .topics.<threadId>`channels.telegram.groups.<id>.*`：每主題覆寫（欄位與群組相同）。
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`：每主題覆寫 groupPolicy（`open | allowlist | disabled`）。
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`：每主題提及閘控覆寫。

- `channels.telegram.capabilities.inlineButtons`：`off | dm | group | all | allowlist`（預設：允許清單）。

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`：每帳號覆寫。

- `channels.telegram.replyToMode`：`off | first | all`（預設：`first`）。

- `channels.telegram.textChunkLimit`：對外分塊大小（字元）。

- `channels.telegram.chunkMode`：`length`（預設）或 `newline`，在長度分塊前先依空白行（段落邊界）分割。

- `channels.telegram.linkPreview`：切換對外訊息的連結預覽（預設：true）。

- `channels.telegram.streamMode`：`off | partial | block`（草稿串流）。

- `channels.telegram.mediaMaxMb`：進站／對外媒體上限（MB）。

- `channels.telegram.retry`：對外 Telegram API 呼叫的重試策略（次數、minDelayMs、maxDelayMs、jitter）。

- `channels.telegram.network.autoSelectFamily`：覆寫 Node 的 autoSelectFamily（true＝啟用，false＝停用）。Node 22 預設停用以避免 Happy Eyeballs 逾時。 4. 在 Node 22 中預設為停用，以避免 Happy Eyeballs 逾時。

- `channels.telegram.proxy`：Bot API 呼叫的代理 URL（SOCKS／HTTP）。

- `channels.telegram.webhookUrl`：啟用 webhook 模式（需要 `channels.telegram.webhookSecret`）。

- `channels.telegram.webhookSecret`：webhook 密鑰（設定 webhookUrl 時必填）。

- `channels.telegram.webhookPath`：本地 webhook 路徑（預設 `/telegram-webhook`）。

- 本地監聽器會綁定至 `0.0.0.0:8787`，預設提供 `POST /telegram-webhook`。

- `channels.telegram.actions.reactions`：閘控 Telegram 工具反應。

- `channels.telegram.actions.sendMessage`：閘控 Telegram 工具訊息傳送。

- `channels.telegram.actions.deleteMessage`：閘控 Telegram 工具訊息刪除。

- `channels.telegram.actions.sticker`：閘控 Telegram 貼圖動作 — 傳送與搜尋（預設：false）。

- `channels.telegram.reactionNotifications`：`off | own | all` — 控制哪些反應會觸發系統事件（未設定時預設：`own`）。

- `channels.telegram.reactionLevel`：`off | ack | minimal | extensive` — 控制代理程式的反應能力（未設定時預設：`minimal`）。

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

Telegram 專屬高重要性欄位：

- startup/auth：`enabled`、`botToken`、`tokenFile`、`accounts.*`
- 存取控制：`dmPolicy`、`allowFrom`、`groupPolicy`、`groupAllowFrom`、`groups`、`groups.*.topics.*`
- 命令／選單：`commands.native`、`customCommands`
- 執行緒／回覆：`replyToMode`
- 可選（僅適用於 `streamMode: "block"`）：
- 格式／傳送：`textChunkLimit`、`chunkMode`、`linkPreview`、`responsePrefix`
- 媒體／網路：`mediaMaxMb`、`timeoutSeconds`、`retry`、`network.autoSelectFamily`、`proxy`
- webhook：`webhookUrl`、`webhookSecret`、`webhookPath`、`webhookHost`
- 動作／功能：`capabilities.inlineButtons`、`actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- 反應：`reactionNotifications`、`reactionLevel`
- 寫入／歷史：`configWrites`、`historyLimit`、`dmHistoryLimit`、`dms.*.historyLimit`

## 範圍：

- Details: [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- 更多協助：[頻道疑難排解](/channels/troubleshooting)。

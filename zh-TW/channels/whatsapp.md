---
summary: "WhatsApp 頻道支援、存取控制、傳送行為與營運作業"
read_when:
  - 處理 WhatsApp／網頁頻道行為或收件匣路由時
title: "WhatsApp"
---

# WhatsApp（網頁頻道）

Status: WhatsApp Web via Baileys only. Gateway owns the session(s).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    預設的 DM 政策為對未知發送者進行配對。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    跨頻道診斷與修復操作手冊。
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    完整的頻道設定模式與範例。
  
</Card>
</CardGroup>

## 設定速查

<Steps>
  <Step title="Configure WhatsApp access policy">

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

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
多帳號登入：`openclaw channels login --account <id>`（`<id>` = `accountId`）。
```

    ```
    針對特定帳戶：
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
使用：`openclaw pairing approve whatsapp <code>` 核准（清單使用 `openclaw pairing list whatsapp`）。
```

    ```
    Codes expire after 1 hour; pending requests are capped at 3 per channel.
    ```

  
</Step>
</Steps>

<Note>
若你在**個人 WhatsApp 號碼**上執行 OpenClaw，請啟用 `channels.whatsapp.selfChatMode`（見上方範例）。 
（該設定的頻道中繼資料與導覽流程已最佳化，但也支援個人號碼設定。）
</Note>

## 部署模式

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    這是最乾淨的營運模式：

    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    導覽流程支援個人號碼模式，並會寫入適用於自我對話的基準設定：

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    在目前的 OpenClaw 頻道架構中，訊息平台頻道是基於 WhatsApp Web（`Baileys`）。

    ```
    內建的聊天頻道註冊表中沒有獨立的 Twilio WhatsApp 訊息頻道。
    ```

  
</Accordion>
</AccordionGroup>

## 執行階段模型

- Gateway 負責管理 WhatsApp socket 與重新連線迴圈。
- 對外發送訊息需要目標帳戶有啟用中的 WhatsApp 監聽器。
- 狀態與廣播聊天會被忽略（`@status`、`@broadcast`）。
- 私聊使用 DM 工作階段規則（`session.dmScope`；預設 `main` 會將 DM 合併至代理的主工作階段）。
- 群組對應至 `agent:<agentId>:whatsapp:group:<jid>` 工作階段。

## 存取控制與啟用

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` 控制私聊存取：

    ```
    `channels.whatsapp.dmPolicy`（私訊政策：配對／允許清單／開放／停用）。
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    群組存取有兩層機制：

    ```
    群組政策：`channels.whatsapp.groupPolicy = open|disabled|allowlist`（預設 `allowlist`）。
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    預設情況下，群組回覆需要被提及（mention）。

    提及偵測包含：

- 明確提及機器人身分的 WhatsApp mention
- 已設定的 mention 正則表達式模式（`agents.list[].groupChat.mentionPatterns`，備援為 `messages.groupChat.mentionPatterns`）
- 隱式的回覆機器人偵測（回覆發送者與機器人身分相符）

工作階段層級的啟用指令：

- `/activation mention`
- `/activation always`

`activation` 會更新工作階段狀態（非全域設定）。此操作僅限擁有者執行。

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

## 當已連結的自身號碼同時存在於 `allowFrom` 中時，將啟用 WhatsApp 自我對話保護機制：

忽略原本可能會對自己觸發提醒的 mention-JID 自動觸發行為

- Read receipts sent for non-self-chat DMs.
- 訊息正規化與內容上下文
- 自我聊天回覆在設定 `[{identity.name}]` 時會成為預設（否則為 `[openclaw]`），
  前提是 `messages.responsePrefix` 未設定。可明確設定以自訂或停用
  該前綴（使用 `""` 以移除）。 Set it explicitly to customize or disable
  the prefix (use `""` to remove it).

## ```
傳入的 WhatsApp 訊息會包裝為共用的 inbound envelope。
```

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    若存在引用回覆，內容會以下列形式附加：

```text
[Replying to <sender> id:<stanzaId>]
<quoted body or media placeholder>
[/Replying]
```

    若可取得，也會填入回覆中繼資料欄位（`ReplyToId`、`ReplyToBody`、`ReplyToSender`、sender JID/E.164）。

    ```
    
        僅含媒體的傳入訊息會以如下佔位符正規化：
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
        位置與聯絡人 payload 會在路由前轉換為文字內容上下文。
    ```

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    對於群組，尚未處理的訊息可以被緩衝，並在機器人最終被觸發時作為上下文注入。

    ```
      
</Accordion>
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">傳送、分段與媒體

    ```
    最近的_未處理_訊息（預設 50）插入於：
    `[Chat messages since your last reply - for context]`（已在工作階段中的訊息不會重新注入）
    ```

  
</Accordion>

  <Accordion title="Read receipts">By default, the gateway marks inbound WhatsApp messages as read (blue ticks) once they are accepted.

    ```
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

  
</Accordion>
</AccordionGroup>

## Delivery, chunking, and media

<AccordionGroup>
  <Accordion title="Text chunking">選用換行分段：設定 `channels.whatsapp.chunkMode="newline"`，在長度分段前依空白行（段落邊界）切分。
</Accordion>

  <Accordion title="Outbound media behavior">
    - supports image, video, audio (PTT voice-note), and document payloads
    - `audio/ogg` is rewritten to `audio/ogg; codecs=opus` for voice-note compatibility
    - animated GIF playback is supported via `gifPlayback: true` on video sends
    - captions are applied to the first media item when sending multi-media reply payloads
    - media source can be HTTP(S), `file://`, or local paths
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - 入站媒體儲存上限：`channels.whatsapp.mediaMaxMb`（預設 `50`）
    - 自動回覆的出站媒體上限：`agents.defaults.mediaMaxMb`（預設 `5MB`）
    - 圖片會自動最佳化（調整尺寸／品質掃描）以符合限制
    - 當媒體傳送失敗時，會以首項備援改為傳送文字警告，而不會靜默捨棄回應
  
</Accordion>
</AccordionGroup>

## 確認回應（Acknowledgment）反應

`channels.whatsapp.ackReaction`（收件即自動反應：`{emoji, direct, group}`）。

```json5
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

注意事項：

- 在接受入站訊息後立即傳送（回覆前）
- 失敗會被記錄，但不會阻止正常回覆傳送
- 群組模式 `mentions` 會在提及觸發的回合中回應；群組啟用 `always` 則會繞過此檢查
- WhatsApp 會忽略 `messages.ackReaction`；請改用 `channels.whatsapp.ackReaction`。

## 多帳號與憑證

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*`（每帳號設定 + 選用 `authDir`）。
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">憑證儲存在 `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`。<accountId>/creds.json`
    - 備份檔案：`creds.json.bak`
    - 傳統預設驗證位於 `~/.openclaw/credentials/` 仍會在預設帳號流程中被識別／遷移
  
</Accordion>

  <Accordion title="Logout behavior">登出：`openclaw channels logout`（或 `--account <id>`）會刪除 WhatsApp 驗證狀態（但保留共用的 `oauth.json`）。<id>]` 會清除此帳號的 WhatsApp 驗證狀態。

    ```
    在舊版驗證目錄中，`oauth.json` 會被保留，而 Baileys 驗證檔案會被移除。
    ```

  
</Accordion>
</AccordionGroup>

## 工具、動作與設定寫入

- Agent 工具支援包含 WhatsApp reaction 動作（`react`）。
- 動作閘控：
  - `channels.whatsapp.actions.reactions`（WhatsApp 工具反應門控）。
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## 疑難排解

<AccordionGroup>
  <Accordion title="Not linked (QR required)">症狀：`channels status` 顯示 `linked: false` 或警告「Not linked」。

    ```
    修復：在 閘道器主機 上執行 `openclaw channels login` 並掃描 QR（WhatsApp → 設定 → 已連結裝置）。
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    症狀：已連結帳號反覆斷線或嘗試重新連線。

    ```
    Fix: `openclaw doctor` (or restart the gateway). If it persists, relink via `channels login` and inspect `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">
    當目標帳號沒有有效的 gateway 監聽器時，出站傳送會立即失敗。

    ```
    請確認 gateway 正在執行，且帳號已連結。
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    請依以下順序檢查：

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` 允許清單項目
    - 提及閘控（`requireMention` + 提及模式）
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway 執行環境應使用 Node。 **不建議** 使用 Bun。WhatsApp（Baileys）與 Telegram 在 Bun 上不穩定。
  請以 **Node** 執行 Gateway 閘道器。（請見 入門指南 的執行環境說明。）
  Run the gateway with **Node**.
  
</Accordion>
</AccordionGroup>

## 設定參考指引

主要參考：

- 預設允許 WhatsApp 寫入由 `/config set|unset` 觸發的設定更新（需要 `commands.config: true`）。

高重要度的 WhatsApp 欄位：

- 存取：`dmPolicy`、`allowFrom`、`groupPolicy`、`groupAllowFrom`、`groups`
- 傳送：`textChunkLimit`、`chunkMode`、`mediaMaxMb`、`sendReadReceipts`、`ackReaction`
- 多帳號：`accounts.<id> .enabled`、`accounts.<id> .authDir`、帳號層級覆寫
- 操作：`configWrites`、`debounceMs`、`web.enabled`、`web.heartbeatSeconds`、`web.reconnect.*`
- 工作階段行為：`session.dmScope`、`historyLimit`、`dmHistoryLimit`、`dms.<id>`messages.groupChat.historyLimit\`

## 相關

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)


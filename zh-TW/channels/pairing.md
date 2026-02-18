---
title: "配對"
---

# 配對

「配對」是 OpenClaw 明確的 **擁有者核准** 步驟。
它用於兩個地方：
It is used in two places:

1. **私訊配對**（誰被允許與機器人對話）
2. **節點配對**（哪些裝置／節點被允許加入 gateway 網路）

安全性背景： [Security](/gateway/security)

## 1. 私訊配對（入站聊天存取）

當頻道的私訊政策設定為 `pairing` 時，未知的傳送者會收到一組簡短代碼，且其訊息在你核准之前**不會被處理**。

預設的私訊政策記載於：[安全性](/gateway/security)

配對碼：

- 8 個字元，全大寫，無易混淆字元（`0O1I`）。
- **1 小時後過期**。機器人僅在建立新的配對請求時才會傳送配對訊息（大約每位傳送者每小時一次）。
- 待處理的私訊配對請求預設每個頻道最多 **3 筆**；在其中一筆過期或獲得核准之前，其他請求將會被忽略。

### 核准傳送者

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

支援的頻道： `telegram`、`whatsapp`、`signal`、`imessage`、`discord`、`slack`。

### Where the state lives

儲存在 `~/.openclaw/credentials/` 之下：

- 待處理請求： `<channel>-pairing.json`
- 已核准的允許清單儲存區： `<channel>-allowFrom.json`

Treat these as sensitive (they gate access to your assistant).

## 2. 節點裝置配對（iOS / Android / macOS / 無介面節點）

節點會以 **裝置** 的形式連線到 Gateway 閘道器，並使用 `role: node`。Gateway 閘道器
會建立一個裝置配對請求，必須先被核准。 The Gateway
creates a device pairing request that must be approved.

### Pair via Telegram (recommended for iOS)

If you use the `device-pair` plugin, you can do first-time device pairing entirely from Telegram:

1. In Telegram, message your bot: `/pair`
2. The bot replies with two messages: an instruction message and a separate **setup code** message (easy to copy/paste in Telegram).
3. On your phone, open the OpenClaw iOS app → Settings → Gateway.
4. Paste the setup code and connect.
5. Back in Telegram: `/pair approve`

The setup code is a base64-encoded JSON payload that contains:

- `url`: the Gateway WebSocket URL (`ws://...` or `wss://...`)
- `token`: a short-lived pairing token

Treat the setup code like a password while it is valid.

### Approve a node device

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Node pairing state storage

儲存在 `~/.openclaw/devices/` 之下：

- `pending.json`（短期存在；待處理請求會到期）
- `paired.json`（已配對的裝置與權杖）

### 注意事項

- 舊版的 `node.pair.*` API（CLI： `openclaw nodes pending/approve`）是
  一個獨立、由 Gateway 閘道器 擁有的配對儲存區。WS 節點仍然需要進行裝置配對。 1. WS 節點仍然需要裝置配對。

## 2. 相關文件

- 安全性模型與提示注入： [Security](/gateway/security)
- 安全更新（執行 doctor）： [Updating](/install/updating)
- 頻道設定：
  - Telegram： [Telegram](/channels/telegram)
  - WhatsApp： [WhatsApp](/channels/whatsapp)
  - Signal： [Signal](/channels/signal)
  - BlueBubbles（iMessage）： [BlueBubbles](/channels/bluebubbles)
  - iMessage（舊版）： [iMessage](/channels/imessage)
  - Discord： [Discord](/channels/discord)
  - Slack： [Slack](/channels/slack)



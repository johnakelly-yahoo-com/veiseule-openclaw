---
summary: "Slack 的 Socket 或 HTTP webhook 模式設定"
read_when:
  - 設定 Slack 或除錯 Slack Socket／HTTP 模式
title: "Slack"
---

# Slack

狀態：已可於正式環境使用，支援 DMs 與透過 Slack 應用程式整合的頻道。
HTTP 模式（Events API）

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DMs 預設為配對模式。
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    原生命令行為與命令目錄。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    跨頻道診斷與修復操作手冊。
  
</Card>
</CardGroup>

## 快速設定（新手）

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        在 Slack 應用程式設定中：

        ```
        建立 **App Token**（`xapp-...`）與 **Bot Token**（`xoxb-...`）。
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            環境變數備援（僅適用於預設帳號）：
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Subscribe app events">
          訂閱機器人事件：
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          另外為 DMs 啟用 App Home **Messages Tab**。
        
</Step>
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        當你的 Gateway 閘道器可透過 HTTPS 被 Slack 存取時，請使用 HTTP webhook 模式（典型的伺服器部署）。
        HTTP 模式使用 Events API＋Interactivity＋Slash Commands，並共用同一個請求 URL。
        HTTP mode uses the Events API + Interactivity + Slash Commands with a shared request URL.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      多帳號 HTTP 模式：設定 `channels.slack.accounts.<id> .mode = "http"`，並為每個帳號提供唯一的
      `webhookPath`，讓每個 Slack 應用程式都指向自己的 URL。

  
</Tab>
</Tabs>

## 權杖模型

- Socket Mode 需要 `botToken` + `appToken`。
- HTTP 模式需要 `botToken` + `signingSecret`。
- 設定中的權杖會覆蓋環境變數備援。
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` 環境變數備援僅適用於預設帳號。
- 明確設定 userTokenReadOnly（允許使用者權杖寫入）的範例：
- 選用：若希望傳送的訊息使用目前啟用的代理身分（自訂 `username` 與圖示），請加入 `chat:write.customize`。 `icon_emoji` 使用 `:emoji_name:` 語法。

<Tip>
在已設定的情況下，進行動作／目錄讀取時可優先使用使用者權杖。 Setting `userTokenReadOnly: false` allows the user token to be used for write
  operations when a bot token is unavailable, which means actions run with the
  installing user's access. Treat the user token as highly privileged and keep
  action gates and allowlists tight.
</Tip>

## 存取控制與路由

<Tabs>
  <Tab title="DM policy">允許任何人：設定 `channels.slack.dm.policy="open"` 與 `channels.slack.dm.allowFrom=["*"]`。

    ```
    若 Slack 未提供 `channel_type`，OpenClaw 會依頻道 ID 前綴（`D`、`C`、`G`）推斷，並預設為 `channel` 以保持工作階段鍵穩定。
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` 控制頻道處理（`open|disabled|allowlist`）。

    ```
    若你只設定 `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN`，且從未建立 `channels.slack` 區段，
      執行時會將 `groupPolicy` 預設為 `open`。請新增 `channels.slack.groupPolicy`、
    `channels.defaults.groupPolicy`，或頻道允許清單以鎖定行為。 Add `channels.slack.groupPolicy`,
    `channels.defaults.groupPolicy`, or a channel allowlist to lock it down.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    頻道訊息預設需提及（mention）才會觸發。


    ```
    提及門檻由 `channels.slack.channels` 控制（將 `requireMention` 設為 `true`）；`agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）也會被視為提及。
    ```

  
</Tab>
</Tabs>

## 命令與斜線指令行為

- Slack 預設**不會**啟用原生命令自動模式（`commands.native: "auto"` 不會啟用 Slack 原生命令）。
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- 啟用原生命令時，請在 Slack 中註冊對應的斜線指令（`/<command>` 名稱）。
- 若未啟用原生命令，可透過 `channels.slack.slashCommand` 執行單一已設定的斜線指令。

預設斜線指令設定：

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

斜線指令工作階段使用隔離的金鑰：

- Slash 命令使用 `agent:<agentId>:slack:slash:<userId>` 工作階段（前綴可透過 `channels.slack.slashCommand.sessionPrefix` 設定）。

並且仍會將指令執行路由至目標對話工作階段（`CommandTargetSessionKey`）。

## 執行緒、工作階段與回覆標籤

- DM 會路由為 `direct`；頻道為 `channel`；MPIM 為 `group`。
- 在預設 `session.dmScope=main` 下，Slack DM 會合併至 agent 的主工作階段。
- 頻道對應到 `agent:<agentId>:slack:channel:<channelId>` 工作階段。
- 在適用情況下，執行緒回覆可建立執行緒工作階段後綴（`:thread:<threadTs>`）。
- `channels.slack.thread.historyScope` 預設為 `thread`；`thread.inheritParent` 預設為 `false`。
- `channels.slack.historyLimit`（或 `channels.slack.accounts.*.historyLimit`）控制要將多少最近的頻道／群組訊息包入提示中。

回覆執行緒控制：

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- 舊版的 `channels.slack.dm.replyToMode` 仍可作為 `direct` 的備援，當未設定聊天類型覆寫時使用。

支援手動回覆標籤：

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — 回覆到指定的訊息 ID。

注意：`replyToMode="off"` 會停用隱式回覆串接。 明確的 `[[reply_to_*]]` 標籤仍會被套用。

## 媒體、分段與傳遞

<AccordionGroup>
  <Accordion title="Inbound attachments">    
    Slack 檔案附件會從 Slack 託管的私人 URL 下載（以權杖驗證的請求流程），在成功取得且符合大小限制時，寫入媒體儲存區。

    ```
    執行階段的傳入大小上限預設為 `20MB`，除非由 `channels.slack.mediaMaxMb` 覆寫。
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">外送文字會被分段為 `channels.slack.textChunkLimit`（預設 4000）。
</Accordion>

  <Accordion title="Delivery targets">    
    建議使用明確的目標：

    ```
    - `user:<id>` 用於 DM
    - `channel:<id>` 用於頻道
    
    當傳送至使用者目標時，Slack DM 會透過 Slack 對話 API 開啟。
    ```

  
</Accordion>
</AccordionGroup>

## 動作與閘門

Slack 工具動作可透過 `channels.slack.actions.*` 設限：

目前 Slack 工具中可用的動作群組：

| 動作群組       | Default |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

## 事件與操作行為

- 訊息編輯／刪除／執行緒廣播會對應為系統事件。
- 新增／移除反應事件會對應為系統事件。
- 成員加入／離開、頻道建立／重新命名，以及釘選新增／移除事件會對應為系統事件。
- 當啟用 `configWrites` 時，`channel_id_changed` 可遷移頻道設定金鑰。
- 頻道 topic／purpose 中繼資料被視為不受信任的內容，並可注入至路由內容中。

## Ack 反應

`ackReaction` 會在 OpenClaw 處理傳入訊息時傳送一個確認 emoji。

解析順序：

- `或`channels.slack.channels.<name>.ackReaction\`
- `channels.slack.ackReaction`
- `replyToMode`
- agent 身分 emoji 備援（`agents.list[].identity.emoji`，否則為 "👀"）

注意事項

- Slack 需要使用短代碼（例如 `"eyes"`）。
- 使用 `""` 可在頻道或帳號層級停用該反應。

## Manifest 與權限範圍檢查清單

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">若你設定 `channels.slack.userToken`，請在 **User Token Scopes** 下新增以下項目。

    ```
    `channels:history`、`groups:history`、`im:history`、`mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="No replies in channels">
    依序檢查：

    ```
    `allowlist` 要求頻道必須列在 `channels.slack.channels` 中。
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    檢查：

    ```
    若要 **不允許任何頻道**，請設定 `channels.slack.groupPolicy: "disabled"`（或保留空的允許清單）。
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    在 Slack 應用程式設定中，驗證 bot 與 app tokens，以及是否已啟用 Socket Mode。
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    驗證：

    ```
    **Event Subscriptions** → 啟用事件，並將 **Request URL** 設為你的 Gateway 閘道器 webhook 路徑（預設 `/slack/events`）。
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    確認您是否原本打算：

    ```
    Native command registration uses `commands.native` (global default `"auto"` → Slack off) and can be overridden per-workspace with `channels.slack.commands.native`. Text commands require standalone `/...` messages and can be disabled with `commands.text: false`. Slack slash commands are managed in the Slack app and are not removed automatically. Use `commands.useAccessGroups: false` to bypass access-group checks for commands.
    ```

  
</Accordion>
</AccordionGroup>

## 設定參考指引

優先順序：

- 在 [https://api.slack.com/apps](https://api.slack.com/apps) 建立 Slack 應用程式（From scratch）。

  高重要性的 Slack 欄位：

  - mode/auth：`mode`、`botToken`、`appToken`、`signingSecret`、`webhookPath`、`accounts.*`
  - DM 存取：`dm.enabled`、`dmPolicy`、`allowFrom`（舊版：`dm.policy`、`dm.allowFrom`）、`dm.groupEnabled`、`dm.groupChannels`
  - `allow`：在 `groupPolicy="allowlist"` 時允許／拒絕該頻道。
  - threading/history：`replyToMode`、`replyToModeByChatType`、`thread.*`、`historyLimit`、`dmHistoryLimit`、`dms.*.historyLimit`
  - 傳送設定：`textChunkLimit`、`chunkMode`、`mediaMaxMb`
  - 營運／功能：`configWrites`、`commands.native`、`slashCommand.*`、`actions.*`、`userToken`、`userTokenReadOnly`

## 相關內容

- [配對](/channels/pairing)
- `channel`：一般頻道（公開／私有）
- 分流流程請見：[/channels/troubleshooting](/channels/troubleshooting)。
- [設定](/gateway/configuration)
- 完整命令清單與設定：[Slash commands](/tools/slash-commands)


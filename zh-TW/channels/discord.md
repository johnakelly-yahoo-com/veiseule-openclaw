---
summary: "Discord bot support status, capabilities, and configuration"
read_when:
  - 進行 Discord 頻道功能開發時
title: "Discord"
---

# Discord（Bot API）

Status: ready for DM and guild text channels via the official Discord bot gateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord 私訊預設為配對模式。
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    原生命令行為與命令目錄。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    跨通道診斷與修復流程。
  
</Card>
</CardGroup>

## 快速設定（新手）

<Steps>
  <Step title="Create a Discord bot and enable intents">在 Discord Developer Portal 中建立一個應用程式，新增一個 bot，然後啟用：

    ```
    - **Message Content Intent**
    - **Server Members Intent**（角色允許清單與基於角色的路由必須啟用；建議用於名稱對 ID 的允許清單比對）
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    預設帳號的環境變數回退：
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Invite the bot to your server with the permissions required to read/send messages where you want to use it.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    配對代碼將於 1 小時後過期。
    ```

  
</Step>
</Steps>

<Note>
Token 解析具備帳號感知能力。 設定檔中的 token 值優先於環境變數回退。 `DISCORD_BOT_TOKEN` 僅用於預設帳號。
</Note>

## 執行時模型

- Gateway 負責管理 Discord 連線。
- 回覆路由為確定性：從 Discord 收到的回覆會回到 Discord。
- 預設情況下（`session.dmScope=main`），私訊會共用 agent 主工作階段（`agent:main:main`）。
- Guild 頻道使用隔離的工作階段鍵（`agent:<agentId>:discord:channel:<channelId>`）。
- 群組私訊預設會被忽略；可透過 `channels.discord.dm.groupEnabled` 啟用，並可選擇以 `channels.discord.dm.groupChannels` 進行限制。
- 原生命令使用隔離的工作階段鍵（`agent:<agentId>:discord:slash:<userId>`），而非共用的 `main` 工作階段。

## 存取控制與路由

<Tabs>
  <Tab title="DM policy">若要維持舊有「對任何人開放」行為：設定 `channels.discord.dm.policy="open"` 與 `channels.discord.dm.allowFrom=["*"]`。

    ```
    若要嚴格允許清單：設定 `channels.discord.dm.policy="allowlist"`，並在 `channels.discord.dm.allowFrom` 列出寄件者。
    ```

  
</Tab>

  <Tab title="Guild policy">行為由 `channels.discord.replyToMode` 控制：

    ```
    若要**不允許任何頻道**，請設定 `channels.discord.groupPolicy: "disabled"`（或保持允許清單為空）。
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    若你只設定了 `DISCORD_BOT_TOKEN`，卻從未建立 `channels.discord` 區段，執行時
        會將 `groupPolicy` 預設為 `open`。請加入 `channels.discord.groupPolicy`、
    `channels.defaults.groupPolicy`，或伺服器/頻道允許清單以鎖定行為。 1.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    預設情況下，Guild 訊息需要被提及（mention-gated）。

    ```
    提及偵測包含：
    
    - 明確提及機器人
    - 已設定的提及樣式（`agents.list[].groupChat.mentionPatterns`，回退至 `messages.groupChat.mentionPatterns`）
    - 在支援情境下的隱式回覆機器人行為
    
    `requireMention` 依 Guild/頻道設定（`channels.discord.guilds...`）。
    
    群組私訊（Group DMs）：
    
    - 預設：忽略（`dm.groupEnabled=false`）
    - 可選：透過 `dm.groupChannels` 設定允許清單（頻道 ID 或 slug）
    ```

  
</Tab>
</Tabs>

### 以角色為基礎的 agent 路由

使用 `bindings[].match.roles` 依角色 ID 將 Discord Guild 成員路由至不同的 agent。 基於角色的綁定僅接受角色 ID，並在 peer 或 parent-peer 綁定之後、guild-only 綁定之前進行評估。 若綁定同時設定其他 match 欄位（例如 `peer` + `guildId` + `roles`），則所有已設定欄位都必須符合。

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Developer Portal 設定

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">在 **Bot** → **Privileged Gateway Intents** 中啟用：

    ```
    - Message Content Intent
    - Server Members Intent（建議）
    
    Presence intent 為選用，僅在需要接收 presence 更新時才需啟用。設定機器人 presence（`setPresence`）不需要為成員啟用 presence 更新。
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">在你的應用程式中：**OAuth2** → **URL Generator**

    ```
    - scopes：`bot`、`applications.commands`
    
    典型的基礎權限：
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions（選用）
    
    除非明確需要，否則避免使用 `Administrator`。
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    啟用 Discord Developer Mode，然後複製：

    ```
    - server ID
    - channel ID
    - user ID
    
    在 OpenClaw 設定中建議使用數字 ID，以利可靠的稽核與探測。
    ```

  
</Accordion>
</AccordionGroup>

## 原生命令與命令驗證

- `commands.native` 預設為 `"auto"`，並於 Discord 啟用。
- 設定中的 `channels.discord.execApprovals.enabled: true`。
- `commands.native=false` 會明確清除先前已註冊的 Discord 原生命令。
- 原生命令驗證使用與一般訊息處理相同的 Discord 允許清單／政策。
- Slash 命令仍可能在 Discord UI 中對未列入允許清單的使用者可見；OpenClaw 會在執行時強制檢查允許清單，並回覆「未授權」。

參考 [Exec approvals](/tools/exec-approvals) 與 [Slash commands](/tools/slash-commands) 以了解更完整的核准與命令流程。

## 功能細節

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord 支援在 agent 輸出中使用回覆標籤：

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    由 `channels.discord.replyToMode` 控制：
    
    - `off`（預設）
    - `first`
    - `all`
    
    注意：`off` 會停用隱式回覆串接。明確的 `[[reply_to_*]]` 標籤仍會被處理。
    
    訊息 ID 會顯示於內容／歷史中，使 agent 能針對特定訊息。
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild 歷史內容：

    ```
    選用的伺服器情境歷史：設定 `channels.discord.historyLimit`（預設 20，回退至 `messages.groupChat.historyLimit`），在回覆提及時包含最近 N 則伺服器訊息作為情境。設定 `0` 可停用。 Set `0` to disable.
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`：

    ```
    - `off`
    - `own`（預設）
    - `all`
    - `allowlist`（使用 `guilds.<id>.users`）
    
    反應事件會轉換為系統事件，並附加至已路由的 Discord 工作階段。
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` 會在 OpenClaw 處理傳入訊息時傳送一個確認表情符號。

    ```
    解析順序：
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - agent 身分表情符號回退（`agents.list[].identity.emoji`，否則為 "👀"）
    
    注意：
    
    - Discord 接受 Unicode 表情符號或自訂表情符號名稱。
    - 使用 `""` 可在頻道或帳號中停用該反應。
    ```

  
</Accordion>

  <Accordion title="Config writes">
    預設啟用由頻道發起的設定寫入。

    ```
    這會影響 `/config set|unset` 流程（當啟用命令功能時）。
    
    停用：
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    透過 `channels.discord.proxy` 將 Discord gateway WebSocket 流量路由至 HTTP(S) 代理。

```json5
私訊會合併至代理程式的主要工作階段（預設 `agent:main:main`）；伺服器頻道則保持隔離為 `agent:<agentId>:discord:channel:<channelId>`（顯示名稱使用 `discord:<guildSlug>#<channelSlug>`）。
```

    ```
    每個帳號可個別覆寫：
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    啟用 PluralKit 解析，將代理訊息對應至系統成員身分：

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    注意：
    
    - 允許清單可使用 `pk:<memberId>`
    - 成員顯示名稱會依名稱／slug 進行比對
    - 查詢使用原始訊息 ID，並受時間視窗限制
    - 若查詢失敗，代理訊息會被視為機器人訊息並捨棄，除非 `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    僅在設定狀態或活動欄位時才會套用 Presence 更新。

    ```
    僅狀態範例：
    ```

```json5
頻道（例如 `#help`）→ **Copy Channel ID**
```

    ```
    活動範例（自訂狀態為預設活動類型）：
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    直播範例：
    ```

```json5
使用 `channels.discord.token`（或 `DISCORD_BOT_TOKEN` 作為回退）設定 OpenClaw。
```

    ```
    活動類型對照表：
    
    - 0: Playing
    - 1: Streaming（需要 `activityUrl`）
    - 2: Listening
    - 3: Watching
    - 4: Custom（使用活動文字作為狀態內容；emoji 為選填）
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    Discord 在私訊中支援以按鈕為基礎的 exec 核准，並可選擇在原始頻道中發布核准提示。

    ```
    **權限稽核**（`channels status --probe`）只會檢查數字型頻道 ID。 **權限稽核**（`channels status --probe`）只會檢查數字頻道 ID。若你使用 slug/名稱作為 `channels.discord.guilds.*.channels` 的鍵，稽核將無法驗證權限。
    ```

  
</Accordion>
</AccordionGroup>

## 工具動作

Discord 訊息動作包含傳送訊息、頻道管理、版務管理、狀態（presence）以及中繼資料相關操作。

核心範例：

- `readMessages`、`sendMessage`、`editMessage`、`deleteMessage`
- reactions：`react`、`reactions`、`emojiList`
- `timeout`、`kick`、`ban`
- presence：`setPresence`

動作閘門位於 `channels.discord.actions.*`。

預設閘門行為：

| 動作群組                                                                                                                                                      | Default |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| reactions、messages、threads、pins、polls、search、memberInfo、roleInfo、channelInfo、channels、voiceStatus、events、stickers、emojiUploads、stickerUploads、permissions | 啟用      |
| 角色                                                                                                                                                        | 停用      |
| moderation                                                                                                                                                | 停用      |
| presence                                                                                                                                                  | 停用      |

## Components v2 UI

OpenClaw 使用 Discord components v2 來處理 exec 核准與跨情境標記。 Discord 訊息動作也可接受 `components` 以建立自訂 UI（進階；需要 Carbon component 實例），舊版 `embeds` 仍可使用，但不建議採用。

- `channels.discord.ui.components.accentColor` 用於設定 Discord 元件容器使用的強調色（hex）。
- 可針對每個帳號於 `channels.discord.accounts.<id>` 設定.ui.components.accentColor\`。
- 當存在 components v2 時，`embeds` 會被忽略。

範例：

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## messages

Discord 語音訊息會顯示波形預覽，並需要 OGG/Opus 音訊及其中繼資料。 OpenClaw 會自動產生波形，但需要在 gateway 主機上提供 `ffmpeg` 與 `ffprobe`，以檢查與轉換音訊檔案。

需求與限制：

- 請提供**本機檔案路徑**（不接受 URL）。
- 請省略文字內容（Discord 不允許在同一個 payload 中同時包含文字與語音訊息）。
- 接受任何音訊格式；必要時 OpenClaw 會轉換為 OGG/Opus。

範例：

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - 啟用 Message Content Intent
    - 當依賴使用者／成員解析時，啟用 Server Members Intent
    - 變更 intents 後重新啟動 gateway
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - 驗證 `groupPolicy`
    - 驗證 `channels.discord.guilds` 下的 guild allowlist
    - 若存在 guild 的 `channels` 對應表，僅允許列出的頻道
    - 驗證 `requireMention` 行為與提及（mention）模式
    
    Useful checks:
    ```

```bash
首先：執行 `openclaw doctor` 與 `openclaw channels status --probe`（可採取行動的警告 + 快速稽核）。
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">    常見原因：

    ```
    `groupPolicy`：控制伺服器頻道處理（`open|disabled|allowlist`）；`allowlist` 需要頻道允許清單。
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">    `channels status --probe` 的權限檢查僅適用於數字頻道 ID。

    ```
    如果使用 slug keys，執行階段仍可正常比對，但 probe 無法完整驗證權限。
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **私訊無法運作**：`channels.discord.dm.enabled=false`、`channels.discord.dm.policy="disabled"`，或你尚未被核准（`channels.discord.dm.policy="pairing"`）。
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">    預設會忽略由機器人發送的訊息。

    ```
    如果設定 `channels.discord.allowBots=true`，請使用嚴格的 mention 與 allowlist 規則以避免迴圈行為。
    ```

  
</Accordion>
</AccordionGroup>

## 設定參考指引

主要參考：

- [設定參考 - Discord](/gateway/configuration-reference#discord)

高重要性的 Discord 欄位：

- startup/auth：`enabled`、`token`、`accounts.*`、`allowBots`
- `guilds.<id> .channels.<channel> .allow`：當 `groupPolicy="allowlist"` 時允許/拒絕頻道。
- Use `commands.useAccessGroups: false` to bypass access-group checks for commands.
- `dmHistoryLimit`：以使用者回合數計的私訊歷史上限。 `dmHistoryLimit`：私訊歷史上限（以使用者回合計）。每使用者覆寫：`dms["<user_id>"].historyLimit`。
- delivery：`textChunkLimit`、`chunkMode`、`maxLinesPerMessage`
- media/retry：`mediaMaxMb`、`retry`
- actions：`actions.*`
- presence：`activity`、`status`、`activityType`、`activityUrl`
- UI：`ui.components.accentColor`
- features：`pluralkit`、`execApprovals`、`intents`、`agentComponents`、`heartbeat`、`responsePrefix`

## 安全性與營運

- 將機器人權杖視為機密（在受監管環境中建議使用 `DISCORD_BOT_TOKEN`）。
- 僅授予最小必要的 Discord 權限。
- 若指令部署／狀態過期，請重新啟動 gateway，並使用 `openclaw channels status --probe` 再次檢查。

## 相關

- 頻道/類別管理
- [Channel 路由](/channels/channel-routing)
- [疑難排解](/channels/troubleshooting)
- 完整命令清單與設定：[Slash commands](/tools/slash-commands)

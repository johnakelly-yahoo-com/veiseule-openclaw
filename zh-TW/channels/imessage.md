---
summary: "透過 imsg 的舊版 iMessage 支援（以 stdio 進行 JSON-RPC）。新的設定應使用 BlueBubbles。 透過 imsg 的舊版 iMessage 支援（以 stdio 進行 JSON-RPC）。新的設定應使用 BlueBubbles。 New setups should use BlueBubbles."
read_when:
  - 設定 iMessage 支援
  - 除錯 iMessage 傳送／接收
title: "iMessage"
---

# iMessage（舊版：imsg）

<Warning>
**建議：** 新的 iMessage 設定請使用 [BlueBubbles](/channels/bluebubbles)。

`imsg` 整合屬於舊版功能，未來版本可能會移除。 
</Warning>

Status: legacy external CLI integration. 狀態：舊版外部 CLI 整合。Gateway 會啟動 `imsg rpc`（以 stdio 進行 JSON-RPC）。

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    新設置建議使用的 iMessage 路徑。
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">Pairing is the default token exchange for iMessage DMs.
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    完整的 iMessage 欄位參考。
  
</Card>
</CardGroup>

## 設定（快速路徑）

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="設定 OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="啟動 gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="核准首次 DM 配對（預設 dmPolicy）">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            配對請求會在 1 小時後過期。
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">若你希望在另一台 Mac 上使用 iMessage，請將 `channels.imessage.cliPath` 設為一個包裝器，透過 SSH 在遠端 macOS 主機上執行 `imsg`。OpenClaw 只需要 stdio。 OpenClaw only needs stdio.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    啟用附件時建議的設定：
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    若未設定 `remoteHost`，OpenClaw 會嘗試解析你的包裝腳本中的 SSH 指令來自動偵測。為了可靠性，建議明確設定。 Explicit configuration is recommended for reliability.
    ```

  
</Tab>
</Tabs>

## 需求與權限（macOS）

- 確認此 Mac 上的「訊息」已登入。
- 執行 OpenClaw/`imsg` 的程序環境需要「完整磁碟存取」權限（用於存取 Messages 資料庫）。
- 需要「自動化」權限，才能透過 Messages.app 傳送訊息。

<Tip>
權限是依據各個程序環境分別授予。 如果 gateway 以無頭模式執行（LaunchAgent/SSH），請在相同的程序環境中執行一次互動式指令，以觸發權限提示：

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## 存取控制與路由

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` 控制私訊：

    ```
    `channels.imessage.groupPolicy`：`open | allowlist | disabled`（預設：允許清單）。
    ```

  
</Tab>

  <Tab title="Group policy + mentions">當設定 `allowlist` 時，`channels.imessage.groupAllowFrom` 會控制誰可以在群組中觸發。

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - 私訊使用直接路由；群組使用群組路由。
    - 在預設 `session.dmScope=main` 下，iMessage 私訊會合併到 agent 的主工作階段。
    - 群組工作階段是隔離的（`agent:<agentId>群組：<chat_id>`）。
    - 回覆會使用來源的 channel/target 中繼資料路由回 iMessage。

    ```
    若多參與者執行緒以 `is_group=false` 抵達，你仍可透過 `channels.imessage.groups` 使用 `chat_id` 進行隔離（見下方「類群組執行緒」）。
    ```

  
</Tab>
</Tabs>

## 部署模式

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    使用專用的 Apple ID 與 macOS 使用者帳號，讓機器人流量與您的個人 Messages 設定檔隔離。

    ```
    使用 `channels.imessage.cliPath` 與 `channels.imessage.dbPath` 設定 OpenClaw。
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    常見拓撲：

    ```
    若 Gateway 在 Linux 主機／VM 上執行，但 iMessage 必須在 Mac 上運作，Tailscale 是最簡單的橋接方式：Gateway 透過 tailnet 與 Mac 通訊，經由 SSH 執行 `imsg`，並將附件以 SCP 傳回。
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    使用 SSH 金鑰，讓 SSH 與 SCP 皆可非互動式執行。
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">由 macOS 上的 `imsg` 支援的 iMessage 頻道。

    ```
    每個帳號都可以覆寫欄位，例如 `cliPath`、`dbPath`、`allowFrom`、`groupPolicy`、`mediaMaxMb` 以及歷史記錄設定。
    ```

  
</Accordion>
</AccordionGroup>

## 媒體、分段與傳送目標

<AccordionGroup>
  <Accordion title="Attachments and media">透過 `channels.imessage.mediaMaxMb` 的媒體上限。
</Accordion>

  <Accordion title="Outbound chunking">`channels.imessage.chunkMode`：`length`（預設）或 `newline`，在依長度分段前先依空白行（段落邊界）分割。
</Accordion>

  <Accordion title="Addressing formats">
    建議使用明確指定的目標：

    ```
    - `chat_id:123`（建議用於穩定路由）
    - `chat_guid:...`
    - `chat_identifier:...`
    
    也支援 handle 目標：
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## 設定寫入

預設允許 iMessage 寫入由 `/config set|unset` 觸發的設定更新（需要 `commands.config: true`）。

停用方式：

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## 疑難排解

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    驗證二進位檔與 RPC 支援：

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    如果 probe 顯示不支援 RPC，請更新 `imsg`。
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">注意事項：

    ```
    `channels.imessage.dmPolicy`：`pairing | allowlist | open | disabled`（預設：配對）。
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">檢查清單：

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`。
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Approve via:

    ```
    `channels.imessage.remoteHost`：當 `cliPath` 指向遠端 Mac（例如 `user@gateway-host`）時，用於 SCP 附件傳輸的 SSH 主機。若未設定，將從 SSH 包裝器自動偵測。 Auto-detected from SSH wrapper if not set.
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    在相同使用者／工作階段環境下，於互動式 GUI 終端機重新執行並核准提示：

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    確認執行 OpenClaw/`imsg` 的程序環境已授予「完整磁碟存取」與「自動化」權限。
    ```

  
</Accordion>
</AccordionGroup>

## 設定參考指引

- 設定 iMessage 並啟動 Gateway。
- 完整設定：[設定](/gateway/configuration)
- Details: [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)


---
title: IRC
description: 將 OpenClaw 連接到 IRC 頻道與私訊。
---

當你希望 OpenClaw 在傳統頻道（`#room`）與私訊中運作時，請使用 IRC。
IRC 以擴充外掛形式提供，但需在主設定檔的 `channels.irc` 中進行設定。

## 快速開始

1. 在 `~/.openclaw/openclaw.json` 中啟用 IRC 設定。
2. 至少設定：

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. 啟動／重新啟動 gateway：

```bash
openclaw gateway run
```

## 安全性預設值

- `channels.irc.dmPolicy` 預設為 `"pairing"`。
- `channels.irc.groupPolicy` 預設為 `"allowlist"`。
- 當 `groupPolicy="allowlist"` 時，請設定 `channels.irc.groups` 以定義允許的頻道。
- 除非你刻意接受明文傳輸，否則請使用 TLS（`channels.irc.tls=true`）。

## 存取控制

IRC 頻道有兩個獨立的「關卡」：

1. **頻道存取**（`groupPolicy` + `groups`）：機器人是否接受來自該頻道的訊息。
2. **發送者存取**（`groupAllowFrom` / 每個頻道的 `groups["#channel"].allowFrom`）：在該頻道中誰可以觸發機器人。

設定鍵：

- DM 允許清單（DM 發送者存取）：`channels.irc.allowFrom`
- 群組發送者允許清單（頻道發送者存取）：`channels.irc.groupAllowFrom`
- 每個頻道的控制（頻道 + 發送者 + 提及規則）：`channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` 允許未設定的頻道（**預設仍需提及才會回應**）

允許清單項目可使用 nick 或 `nick!user@host` 格式。

### 常見錯誤：`allowFrom` 用於 DM，而非頻道

如果你看到類似以下的日誌：

- `irc: drop group sender alice!ident@host (policy=allowlist)`

……這表示該發送者未被允許在**群組／頻道**訊息中使用。 可透過以下方式修正：

- 設定 `channels.irc.groupAllowFrom`（對所有頻道全域生效），或
- 設定每個頻道的發送者允許清單：`channels.irc.groups["#channel"].allowFrom`

範例（允許 `#tuirc-dev` 中的任何人與機器人互動）：

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## 回應觸發（提及）

即使某個頻道已被允許（透過 `groupPolicy` + `groups`）且發送者也被允許，OpenClaw 在群組情境中預設仍採用**提及限制**。

這表示你可能會看到類似 `drop channel …` 的日誌 `(missing-mention)`，除非訊息中包含符合機器人的提及模式。

若要讓機器人在 IRC 頻道中**不需提及即可回應**，請為該頻道停用提及限制：

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

或者允許 **所有** IRC 頻道（不使用每頻道允許清單），且在未被提及時仍可回覆：

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## 安全性注意事項（建議用於公開頻道）

如果你在公開頻道中設定 `allowFrom: ["*"]`，任何人都可以向機器人發送提示。
為降低風險，請限制該頻道可使用的工具。

### 頻道內所有人使用相同工具

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### 依發送者使用不同工具（擁有者擁有更多權限）

使用 `toolsBySender` 對 `"*"` 套用較嚴格的政策，並對你的暱稱套用較寬鬆的政策：

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

注意事項：

- `toolsBySender` 的鍵可以是暱稱（例如 `"eigen"`），或是完整的 hostmask（`"eigen!~eigen@174.127.248.171"`），以進行更嚴格的身分比對。
- 第一個符合的發送者政策將會生效；`"*"` 為萬用的預設規則。

如需了解群組存取與提及限制（mention-gating）之間的差異與互動方式，請參閱：[/channels/groups](/channels/groups)。

## NickServ

連線後使用 NickServ 進行身分驗證：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

可選：連線時進行一次性註冊：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

暱稱註冊完成後，請停用 `register` 以避免重複嘗試 REGISTER。

## 環境變數

預設帳戶支援：

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS`（以逗號分隔）
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## 疑難排解

- 如果機器人已連線但在頻道中從未回覆，請確認 `channels.irc.groups` **以及** 是否因提及限制（`missing-mention`）而丟棄訊息。 若希望在未被提及（無 ping）時也能回覆，請為該頻道設定 `requireMention:false`。
- 如果登入失敗，請確認暱稱是否可用以及伺服器密碼是否正確。
- 若在自訂網路上 TLS 連線失敗，請確認主機／連接埠與憑證設定是否正確。

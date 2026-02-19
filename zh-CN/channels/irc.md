---
title: IRC
description: 将 OpenClaw 连接到 IRC 频道和私信。
---

当你希望 OpenClaw 加入经典频道（`#room`）和私信时，请使用 IRC。
IRC 作为扩展插件提供，但在主配置中的 `channels.irc` 下进行配置。

## 快速开始

1. 在 `~/.openclaw/openclaw.json` 中启用 IRC 配置。
2. 至少设置：

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

3. 启动/重启 gateway：

```bash
openclaw gateway run
```

## 安全默认设置

- `channels.irc.dmPolicy` 默认值为 `"pairing"`。
- `channels.irc.groupPolicy` 默认值为 `"allowlist"`。
- 当 `groupPolicy="allowlist"` 时，设置 `channels.irc.groups` 以定义允许的频道。
- 除非你有意接受明文传输，否则请使用 TLS（`channels.irc.tls=true`）。

## 访问控制

IRC 频道有两个独立的“门控”机制：

1. **频道访问**（`groupPolicy` + `groups`）：机器人是否接受来自某个频道的消息。
2. **发送者访问**（`groupAllowFrom` / 每频道 `groups["#channel"].allowFrom`）：在该频道内谁可以触发机器人。

配置键：

- DM 允许列表（DM 发送者访问）：`channels.irc.allowFrom`
- 群组发送者允许列表（频道发送者访问）：`channels.irc.groupAllowFrom`
- 每频道控制（频道 + 发送者 + 提及规则）：`channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` 允许未配置的频道（**默认仍受提及门控限制**）

允许列表条目可以使用 nick 或 `nick!user@host` 形式。

### 常见陷阱：`allowFrom` 适用于 DM，而不适用于频道

如果你看到如下日志：

- `irc: drop group sender alice!ident@host (policy=allowlist)`

……这表示该发送者不被允许发送**群组/频道**消息。 可以通过以下方式之一修复：

- 设置 `channels.irc.groupAllowFrom`（对所有频道全局生效），或
- 为每个频道单独设置发送者允许列表：`channels.irc.groups["#channel"].allowFrom`

示例（允许 `#tuirc-dev` 中的任何人与机器人对话）：

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

## 回复触发（提及）

即使某个频道已被允许（通过 `groupPolicy` + `groups`）且发送者也被允许，OpenClaw 在群组场景中默认启用**提及门控（mention-gating）**。

这意味着你可能会看到类似 `drop channel …` 的日志 `(missing-mention)`，除非消息中包含与机器人匹配的提及模式。

要让机器人在 IRC 频道中**无需提及即可回复**，请为该频道禁用提及门控：

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

或者允许**所有** IRC 频道（不使用按频道允许列表），并且无需提及即可回复：

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

## 安全说明（建议用于公共频道）

如果你在公共频道中设置 `allowFrom: ["*"]`，任何人都可以向机器人发送提示。
为降低风险，请限制该频道可用的工具。

### 频道内所有人使用相同的工具

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

### 为不同发送者分配不同工具（owner 拥有更多权限）

使用 `toolsBySender` 为 `"*"` 应用更严格的策略，而为你的昵称应用更宽松的策略：

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

说明：

- `toolsBySender` 的键可以是昵称（例如 `"eigen"`），也可以是完整的 hostmask（`"eigen!~eigen@174.127.248.171"`），以实现更强的身份匹配。
- 第一个匹配到的发送者策略将生效；`"*"` 是通配符兜底规则。

有关群组访问与提及门控（以及它们如何相互作用）的更多信息，请参阅：[/channels/groups](/channels/groups)。

## NickServ

在连接后通过 NickServ 进行身份识别：

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

可选：连接时进行一次性注册：

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

昵称注册完成后，请禁用 `register`，以避免重复尝试 REGISTER。

## 环境变量

默认账户支持：

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS`（用逗号分隔）
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## 故障排查

- 如果机器人已连接但在频道中从不回复，请检查 `channels.irc.groups` **以及** 是否因为提及限制而丢弃了消息（`missing-mention`）。 如果希望在无需 @ 提及的情况下回复，请为该频道设置 `requireMention:false`。
- 如果登录失败，请检查昵称是否可用以及服务器密码是否正确。
- 如果在自定义网络上 TLS 失败，请检查主机/端口以及证书配置。

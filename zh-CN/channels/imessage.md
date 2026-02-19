---
summary: "通过 imsg（基于 stdio 的 JSON-RPC）实现 iMessage 支持、设置及 chat_id 路由 22. 新的部署应使用 BlueBubbles。"
read_when:
  - 设置 iMessage 支持
  - 调试 iMessage 发送/接收
title: "iMessage"
---

# iMessage (imsg)

<Warning>
对于新的 iMessage 部署，请使用 <a href="/channels/bluebubbles">BlueBubbles</a>。

`imsg` 集成为旧版功能，可能会在未来版本中移除。 
</Warning>

29. 状态：旧的外部 CLI 集成。 状态：外部 CLI 集成。Gateway 网关生成 `imsg rpc`（基于 stdio 的 JSON-RPC）。

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">配置参考（iMessage）
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage 私信默认使用配对模式。
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">`channels.imessage.region`：短信区域。
</Card>
</CardGroup>

## 快速设置（新手）

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="Configure OpenClaw">

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
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Approve first DM pairing (default dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            配对请求将在 1 小时后过期。
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">如果你想在另一台 Mac 上使用 iMessage，请将 `channels.imessage.cliPath` 设置为通过 SSH 在远程 macOS 主机上运行 `imsg` 的包装脚本。OpenClaw 只需要 stdio。

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    启用附件时的推荐配置：
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
    如果未设置 `remoteHost`，OpenClaw 会尝试通过解析包装脚本中的 SSH 命令自动检测。建议显式配置以提高可靠性。
    ```

  
</Tab>
</Tabs>

## 要求与权限（macOS）

- 确保在此 Mac 上已登录"信息"。
- OpenClaw + `imsg` 的完全磁盘访问权限（访问"信息"数据库）。
- 发送时需要自动化权限。

<Tip>
权限按进程上下文授予。 如果 gateway 以无头模式运行（LaunchAgent/SSH），请在相同的上下文中运行一次交互式命令以触发权限提示：

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## 访问控制与路由

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` 用于控制私信：

    ```
    `channels.imessage.allowFrom`：私信允许列表（handle、邮箱、E.164 号码或 `chat_id:*`）。`open` 需要 `"*"`。iMessage 没有用户名；使用 handle 或聊天目标。
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupPolicy = open | allowlist | disabled`。

    ```
    设置 `allowlist` 时，`channels.imessage.groupAllowFrom` 控制谁可以在群组中触发。
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - 私信使用直接路由；群组使用群组路由。
    - 在默认 `session.dmScope=main` 下，iMessage 私信会合并到代理的主会话中。
    - 群组会话是隔离的（`agent:<agentId>群组：<chat_id>`）。
    - 回复会通过原始通道/目标元数据路由回 iMessage。

    ```
    如果多参与者会话以 `is_group=false` 到达，你仍可使用 `channels.imessage.groups` 按 `chat_id` 隔离（参见下方"类群组会话"）。
    ```

  
</Tab>
</Tabs>

## 部署模式

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">如果你希望机器人从**独立的 iMessage 身份**发送（并保持你的个人"信息"整洁），请使用专用 Apple ID + 专用 macOS 用户。

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

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    常见拓扑：

    ```
    如果 Gateway 网关运行在 Linux 主机/虚拟机上但 iMessage 必须运行在 Mac 上，Tailscale 是最简单的桥接方式：Gateway 网关通过 tailnet 与 Mac 通信，通过 SSH 运行 `imsg`，并通过 SCP 获取附件。
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
    使用 SSH 密钥以确保 SSH 和 SCP 均为非交互式。
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">对于单账户设置，使用扁平选项（`channels.imessage.cliPath`、`channels.imessage.dbPath`）而不是 `accounts` 映射。

    ```
    每个账户都可以覆盖 `cliPath`、`dbPath`、`allowFrom`、`groupPolicy`、`mediaMaxMb` 以及历史记录设置等字段。
    ```

  
</Accordion>
</AccordionGroup>

## 媒体、分块与投递目标

<AccordionGroup>
  <Accordion title="Attachments and media">媒体上传受 `channels.imessage.mediaMaxMb` 限制（默认 16）。
</Accordion>

  <Accordion title="Outbound chunking">`channels.imessage.chunkMode`：`length`（默认）或 `newline` 在长度分块前按空行（段落边界）分割。
</Accordion>

  <Accordion title="Addressing formats">
    首选显式目标：

    ```
    直接 handle：`imessage:+1555` / `sms:+1555` / `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## 配置写入

默认情况下，iMessage 允许写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。

禁用方式：

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## 故障排查

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    验证二进制文件和 RPC 支持：

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    如果探测结果显示不支持 RPC，请更新 `imsg`。
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">
    检查：

    ```
    `channels.imessage.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）。
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    检查：

    ```
    `channels.imessage.groupPolicy`：`open | allowlist | disabled`（默认：allowlist）。
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">
    检查：

    ```
    `channels.imessage.remoteHost`：当 `cliPath` 指向远程 Mac 时用于 SCP 附件传输的 SSH 主机（例如 `user@gateway-host`）。如未设置则从 SSH 包装脚本自动检测。
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    在相同用户/会话上下文的交互式 GUI 终端中重新运行，并批准权限提示：

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    确认为运行 OpenClaw/`imsg` 的进程上下文授予了“完全磁盘访问”和“自动化”权限。
    ```

  
</Accordion>
</AccordionGroup>

## 配置参考指引

- 配置 iMessage 并启动 Gateway 网关。
- 完整配置：[配置](/gateway/configuration)
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

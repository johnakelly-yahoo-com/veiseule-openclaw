---
summary: "WhatsApp 渠道支持、访问控制、消息投递行为与运维"
read_when:
  - 处理 WhatsApp/网页渠道行为或收件箱路由时
title: "WhatsApp"
---

# WhatsApp（网页渠道）

状态：通过 WhatsApp Web（Baileys）达到生产就绪。 Gateway 拥有已关联的会话。

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">默认 DM 策略是对未知发送者进行配对（pairing）。
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">跨渠道诊断与修复操作手册。
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">完整的渠道配置模式与示例。
</Card>
</CardGroup>

## 快速设置（新手）

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
多账户登录：`openclaw channels login --account <id>`（`<id>` = `accountId`）。
```

    ```
    针对特定账户：
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
使用以下命令批准：`openclaw pairing approve whatsapp <code>`（使用 `openclaw pairing list whatsapp` 列出）。
```

    ```
    码在 1 小时后过期；每个渠道的待处理请求上限为 3 个。
    ```

  
</Step>
</Steps>

<Note>
如果你在**个人 WhatsApp 号码**上运行 OpenClaw，请启用 `channels.whatsapp.selfChatMode`（见上面的示例）。 （该渠道的元数据和引导流程针对该设置进行了优化，但也支持个人号码设置。）
</Note>

## 部署模式

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">这是最简洁的运维模式：

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

  <Accordion title="Personal-number fallback">引导流程支持个人号码模式，并写入适用于自聊天的基础配置：

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

  <Accordion title="WhatsApp Web-only channel scope">当前 OpenClaw 渠道架构中的消息平台渠道基于 WhatsApp Web（`Baileys`）。

    ```
    在内置的聊天渠道注册表中，没有单独的 Twilio WhatsApp 消息渠道。
    ```

  
</Accordion>
</AccordionGroup>

## 运行时模型

- Gateway 负责管理 WhatsApp socket 和重连循环。
- 出站消息发送需要目标账户存在一个活跃的 WhatsApp 监听器。
- 状态/广播聊天被忽略。
- 私聊使用 DM 会话规则（`session.dmScope`；默认 `main` 会将 DM 合并到代理的主会话）。
- 群组映射到 `agent:<agentId>:whatsapp:group:<jid>` 会话。

## 访问控制与激活

<Tabs>
  <Tab title="DM policy">**私信策略**：`channels.whatsapp.dmPolicy` 控制直接聊天访问（默认：`pairing`）。

    ```
    `channels.whatsapp.dmPolicy`（私信策略：pairing/allowlist/open/disabled）。
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">群组访问有两层控制：

    ```
    `channels.whatsapp.groupAllowFrom`（群组发送者允许列表）。
    ```

  
</Tab>

  <Tab title="Mentions + /activation">默认情况下，群组回复需要被提及（mention）。

    ```
    提及检测包括：
    
    - 明确提及 WhatsApp 中的机器人身份
    - 配置的提及正则模式（`agents.list[].groupChat.mentionPatterns`，回退为 `messages.groupChat.mentionPatterns`）
    - 隐式回复机器人检测（回复发送者与机器人身份匹配）
    
    会话级激活命令：
    
    - `/activation mention`
    - `/activation always`
    
    `activation` 会更新会话状态（而非全局配置），并且仅限所有者执行。
    ```

  
</Tab>
</Tabs>

## 个人号码与自聊行为

当已关联的个人号码同时出现在 `allowFrom` 中时，将启用 WhatsApp 自聊保护机制：

- 跳过自聊回合的已读回执
- 忽略提及 JID 的自动触发行为，避免误提醒自己
- 当设置了 `identity.name` 时，自聊天回复默认为 `[{identity.name}]`（否则为 `[openclaw]`），
  前提是 `messages.responsePrefix` 未设置。明确设置它可以自定义或禁用
  前缀（使用 `""` 来移除）。

## 消息标准化与上下文

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">    传入的 WhatsApp 消息会被封装为统一的入站信封结构。

    ````
    如果存在引用回复，将按以下格式附加上下文：
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    在可用情况下，也会填充回复元数据字段（`ReplyToId`、`ReplyToBody`、`ReplyToSender`、发送者 JID/E.164）。
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">纯媒体入站消息使用占位符：

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    位置信息和联系人负载会在路由前标准化为文本上下文。
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">    对于群组，未处理的消息可以被缓冲，并在机器人最终被触发时作为上下文注入。

    ```
    最近*未处理*的消息（默认 50 条）插入在：
    `[Chat messages since your last reply - for context]`（已在会话中的消息不会重新注入）
    ```

  
</Accordion>

  <Accordion title="Read receipts">    默认情况下，对已接受的入站 WhatsApp 消息启用已读回执。

    ```
    登出：`openclaw channels logout`（或 `--account <id>`）删除 WhatsApp 认证状态（但保留共享的 `oauth.json`）。
    ```

  
</Accordion>
</AccordionGroup>

## 投递、分块与媒体

<AccordionGroup>
  <Accordion title="Text chunking">可选换行分块：设置 `channels.whatsapp.chunkMode="newline"` 在长度分块之前按空行（段落边界）分割。
</Accordion>

  <Accordion title="Outbound media behavior">    - 支持图片、视频、音频（PTT 语音消息）和文档负载
    - 为兼容语音消息，`audio/ogg` 会被重写为 `audio/ogg; codecs=opus`
    - 发送视频时通过设置 `gifPlayback: true` 支持动画 GIF 播放
    - 发送多媒体回复时，标题说明会应用到第一项媒体
    - 媒体来源可以是 HTTP(S)、`file://` 或本地路径
</Accordion>

  <Accordion title="Media size limits and fallback behavior">    - 入站媒体保存上限：`channels.whatsapp.mediaMaxMb`（默认 `50`）
    - 自动回复的出站媒体上限：`agents.defaults.mediaMaxMb`（默认 `5MB`）
    - 图片会自动优化（尺寸调整/质量压缩）以符合限制
    - 媒体发送失败时，将回退为发送文本警告，而不是静默丢弃响应
</Accordion>
</AccordionGroup>

## 确认反应

`channels.whatsapp.ackReaction`（消息收到时的自动回应：`{emoji, direct, group}`）。

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

行为：

- 在入站消息被接受后立即发送（回复前）
- 失败会被记录日志，但不会阻止正常回复投递
- 群组模式 `mentions` 仅在提及时触发的回合发送反应；群组激活模式 `always` 会绕过此检查
- WhatsApp 忽略 `messages.ackReaction`；请改用 `channels.whatsapp.ackReaction`。

## 多账号与凭证

<AccordionGroup>
  <Accordion title="Account selection and defaults">默认账户（省略 `--account` 时）：如果存在则为 `default`，否则为第一个配置的账户 id（排序后）。
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">凭证存储在 `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`。<accountId>/creds.json`
    - 备份文件：`creds.json.bak`
    - 仍会识别/迁移位于 `~/.openclaw/credentials/` 的旧版默认认证，用于默认账号流程
</Accordion>

  <Accordion title="Logout behavior">`channels.whatsapp.accounts.<accountId><id>]` 会清除该账号的 WhatsApp 认证状态。

    ```
    在旧版认证目录中，会保留 `oauth.json`，同时移除 Baileys 认证文件。
    ```

  
</Accordion>
</AccordionGroup>

## 工具、操作与配置写入

- Agent 工具支持包括 WhatsApp 反应操作（`react`）。
- 操作门控：
  - `channels.whatsapp.actions.reactions`（门控 WhatsApp 工具表情回应）。
  - channels/whatsapp.md
- {
  channels: { whatsapp: { configWrites: false } },
  }

## 故障排查

<AccordionGroup>
  <Accordion title="Not linked (QR required)">症状：`channels status` 显示 `linked: false` 或警告"Not linked"。

    ```
    修复：在 Gateway 网关主机上运行 `openclaw channels login` 并扫描二维码（WhatsApp → 设置 → 关联设备）。
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">    症状：已关联账号反复断开或尝试重连。

    ```
    修复：`openclaw doctor`（或重启 Gateway 网关）。如果问题持续，通过 `channels login` 重新关联并检查 `openclaw logs --follow`。
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">当目标账号不存在活动的 gateway 监听器时，出站发送会快速失败。

    ```
    请确保 gateway 正在运行且账号已关联。
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">    请按以下顺序检查：

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` 允许列表条目
    - 提及门控（`requireMention` + 提及模式）
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway 运行时应使用 Node。 Bun 被标记为与稳定运行 WhatsApp/Telegram gateway 不兼容。
  
</Accordion>
</AccordionGroup>

## 配置参考指引

主要参考：

- WhatsApp Web 发送标准消息（当前 Gateway 网关无引用回复线程）。

高频使用的 WhatsApp 字段：

- 访问控制：`dmPolicy`、`allowFrom`、`groupPolicy`、`groupAllowFrom`、`groups`
- 投递相关：`textChunkLimit`、`chunkMode`、`mediaMaxMb`、`sendReadReceipts`、`ackReaction`
- 多账号：`accounts.<id>`.enabled`, `accounts.<id>`.authDir`、账户级覆盖
- 操作：`configWrites`、`debounceMs`、`web.enabled`、`web.heartbeatSeconds`、`web.reconnect.*`
- 会话行为：`session.dmScope`、`historyLimit`、`dmHistoryLimit`、`dms.<id>``messages.groupChat.historyLimit`

## 相关

- [配对](/channels/pairing)
- [频道路由](/channels/channel-routing)
- [故障排查](/channels/troubleshooting)


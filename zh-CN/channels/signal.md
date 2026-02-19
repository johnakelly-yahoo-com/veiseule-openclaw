---
summary: "通过 signal-cli（JSON-RPC + SSE）支持 Signal，设置和号码模型"
read_when:
  - 设置 Signal 支持
  - 调试 Signal 发送/接收
title: "Signal"
---

# Signal (signal-cli)

Status: external CLI integration. 状态：外部 CLI 集成。Gateway 网关通过 HTTP JSON-RPC + SSE 与 `signal-cli` 通信。

## 前提条件

- 在你的服务器上已安装 OpenClaw（以下 Linux 流程在 Ubuntu 24 上测试通过）。
- 在运行 gateway 的主机上可用 `signal-cli`。
- 一个可以接收一次验证短信的电话号码（用于短信注册流程）。
- 在注册期间可通过浏览器访问 Signal 验证码页面（`signalcaptchas.org`）。

## 快速设置（初学者）

1. 为 bot 使用**单独的 Signal 号码**（推荐）。
2. 安装 `signal-cli`（需要 Java）。
3. 选择以下其中一种设置路径：
   - `signal-cli link -n "OpenClaw"`
   - **路径 B（短信注册）：** 使用验证码 + 短信验证注册一个专用号码。
4. 配置 OpenClaw 并启动 Gateway 网关。
5. 发送第一条私信并批准配对（`openclaw pairing approve signal <CODE>`）。

最小配置：

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

字段说明：

| 字段          | 说明                                                  |
| ----------- | --------------------------------------------------- |
| `account`   | 机器人电话号码，采用 E.164 格式（`+15551234567`） |
| 设置（快速路径）    | 配对是 Signal 私信的默认令牌交换方式。详情：[配对](/channels/pairing)   |
| `dmPolicy`  | 私信访问策略（推荐使用 `pairing`）                              |
| `allowFrom` | 允许发送私信的电话号码或 `uuid:&lt;id&gt;` 值                          |

## 它是什么

- 通过 `signal-cli` 的 Signal 渠道（非嵌入式 libsignal）。
- 确定性路由：回复始终返回到 Signal。
- 私信共享智能体的主会话；群组是隔离的（`agent:<agentId>:signal:group:<groupId>`）。

## 配置写入

默认情况下，Signal 允许写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。

禁用方式：

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## 号码模型（重要）

- Gateway 网关连接到一个 **Signal 设备**（`signal-cli` 账户）。
- 如果你在**个人 Signal 账户**上运行 bot，它会忽略你自己的消息（循环保护）。
- 要实现"我发消息给 bot 然后它回复"，请使用**单独的 bot 号码**。

## 设置路径 A：关联现有 Signal 账户（QR）

1. 安装 `signal-cli`（需要 Java）。
2. 链接 bot 账户：
   - `signal-cli link -n "OpenClaw"` 然后在 Signal 中扫描二维码。
3. 配置 Signal 并启动 Gateway 网关。

示例：

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

多账户支持：使用 `channels.signal.accounts` 配置每个账户及可选的 `name`。共享模式请参见 [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)。 See [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) for the shared pattern.

## 设置路径 B：注册专用机器人号码（短信，Linux）

当你希望使用专用机器人号码而不是关联现有 Signal 应用账户时，请使用此方式。

1. 获取一个可以接收短信的号码（或用于座机的语音验证）。
   - 使用专用机器人号码以避免账户/会话冲突。
2. 在 gateway 主机上安装 `signal-cli`：

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

如果你使用 JVM 构建版本（`signal-cli-${VERSION}.tar.gz`），请先安装 JRE 25+。
请保持 `signal-cli` 为最新版本；上游说明旧版本可能会因 Signal 服务器 API 变更而失效。

3. 注册并验证该号码：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

如果需要验证码：

1. 打开 `https://signalcaptchas.org/registration/generate.html`。
2. 完成验证码验证，从“Open Signal”中复制 `signalcaptcha://...` 链接目标地址。
3. 尽可能使用与浏览器会话相同的外部 IP 运行。
4. 立即再次运行注册（验证码令牌很快会过期）：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. 这会跳过自动启动和 OpenClaw 内部的启动等待。对于自动启动时的慢启动，请设置 `channels.signal.startupTimeoutMs`。

```bash
# 如果你将 gateway 作为用户级 systemd 服务运行：
systemctl --user restart openclaw-gateway

# 然后验证：
openclaw doctor
openclaw channels status --probe
```

5. 配对你的 DM 发送者：
   - 向机器人号码发送任意消息。
   - 在服务器上批准代码：`openclaw pairing approve signal <PAIRING_CODE>`。
   - 将机器人号码保存为手机联系人，以避免显示为“Unknown contact”。

重要：使用 `signal-cli` 注册手机号账户可能会使该号码的主 Signal 应用会话失去认证。 建议使用专用的机器人号码，或者如果需要保留现有手机应用设置，请使用 QR 链接模式。

上游参考资料：

- `signal-cli` README：`https://github.com/AsamK/signal-cli`
- 验证码流程：`https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- 链接流程：`https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## 外部守护进程模式（httpUrl）

如果你想自己管理 `signal-cli`（JVM 冷启动慢、容器初始化或共享 CPU），请单独运行守护进程并将 OpenClaw 指向它：

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

This skips auto-spawn and the startup wait inside OpenClaw. For slow starts when auto-spawning, set `channels.signal.startupTimeoutMs`.

## 访问控制（私信 + 群组）

DMs:

- 默认：`channels.signal.dmPolicy = "pairing"`。
- 未知发送者会收到配对码；消息在批准前会被忽略（配对码 1 小时后过期）。
- 通过以下方式批准：
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Pairing is the default token exchange for Signal DMs. Details: [Pairing](/channels/pairing)
- 仅有 UUID 的发送者（来自 `sourceUuid`）在 `channels.signal.allowFrom` 中存储为 `uuid:<id>`。

群组：

- `channels.signal.groupPolicy = open | allowlist | disabled`。
- 当设置为 `allowlist` 时，`channels.signal.groupAllowFrom` 控制谁可以在群组中触发。

## How it works (behavior)

- `signal-cli` 作为守护进程运行；Gateway 网关通过 SSE 读取事件。
- 入站消息被规范化为共享渠道信封。
- 回复始终路由回同一号码或群组。

## 媒体 + 限制

- 出站文本按 `channels.signal.textChunkLimit` 分块（默认 4000）。
- 可选换行分块：设置 `channels.signal.chunkMode="newline"` 在长度分块前按空行（段落边界）分割。
- 支持附件（从 `signal-cli` 获取 base64）。
- 默认媒体上限：`channels.signal.mediaMaxMb`（默认 8）。
- 使用 `channels.signal.ignoreAttachments` 跳过下载媒体。
- 群组历史上下文使用 `channels.signal.historyLimit`（或 `channels.signal.accounts.*.historyLimit`），回退到 `messages.groupChat.historyLimit`。设置 `0` 禁用（默认 50）。 Set `0` to disable (default 50).

## 输入指示器 + 已读回执

- **输入指示器**：OpenClaw 通过 `signal-cli sendTyping` 发送输入信号，并在回复运行时刷新它们。
- **已读回执**：当 `channels.signal.sendReadReceipts` 为 true 时，OpenClaw 为允许的私信转发已读回执。
- Signal-cli 不暴露群组的已读回执。

## 表情回应（message 工具）

- 使用 `message action=react` 配合 `channel=signal`。
- 目标：发送者 E.164 或 UUID（使用配对输出中的 `uuid:<id>`；裸 UUID 也可以）。
- `messageId` 是你要回应的消息的 Signal 时间戳。
- 群组表情回应需要 `targetAuthor` 或 `targetAuthorUuid`。

示例：

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

配置：

- `channels.signal.actions.reactions`：启用/禁用表情回应操作（默认 true）。
- `channels.signal.reactionLevel`：`off | ack | minimal | extensive`。
  - `off`/`ack` 禁用智能体表情回应（message 工具 `react` 会报错）。
  - `minimal`/`extensive` 启用智能体表情回应并设置指导级别。
- 每账户覆盖：`channels.signal.accounts.<id>.actions.reactions`、`channels.signal.accounts.<id>.reactionLevel`。

## 投递目标（CLI/cron）

- 私信：`signal:+15551234567`（或纯 E.164）。
- UUID 私信：`uuid:<id>`（或裸 UUID）。
- 群组：`signal:group:<groupId>`。
- 用户名：`username:<name>`（如果你的 Signal 账户支持）。

## Troubleshooting

Run this ladder first:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Then confirm DM pairing state if needed:

```bash
openclaw pairing list signal
```

Common failures:

- Daemon reachable but no replies: verify account/daemon settings (`httpUrl`, `account`) and receive mode.
- DMs ignored: sender is pending pairing approval.
- 群组消息被忽略：群组发送者/提及门控阻止了消息投递。
- 编辑后出现配置校验错误：运行 `openclaw doctor --fix`。
- 诊断中未显示 Signal：确认 `channels.signal.enabled: true`。

额外检查：

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

For triage flow: [/channels/troubleshooting](/channels/troubleshooting).

## 安全说明

- `signal-cli` 会在本地存储账户密钥（通常位于 `~/.local/share/signal-cli/data/`）。
- 在服务器迁移或重建前，请备份 Signal 账户状态。
- `channels.signal.allowFrom`：私信允许列表（E.164 或 `uuid:<id>`）。`open` 需要 `"*"`。Signal 没有用户名；使用电话/UUID id。
- SMS 验证仅在注册或恢复流程中需要，但如果失去对该号码/账户的控制，可能会使重新注册变得复杂。

## 配置参考（Signal）

完整配置：[配置](/gateway/configuration)

提供商选项：

- `channels.signal.enabled`：启用/禁用渠道启动。
- `channels.signal.account`：bot 账户的 E.164。
- `channels.signal.cliPath`：`signal-cli` 的路径。
- `channels.signal.httpUrl`：完整守护进程 URL（覆盖 host/port）。
- `channels.signal.httpHost`、`channels.signal.httpPort`：守护进程绑定（默认 127.0.0.1:8080）。
- `channels.signal.autoStart`：自动启动守护进程（如果未设置 `httpUrl` 则默认 true）。
- `channels.signal.startupTimeoutMs`：启动等待超时（毫秒）（上限 120000）。
- `channels.signal.receiveMode`：`on-start | manual`。
- `channels.signal.ignoreAttachments`：跳过附件下载。
- `channels.signal.ignoreStories`: ignore stories from the daemon.
- `channels.signal.sendReadReceipts`：转发已读回执。
- `channels.signal.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）。
- `channels.signal.allowFrom`: DM allowlist (E.164 or `uuid:<id>`). `open` requires `"*"`. Signal has no usernames; use phone/UUID ids.
- `channels.signal.groupPolicy`：`open | allowlist | disabled`（默认：allowlist）。
- `channels.signal.groupAllowFrom`：群组发送者允许列表。
- `channels.signal.historyLimit`：作为上下文包含的最大群组消息数（0 禁用）。
- `channels.signal.dmHistoryLimit`：以用户轮次计的私信历史上限。 `channels.signal.dmHistoryLimit`：私信历史限制（用户轮次）。每用户覆盖：`channels.signal.dms["<phone_or_uuid>"].historyLimit`。
- `channels.signal.textChunkLimit`：出站分块大小（字符）。
- `channels.signal.chunkMode`：`length`（默认）或 `newline` 在长度分块前按空行（段落边界）分割。
- `channels.signal.mediaMaxMb`：入站/出站媒体上限（MB）。

相关全局选项：

- `agents.list[].groupChat.mentionPatterns`（Signal 不支持原生提及）。
- `messages.groupChat.mentionPatterns`（全局回退）。
- `messages.responsePrefix`。


---
summary: "用于唤醒和隔离智能体运行的 Webhook 入口"
read_when:
  - 添加或更改 webhook 端点
  - 将外部系统接入 OpenClaw
title: "Webhooks"
---

# Webhooks

Gateway 网关可以暴露一个小型 HTTP webhook 端点用于外部触发。

## 启用

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

注意事项：

- 当 `hooks.enabled=true` 时，`hooks.token` 为必填项。
- `hooks.path` 默认为 `/hooks`。

## 认证

每个请求必须包含 hook 令牌。推荐使用请求头： Prefer headers:

- `Authorization: Bearer <token>`（推荐）
- `x-openclaw-token: <token>`
- 查询字符串中的 token 会被拒绝（`?token=...` 返回 `400`）。

## 端点

### `POST /hooks/wake`

Payload:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **必填**（字符串）：事件描述（例如"收到新邮件"）。
- `mode` 可选（`now` | `next-heartbeat`）：是否立即触发心跳（默认 `now`）或等待下一次定期检查。

效果：

- 为**主**会话加入一个系统事件队列
- 如果 `mode=now`，则立即触发心跳

### `POST /hooks/agent`

Payload:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **必填**（字符串）：智能体要处理的提示或消息。
- `name` 可选（字符串）：hook 的可读名称（例如"GitHub"），用作会话摘要的前缀。
- `agentId` 可选（字符串）：将此 hook 路由到特定代理。 未知 ID 将回退到默认代理。 设置后，hook 将使用解析后的代理 workspace 和配置运行。
- `sessionKey` optional (string): The key used to identify the agent's session. 默认情况下，除非设置 `hooks.allowRequestSessionKey=true`，否则此字段会被拒绝。
- `wakeMode` 可选（`now` | `next-heartbeat`）：是否立即触发心跳（默认 `now`）或等待下一次定期检查。
- `deliver` 可选（布尔值）：如果为 `true`，智能体的响应将发送到消息渠道。默认为 `true`。仅为心跳确认的响应会自动跳过。 Defaults to `true`. Responses that are only heartbeat acknowledgments are automatically skipped.
- `channel` optional (string): The messaging channel for delivery. `channel` 可选（字符串）：用于投递的消息渠道。可选值：`last`、`whatsapp`、`telegram`、`discord`、`slack`、`mattermost`（插件）、`signal`、`imessage`、`msteams`。默认为 `last`。 Defaults to `last`.
- `to` 可选（字符串）：渠道的接收者标识符（例如 WhatsApp/Signal 的电话号码、Telegram 的聊天 ID、Discord/Slack/Mattermost（插件）的频道 ID、MS Teams 的会话 ID）。默认为主会话中的最后一个接收者。 Defaults to the last recipient in the main session.
- `model` 可选（字符串）：模型覆盖（例如 `anthropic/claude-3-5-sonnet` 或别名）。如果有限制，必须在允许的模型列表中。 Must be in the allowed model list if restricted.
- `thinking` 可选（字符串）：思考级别覆盖（例如 `low`、`medium`、`high`）。
- `timeoutSeconds` 可选（数字）：智能体运行的最大持续时间（秒）。

效果：

- 运行一个**隔离的**智能体回合（独立的会话键）
- 始终在**主**会话中发布摘要
- 如果 `wakeMode=now`，则立即触发心跳

## Session key 策略（破坏性变更）

`/hooks/agent` 负载中的 `sessionKey` 覆盖默认已禁用。

- 推荐：设置固定的 `hooks.defaultSessionKey`，并关闭请求覆盖。
- 可选：仅在必要时允许请求覆盖，并限制前缀。

推荐配置：

```json5
`sessionKey` 可选（字符串）：用于标识智能体会话的键。默认为随机的 `hook:<uuid>`。使用一致的键可以在 hook 上下文中进行多轮对话。
```

兼容性配置（旧版行为）：

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // 强烈建议
  },
}
```

### `POST /hooks/<name>`（映射）

Custom hook names are resolved via `hooks.mappings` (see configuration). A mapping can
turn arbitrary payloads into `wake` or `agent` actions, with optional templates or
code transforms.

映射选项（摘要）：

- `hooks.presets: ["gmail"]` 启用内置的 Gmail 映射。
- `hooks.mappings` 允许你在配置中定义 `match`、`action` 和模板。
- `hooks.transformsDir` + `transform.module` 加载 JS/TS 模块用于自定义逻辑。
  - 如果设置了 `hooks.transformsDir`，则必须位于 OpenClaw 配置目录下的 transforms 根目录内（通常为 `~/.openclaw/hooks/transforms`）。
  - `transform.module` 必须在有效的 transforms 目录内解析（拒绝路径遍历/逃逸路径）。
- 使用 `match.source` 保持通用的接收端点（基于请求体的路由）。
- TS 转换需要 TS 加载器（例如 `bun` 或 `tsx`）或运行时预编译的 `.js`。
- 在映射上设置 `deliver: true` + `channel`/`to` 可将回复路由到聊天界面（`channel` 默认为 `last`，回退到 WhatsApp）。
- `agentId` 会将 hook 路由到特定代理；未知 ID 将回退到默认代理。
- `hooks.allowedAgentIds` 限制显式的 `agentId` 路由。 省略该项（或包含 `*`）以允许任意代理。 设置为 `[]` 以拒绝显式的 `agentId` 路由。
- `hooks.defaultSessionKey` 在未提供显式 key 时，为 hook 代理运行设置默认会话。
- `hooks.allowRequestSessionKey` 控制 `/hooks/agent` 负载是否可以设置 `sessionKey`（默认：`false`）。
- `hooks.allowedSessionKeyPrefixes` 可选地限制来自请求负载和映射的显式 `sessionKey` 值。
- `allowUnsafeExternalContent: true` 禁用该 hook 的外部内容安全包装（危险；仅用于受信任的内部来源）。
- `openclaw webhooks gmail setup` 为 `openclaw webhooks gmail run` 写入 `hooks.gmail` 配置。完整的 Gmail 监听流程请参阅 [Gmail Pub/Sub](/automation/gmail-pubsub)。
  See [Gmail Pub/Sub](/automation/gmail-pubsub) for the full Gmail watch flow.

## 响应

- `200` 用于 `/hooks/wake`
- `202` 用于 `/hooks/agent`（异步运行已启动）
- `401` 认证失败
- 同一客户端在多次身份验证失败后返回 `429`（检查 `Retry-After`）。
- `400` 请求体无效
- `413` 请求体过大

## 示例

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### 使用不同的模型

在智能体请求体（或映射）中添加 `model` 以覆盖该次运行的模型：

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

如果你启用了 `agents.defaults.models` 限制，请确保覆盖的模型包含在其中。

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## 安全

- 将 hook 端点保持在 loopback、tailnet 或受信任的反向代理之后。
- 使用专用的 hook 令牌；不要复用 Gateway 网关认证令牌。
- 为减缓暴力破解尝试，针对同一客户端地址的重复身份验证失败将被限流。
- 如果使用多代理路由，请设置 `hooks.allowedAgentIds` 以限制显式的 `agentId` 选择。
- 除非需要由调用方选择会话，否则保持 `hooks.allowRequestSessionKey=false`。
- 如果启用请求 `sessionKey`，请限制 `hooks.allowedSessionKeyPrefixes`（例如，`["hook:"]`）。
- 避免在 webhook 日志中包含敏感的原始请求体。
- Hook payloads are treated as untrusted and wrapped with safety boundaries by default.
  Hook 请求体默认被视为不受信任并使用安全边界包装。如果你必须为特定 hook 禁用此功能，请在该 hook 的映射中设置 `allowUnsafeExternalContent: true`（危险）。

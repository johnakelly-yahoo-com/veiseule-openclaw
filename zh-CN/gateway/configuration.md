---
summary: "配置概览：常见任务、快速设置以及完整参考文档的链接"
read_when:
  - 首次设置 OpenClaw
  - 查找常见的配置模式
  - 导航到特定的配置部分
title: "Configuration"
---

# 配置

OpenClaw 从 `~/.openclaw/openclaw.json` 读取一个可选的 <Tooltip tip="JSON5 supports comments and trailing commas">**JSON5**</Tooltip> 配置文件。

如果该文件不存在，OpenClaw 将使用安全的默认设置。 添加配置的常见原因：

- 连接各个渠道并控制谁可以向机器人发送消息
- 设置模型、工具、沙箱或自动化（cron、hooks）
- 调整会话、媒体、网络或 UI

有关所有可用字段，请参阅[完整参考](/gateway/configuration-reference)。

<Tip>
**刚接触配置？** 可以从 `openclaw onboard` 开始进行交互式设置，或查看 [Configuration Examples](/gateway/configuration-examples) 指南获取完整的可复制粘贴配置。
</Tip>

## 最小配置

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## 编辑配置

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # 完整设置向导
    openclaw configure     # 配置向导
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    打开 [http://127.0.0.1:18789](http://127.0.0.1:18789) 并使用 **Config** 选项卡。
    Control UI 会根据配置 schema 渲染表单，并提供一个 **Raw JSON** 编辑器作为兜底选项。
  
</Tab>
  <Tab title="Direct edit">
    直接编辑 `~/.openclaw/openclaw.json`。 Gateway 会监听该文件并自动应用更改（参见 [hot reload](#config-hot-reload)）。
  
</Tab>
</Tabs>

## 严格校验

<Warning>
OpenClaw only accepts configurations that fully match the schema. 未知键、类型格式错误或无效值会导致 Gateway **拒绝启动**。 唯一的根级例外是 `$schema`（字符串），以便编辑器附加 JSON Schema 元数据。
</Warning>

When validation fails:

- Gateway 无法启动
- 仅诊断命令可用（`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`）
- 运行 `openclaw doctor` 查看具体问题
- 运行 `openclaw doctor --fix`（或 `--yes`）以应用修复

## 常见任务

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    每个 channel 在 `channels.` 下都有各自的配置部分。<provider>`. 请参阅对应 channel 页面获取设置步骤：

    ````
    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`
    
    所有 channel 共享相同的 DM 策略模式：
    
    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // 仅用于 allowlist/open
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Choose and configure models">
    设置主模型以及可选的备用模型：

    ````
    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```
    
    - `agents.defaults.models` 定义模型目录，并作为 `/model` 的 allowlist。
    - 模型引用使用 `provider/model` 格式（例如 `anthropic/claude-opus-4-6`）。
    - 有关在聊天中切换模型，请参见 [Models CLI](/concepts/models)；有关认证轮换和回退行为，请参见 [Model Failover](/concepts/model-failover)。
    - 对于自定义/自托管 provider，请参见参考文档中的 [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls)。
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    通过 `dmPolicy` 按 channel 控制 DM 访问：

    ```
    - `"pairing"`（默认）：未知发送者会收到一次性配对代码以供批准
    - `"allowlist"`：仅允许 `allowFrom`（或已配对的允许列表存储）中的发送者
    - `"open"`：允许所有传入 DM（需要设置 `allowFrom: ["*"]`）
    - `"disabled"`：忽略所有 DM
    
    对于群组，请使用 `groupPolicy` + `groupAllowFrom` 或 channel 特定的 allowlist。
    
    有关各 channel 的详细信息，请参见 [full reference](/gateway/configuration-reference#dm-and-group-access)。
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    群组消息默认设置为 **需要提及**。 为每个 agent 配置模式：

    ````
    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```
    
    - **元数据提及**：原生 @ 提及（WhatsApp 点按提及、Telegram @bot 等）
    - **文本模式**：`mentionPatterns` 中的正则表达式模式
    - 有关各 channel 覆盖设置和自聊模式，请参见 [full reference](/gateway/configuration-reference#group-chat-mention-gating)。
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    会话控制对话的连续性和隔离性：

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // 推荐用于多用户
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`：`main`（共享）| `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - 有关作用域、身份关联和发送策略，请参见 [Session Management](/concepts/session)。
    - 有关所有字段，请参见 [full reference](/gateway/configuration-reference#session)。
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    在隔离的 Docker 容器中运行 agent 会话：

    ````
    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```
    
    请先构建镜像：`scripts/sandbox-setup.sh`
    
    完整指南请参见 [Sandboxing](/gateway/sandboxing)，所有选项请参见 [full reference](/gateway/configuration-reference#sandbox)。
    ````

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```


    ```
    - `every`：时长字符串（`30m`、`2h`）。设置为 `0m` 可禁用。
    - `target`：`last` | `whatsapp` | `telegram` | `discord` | `none`
    - 完整指南请参见 [Heartbeat](/gateway/heartbeat)。
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```


    ```
    有关功能概览和 CLI 示例，请参阅 [Cron jobs](/automation/cron-jobs)。
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">
    在 Gateway 上启用 HTTP webhook 端点：

    ````
    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```
    
    有关所有映射选项和 Gmail 集成，请参见 [full reference](/gateway/configuration-reference#hooks)。
    ````

  
</Accordion>

  <Accordion title="Configure multi-agent routing">
    运行多个相互隔离的 agent，分别使用独立的工作区和会话：

    ````
    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```
    
    有关绑定规则和每个 agent 的访问配置文件，请参见 [Multi-Agent](/concepts/multi-agent) 和 [full reference](/gateway/configuration-reference#multi-agent-routing)。
    ````

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">
    使用 `$include` 来组织大型配置：

    ````
    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```
    
    - **单个文件**：替换包含它的对象
    - **文件数组**：按顺序进行深度合并（后者优先生效）
    - **同级键**：在 include 之后合并（覆盖已包含的值）
    - **嵌套 include**：最多支持 10 层嵌套
    - **相对路径**：相对于包含它的文件进行解析
    - **错误处理**：对缺失文件、解析错误和循环 include 提供清晰的错误提示
    ````

  
</Accordion>
</AccordionGroup>

## 配置热重载

Gateway 会监视 `~/.openclaw/openclaw.json` 并自动应用更改——大多数设置无需手动重启。

### 重新加载模式

| 模式               | 行为                             |
| ---------------- | ------------------------------ |
| **`hybrid`**（默认） | 即时热应用安全的更改。 对关键更改自动重启。         |
| **`hot`**        | 仅热应用安全的更改。 当需要重启时记录警告——由你自行处理。 |
| **`restart`**    | 在任何配置更改时重启 Gateway，无论是否安全。     |
| **`off`**        | 禁用文件监视。 更改将在下次手动重启时生效。         |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### 哪些可以热应用，哪些需要重启

大多数字段可在无停机的情况下热应用。 在 `hybrid` 模式下，需要重启的更改会自动处理。

| 类别          | 字段                                             | 需要重启？ |
| ----------- | ---------------------------------------------- | ----- |
| 通道          | `channels.*`, `web`（WhatsApp）——所有内置和扩展通道       | 否     |
| Agent 与模型   | `agent`, `agents`, `models`, `routing`         | 否     |
| 自动化         | `hooks`, `cron`, `agent.heartbeat`             | 否     |
| 会话与消息       | `session`, `messages`                          | 否     |
| 工具与媒体       | `tools`, `browser`, `skills`, `audio`, `talk`  | 否     |
| 界面与其他       | `ui`, `logging`, `identity`, `bindings`        | 否     |
| Gateway 服务器 | `gateway.*`（port、bind、auth、tailscale、TLS、HTTP） | **是** |
| 基础设施        | `discovery`、`canvasHost`、`plugins`             | **是** |

<Note>
`gateway.reload` 和 `gateway.remote` 是例外——更改它们**不会**触发重启。
</Note>

## 配置 RPC（以编程方式更新）

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    在一步操作中完成校验 + 写入完整配置并重启 Gateway。


    ````
    <Warning>
    `config.apply` 会替换**整个配置**。部分更新请使用 `config.patch`，或使用 `openclaw config set` 设置单个键。
    
</Warning>
    
    参数：
    
    - `raw`（string）— 整个配置的 JSON5 负载
    - `baseHash`（可选）— 来自 `config.get` 的配置哈希（当配置已存在时为必需）
    - `sessionKey`（可选）— 重启后唤醒 ping 的会话键
    - `note`（可选）— 重启哨兵的备注
    - `restartDelayMs`（可选）— 重启前的延迟时间（默认 2000）
    
    ```bash
    openclaw gateway call config.get --params '{}'  # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    将部分更新合并到现有配置中（JSON merge patch 语义）：

    ````
    - 对象递归合并
    - `null` 删除键
    - 数组整体替换
    
    参数：
    
    - `raw`（string）— 仅包含需要更改键的 JSON5
    - `baseHash`（必需）— 来自 `config.get` 的配置哈希
    - `sessionKey`、`note`、`restartDelayMs` — 与 `config.apply` 相同
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## 环境变量

OpenClaw 从父进程读取环境变量，此外还包括：

- `.env` from the current working directory (if present)
- `~/.openclaw/.env`（全局回退）

两个文件都不会覆盖已存在的环境变量。 你也可以在配置中内联设置环境变量：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  如果启用且预期的键未设置，OpenClaw 会运行你的登录 shell，并仅导入缺失的键：

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

等效的环境变量：`OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  在任何配置字符串值中使用 `${VAR_NAME}` 引用环境变量：

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

规则：

- 仅匹配大写名称：`[A-Z_][A-Z0-9_]*`
- 缺失或为空的变量会在加载时抛出错误
- 使用 `$${VAR}` 进行转义以输出字面量
- 在 `$include` 文件中同样生效
- 内联替换：`"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

完整的优先级和来源请参阅 [Environment](/help/environment)。

## 完整参考

有关逐字段的完整参考，请参阅 **[Configuration Reference](/gateway/configuration-reference)**。

---

_相关： [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_

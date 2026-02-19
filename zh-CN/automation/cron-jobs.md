---
summary: "Gateway网关调度器的定时任务与唤醒"
read_when:
  - 调度后台任务或唤醒
  - 配置需要与心跳一起或并行运行的自动化
  - 在心跳和定时任务之间做选择
title: "Cron 作业"
---

# 定时任务（Gateway网关调度器）

> **定时任务还是心跳？** 请参阅[定时任务与心跳对比](/automation/cron-vs-heartbeat)了解何时使用哪种方式。

Cron 是 Gateway 的内置调度器。 它会持久化作业，在合适的时间唤醒代理，并且可选地将输出回传到聊天中。

如果你想要 _"每天早上运行"_ 或 _"20 分钟后提醒智能体"_，定时任务就是对应的机制。

故障排查：[/automation/troubleshooting](/automation/troubleshooting)

## TL;DR

- 定时任务运行在 **Gateway网关内部**（而非模型内部）。
- 任务持久化存储在 `~/.openclaw/cron/` 下，因此重启不会丢失计划。
- 两种执行方式：
  - **主会话**：入队一个系统事件，然后在下一次心跳时运行。
  - **隔离式**：在 `cron:<jobId>` 中运行专用智能体轮次，可投递摘要（默认 announce）或不投递。
- 唤醒是一等功能：任务可以请求"立即唤醒"或"下次心跳时"。

## 快速开始（可操作）

创建一个一次性提醒，验证其存在，然后立即运行：

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id> --force
openclaw cron runs --id <job-id>
```

调度一个带投递功能的周期性隔离任务：

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## 工具调用等价形式（Gateway网关定时任务工具）

有关规范的 JSON 结构和示例，请参阅[工具调用的 JSON 模式](/automation/cron-jobs#json-schema-for-tool-calls)。

## 定时任务的存储位置

定时任务默认持久化存储在 Gateway网关主机的 `~/.openclaw/cron/jobs.json` 中。Gateway网关将文件加载到内存中，并在更改时写回，因此仅在 Gateway网关停止时手动编辑才是安全的。请优先使用 `openclaw cron add/edit` 或定时任务工具调用 API 进行更改。
Gateway 会将该文件加载到内存中，并在发生更改时写回，因此只有在 Gateway 停止时手动编辑才是安全的。 5. 更改时优先使用 `openclaw cron add/edit` 或 cron
工具调用 API。

## 新手友好概述

将定时任务理解为：**何时**运行 + **做什么**。

1. **选择调度计划**
   - 一次性提醒 → `schedule.kind = "at"`（CLI：`--at`）
   - 重复任务 → `schedule.kind = "every"` 或 `schedule.kind = "cron"`
   - 如果你的 ISO 时间戳省略了时区，将被视为 **UTC**。

2. **选择运行位置**
   - `sessionTarget: "main"` → 在下一次心跳时使用主会话上下文运行。
   - `sessionTarget: "isolated"` → 在 `cron:<jobId>` 中运行专用智能体轮次。

3. **选择负载**
   - 主会话 → `payload.kind = "systemEvent"`
   - 隔离会话 → `payload.kind = "agentTurn"`

可选：一次性任务（`schedule.kind = "at"`）默认会在成功运行后删除。设置
`deleteAfterRun: false` 可保留它（成功后会禁用）。 19. 设置
`deleteAfterRun: false` 以保留它们（成功后将被禁用）。

## 概念

### 21. 作业

定时任务是一条存储记录，包含：

- 一个**调度计划**（何时运行），
- 一个**负载**（做什么），
- 可选的**投递**（输出发送到哪里）。
- 26. 可选的 **代理绑定**（`agentId`）：在特定代理下运行作业；如果
      缺失或未知，Gateway 将回退到默认代理。

27. 作业通过稳定的 `jobId` 标识（由 CLI/Gateway API 使用）。
28. 在代理工具调用中，`jobId` 是规范字段；为兼容性也接受旧的 `id`。
29. 一次性作业默认在成功后自动删除；设置 `deleteAfterRun: false` 可保留它们。

### 调度计划

定时任务支持三种调度类型：

- `at`：一次性时间戳（ISO 8601 字符串）。
- `every`：固定间隔（毫秒）。
- `cron`：5 字段 cron 表达式，可选 IANA 时区。

Cron 表达式使用 `croner`。如果省略时区，将使用 Gateway网关主机的本地时区。 36. 如果省略时区，则使用 Gateway 主机的
本地时区。

### 主会话与隔离式执行

#### 主会话任务（系统事件）

39. 主作业会入队一个系统事件，并可选择唤醒心跳运行器。
40. 它们必须使用 `payload.kind = "systemEvent"`。

- `wakeMode: "now"`：事件触发立即心跳运行。
- `wakeMode: "next-heartbeat"`（默认）：事件等待下一次计划心跳。

43. 当你需要正常的心跳提示 + 主会话上下文时，这是最佳选择。
44. 参见 [Heartbeat](/gateway/heartbeat)。

#### 隔离任务（专用定时会话）

隔离任务在会话 `cron:<jobId>` 中运行专用智能体轮次。

关键行为：

- 提示以 `[cron:<jobId> <任务名称>]` 为前缀，便于追踪。
- 每次运行都会启动一个**全新的会话 ID**（不继承之前的对话）。
- 任务通过稳定的 `jobId` 标识（用于 CLI/Gateway网关 API）。
  在智能体工具调用中，`jobId` 是规范字段；旧版 `id` 仍可兼容使用。
  一次性任务默认会在成功运行后自动删除；设置 `deleteAfterRun: false` 可保留它。
- 2. `announce`：将摘要投递到目标频道，并在主会话中发布一条简要摘要。
  - 3. `none`：仅内部使用（不投递，也不发布主会话摘要）。
  - 4. `wakeMode` 控制主会话摘要何时发布：
- 5. `now`：立即心跳。
  - 6. `next-heartbeat`：等待下一个计划的心跳。
  - 当你需要正常的心跳提示 + 主会话上下文时，这是最佳选择。参见[心跳](/gateway/heartbeat)。

对于嘈杂、频繁或"后台杂务"类任务，使用隔离任务可以避免污染你的主聊天记录。

### 负载结构（运行内容）

支持两种负载类型：

- `systemEvent`：仅限主会话，通过心跳提示路由。
- `agentTurn`：仅限隔离会话，运行专用智能体轮次。

常用 `agentTurn` 字段：

- `message`：必填文本提示。
- `model` / `thinking`：可选覆盖（见下文）。
- `timeoutSeconds`：可选超时覆盖。

17. `delivery.mode`：`none` | `announce`。

- `delivery.mode` 可选 `announce`（投递摘要）或 `none`（内部运行）。
- 当启用 announce 投递时，该轮次会抑制消息工具发送；请使用 `delivery.channel`/`delivery.to` 来指定目标。
- Telegram 通过 `message_thread_id` 支持论坛主题。对于定时任务投递，你可以将主题/帖子编码到 `to` 字段中：
- 21. 公告投递会抑制该运行中的消息工具发送；请使用 `delivery.channel`/`delivery.to`
      将消息定向到聊天。

22. 当 `delivery.mode = "none"` 时，不会向主会话发布摘要。 23. 如果隔离作业省略了 `delivery`，OpenClaw 默认使用 `announce`。

如果未指定 `delivery`，隔离任务会默认以“announce”方式投递摘要。

#### 25. 当 `delivery.mode = "announce"` 时，cron 会通过出站频道适配器直接投递。

26. 不会启动主代理来编写或转发消息。
27. 行为细节：

简要概述

- 29. 仅心跳响应（没有实际内容的 `HEARTBEAT_OK`）不会被投递。
- 30. 如果隔离运行已通过消息工具向同一目标发送过消息，为避免重复，将跳过投递。
- 31. 缺失或无效的投递目标会使作业失败，除非 `delivery.bestEffort = true`。
- `delivery.bestEffort`：投递失败时避免任务失败
- 33. 主会话摘要遵循 `wakeMode`：`now` 会触发立即心跳，
      `next-heartbeat` 则等待下一个计划的心跳。
- 主会话任务入队一个系统事件，并可选择唤醒心跳运行器。它们必须使用 `payload.kind = "systemEvent"`。

### 模型和思维覆盖

隔离任务（`agentTurn`）可以覆盖模型和思维级别：

- `model`：提供商/模型字符串（例如 `anthropic/claude-sonnet-4-20250514`）或别名（例如 `opus`）
- `thinking`：思维级别（`off`、`minimal`、`low`、`medium`、`high`、`xhigh`；仅限 GPT-5.2 + Codex 模型）

注意：你也可以在主会话任务上设置 `model`，但这会更改共享的主会话模型。我们建议仅对隔离任务使用模型覆盖，以避免意外的上下文切换。 40. 解析优先级：

优先级解析顺序：

1. 任务负载覆盖（最高优先级）
2. 钩子特定默认值（例如 `hooks.gmail.model`）
3. 44. 投递（频道 + 目标）

### 投递（渠道 + 目标）

隔离任务可以通过顶层 `delivery` 配置投递输出：

- `delivery.mode`：`announce`（投递摘要）或 `none`
- `delivery.channel`：`whatsapp` / `telegram` / `discord` / `slack` / `mattermost`（插件）/ `signal` / `imessage` / `last`
- `delivery.to`：渠道特定的接收目标

50. 如果省略 `delivery.channel` 或 `delivery.to`，cron 可以回退到主会话的
    “最后路由”（代理最后一次回复的位置）。

如果省略 `delivery.channel` 或 `delivery.to`，定时任务会回退到主会话的“最后路由”（智能体最后回复的位置）。

目标格式提醒：

- Slack/Discord/Mattermost（插件）目标应使用明确前缀（例如 `channel:<id>`、`user:<id>`）以避免歧义。
- Telegram 主题应使用 `:topic:` 格式（见下文）。

#### Telegram 投递目标（主题/论坛帖子）

5. Telegram 通过 `message_thread_id` 支持论坛主题。 6. 对于 cron 投递，你可以将主题/线程编码到 `to` 字段中：

- `-1001234567890`（仅聊天 ID）
- `-1001234567890:topic:123`（推荐：明确的主题标记）
- `-1001234567890:123`（简写：数字后缀）

带前缀的目标如 `telegram:...` / `telegram:group:...` 也可接受：

- `telegram:group:-1001234567890:topic:123`

## 工具调用的 JSON 模式

13. 直接调用 Gateway `cron.*` 工具时（代理工具调用或 RPC）请使用以下结构。
    直接调用 Gateway网关 `cron.*` 工具（智能体工具调用或 RPC）时使用这些结构。CLI 标志接受人类可读的时间格式如 `20m`，但工具调用应使用 ISO 8601 字符串作为 `schedule.at`，并使用毫秒作为 `schedule.everyMs`。

### cron.add 参数

一次性主会话任务（系统事件）：

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

带投递的周期性隔离任务：

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates."
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

说明：

- `schedule.kind`：`at`（`at`）、`every`（`everyMs`）或 `cron`（`expr`，可选 `tz`）。
- `schedule.at` 接受 ISO 8601（可省略时区；省略时按 UTC 处理）。
- `everyMs` 为毫秒数。
- `sessionTarget` 必须为 `"main"` 或 `"isolated"`，且必须与 `payload.kind` 匹配。
- 可选字段：`agentId`、`description`、`enabled`、`deleteAfterRun`、`delivery`。
- `wakeMode` 省略时默认为 `"next-heartbeat"`。

### cron.update 参数

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

说明：

- `jobId` 是规范字段；`id` 可兼容使用。
- 在补丁中使用 `agentId: null` 可清除智能体绑定。

### cron.run 和 cron.remove 参数

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## 存储与历史

- 任务存储：`~/.openclaw/cron/jobs.json`（Gateway网关管理的 JSON）。
- 运行历史：`~/.openclaw/cron/runs/<jobId>.jsonl`（JSONL，自动清理）。
- 覆盖存储路径：配置中的 `cron.store`。

## 配置

```json5
{
  cron: {
    enabled: true, // 默认 true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // 默认 1
  },
}
```

完全禁用定时任务：

- `cron.enabled: false`（配置）
- `OPENCLAW_SKIP_CRON=1`（环境变量）

## CLI 快速开始

一次性提醒（UTC ISO，成功后自动删除）：

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

一次性提醒（主会话，立即唤醒）：

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

周期性隔离任务（投递到 WhatsApp）：

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

周期性隔离任务（投递到 Telegram 主题）：

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

带模型和思维覆盖的隔离任务：

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Agent selection (multi-agent setups):

```bash
# 将任务绑定到智能体 "ops"（如果该智能体不存在则回退到默认智能体）
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# 切换或清除现有任务的智能体
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

Manual run (force is the default, use `--due` to only run when due):

```bash
openclaw cron run <jobId> --force
```

编辑现有任务（补丁字段）：

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

运行历史：

```bash
openclaw cron runs --id <jobId> --limit 50
```

不创建任务直接发送系统事件：

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

## Gateway网关 API 接口

- `cron.list`、`cron.status`、`cron.add`、`cron.update`、`cron.remove`
- `cron.run`（强制或到期）、`cron.runs`
  如需不创建任务直接发送系统事件，请使用 [`openclaw system event`](/cli/system)。

## 故障排除

### "没有任何任务运行"

- 检查定时任务是否已启用：`cron.enabled` 和 `OPENCLAW_SKIP_CRON`。
- 检查 Gateway网关是否持续运行（定时任务运行在 Gateway网关进程内部）。
- 对于 `cron` 调度：确认时区（`--tz`）与主机时区的关系。

### A recurring job keeps delaying after failures

- OpenClaw applies exponential retry backoff for recurring jobs after consecutive errors:
  30s, 1m, 5m, 15m, then 60m between retries.
- Backoff resets automatically after the next successful run.
- One-shot (`at`) jobs disable after a terminal run (`ok`, `error`, or `skipped`) and do not retry.

### Telegram 投递到了错误的位置

- 对于论坛主题，使用 `-100…:topic:<id>` 以确保明确无歧义。
- 如果你在日志或存储的"最后路由"目标中看到 `telegram:...` 前缀，这是正常的；定时任务投递接受这些前缀并仍能正确解析主题 ID。


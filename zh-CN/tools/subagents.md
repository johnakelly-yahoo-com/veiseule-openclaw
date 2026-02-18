---
title: 子智能体
x-i18n:
  generated_at: "2026-02-03T10:12:07Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 3c83eeed69a65dbbb6b21a386f3ac363d3ef8f077f0e03b834c3f0a9911dca7c
  source_path: tools/subagents.md
  workflow: 15
---

# 子智能体

子智能体是从现有智能体运行中生成的后台智能体运行。它们在自己的会话中运行（`agent:<agentId>:subagent:<uuid>`），完成后将结果**通告**回请求者的聊天渠道。

## 斜杠命令

使用 `/subagents` 检查或控制**当前会话**的子智能体运行：

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` 显示运行元数据（状态、时间戳、会话 id、转录路径、清理）。

主要目标：

- 并行化“研究 / 长任务 / 慢工具”工作，而不阻塞主运行。
- 默认保持子智能体隔离（会话分离 + 可选沙箱隔离）。
- 保持工具接口难以滥用：子智能体默认**不**获得会话工具。
- 支持可配置的嵌套深度，用于编排器模式。

成本说明：每个子智能体都有**自己的**上下文和 token 使用量。对于繁重或重复的任务，为子智能体设置更便宜的模型，而让主智能体使用更高质量的模型。你可以通过 `agents.defaults.subagents.model` 或每智能体覆盖来配置。

## 工具

使用 `sessions_spawn`：

- 启动子智能体运行（`deliver: false`，全局队列：`subagent`）
- 然后运行通告步骤，并将通告回复发布到请求者的聊天渠道
- 默认模型：继承调用者，除非你设置了 `agents.defaults.subagents.model`（或每智能体的 `agents.list[].subagents.model`）；显式的 `sessions_spawn.model` 仍然优先。
- 默认思考：继承调用者，除非你设置了 `agents.defaults.subagents.thinking`（或每智能体的 `agents.list[].subagents.thinking`）；显式的 `sessions_spawn.thinking` 仍然优先。

工具参数：

- `task`（必需）
- `label?`（可选）
- `agentId?`（可选；如果允许，在另一个智能体 id 下生成）
- `model?`（可选；覆盖子智能体模型；无效值会被跳过，子智能体将使用默认模型运行并在工具结果中显示警告）
- `thinking?`（可选；覆盖子智能体运行的思考级别）
- `runTimeoutSeconds?`（默认 `0`；设置后，子智能体运行在 N 秒后中止）
- `cleanup?`（`delete|keep`，默认 `keep`）

允许列表：

- `agents.list[].subagents.allowAgents`：可以通过 `agentId` 指定的智能体 id 列表（`["*"]` 允许任意）。默认：仅限请求者智能体。

发现：

- 使用 `agents_list` 查看当前允许用于 `sessions_spawn` 的智能体 id。

自动归档：

- 子智能体会话在 `agents.defaults.subagents.archiveAfterMinutes` 后自动归档（默认：60）。
- 归档使用 `sessions.delete` 并将转录重命名为 `*.deleted.<timestamp>`（同一文件夹）。
- `cleanup: "delete"` 在通告后立即归档（仍通过重命名保留转录）。
- 自动归档是尽力而为的；如果 Gateway 网关重启，待处理的定时器会丢失。
- `runTimeoutSeconds` **不会**自动归档；它只停止运行。会话会保留直到自动归档。
- 自动归档同样适用于深度 1 和深度 2 会话。

## 嵌套子智能体

默认情况下，子智能体不能生成自己的子智能体（`maxSpawnDepth: 1`）。你可以通过设置 `maxSpawnDepth: 2` 启用一层嵌套，从而支持**编排器模式**：主 → 编排子智能体 → 工作子子智能体。

### 如何启用

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // 允许子智能体生成子级（默认: 1）
        maxChildrenPerAgent: 5, // 每个智能体会话的最大活动子级数（默认: 5）
        maxConcurrent: 8, // 全局并发通道上限（默认: 8）
      },
    },
  },
}
```

### 深度级别

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | 主智能体                                     | 始终可以                     |
| 1     | `agent:<id>:subagent:<uuid>`                 | 子智能体（当允许深度 2 时为编排器）           | 仅当 `maxSpawnDepth >= 2`    |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | 子子智能体（叶子工作者）                      | 永远不可以                   |

### 通告链

结果会沿链路向上回传：

1. 深度 2 工作智能体完成 → 向其父级（深度 1 编排器）通告
2. 深度 1 编排器接收通告，汇总结果，完成后 → 向主智能体通告
3. 主智能体接收通告并将结果发送给用户

每一层只会看到其直接子级的通告。

### 按深度划分的工具策略

- **深度 1（编排器，当 `maxSpawnDepth >= 2`）**：获得 `sessions_spawn`、`subagents`、`sessions_list`、`sessions_history`，以便管理其子级。其他会话/系统工具仍被拒绝。
- **深度 1（叶子，当 `maxSpawnDepth == 1`）**：没有会话工具（当前默认行为）。
- **深度 2（叶子工作者）**：没有会话工具 —— 在深度 2 时始终拒绝 `sessions_spawn`，不能再生成子级。

### 每智能体生成上限

每个智能体会话（任意深度）在任一时间最多只能有 `maxChildrenPerAgent`（默认：5）个活动子级。这可防止单个编排器出现失控扇出。

### 级联停止

停止深度 1 编排器会自动停止其所有深度 2 子级：

- 在主聊天中发送 `/stop` 会停止所有深度 1 智能体，并级联到其深度 2 子级。
- `/subagents kill <id>` 会停止特定子智能体，并级联到其子级。
- `/subagents kill all` 会停止请求者的所有子智能体，并级联停止。

## 认证

子智能体认证按**智能体 id** 解析，而不是按会话类型：

- 子智能体会话键是 `agent:<agentId>:subagent:<uuid>`。
- 认证存储从该智能体的 `agentDir` 加载。
- 主智能体的认证配置文件作为**回退**合并；智能体配置文件在冲突时覆盖主配置文件。

注意：合并是累加的，所以主配置文件始终可用作回退。目前尚不支持每智能体完全隔离的认证。

## 通告

子智能体通过通告步骤报告：

- 通告步骤在子智能体会话中运行（不是请求者会话）。
- 如果子智能体精确回复 `ANNOUNCE_SKIP`，则不发布任何内容。
- 否则，通告回复通过后续的 `agent` 调用（`deliver=true`）发布到请求者的聊天渠道。
- 通告回复在可用时保留线程/话题路由（Slack 线程、Telegram 话题、Matrix 线程）。
- 通告消息被规范化为稳定模板：
  - `Status:` 从运行结果派生（`success`、`error`、`timeout` 或 `unknown`）。
  - `Result:` 通告步骤的摘要内容（如果缺失则为 `(not available)`）。
  - `Notes:` 错误详情和其他有用的上下文。
- `Status` 不是从模型输出推断的；它来自运行时结果信号。

通告负载在末尾包含统计行（即使被包装）：

- 运行时间（例如 `runtime 5m12s`）
- Token 使用量（输入/输出/总计）
- 配置模型定价时的估计成本（`models.providers.*.models[].cost`）
- `sessionKey`、`sessionId` 和转录路径（以便主智能体可以通过 `sessions_history` 获取历史记录或检查磁盘上的文件）

## 工具策略（子智能体工具）

默认情况下，子智能体获得**除会话工具和系统工具外的所有工具**：

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

当 `maxSpawnDepth >= 2` 时，深度 1 的编排器子智能体还会额外获得 `sessions_spawn`、`subagents`、`sessions_list` 和 `sessions_history`，以便管理其子级。

通过配置覆盖：

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny 优先
        deny: ["gateway", "cron"],
        // 如果设置了 allow，则变为仅允许模式（deny 仍然优先）
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## 并发

子智能体使用专用的进程内队列通道：

- 通道名称：`subagent`
- 并发数：`agents.defaults.subagents.maxConcurrent`（默认 `8`）

## 停止

- 在请求者聊天中发送 `/stop` 会中止请求者会话，并停止从中生成的任何活动子智能体运行，级联至嵌套子级。
- `/subagents kill <id>` 会停止特定子智能体，并级联至其子级。

## 限制

- 子智能体通告是**尽力而为**的。如果 Gateway 网关重启，待处理的“通告回复”工作会丢失。
- 子智能体仍然共享相同的 Gateway 网关进程资源；将 `maxConcurrent` 视为安全阀。
- `sessions_spawn` 始终是非阻塞的：它立即返回 `{ status: "accepted", runId, childSessionKey }`。
- 子智能体上下文仅注入 `AGENTS.md` + `TOOLS.md`（无 `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 或 `BOOTSTRAP.md`）。
- 最大嵌套深度为 5（`maxSpawnDepth` 范围：1–5）。大多数用例推荐使用深度 2。
- `maxChildrenPerAgent` 限制每个会话的活动子级数量（默认：5，范围：1–20）。


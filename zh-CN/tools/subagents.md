---
summary: "子智能体：生成隔离的智能体运行，并将结果通告回请求者聊天"
read_when:
  - 44. 当你希望通过代理进行后台/并行工作时
  - 你正在更改 sessions_spawn 或子智能体工具策略
title: "46. 子代理"
---

# 子代理

子代理是从现有代理运行中派生出的后台代理运行。 它们在各自的会话中运行（`agent:<agentId>:subagent:<uuid>`），并在完成后将其结果**通告**回请求者所在的聊天频道。

## 斜杠命令

使用 `/subagents` 检查或控制**当前会话**的子智能体运行：

- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` 显示运行元数据（状态、时间戳、会话 id、转录路径、清理）。

主要目标：

- 并行化"研究 / 长任务 / 慢工具"工作，而不阻塞主运行。
- 默认保持子智能体隔离（会话分离 + 可选沙箱隔离）。
- 保持工具接口难以被误用：子代理默认**不**会获得会话工具。
- 支持为编排模式配置嵌套深度。

成本说明：每个子代理都有其**独立**的上下文和令牌消耗。 对于繁重或重复性的任务，可为子代理设置更便宜的模型，而主代理使用更高质量的模型。
你可以通过 `agents.defaults.subagents.model` 或按代理进行覆盖配置。

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

- `agents.list[].subagents.allowAgents`：可通过 `agentId` 指定的代理 ID 列表（`["*"]` 表示允许任意）。 默认：仅请求者代理。

发现：

- 使用 `agents_list` 查看当前允许用于 `sessions_spawn` 的智能体 id。

自动归档：

- 子智能体会话在 `agents.defaults.subagents.archiveAfterMinutes` 后自动归档（默认：60）。
- 归档使用 `sessions.delete` 并将转录重命名为 `*.deleted.<timestamp>`（同一文件夹）。
- `cleanup: "delete"` 在通告后立即归档（仍通过重命名保留转录）。
- 自动归档是尽力而为的；如果 Gateway 网关重启，待处理的定时器会丢失。
- `runTimeoutSeconds` **不会**自动归档；它只停止运行。会话会保留直到自动归档。 会话将持续存在，直到自动归档。
- 自动归档同样适用于深度为 1 和 2 的会话。

## 嵌套子代理

默认情况下，子代理不能再生成自己的子代理（`maxSpawnDepth: 1`）。 你可以通过设置 `maxSpawnDepth: 2` 启用一层嵌套，从而支持**编排模式**：主代理 → 编排子代理 → 工作子子代理。

### 如何启用

```json5
`agents.list[].subagents.allowAgents`：可以通过 `agentId` 指定的智能体 id 列表（`["*"]` 允许任意）。默认：仅限请求者智能体。
```

### 深度级别

| 深度 | 会话键格式                                        | 角色                  | 可以生成？                   |
| -- | -------------------------------------------- | ------------------- | ----------------------- |
| 0  | `agent:&lt;id&gt;:main`                            | 主代理                 | 始终可以                    |
| 1  | `agent:&lt;id&gt;:subagent:<uuid>`                 | 子代理（当允许深度为 2 时为编排者） | 仅当 `maxSpawnDepth >= 2` |
| 2  | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | 子子代理（叶子工作者）         | 从不                      |

### Announce 链路

结果沿链路向上回传：

1. Depth-2 worker 完成 → 向其父级（depth-1 orchestrator）发送 announce
2. Depth-1 orchestrator 接收 announce，汇总结果并完成 → 向 main 发送 announce
3. Main agent 接收 announce 并将结果交付给用户

每一层只能看到来自其直属子级的 announce。

### 按深度划分的工具策略

- **Depth 1（orchestrator，当 `maxSpawnDepth >= 2` 时）**：获得 `sessions_spawn`、`subagents`、`sessions_list`、`sessions_history`，以便管理其子级。 其他 session/system 工具仍然被拒绝。
- **Depth 1（leaf，当 `maxSpawnDepth == 1` 时）**：无 session 工具（当前默认行为）。
- **Depth 2（leaf worker）**：无 session 工具 —— 在 depth 2 时始终拒绝 `sessions_spawn`。 不能继续创建子级。

### 每个 agent 的 spawn 限制

每个 agent session（任意深度）在任意时刻最多只能拥有 `maxChildrenPerAgent`（默认：5）个活跃子级。 这可防止单个 orchestrator 出现失控的扇出。

### 级联停止

停止一个 depth-1 orchestrator 会自动停止其所有 depth-2 子级：

- 在主聊天中执行 `/stop` 会停止所有 depth-1 agents，并级联停止其 depth-2 子级。
- `/subagents kill <id>` 会停止指定的 sub-agent，并级联停止其子级。
- `/subagents kill all` 会停止请求者的所有 sub-agents，并进行级联停止。

## 认证

49. 子代理认证是通过 **代理 ID** 解析的，而不是通过会话类型：

- 子智能体会话键是 `agent:<agentId>:subagent:<uuid>`。
- 认证存储从该智能体的 `agentDir` 加载。
- 主 agent 的 auth profiles 会作为\*\*回退（fallback）\*\*合并；当发生冲突时，agent profiles 会覆盖主 profiles。

注意：该合并为累加式，因此主 profiles 始终作为回退可用。 目前尚不支持每个 agent 完全隔离的 auth。

## 通告

Sub-agents 通过 announce 步骤进行回报：

- announce 步骤在 sub-agent session 内运行（而非请求者 session）。
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

默认情况下，子智能体获得**除会话工具外的所有工具**：

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

当 `maxSpawnDepth >= 2` 时，depth-1 orchestrator 类型的 sub-agents 还会获得 `sessions_spawn`、`subagents`、`sessions_list` 和 `sessions_history`，以便管理其子级。

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

## Concurrency

子智能体使用专用的进程内队列通道：

- 通道名称：`subagent`
- 并发数：`agents.defaults.subagents.maxConcurrent`（默认 `8`）

## 停止

- 在请求者聊天中发送 `/stop` 会中止请求者会话并停止从中生成的任何活动子智能体运行。
- `/subagents kill <id>` 会停止指定的 sub-agent，并级联停止其子级。

## 限制

- Sub-agent announce 为**尽力而为（best-effort）**。 如果 gateway 重启，待处理的“announce 回传”任务将会丢失。
- 子智能体仍然共享相同的 Gateway 网关进程资源；将 `maxConcurrent` 视为安全阀。
- `sessions_spawn` 始终是非阻塞的：它立即返回 `{ status: "accepted", runId, childSessionKey }`。
- 子智能体上下文仅注入 `AGENTS.md` + `TOOLS.md`（无 `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 或 `BOOTSTRAP.md`）。
- 最大嵌套深度为 5（`maxSpawnDepth` 范围：1–5）。 大多数使用场景推荐使用 Depth 2。
- `maxChildrenPerAgent` 限制每个 session 的活跃子级数量（默认：5，范围：1–20）。

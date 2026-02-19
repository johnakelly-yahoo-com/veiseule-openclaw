---
summary: "菜单栏状态逻辑及向用户展示的内容"
read_when:
  - 调整 Mac 菜单 UI 或状态逻辑
title: "菜单栏"
---

# 菜单栏状态逻辑

## 显示内容

- 我们在菜单栏图标和菜单的第一行状态行中展示当前智能体的工作状态。
- 工作活跃时隐藏健康状态；当所有会话空闲时恢复显示。
- 菜单中的"节点"区块仅列出**设备**（通过 `node.list` 配对的节点），不包括客户端/在线状态条目。
- 20. 状态模型

## 状态模型

- 会话：事件携带 `runId`（每次运行）以及载荷中的 `sessionKey`。"main" 会话的键为 `main`；如果不存在，则回退到最近更新的会话。 23. 优先级：main 始终优先。
- 24. 如果 main 处于活动状态，则立即显示其状态。 25. 如果 main 处于空闲状态，则显示最近一次处于活动状态的非 main 会话。 26. 我们不会在活动进行中来回切换；只有当当前会话变为空闲或 main 变为活动时才会切换。 27. 活动类型：
- 活动类型：
  - `job`：高层命令执行（`state: started|streaming|done|error`）。
  - `tool`：`phase: start|result`，包含 `toolName` 和 `meta/args`。

## IconState 枚举（Swift）

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)`（调试覆盖）

### ActivityKind → 图标符号

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- 默认 → 🛠️

### 视觉映射

- `idle`：正常小动物图标。
- `workingMain`：带图标符号的徽章，完整色调，腿部"工作"动画。
- 46. `overridden`：无论活动如何，使用选定的图标/色调。
- `overridden`：无论活动状态如何，使用所选的图标符号/色调。

## 状态行文本（菜单）

- 工作活跃时：`<会话角色> · <活动标签>`
  - 示例：`Main · exec: pnpm test`、`Other · read: apps/macos/Sources/OpenClaw/AppState.swift`。
- 空闲时：回退显示健康摘要。

## 事件接收

- 来源：控制渠道 `agent` 事件（`ControlChannel.handleAgentEvent`）。
- 解析字段：
  - `stream: "job"`，包含 `data.state` 用于启动/停止。
  - `stream: "tool"`，包含 `data.phase`、`name`，可选 `meta`/`args`。
- 标签：
  - `exec`：`args.command` 的第一行。
  - `read`/`write`：缩短的路径。
  - `edit`：路径加上从 `meta`/diff 计数推断的变更类型。
  - 回退：工具名称。

## Debug override

- 设置 ▸ 调试 ▸ "图标覆盖" 选择器：
  - `系统（自动）`（默认）
  - `工作中：main`（按工具类型）
  - `工作中：other`（按工具类型）
  - `Idle`
- 通过 `@AppStorage("iconOverride")` 存储；映射到 `IconState.overridden`。

## 测试清单

- 触发 main 会话任务：验证图标立即切换且状态行显示 main 标签。
- main 空闲时触发非 main 会话任务：图标/状态显示非 main；保持稳定直到完成。
- 在 other 活跃时启动 main：图标立即切换到 main。
- 快速连续工具调用：确保徽章不会闪烁（工具结果的 TTL 宽限期）。
- 所有会话空闲后健康行重新出现。


---
summary: "计划：使用 CDP 将 browser act:evaluate 与 Playwright 队列隔离，设置端到端超时限制并采用更安全的 ref 解析"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor 计划

## 背景

`act:evaluate` 在页面中执行用户提供的 JavaScript。 目前它通过 Playwright
（`page.evaluate` 或 `locator.evaluate`）运行。 Playwright 会按页面串行化 CDP 命令，因此一个
卡住或长时间运行的 evaluate 可能会阻塞页面命令队列，使该标签页上的后续操作
看起来都“卡住”。

PR #13498 添加了一个务实的安全保障（受限的 evaluate、abort 传播以及尽力恢复）。 本文档描述了一项更大的重构，使 `act:evaluate` 从架构上
与 Playwright 隔离，从而即使 evaluate 卡住，也不会阻塞正常的 Playwright 操作。

## 目标

- `act:evaluate` 不能永久阻塞同一标签页上的后续浏览器操作。
- 超时应作为端到端的单一事实来源，使调用方可以依赖统一的时间预算。
- 在 HTTP 和进程内分发中，Abort 和超时应以相同方式处理。
- 在不完全脱离 Playwright 的情况下，支持对元素进行 evaluate 定位。
- 为现有调用方和负载保持向后兼容。

## 非目标

- 将所有浏览器操作（click、type、wait 等） 替换为 CDP 实现。
- 移除在 PR #13498 中引入的现有安全兜底机制（它仍然是一个有用的回退方案）。
- 在现有 `browser.evaluateEnabled` 开关之外，引入新的不安全能力。
- 为 evaluate 增加进程隔离（worker 进程/线程）。 如果在本次重构之后我们仍然看到难以恢复的
  卡死状态，这是一个后续可以跟进的想法。

## 当前架构（为何会卡死）

从高层来看：

- 调用方将 `act:evaluate` 发送到浏览器控制服务。
- 路由处理器调用 Playwright 来执行 JavaScript。
- Playwright 会串行化页面命令，因此一个永远无法完成的 evaluate 会阻塞队列。
- 队列卡住意味着后续在该标签页上的点击/输入/等待等操作可能看起来像是挂起。

## 建议的架构

### 1. 截止时间传播

引入一个统一的预算概念，并从中派生所有相关逻辑：

- 调用方设置 `timeoutMs`（或一个未来的截止时间）。
- 外层请求超时、路由处理逻辑以及页面内部的执行预算
  都使用同一个预算，并在需要时为序列化开销预留少量余量。
- 通过 `AbortSignal` 在各处传播中止信号，以确保取消行为一致。

实现方向：

- 添加一个小型辅助函数（例如 `createBudget({ timeoutMs, signal })`），返回：
  - `signal`：关联的 AbortSignal
  - `deadlineAtMs`：绝对截止时间
  - `remainingMs()`：子操作的剩余预算
- 在以下位置使用该辅助函数：
  - `src/browser/client-fetch.ts`（HTTP 和进程内分发）
  - `src/node-host/runner.ts`（代理路径）
  - 浏览器操作实现（Playwright 和 CDP）

### 2. 独立的 Evaluate 引擎（CDP 路径）

新增一个基于 CDP 的 evaluate 实现，不与 Playwright 的页面级命令
队列共享。 关键特性是 evaluate 的传输使用独立的 WebSocket 连接
以及附加到目标上的独立 CDP 会话。

实现方向：

- 新增模块，例如 `src/browser/cdp-evaluate.ts`，其功能包括：
  - 连接到已配置的 CDP 端点（浏览器级 socket）。
  - 使用 `Target.attachToTarget({ targetId, flatten: true })` 获取一个 `sessionId`。
  - 执行以下之一：
    - 页面级 evaluate 使用 `Runtime.evaluate`，或
    - 元素级 evaluate 使用 `DOM.resolveNode` 加 `Runtime.callFunctionOn`。
  - 在超时或中止时：
    - 尽最大努力为该会话发送 `Runtime.terminateExecution`。
    - 关闭 WebSocket 并返回清晰的错误信息。

Notes:

- 这仍然会在页面中执行 JavaScript，因此终止操作可能会产生副作用。 其优势在于它不会阻塞 Playwright 队列，并且可以通过终止 CDP 会话在传输层取消。

### 3. Ref 方案（无需全面重写的元素定位）

难点在于元素定位。 CDP 需要一个 DOM 句柄或 `backendDOMNodeId`，而目前大多数浏览器操作使用基于快照中 ref 的 Playwright 定位器。

推荐方案：保留现有的 refs，但附加一个可选的可由 CDP 解析的 id。

#### 3.1 扩展存储的 Ref 信息

扩展已存储的角色 ref 元数据，使其可选地包含一个 CDP id：

- Today: `{ role, name, nth }`
- Proposed: `{ role, name, nth, backendDOMNodeId?: number }`

这可以保持所有现有基于 Playwright 的操作正常工作，并在 `backendDOMNodeId` 可用时允许 CDP evaluate 接受相同的 `ref` 值。

#### 3.2 在快照阶段填充 backendDOMNodeId

在生成角色快照时：

1. 像当前一样生成现有的角色 ref 映射（role, name, nth）。
2. 通过 CDP 获取 AX 树（`Accessibility.getFullAXTree`），并使用相同的重复处理规则计算一个并行映射 `(role, name, nth) -> backendDOMNodeId`。
3. 将该 id 合并回当前标签页存储的 ref 信息中。

如果某个 ref 的映射失败，则将 `backendDOMNodeId` 保持为 undefined。 这使该功能成为尽力而为（best-effort）且可安全逐步推出的方案。

#### 3.3 使用 Ref 的 Evaluate 行为

在 `act:evaluate` 中：

- 如果存在 `ref` 且包含 `backendDOMNodeId`，则通过 CDP 对元素执行 evaluate。
- 如果存在 `ref` 但没有 `backendDOMNodeId`，则回退到 Playwright 路径（带安全保护机制）。

可选的逃生通道：

- 扩展请求结构以支持直接接受 `backendDOMNodeId`（用于高级调用方和调试），同时保留 `ref` 作为主要接口。

### 4. 保留最后的恢复路径

即使使用 CDP evaluate，仍然有其他方式可能导致标签页或连接卡死。 保留现有的恢复机制（终止执行 + 断开 Playwright 连接）作为以下情况的最后手段：

- 遗留调用方
- CDP attach 被阻止的环境
- Playwright 的意外边缘情况

## 实施计划（单次迭代）

### 交付内容

- 一个基于 CDP 的 evaluate 引擎，在 Playwright 的每页面命令队列之外运行。
- 一个由调用方和处理程序一致使用的端到端 timeout/abort 预算。
- 可选携带 `backendDOMNodeId` 用于元素 evaluate 的 ref 元数据。
- `act:evaluate` 在可能时优先使用 CDP 引擎，否则回退到 Playwright。
- 用于证明卡死的 evaluate 不会阻塞后续操作的测试。
- 用于让故障和回退路径可见的日志/指标。

### 实施检查清单

1. 添加一个共享的“budget”辅助函数，将 `timeoutMs` 和上游 `AbortSignal` 关联为：
   - 单个 `AbortSignal`
   - 一个绝对截止时间
   - 用于下游操作的 `remainingMs()` 辅助函数
2. 更新所有调用路径以使用该辅助函数，使 `timeoutMs` 在各处含义一致：
   - `src/browser/client-fetch.ts`（HTTP 和进程内分发）
   - `src/node-host/runner.ts`（node 代理路径）
   - 调用 `/act` 的 CLI 包装器（为 `browser evaluate` 添加 `--timeout-ms`）
3. 实现 `src/browser/cdp-evaluate.ts`：
   - 连接到浏览器级别的 CDP socket
   - 使用 `Target.attachToTarget` 获取 `sessionId`
   - 为页面 evaluate 运行 `Runtime.evaluate`
   - 为元素 evaluate 运行 `DOM.resolveNode` + `Runtime.callFunctionOn`
   - 在超时/中止时：尽最大努力执行 `Runtime.terminateExecution`，然后关闭 socket
4. 扩展已存储的角色引用元数据，使其可选包含 `backendDOMNodeId`：
   - 保留现有的 `{ role, name, nth }` 行为以支持 Playwright 操作
   - 为 CDP 元素定位添加 `backendDOMNodeId?: number`
5. 在创建快照期间填充 `backendDOMNodeId`（尽最大努力）：
   - 通过 CDP 获取 AX 树（`Accessibility.getFullAXTree`）
   - 计算 `(role, name, nth) -> backendDOMNodeId` 并合并到已存储的引用映射中
   - 如果映射存在歧义或缺失，则保持 id 为 undefined
6. 更新 `act:evaluate` 路由：
   - 如果没有 `ref`：始终使用 CDP evaluate
   - 如果 `ref` 解析为 `backendDOMNodeId`：使用 CDP 元素 evaluate
   - 否则：回退到 Playwright evaluate（仍然受限且可中止）
7. 保留现有的“最后手段”恢复路径作为回退方案，而非默认路径。
8. 添加测试：
   - 卡住的 evaluate 会在预算时间内超时，且后续 click/type 能成功执行
   - abort 会取消 evaluate（客户端断开或超时），并解除对后续操作的阻塞
   - 映射失败时能干净地回退到 Playwright
9. 添加可观测性：
   - evaluate 持续时间和超时计数器
   - terminateExecution 使用情况
   - 回退率（CDP -> Playwright）及原因

### 验收标准

- 一个故意挂起的 `act:evaluate` 会在调用方预算时间内返回，并且不会使
  该标签页在后续操作中卡死。
- `timeoutMs` 在 CLI、agent 工具、node 代理和进程内调用之间表现一致。
- 如果 `ref` 可以映射到 `backendDOMNodeId`，元素 evaluate 使用 CDP；否则
  回退路径仍然受限且可恢复。

## 测试计划

- 单元测试：
  - `(role, name, nth)` 在 role 引用与 AX 树节点之间的匹配逻辑。
  - 预算辅助工具行为（余量、剩余时间计算）。
- 集成测试：
  - CDP evaluate 超时应在预算内返回，且不会阻塞下一个操作。
  - Abort 会取消 evaluate 并尽最大努力触发终止。
- 契约测试：
  - 确保 `BrowserActRequest` 和 `BrowserActResponse` 保持兼容。

## 风险与缓解措施

- 映射并不完美：
  - 缓解措施：尽力映射，回退到 Playwright evaluate，并添加调试工具。
- `Runtime.terminateExecution` 存在副作用：
  - 缓解措施：仅在超时/Abort 时使用，并在错误中记录该行为。
- 额外开销：
  - 缓解措施：仅在请求快照时获取 AX 树，按 target 缓存，并保持
    CDP 会话为短生命周期。
- 扩展中继限制：
  - 缓解措施：当无法使用每页 socket 时，使用浏览器级别的 attach API，且
    保留当前的 Playwright 路径作为回退方案。

## 未决问题

- 新引擎是否应配置为 `playwright`、`cdp` 或 `auto`？
- 是否要为高级用户公开新的 "nodeRef" 格式，还是仅保留 `ref`？
- 帧快照和选择器范围快照应如何参与 AX 映射？

---
title: 形式化验证（安全模型）
permalink: /security/formal-verification/
---

# 形式化验证（安全模型）

本页面记录 OpenClaw 的**形式化安全模型**（当前为 TLA+/TLC；根据需要会增加更多）。

> 注意：某些旧链接可能仍引用之前的项目名称。

**目标（北极星）：**在明确假设前提下，提供一个经机器校验的论证，证明 OpenClaw 能够执行其预期的安全策略（授权、会话隔离、工具门控以及配置错误安全性）。

**当前内容：**一个可执行、以攻击者为驱动的**安全回归测试套件**：

- 每个声明都附带一个在有限状态空间上可运行的模型检查。
- 许多声明配有对应的**负向模型**，可针对现实中的漏洞类别生成反例轨迹。

**当前尚未做到：**尚未证明“OpenClaw 在所有方面都是安全的”，也未证明完整的 TypeScript 实现是正确的。

## 模型存放位置

模型维护在一个独立仓库中：[vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

## 重要注意事项

- 这些是**模型**，而不是完整的 TypeScript 实现。模型与代码之间可能存在偏差。
- 结果受限于 TLC 探索的状态空间；“绿色”结果并不意味着在建模假设和边界之外仍然安全。
- 某些声明依赖于明确的环境假设（例如正确部署、正确的配置输入）。

## 结果复现

目前，可通过在本地克隆模型仓库并运行 TLC 来复现结果（见下文）。未来版本可能会提供：

- 在 CI 中运行模型并公开产物（反例轨迹、运行日志）
- 提供一个托管的“运行此模型”工作流，用于小规模、有界检查

快速开始：

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# 需要 Java 11+（TLC 运行在 JVM 上）。
# 仓库内包含固定版本的 `tla2tools.jar`（TLA+ 工具），并提供 `bin/tlc` + Make 目标。

make <target>
```

### 网关暴露与开放网关错误配置

**声明：**在未启用认证的情况下绑定到回环地址以外的接口，可能导致远程入侵成为可能 / 增加暴露面；令牌/密码可阻止未认证攻击者（在模型假设下）。

- 绿色运行：
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- 红色（预期）：
  - `make gateway-exposure-v2-negative`

另见：模型仓库中的 `docs/gateway-exposure-matrix.md`。

### Nodes.run 管道（最高风险能力）

**声明：**`nodes.run` 需要（a）节点命令允许列表及声明命令，以及（b）在配置启用时进行实时审批；审批在模型中采用令牌化以防止重放。

- 绿色运行：
  - `make nodes-pipeline`
  - `make approvals-token`
- 红色（预期）：
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### 配对存储（DM 门控）

**声明：**配对请求遵守 TTL 和待处理请求上限。

- 绿色运行：
  - `make pairing`
  - `make pairing-cap`
- 红色（预期）：
  - `make pairing-negative`
  - `make pairing-cap-negative`

### 入口门控（提及 + 控制命令绕过）

**声明：**在需要提及的群组上下文中，未授权的“控制命令”无法绕过提及门控。

- 绿色：
  - `make ingress-gating`
- 红色（预期）：
  - `make ingress-gating-negative`

### 路由 / 会话键隔离

**声明：**来自不同对等方的私信不会合并为同一会话，除非明确链接/配置。

- 绿色：
  - `make routing-isolation`
- 红色（预期）：
  - `make routing-isolation-negative`

## v1++：附加有界模型（并发、重试、轨迹正确性）

这些是后续模型，用于在真实世界故障模式（非原子更新、重试、消息扇出）方面提高模型的保真度。

### 配对存储并发 / 幂等性

**声明：**配对存储在交错执行情况下也应强制执行 `MaxPending` 和幂等性（即“检查-再写入”必须是原子的 / 加锁；刷新不应创建重复项）。

含义：

- 在并发请求下，某个频道的待处理数量不能超过 `MaxPending`。
- 对同一 `(channel, sender)` 的重复请求/刷新不应创建重复的有效待处理记录。

- 绿色运行：
  - `make pairing-race`（原子/加锁的上限检查）
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- 红色（预期）：
  - `make pairing-race-negative`（非原子 begin/commit 上限竞争）
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### 入口轨迹关联 / 幂等性

**声明：**摄取过程应在扇出过程中保持轨迹关联，并在提供方重试时保持幂等。

含义：

- 当一个外部事件转化为多个内部消息时，每个部分都保留相同的轨迹/事件标识。
- 重试不会导致重复处理。
- 如果缺少提供方事件 ID，去重应回退到安全键（例如 trace ID），以避免误丢弃不同事件。

- 绿色：
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- 红色（预期）：
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### 路由 dmScope 优先级 + identityLinks

**声明：**默认情况下，路由必须保持私信会话隔离，仅在明确配置时才合并会话（频道优先级 + 身份链接）。

含义：

- 特定频道的 dmScope 覆盖必须优先于全局默认值。
- identityLinks 仅应在显式链接组内合并，而不应跨无关对等方合并。

- 绿色：
  - `make routing-precedence`
  - `make routing-identitylinks`
- 红色（预期）：
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`

---
summary: "Gateway 网关服务、生命周期和运维的运行手册"
read_when:
  - 运行或调试 Gateway 网关进程时
title: "Gateway 网关运行手册"
---

# Gateway 网关服务运行手册

本页用于 Gateway 服务的首日启动（day-1）和次日运维（day-2）操作。

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    基于症状优先的诊断指南，提供精确的命令步骤和日志特征。
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    面向任务的设置指南 + 完整配置参考。
  
</Card>
</CardGroup>

## 5 分钟本地启动

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# 在 stdio 中获取完整的调试/追踪日志：
openclaw gateway --port 18789 --verbose
# 如果端口被占用，终止监听器然后启动：
openclaw gateway --force
# 开发循环（TS 更改时自动重载）：
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

健康基线：`Runtime: running` 和 `RPC probe: ok`。

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway 配置重载会监听当前活动的配置文件路径（从 profile/state 默认值解析，或在设置 `OPENCLAW_CONFIG_PATH` 时使用该路径）。
默认模式：`gateway.reload.mode="hybrid"`（热应用安全更改，关键更改时重启）。
</Note>

## 运行时模型

- 一个始终运行的进程，用于路由、控制平面和通道连接。
- 单一复用端口用于：
  - WebSocket 控制/RPC
  - OpenResponses（HTTP）：[`/v1/responses`](/gateway/openresponses-http-api)。
  - 控制 UI 和 hooks
- 默认绑定模式：`loopback`。
- 默认需要 Gateway 网关认证：设置 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）或 `gateway.auth.password`。客户端必须发送 `connect.params.auth.token/password`，除非使用 Tailscale Serve 身份。

### 端口和绑定优先级

| 设置         | 解析顺序                                                          |
| ---------- | ------------------------------------------------------------- |
| Gateway 端口 | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| 绑定模式       | CLI/override → `gateway.bind` → `loopback`                    |

### 热重载模式

| 使用 `gateway.reload.mode="off"` 禁用。 | 保活行为           |
| ---------------------------------- | -------------- |
| `off`                              | 不重新加载配置        |
| `hot`                              | 仅应用可安全热更新的更改   |
| `restart`                          | 在需要重启的更改时执行重启  |
| `hybrid`（默认）                       | 在安全时热应用，在需要时重启 |

## 运维命令集

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## 远程访问

首选 Tailscale/VPN；否则使用 SSH 隧道：
备用方案：SSH 隧道。

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

然后客户端通过隧道连接到 `ws://127.0.0.1:18789`。

<Warning>
如果配置了令牌，即使通过隧道，客户端也必须在 `connect.params.auth.token` 中包含它。
</Warning>

参见：[Remote Gateway](/gateway/remote)、[Authentication](/gateway/authentication)、[Tailscale](/gateway/tailscale)。

## 监控与服务生命周期

在生产类环境中使用受监督运行以提高可靠性。

<Tabs>
  <Tab title="macOS (launchd)">

```bash
`openclaw gateway stop|restart` — 停止/重启受监管的 Gateway 网关服务（launchd/systemd）。
```

LaunchAgent 标签为 `ai.openclaw.gateway`（默认）或 `ai.openclaw.<profile>`（命名 profile）。 `openclaw doctor` 会审计并修复服务配置漂移。

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

启用 lingering（必需，以便用户服务在登出/空闲后继续存活）：

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

对于多用户/始终运行的主机，请使用 system 单元。

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## 多个 Gateway 网关（同一主机）

大多数部署应只运行 **一个** Gateway。
仅在需要严格隔离/冗余时才使用多个（例如救援配置）。

每个实例的检查清单：

- 唯一的 `gateway.port`
- 唯一的 `OPENCLAW_CONFIG_PATH`
- 唯一的 `OPENCLAW_STATE_DIR`
- 唯一的 `agents.defaults.workspace`

示例：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

参见：[Multiple gateways](/gateway/multiple-gateways)。

### Dev 配置文件（`--dev`）

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# 然后定位到 dev 实例：
openclaw --dev status
openclaw --dev health
```

默认包括隔离的状态/配置以及基础 Gateway 端口 `19001`。

## 协议快速参考（运维视角）

- 客户端发送的第一帧必须是 `connect`。
- Gateway 返回 `hello-ok` 快照（`presence`、`health`、`stateVersion`、`uptimeMs`、limits/policy）。
- 请求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- 常见事件：`connect.challenge`、`agent`、`chat`、`presence`、`tick`、`health`、`heartbeat`、`shutdown`。

Agent 运行分为两个阶段：

1. 立即返回已接受确认（`status:"accepted"`）
2. `agent` 响应是两阶段的：首先 `res` 确认 `{runId,status:"accepted"}`，然后在运行完成后发送最终 `res` `{runId,status:"ok"|"error",summary}`；流式输出作为 `event:"agent"` 到达。

完整文档：[Gateway 网关协议](/gateway/protocol) 和 [Bridge 协议（旧版）](/gateway/bridge-protocol)。

## 运维检查

### 存活性

- 打开 WS 并发送 `connect`。
- 期望收到包含快照的 `hello-ok` 响应。

### 就绪性

```bash
`openclaw gateway health|status` — 通过 Gateway 网关 WS 请求 health/status。
```

### 间隙恢复

客户端检测到序列缺口时，应在继续之前刷新（`health` + `system-presence`）。 在出现序列间隙时，先刷新状态（`health`、`system-presence`）再继续。

## 常见故障特征

| 特征                                                             | 可能的问题                                    |
| -------------------------------------------------------------- | ---------------------------------------- |
| `refusing to bind gateway ...` `without auth`                  | 在未提供 token/password 的情况下绑定到非 loopback 地址 |
| `another gateway instance is already listening` / `EADDRINUSE` | 端口冲突                                     |
| `Gateway start blocked: set gateway.mode=local`                | 配置被设置为远程模式                               |
| 在 connect 期间出现 `unauthorized`                                  | 客户端与 Gateway 之间的认证不匹配                    |

完整的诊断步骤请参见 [Gateway Troubleshooting](/gateway/troubleshooting)。

## 安全保证

- 当 Gateway 不可用时，Gateway 协议客户端会快速失败（不会隐式回退到直连通道）。
- 无效或非 connect 的首帧将被拒绝并关闭连接。
- 优雅关闭：关闭前发出 `shutdown` 事件；客户端必须处理关闭 + 重新连接。

---

相关内容：

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)


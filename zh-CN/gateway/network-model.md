---
summary: "Gateway 网关、节点和 canvas 主机如何连接。"
read_when:
  - 你想要简要了解 Gateway 网关的网络模型
title: "网络模型"
---

大多数操作通过 Gateway 网关（`openclaw gateway`）进行，它是一个长期运行的单一进程，负责管理渠道连接和 WebSocket 控制平面。

## 核心规则

- One Gateway per host is recommended. It is the only process allowed to own the WhatsApp Web session. For rescue bots or strict isolation, run multiple gateways with isolated profiles and ports. See [Multiple gateways](/gateway/multiple-gateways).
- 优先使用回环地址：Gateway 网关的 WS 默认为 `ws://127.0.0.1:18789`。即使是回环连接，向导也会默认生成 gateway token。若需通过 tailnet 访问，请运行 `openclaw gateway --bind tailnet --token ...`，因为非回环绑定必须使用 token。 The wizard generates a gateway token by default, even for loopback. For tailnet access, run `openclaw gateway --bind tailnet --token ...` because tokens are required for non-loopback binds.
- Nodes connect to the Gateway WS over LAN, tailnet, or SSH as needed. The legacy TCP bridge is deprecated.
- Canvas 主机是一个 HTTP 文件服务器，运行在 `canvasHost.port`（默认 `18793`）上，提供 `/__openclaw__/canvas/` 路径供节点 WebView 使用。参见 [Gateway 网关配置](/gateway/configuration)（`canvasHost`）。
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    当配置了 `gateway.auth` 且 Gateway 绑定到非 loopback 地址时，这些路由会受到 Gateway 身份验证保护（来自 loopback 的请求不受影响）。 参见 [Gateway configuration](/gateway/configuration)（`canvasHost`、`gateway`）。
- 远程使用通常通过 SSH 隧道或 Tailscale VPN。参见[远程访问](/gateway/remote)和[设备发现](/gateway/discovery)。 See [Remote access](/gateway/remote) and [Discovery](/gateway/discovery).

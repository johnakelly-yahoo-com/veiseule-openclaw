---
summary: "Gateway 网关仪表板（控制 UI）访问和认证"
read_when:
  - 更改仪表板认证或暴露模式
title: "仪表板"
---

# 仪表板（控制 UI）

Gateway 网关仪表板是默认在 `/` 提供的浏览器控制 UI
（通过 `gateway.controlUi.basePath` 覆盖）。

快速打开（本地 Gateway 网关）：

- http://127.0.0.1:18789/（或 http://localhost:18789/）

关键参考：

- [控制 UI](/web/control-ui) 了解使用方法和 UI 功能。
- [Tailscale](/gateway/tailscale) 了解 Serve/Funnel 自动化。
- [Web 界面](/web) 了解绑定模式和安全注意事项。

认证通过 `connect.params.auth`（token 或密码）在 WebSocket 握手时强制执行。
参见 [Gateway 网关配置](/gateway/configuration) 中的 `gateway.auth`。 See `gateway.auth` in [Gateway configuration](/gateway/configuration).

Security note: the Control UI is an **admin surface** (chat, config, exec approvals).
Do not expose it publicly. token 保持本地（仅查询参数）；UI 在首次加载后移除它并保存到 localStorage。
Prefer localhost, Tailscale Serve, or an SSH tunnel.

## 快速路径（推荐）

- After onboarding, the CLI auto-opens the dashboard and prints a clean (non-tokenized) link.
- 随时重新打开：`openclaw dashboard`（复制链接，如果可能则打开浏览器，如果是无头环境则显示 SSH 提示）。
- 在仪表板设置中，粘贴你在 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）中配置的相同 token。

## Token 基础（本地 vs 远程）

- **Localhost**: open `http://127.0.0.1:18789/`.
- **Token 来源**：`gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）；UI 在首次加载后存储它。
- **非 localhost**：使用 Tailscale Serve（如果 `gateway.auth.allowTailscale: true` 则无需 token）、带 token 的 tailnet 绑定，或 SSH 隧道。参见 [Web 界面](/web)。 See [Web surfaces](/web).

## 如果你看到"unauthorized" / 1008

- 确保 Gateway 网关可达（本地：`openclaw status`；远程：SSH 隧道 `ssh -N -L 18789:127.0.0.1:18789 user@host` 然后打开 `http://127.0.0.1:18789/?token=...`）。
- Retrieve the token from the gateway host: `openclaw config get gateway.auth.token` (or generate one: `openclaw doctor --generate-gateway-token`).
- In the dashboard settings, paste the token into the auth field, then connect.

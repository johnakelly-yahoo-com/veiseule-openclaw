---
summary: "将 Gateway 认证委托给受信任的反向代理（Pomerium、Caddy、nginx + OAuth）"
read_when:
  - 在身份感知代理后运行 OpenClaw
  - 在 OpenClaw 前配置 Pomerium、Caddy 或 nginx + OAuth
  - 修复在反向代理配置下出现的 WebSocket 1008 unauthorized 错误
---

# 受信任代理认证

> ⚠️ **安全敏感功能。** 此模式会将认证完全委托给你的反向代理。 配置错误可能会使你的 Gateway 暴露给未授权访问。 启用前请仔细阅读本页面。

## 何时使用

在以下情况下使用 `trusted-proxy` 认证模式：

- 你在 **身份感知代理**（Pomerium、Caddy + OAuth、nginx + oauth2-proxy、Traefik + forward auth）之后运行 OpenClaw
- 你的代理处理所有认证，并通过请求头传递用户身份
- 你处于 Kubernetes 或容器环境中，且代理是访问 Gateway 的唯一入口
- 由于浏览器无法在 WS 负载中传递令牌，导致出现 WebSocket `1008 unauthorized` 错误

## 何时不应使用

- 如果你的代理不进行用户认证（仅作为 TLS 终止器或负载均衡器）
- 如果存在绕过代理直接访问 Gateway 的路径（防火墙漏洞、内部网络访问）
- 如果你不确定代理是否正确剥离/覆盖转发头
- 如果你只需要个人单用户访问（可考虑使用 Tailscale Serve + loopback 以获得更简单的配置）

## 工作原理

1. 你的反向代理对用户进行认证（OAuth、OIDC、SAML 等）
2. 代理添加包含已认证用户身份的请求头（例如 `x-forwarded-user: nick@example.com`）
3. OpenClaw 会检查请求是否来自 **受信任的代理 IP**（在 `gateway.trustedProxies` 中配置）
4. OpenClaw 从配置的请求头中提取用户身份
5. 如果所有检查都通过，请求将被授权

## 配置

```json5
{
  gateway: {
    // Must bind to network interface (not loopback)
    bind: "lan",

    // CRITICAL: Only add your proxy's IP(s) here
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header containing authenticated user identity (required)
        userHeader: "x-forwarded-user",

        // Optional: headers that MUST be present (proxy verification)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optional: restrict to specific users (empty = allow all)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### 配置参考

| 字段                                          | 必填 | 说明                               |
| ------------------------------------------- | -- | -------------------------------- |
| `gateway.trustedProxies`                    | 是  | 受信任的代理 IP 地址数组。 来自其他 IP 的请求将被拒绝。 |
| `gateway.auth.mode`                         | 是  | 必须为 `"trusted-proxy"`            |
| `gateway.auth.trustedProxy.userHeader`      | 是  | 包含已认证用户身份的请求头名称                  |
| `gateway.auth.trustedProxy.requiredHeaders` | 否  | 请求被信任所必须存在的附加请求头                 |
| `gateway.auth.trustedProxy.allowUsers`      | 否  | 用户身份的允许列表。 为空表示允许所有已认证用户。        |

## 代理设置示例

### Pomerium

Pomerium 会在 `x-pomerium-claim-email`（或其他声明头）中传递身份信息，并在 `x-pomerium-jwt-assertion` 中传递 JWT。

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium 的 IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Pomerium 配置片段：

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### 使用 OAuth 的 Caddy

使用 `caddy-security` 插件的 Caddy 可以对用户进行身份验证并传递身份请求头。

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy 的 IP（如果在同一主机上）
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile 配置片段：

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy 对用户进行身份验证，并在 `x-auth-request-email` 中传递身份信息。

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy 的 IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx 配置片段：

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### 使用 Forward Auth 的 Traefik

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik 容器 IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## 安全检查清单

在启用 trusted-proxy 身份验证之前，请确认：

- [ ] **代理是唯一入口路径**：Gateway 端口已通过防火墙限制，仅允许你的代理访问
- [ ] **trustedProxies 配置最小化**：仅填写实际代理 IP，而不是整个子网
- [ ] **代理会清除请求头**：你的代理会覆盖（而非追加）来自客户端的 `x-forwarded-*` 请求头
- [ ] **TLS 终止**：由你的代理处理 TLS；用户通过 HTTPS 连接
- [ ] **已设置 allowUsers**（推荐）：限制为已知用户，而不是允许任何已认证用户

## 安全审计

`openclaw security audit` 会将 trusted-proxy 身份验证标记为 **严重（critical）** 级别的问题。 这是有意为之——用于提醒你已将安全性委托给代理配置。

审计会检查以下内容：

- 缺少 `trustedProxies` 配置
- 缺少 `userHeader` 配置
- 空的 `allowUsers`（允许任何已认证用户）

## 故障排查

### "trusted_proxy_untrusted_source"

请求并非来自 `gateway.trustedProxies` 中的 IP。 检查：

- 代理 IP 是否正确？ （Docker 容器 IP 可能会变化）
- 你的代理前面是否有负载均衡器？
- 使用 `docker inspect` 或 `kubectl get pods -o wide` 查找实际 IP

### "trusted_proxy_user_missing"

用户请求头为空或缺失。 检查：

- 你的代理是否配置为传递身份标头？
- 标头名称是否正确？ （不区分大小写，但拼写必须正确）
- 用户是否已在代理处完成认证？

### "trusted_proxy_missing_header_\*"

缺少必需的标头。 检查：

- 代理中这些特定标头的配置
- 标头是否在传输链路中的某处被移除

### "trusted_proxy_user_not_allowed"

用户已认证，但不在 `allowUsers` 中。 将其添加到列表中或移除允许列表限制。

### WebSocket 仍然失败

请确保你的代理：

- 支持 WebSocket 升级（`Upgrade: websocket`、`Connection: upgrade`）
- 在 WebSocket 升级请求时传递身份标头（不仅仅是 HTTP）
- 没有为 WebSocket 连接设置单独的认证路径

## 从 Token 认证迁移

如果你正从 token 认证迁移到 trusted-proxy：

1. 配置代理以认证用户并传递标头
2. 独立测试代理设置（使用带标头的 curl）
3. 在 OpenClaw 配置中更新为 trusted-proxy 认证
4. 重启 Gateway
5. 从 Control UI 测试 WebSocket 连接
6. 运行 `openclaw security audit` 并查看结果

## 相关内容

- [Security](/gateway/security) — 完整的安全指南
- [Configuration](/gateway/configuration) — 配置参考
- [Remote Access](/gateway/remote) — 其他远程访问模式
- [Tailscale](/gateway/tailscale) — 仅限 tailnet 访问的更简单替代方案


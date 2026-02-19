---
summary: "將 gateway 驗證委派給受信任的反向代理（Pomerium、Caddy、nginx + OAuth）"
read_when:
  - 在具備身分感知功能的代理後方執行 OpenClaw
  - 在 OpenClaw 前方設定搭配 OAuth 的 Pomerium、Caddy 或 nginx
  - 在反向代理設定中修正 WebSocket 1008 unauthorized 錯誤
---

# 受信任的代理驗證

> ⚠️ **安全性高度敏感的功能。** 此模式會將驗證完全委派給您的反向代理。 設定錯誤可能會讓您的 Gateway 暴露於未經授權的存取。 啟用前請仔細閱讀本頁內容。

## 何時使用

在以下情況下使用 `trusted-proxy` 驗證模式：

- 當您在 **具備身分感知功能的代理**（Pomerium、Caddy + OAuth、nginx + oauth2-proxy、Traefik + forward auth）後方執行 OpenClaw 時
- 您的代理負責所有驗證，並透過標頭傳遞使用者身分資訊
- 您處於 Kubernetes 或容器環境中，且代理是通往 Gateway 的唯一入口
- 因為瀏覽器無法在 WS payload 中傳遞權杖，導致出現 WebSocket `1008 unauthorized` 錯誤

## 何時不應使用

- 如果您的代理未驗證使用者（僅為 TLS 終止或負載平衡器）
- 如果存在任何可繞過代理直接存取 Gateway 的路徑（例如防火牆漏洞、內部網路存取）
- 如果您不確定代理是否正確移除或覆寫轉發標頭
- 如果您僅需個人單一使用者存取（可考慮使用 Tailscale Serve + loopback 以簡化設定）

## 運作方式

1. 您的反向代理會驗證使用者（OAuth、OIDC、SAML 等）。
2. 代理會加入包含已驗證使用者身分的標頭（例如：`x-forwarded-user: nick@example.com`）。
3. OpenClaw 會檢查請求是否來自 **受信任的代理 IP**（於 `gateway.trustedProxies` 中設定）。
4. OpenClaw 會從設定的標頭中擷取使用者身分。
5. 若所有檢查皆通過，則授權該請求。

## 設定

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

### 設定參考

| 欄位                                          | 必填 | 說明                               |
| ------------------------------------------- | -- | -------------------------------- |
| `gateway.trustedProxies`                    | 是  | 要信任的代理 IP 位址陣列。 來自其他 IP 的請求將被拒絕。 |
| `gateway.auth.mode`                         | 是  | 必須為 `"trusted-proxy"`            |
| `gateway.auth.trustedProxy.userHeader`      | 是  | 包含已驗證使用者身分的標頭名稱                  |
| `gateway.auth.trustedProxy.requiredHeaders` | 否  | 請求必須包含的其他標頭，才會被視為可信任             |
| `gateway.auth.trustedProxy.allowUsers`      | 否  | 使用者身分允許清單。 留空表示允許所有已驗證的使用者。      |

## 代理設定範例

### Pomerium

Pomerium 會在 `x-pomerium-claim-email`（或其他 claim 標頭）中傳遞身分資訊，並在 `x-pomerium-jwt-assertion` 中傳遞 JWT。

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

Pomerium 設定範例：

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

搭配 `caddy-security` 外掛的 Caddy 可以驗證使用者並傳遞身分標頭。

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy 的 IP（若在同一主機上）
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile 範例：

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

oauth2-proxy 會驗證使用者，並在 `x-auth-request-email` 中傳遞身分資訊。

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

nginx 設定範例：

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

## 安全性檢查清單

在啟用 trusted-proxy 驗證之前，請確認：

- [ ] **代理是唯一的存取路徑**：Gateway 連接埠已設置防火牆，僅允許你的代理存取
- [ ] **trustedProxies 設定最小化**：僅填入實際代理的 IP，而非整個子網段
- [ ] **代理會移除原有標頭**：你的代理會覆寫（而非附加）來自客戶端的 `x-forwarded-*` 標頭
- [ ] **TLS 終止**：由您的 proxy 處理 TLS；使用者透過 HTTPS 連線
- [ ] **已設定 allowUsers**（建議）：限制為已知使用者，而不是允許任何已通過驗證的人

## 安全性稽核

`openclaw security audit` 會將 trusted-proxy 驗證標記為 **critical** 等級的發現。 這是刻意設計的 —— 提醒您正在將安全性委派給您的 proxy 設定。

稽核會檢查：

- 缺少 `trustedProxies` 設定
- 缺少 `userHeader` 設定
- `allowUsers` 為空（允許任何已通過驗證的使用者）

## 疑難排解

### "trusted_proxy_untrusted_source"

該請求不是來自 `gateway.trustedProxies` 中的 IP。 請檢查：

- proxy 的 IP 是否正確？ （Docker 容器 IP 可能會變動）
- 您的 proxy 前面是否有 load balancer？
- 使用 `docker inspect` 或 `kubectl get pods -o wide` 來查詢實際 IP

### "trusted_proxy_user_missing"

使用者標頭為空或遺失。 請檢查：

- 您的 proxy 是否已設定傳遞身分識別標頭？
- 標頭名稱是否正確？ （不區分大小寫，但拼字必須正確）
- 該使用者是否確實已在 proxy 完成驗證？

### "trusted_proxy_missing_header_\*"

缺少必要的標頭。 請檢查：

- 您的 proxy 對這些特定標頭的設定
- 標頭是否在傳遞鏈中的某處被移除

### "trusted_proxy_user_not_allowed"

使用者已通過驗證，但不在 `allowUsers` 中。 請將其加入，或移除允許清單。

### WebSocket 仍然失敗

請確保您的 proxy：

- 支援 WebSocket 升級（`Upgrade: websocket`、`Connection: upgrade`）
- 在 WebSocket 升級請求時也會傳遞身分識別標頭（不僅僅是 HTTP）
- 沒有為 WebSocket 連線設定獨立的驗證路徑

## 從 Token 驗證遷移

如果您正從 token 驗證移轉到 trusted-proxy：

1. 請設定您的 proxy 以驗證使用者並傳遞標頭
2. 獨立測試代理設定（使用帶有標頭的 curl）
3. 使用 trusted-proxy 驗證更新 OpenClaw 設定
4. 重新啟動 Gateway
5. 從 Control UI 測試 WebSocket 連線
6. 執行 `openclaw security audit` 並檢視結果

## 相關內容

- [Security](/gateway/security) — 完整安全指南
- [Configuration](/gateway/configuration) — 設定參考
- [Remote Access](/gateway/remote) — 其他遠端存取模式
- [Tailscale](/gateway/tailscale) — 僅限 tailnet 存取的更簡單替代方案


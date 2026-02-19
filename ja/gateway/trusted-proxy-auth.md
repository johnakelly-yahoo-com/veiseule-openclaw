---
summary: "Gateway の認証を信頼できるリバースプロキシ（Pomerium、Caddy、nginx + OAuth）に委任する"
read_when:
  - アイデンティティ対応プロキシの背後で OpenClaw を実行する
  - OpenClaw の前段に Pomerium、Caddy、または nginx + OAuth を設定する
  - リバースプロキシ構成で WebSocket 1008 unauthorized エラーを修正する
---

# Trusted Proxy Auth

> ⚠️ **セキュリティに影響する機能です。** このモードでは認証を完全にリバースプロキシに委任します。 設定を誤ると、Gateway が不正アクセスにさらされる可能性があります。 有効化する前に、このページを注意深く読んでください。

## 使用するタイミング

次の場合に `trusted-proxy` 認証モードを使用してください：

- OpenClaw を **アイデンティティ対応プロキシ**（Pomerium、Caddy + OAuth、nginx + oauth2-proxy、Traefik + forward auth）の背後で実行している場合
- プロキシがすべての認証を処理し、ヘッダー経由でユーザーの識別情報を渡します
- Kubernetes やコンテナ環境で、プロキシのみが Gateway への唯一の経路となっている場合
- ブラウザが WS ペイロードでトークンを渡せないため、WebSocket `1008 unauthorized` エラーが発生している場合

## 使用しない場合

- プロキシがユーザーを認証しない場合（単なる TLS 終端やロードバランサーなど）
- プロキシをバイパスして Gateway に到達できる経路が存在する場合（ファイアウォールの穴、内部ネットワークアクセスなど）
- プロキシが転送ヘッダーを正しく削除／上書きしているか確信が持てない場合
- 個人の単一ユーザーアクセスのみが必要な場合（より簡単な構成として Tailscale Serve + loopback を検討してください）

## 仕組み

1. リバースプロキシがユーザーを認証します（OAuth、OIDC、SAML など）
2. プロキシが認証済みユーザーの識別情報を含むヘッダーを追加します（例：`x-forwarded-user: nick@example.com`）
3. OpenClaw はリクエストが**信頼されたプロキシ IP**（`gateway.trustedProxies` で設定）から送信されたことを確認します
4. OpenClaw は設定されたヘッダーからユーザー識別情報を抽出します
5. すべての確認が取れれば、リクエストは認可されます

## 設定

```json5
{
  gateway: {
    // ネットワークインターフェースにバインドする必要があります（loopback ではなく）
    bind: "lan",

    // 重要: ここにはプロキシの IP のみを追加してください
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // 認証済みユーザーの識別情報を含むヘッダー（必須）
        userHeader: "x-forwarded-user",

        // 任意: 必ず存在しなければならないヘッダー（プロキシ検証用）
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // 任意: 特定のユーザーのみに制限（空 = すべて許可）
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### 設定リファレンス

| フィールド                                       | 必須  | 説明                                            |
| ------------------------------------------- | --- | --------------------------------------------- |
| `gateway.trustedProxies`                    | はい  | 信頼するプロキシの IP アドレス配列。 その他の IP からのリクエストは拒否されます。 |
| `gateway.auth.mode`                         | はい  | `"trusted-proxy"` である必要があります                  |
| `gateway.auth.trustedProxy.userHeader`      | はい  | 認証済みユーザーの識別情報を含むヘッダー名                         |
| `gateway.auth.trustedProxy.requiredHeaders` | いいえ | リクエストを信頼するために追加で存在が必要なヘッダー                    |
| `gateway.auth.trustedProxy.allowUsers`      | いいえ | ユーザー識別情報の許可リスト。 空の場合は、認証済みのすべてのユーザーを許可します。    |

## プロキシ設定例

### Pomerium

Pomerium は `x-pomerium-claim-email`（またはその他の claim ヘッダー）で識別情報を渡し、`x-pomerium-jwt-assertion` に JWT を含めます。

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium's IP
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

Pomerium の設定スニペット:

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

### OAuth を使用した Caddy

`caddy-security` プラグインを使用した Caddy は、ユーザーを認証し、ID ヘッダーを渡すことができます。

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy's IP (if on same host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile のスニペット:

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

oauth2-proxy はユーザーを認証し、`x-auth-request-email` に ID 情報を渡します。

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx の設定スニペット:

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

### Forward Auth を使用した Traefik

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik container IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## セキュリティチェックリスト

trusted-proxy 認証を有効にする前に、以下を確認してください:

- [ ] **プロキシのみが経路であること**: Gateway のポートが、プロキシ以外からはファイアウォールで遮断されていること
- [ ] **trustedProxies は最小限に設定すること**: サブネット全体ではなく、実際のプロキシ IP のみを指定すること
- [ ] **プロキシがヘッダーを上書きすること**: プロキシがクライアントからの `x-forwarded-*` ヘッダーを追加ではなく上書きしていること
- [ ] **TLS 終端**: プロキシが TLS を処理し、ユーザーは HTTPS 経由で接続していること
- [ ] **allowUsers を設定すること**（推奨）: 認証済みであれば誰でも許可するのではなく、既知のユーザーのみに制限すること

## セキュリティ監査

`openclaw security audit` は、trusted-proxy 認証を **critical** 重大度の問題として警告します。 これは意図的なものです — セキュリティをプロキシ設定に委ねていることを注意喚起するためです。

監査では以下をチェックします:

- `trustedProxies` の設定漏れ
- `userHeader` の設定漏れ
- `allowUsers` が空（認証済みであれば誰でも許可される状態）

## トラブルシューティング

### "trusted_proxy_untrusted_source"

リクエストが `gateway.trustedProxies` に含まれる IP から送信されていません。 確認事項:

- プロキシの IP は正しいですか？ （Docker コンテナの IP は変更される場合があります）
- プロキシの前段にロードバランサーはありますか？
- 実際の IP を確認するには `docker inspect` または `kubectl get pods -o wide` を使用してください

### "trusted_proxy_user_missing"

ユーザーヘッダーが空、または存在しませんでした。 確認事項:

- プロキシはアイデンティティヘッダーを通過させるように設定されていますか？
- ヘッダー名は正しいですか？ （大文字・小文字は区別されませんが、スペルは重要です）
- ユーザーは実際にプロキシで認証されていますか？

### "trusted_proxy_missing_header_\*"

必要なヘッダーが存在しませんでした。 確認事項：

- 該当するヘッダーに対するプロキシ設定
- チェーンのどこかでヘッダーが削除されていないか

### "trusted_proxy_user_not_allowed"

ユーザーは認証されていますが、`allowUsers` に含まれていません。 ユーザーを追加するか、許可リストを削除してください。

### WebSocket が依然として失敗する場合

プロキシが以下を満たしていることを確認してください：

- WebSocket アップグレード（`Upgrade: websocket`, `Connection: upgrade`）をサポートしている
- WebSocket アップグレードリクエスト時にも（HTTP だけでなく）アイデンティティヘッダーを渡している
- WebSocket 接続に対して別の認証パスを使用していない

## トークン認証からの移行

token auth から trusted-proxy へ移行する場合：

1. プロキシでユーザーを認証し、ヘッダーを渡すように設定する
2. プロキシ設定を個別にテストする（ヘッダー付きの curl）
3. OpenClaw の設定を trusted-proxy 認証に更新する
4. Gateway を再起動する
5. Control UI から WebSocket 接続をテストする
6. `openclaw security audit` を実行し、結果を確認する

## 関連項目

- [Security](/gateway/security) — セキュリティ完全ガイド
- [Configuration](/gateway/configuration) — 設定リファレンス
- [Remote Access](/gateway/remote) — その他のリモートアクセスパターン
- [Tailscale](/gateway/tailscale) — tailnet 限定アクセス向けのよりシンプルな代替手段


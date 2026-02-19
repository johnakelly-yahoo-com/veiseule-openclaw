---
summary: "Gateway サービスの運用、ライフサイクル、およびオペレーションのためのランブック"
read_when:
  - ゲートウェイプロセスを実行またはデバッグする場合
title: "Gateway ランブック"
---

# Gateway サービスランブック

このページは、Gateway サービスの day-1 起動および day-2 運用に使用してください。

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    症状別の診断ガイド（正確なコマンド手順およびログシグネチャ付き）。
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    タスク指向のセットアップガイド + 完全な設定リファレンス。
  
</Card>
</CardGroup>

## 5分でローカル起動

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

正常なベースライン: `Runtime: running` および `RPC probe: ok`。

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway の設定リロードは、アクティブな設定ファイルパス（profile/state のデフォルトから解決、または `OPENCLAW_CONFIG_PATH` が設定されている場合はそれ）を監視します。
デフォルトモード: `gateway.reload.mode="hybrid"`（安全な変更はホット適用、重大な変更は再起動）。
</Note>

## ランタイムモデル

- ルーティング、コントロールプレーン、およびチャネル接続のための常時稼働プロセス。
- 以下のための単一の多重化ポート:
  - WebSocket コントロール/RPC
  - OpenResponses（HTTP）: [`/v1/responses`](/gateway/openresponses-http-api)。
  - コントロール UI とフック
- デフォルトのバインドモード: `loopback`。
- Gateway の認証はデフォルトで必須です。`gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）もしくは `gateway.auth.password` を設定してください。Tailscale Serve のアイデンティティを使用しない限り、クライアントは `connect.params.auth.token/password` を送信する必要があります。 クライアントは Tailscale Serve ID を使用しない限り、`connect.params.auth.token/password` を送信する必要があります。

### ポートとバインドの優先順位

| 設定                                                              | 解決順序                                                          |
| --------------------------------------------------------------- | ------------------------------------------------------------- |
| ベースポート = `gateway.port`（または `OPENCLAW_GATEWAY_PORT` / `--port`） | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| バインドモード                                                         | CLI/override → `gateway.bind` → `loopback`                    |

### ホットリロードモード

| `gateway.reload.mode="off"` で無効化できます。 | 動作                    |
| ------------------------------------- | --------------------- |
| `off`                                 | 設定の再読み込みなし            |
| `hot`                                 | ホットセーフな変更のみ適用         |
| `restart`                             | 再起動が必要な変更時に再起動        |
| `hybrid`（デフォルト）                       | 安全な場合はホット適用、必要な場合は再起動 |

## オペレーターコマンドセット

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## リモートアクセス

推奨: Tailscale/VPN。
フォールバック: SSHトンネル。

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

クライアントはトンネル経由で `ws://127.0.0.1:18789` に接続します。

<Warning>
トークンが設定されている場合、トンネル経由であってもクライアントは `connect.params.auth.token` にトークンを含める必要があります。
</Warning>

参照: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale)。

## 監視およびサービスライフサイクル

本番相当の信頼性を確保するには supervised 実行を使用してください。

<Tabs>
  <Tab title="macOS (launchd)">

```bash
`openclaw gateway stop|restart` — スーパーバイザー管理下の Gateway サービスを停止／再起動（launchd/systemd）。
```

LaunchAgent のラベルは `ai.openclaw.gateway`（デフォルト）または `ai.openclaw.<profile>``（名前付きプロファイル）。 `openclaw doctor\` はサービス設定のドリフトを監査し、修復します。

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

ログアウト後も持続させるには、lingering を有効にしてください。

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

マルチユーザー／常時稼働ホストでは system unit を使用してください。

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## 複数 Gateway（同一ホスト）

ほとんどのセットアップでは **1つ** の Gateway を実行するべきです。
厳密な分離や冗長性が必要な場合（例: レスキュープロファイル）のみ複数を使用してください。

インスタンスごとのチェックリスト:

- 一意の `gateway.port`
- 一意の `OPENCLAW_CONFIG_PATH`
- 一意の `OPENCLAW_STATE_DIR`
- 一意の `agents.defaults.workspace`

例:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

参照: [Multiple gateways](/gateway/multiple-gateways)。

### Dev プロファイル（`--dev`）

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

デフォルトでは分離された state/config とベース Gateway ポート `19001` が含まれます。

## プロトコル（オペレーター視点）

- 最初のクライアントフレームは `connect` である必要があります。
- Gateway は `hello-ok` スナップショット（`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy）を返します。
- リクエスト: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- 一般的なイベント: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`。

Agent の実行は2段階です:

1. 即時の受理 ack（`status:"accepted"`）
2. `agent` のレスポンスは 2 段階です。まず `res` の ack `{runId,status:"accepted"}`、次に実行完了後の最終 `res` `{runId,status:"ok"|"error",summary}`。ストリーミング出力は `event:"agent"` として到着します。

完全なドキュメント: [Gateway protocol](/gateway/protocol) および [Bridge protocol（レガシー）](/gateway/bridge-protocol)。

## 運用チェック

### 稼働確認

- WS を開いて `connect` を送信します。
- スナップショット付きの `hello-ok` レスポンスが返ることを確認してください。

### 準備状況

```bash
`openclaw gateway health|status` — Gateway WS 経由でヘルス／ステータスを要求します。
```

### ギャップリカバリー

イベントは再生されません。 シーケンスにギャップが発生した場合は、続行する前に状態（`health`、`system-presence`）を更新してください。

## 一般的な障害シグネチャ

| シグネチャ                                    | \`refusing to bind gateway ... |
| ---------------------------------------- | ------------------------------------------------------------------------------ |
| without auth\` トークン／パスワードなしでの非ループバックバインド | `another gateway instance is already listening` / `EADDRINUSE`                 |
| ポートの競合                                   | `Gateway start blocked: set gateway.mode=local`                                |
| 設定がリモートモードになっている                         | 接続中に `unauthorized` が発生                                                        |
| クライアントとGateway間の認証不一致                    | 完全な診断手順については、[Gateway Troubleshooting](/gateway/troubleshooting) を参照してください。    |

Gatewayが利用できない場合、Gatewayプロトコルクライアントは即座に失敗します（暗黙のダイレクトチャネルフォールバックはありません）。

## 安全性の保証

- 正常終了時には、ソケットが閉じられる前に `shutdown` イベントが送出されます。
- 接続以外の最初のフレームや不正な JSON は拒否され、ソケットはクローズされます。
- 関連:

---

[Troubleshooting](/gateway/troubleshooting)

- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)
- canvasホストはGateway HTTPサーバー（`gateway.port` と同じポート）で提供されます

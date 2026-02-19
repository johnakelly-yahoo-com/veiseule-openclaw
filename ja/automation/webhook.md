---
summary: "ウェブフックによる起動および分離されたエージェント実行のためのインバウンド"
read_when:
  - ウェブフックエンドポイントを追加または変更する場合
  - 外部システムを OpenClaw に接続する場合
title: "Webhooks"
---

# Webhooks

Gateway（ゲートウェイ）は、外部トリガー用に小規模な HTTP ウェブフックエンドポイントを公開できます。

## 有効化

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

注記:

- `hooks.token` は、`hooks.enabled=true` の場合に必須です。
- `hooks.path` のデフォルトは `/hooks` です。

## 認証

すべてのリクエストにはフックトークンを含める必要があります。ヘッダーの使用を推奨します。 ヘッダーを優先:

- `Authorization: Bearer <token>`（推奨）
- `x-openclaw-token: <token>`
- クエリ文字列のトークンは拒否されます（`?token=...` は `400` を返します）。

## エンドポイント

### `POST /hooks/wake`

ペイロード:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **必須**（string）: イベントの説明（例: 「New email received」）。
- `mode` 任意（`now` | `next-heartbeat`）: 即時ハートビートをトリガーするか（デフォルトは `now`）、次回の定期チェックまで待機するか。

効果:

- **main** セッション向けにシステムイベントをキューに追加します。
- `mode=now` の場合、即時ハートビートをトリガーします。

### `POST /hooks/agent`

ペイロード:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **必須**（string）: エージェントが処理するプロンプトまたはメッセージ。
- `name` 任意（string）: フックの人間可読な名前（例: 「GitHub」）。セッションサマリーのプレフィックスとして使用されます。
- `agentId` は任意（文字列）: このフックを特定のエージェントにルーティングします。 不明な ID はデフォルトエージェントにフォールバックします。 設定されている場合、このフックは解決されたエージェントの workspace と設定を使用して実行されます。
- `sessionKey` 任意（string）: エージェントのセッションを識別するためのキー。デフォルトはランダムな `hook:<uuid>` です。一貫したキーを使用すると、フックコンテキスト内でのマルチターン会話が可能になります。 デフォルトでは、このフィールドは `hooks.allowRequestSessionKey=true` が設定されていない限り拒否されます。
- `wakeMode` 任意（`now` | `next-heartbeat`）: 即時ハートビートをトリガーするか（デフォルトは `now`）、次回の定期チェックまで待機するか。
- `deliver` 任意（boolean）: `true` の場合、エージェントの応答がメッセージングチャンネルに送信されます。デフォルトは `true` です。ハートビート確認のみの応答は自動的にスキップされます。 デフォルトは `true` です。 ハートビート認識のみのレスポンスは自動的にスキップされます。
- `channel` optional (string): 配信のためのメッセージングチャネル。 `channel` optional (string): 配信のためのメッセージングチャネル。 `channel` 任意（string）: 配信先のメッセージングチャンネル。次のいずれか: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost`（plugin）, `signal`, `imessage`, `msteams`。デフォルトは `last` です。 デフォルトは `last` です。 デフォルトは `last` です。
- `to` 任意（string）: チャンネルの受信者識別子（例: WhatsApp/Signal の電話番号、Telegram の chat ID、Discord/Slack/Mattermost（plugin）の channel ID、MS Teams の conversation ID）。デフォルトは main セッションの最後の受信者です。 デフォルトは、メインセッションの最後の受信者です。 デフォルトは、メインセッションの最後の受信者です。
- `model` 任意（string）: モデルのオーバーライド（例: `anthropic/claude-3-5-sonnet` またはエイリアス）。制限されている場合は、許可されたモデルリストに含まれている必要があります。 制限されている場合は、許可されているモデルリスト内にある必要があります。
- `thinking` 任意（string）: 思考レベルのオーバーライド（例: `low`, `medium`, `high`）。
- `timeoutSeconds` 任意（number）: エージェント実行の最大時間（秒）。

効果:

- **分離された** エージェントターンを実行します（独自のセッションキー）。
- 常に **main** セッションにサマリーを投稿します。
- `wakeMode=now` の場合、即時ハートビートをトリガーします。

## セッションキーのポリシー（破壊的変更）

`/hooks/agent` ペイロードの `sessionKey` 上書きはデフォルトで無効化されています。

- 推奨: 固定の `hooks.defaultSessionKey` を設定し、リクエストによるオーバーライドは無効のままにしてください。
- 任意: 必要な場合にのみリクエストのオーバーライドを許可し、プレフィックスを制限してください。

推奨設定:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

互換設定（レガシー動作）:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // 強く推奨
  },
}
```

### `POST /hooks/<name>`（mapped）

カスタムフック名は `hooks.mappings` で解決されます (構成を参照)。 カスタムフック名は `hooks.mappings`（設定を参照）によって解決されます。マッピングにより、
任意のペイロードを `wake` または `agent` アクションに変換できます。テンプレートや
コード変換は任意です。

マッピングオプション（概要）:

- `hooks.presets: ["gmail"]` は、組み込みの Gmail マッピングを有効にします。
- `hooks.mappings` を使用すると、設定内で `match`, `action`, およびテンプレートを定義できます。
- `hooks.transformsDir` + `transform.module` は、カスタムロジック用の JS/TS モジュールを読み込みます。
  - `hooks.transformsDir`（設定している場合）は、OpenClaw の設定ディレクトリ配下の transforms ルート（通常は `~/.openclaw/hooks/transforms`）内に配置する必要があります。
  - `transform.module` は有効な transforms ディレクトリ内で解決される必要があります（ディレクトリトラバーサルやエスケープパスは拒否されます）。
- `match.source` を使用して、汎用的なインジェストエンドポイント（ペイロード駆動のルーティング）を維持できます。
- TS 変換には、TS ローダー（例: `bun` または `tsx`）または、実行時に事前コンパイルされた `.js` が必要です。
- マッピングに `deliver: true` + `channel`/`to` を設定すると、返信をチャットサーフェスへルーティングします
  （`channel` のデフォルトは `last` で、WhatsApp にフォールバックします）。
- `agentId` はフックを特定のエージェントにルーティングします。不明な ID はデフォルトエージェントにフォールバックします。
- `hooks.allowedAgentIds` は明示的な `agentId` ルーティングを制限します。 任意のエージェントを許可するには、これを省略する（または `*` を含める）してください。 明示的な `agentId` ルーティングを拒否するには `[]` を設定してください。
- `sessionKey` 任意（string）: エージェントのセッションを識別するためのキー。デフォルトはランダムな `hook:<uuid>` です。一貫したキーを使用すると、フックコンテキスト内でのマルチターン会話が可能になります。 デフォルトはランダムな `フック:<uuid> ` です。 一貫性のあるキーを使用すると、フックコンテキスト内で複数回会話が可能になります。
- `hooks.allowRequestSessionKey` は `/hooks/agent` のペイロードで `sessionKey` を設定できるかどうかを制御します（デフォルト: `false`）。
- `hooks.allowedSessionKeyPrefixes` は、リクエストペイロードやマッピングから指定される `sessionKey` の値を任意で制限します。
- `allowUnsafeExternalContent: true` は、そのフックに対して外部コンテンツ安全ラッパーを無効化します
  （危険です。信頼された内部ソースのみに使用してください）。
- `openclaw webhooks gmail setup` は、`openclaw webhooks gmail run` 用の `hooks.gmail` 設定を書き込みます。
  Gmail の完全なウォッチフローについては [Gmail Pub/Sub](/automation/gmail-pubsub) を参照してください。
  Gmail のウォッチフローについては、[Gmail Pub/Sub](/automation/gmail-pubsub)を参照してください。
  Gmail のウォッチフローについては、[Gmail Pub/Sub](/automation/gmail-pubsub)を参照してください。

## Responses

- `/hooks/wake` に対して `200`
- `/hooks/agent` に対して `202`（非同期実行が開始されました）
- 認証失敗時は `401`
- 同一クライアントから認証失敗が繰り返された後の `429`（`Retry-After` を確認してください）
- 無効なペイロード時は `400`
- ペイロードが過大な場合は `413`

## Examples

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Use a different model

エージェントのペイロード（またはマッピング）に `model` を追加すると、その実行に対してモデルをオーバーライドできます。

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

`agents.defaults.models` を強制している場合は、オーバーライドするモデルがそこに含まれていることを確認してください。

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Security

- フックエンドポイントは、loopback、tailnet、または信頼されたリバースプロキシの背後に配置してください。
- 専用のフックトークンを使用し、ゲートウェイ認証トークンを再利用しないでください。
- ブルートフォース攻撃を遅らせるため、認証失敗の繰り返しはクライアントアドレスごとにレート制限されます。
- マルチエージェントルーティングを使用する場合は、明示的な `agentId` 選択を制限するために `hooks.allowedAgentIds` を設定してください。
- 呼び出し元がセッションを選択する必要がない限り、`hooks.allowRequestSessionKey=false` を維持してください。
- リクエストの `sessionKey` を有効にする場合は、`hooks.allowedSessionKeyPrefixes`（例: `["hook:"]`）を制限してください。
- ウェブフックログに機密性の高い生のペイロードを含めないようにしてください。
- フックペイロードは、デフォルトで信頼されていないものとして扱われ、安全境界でラップされます。
  特定のフックに対してこれを無効にする必要がある場合は、そのフックのマッピングに `allowUnsafeExternalContent: true`
  を設定してください(危険)。


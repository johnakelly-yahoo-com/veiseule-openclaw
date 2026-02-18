---
title: "設定"
---

# 設定

OpenClaw は、`~/.openclaw/openclaw.json` から任意の <Tooltip tip="JSON5 supports comments and trailing commas">**JSON5**</Tooltip> 設定を読み込みます。

ファイルが存在しない場合、OpenClaw は安全なデフォルトを使用します。設定を追加する一般的な理由：

- チャンネルを接続し、誰がボットにメッセージできるかを制御する
- モデル、ツール、サンドボックス、自動化（cron、hooks）を設定する
- セッション、メディア、ネットワーク、UI を調整する

利用可能なすべてのフィールドについては、[full reference](/gateway/configuration-reference) を参照してください。

<Tip>
**設定が初めてですか？** `openclaw onboard` で対話型セットアップを開始するか、完全なコピーペースト用設定が載っている [Configuration Examples](/gateway/configuration-examples) ガイドをご覧ください。
</Tip>

## Minimal config

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Editing config

<Tabs>
  <Tab title="対話型ウィザード">
    ```bash
    openclaw onboard       # フルセットアップウィザード
    openclaw configure     # 設定ウィザード
    ```
  </Tab>
  <Tab title="CLI（ワンライナー）">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) を開き、**Config** タブを使用します。  
    Control UI は設定スキーマからフォームを生成し、エスケープハッチとして **Raw JSON** エディタを提供します。
  </Tab>
  <Tab title="直接編集">
    `~/.openclaw/openclaw.json` を直接編集します。Gateway はファイルを監視し、変更を自動適用します（[hot reload](#config-hot-reload) を参照）。
  </Tab>
</Tabs>

## Strict validation

<Warning>
OpenClaw はスキーマに完全一致する設定のみを受け付けます。未知のキー、不正な型、無効な値がある場合、Gateway は **起動を拒否** します。ルートレベルでの唯一の例外は `$schema`（string）で、エディタが JSON Schema メタデータを付与するために使用できます。
</Warning>

検証に失敗した場合：

- Gateway は起動しません
- 診断コマンドのみ使用可能です（`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`）
- `openclaw doctor` を実行して正確な問題を確認してください
- `openclaw doctor --fix`（または `--yes`）で修復を適用します

## Common tasks

<AccordionGroup>
  <Accordion title="チャンネルを設定する（WhatsApp、Telegram、Discord など）">
    各チャンネルには `channels.<provider>` 配下に専用の設定セクションがあります。セットアップ手順は各チャンネルページを参照してください：

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    すべてのチャンネルは同じ DM ポリシーパターンを共有します：

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="モデルを選択・設定する">
    プライマリモデルとオプションのフォールバックを設定します：

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` はモデルカタログを定義し、`/model` の許可リストとして機能します。
    - モデル参照は `provider/model` 形式です（例：`anthropic/claude-opus-4-6`）。
    - チャット中のモデル切替は [Models CLI](/concepts/models)、認証ローテーションやフォールバック動作は [Model Failover](/concepts/model-failover) を参照してください。
    - カスタム／セルフホスト型プロバイダーは、リファレンスの [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) を参照してください。

  </Accordion>

  <Accordion title="誰がボットにメッセージできるかを制御する">
    DM アクセスはチャンネルごとに `dmPolicy` で制御します：

    - `"pairing"`（デフォルト）：未知の送信者は承認用のワンタイムペアリングコードを受け取ります
    - `"allowlist"`：`allowFrom`（またはペア済みストア）の送信者のみ許可
    - `"open"`：すべての受信 DM を許可（`allowFrom: ["*"]` が必要）
    - `"disabled"`：すべての DM を無視

    グループの場合は `groupPolicy` + `groupAllowFrom`、またはチャンネル固有の許可リストを使用します。

    詳細は [full reference](/gateway/configuration-reference#dm-and-group-access) を参照してください。

  </Accordion>

  <Accordion title="グループチャットのメンションゲートを設定する">
    グループメッセージはデフォルトで **メンション必須** です。エージェントごとにパターンを設定できます：

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **メタデータメンション**：ネイティブ @メンション（WhatsApp のタップメンション、Telegram の @bot など）
    - **テキストパターン**：`mentionPatterns` の正規表現パターン
    - チャンネル別の上書きやセルフチャットモードは [full reference](/gateway/configuration-reference#group-chat-mention-gating) を参照してください。

  </Accordion>

  <Accordion title="セッションとリセットを設定する">
    セッションは会話の継続性と分離を制御します：

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // マルチユーザーに推奨
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main`（共有）| `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - スコープ、IDリンク、送信ポリシーは [Session Management](/concepts/session) を参照してください。
    - 全フィールドは [full reference](/gateway/configuration-reference#session) を参照してください。

  </Accordion>

  <Accordion title="サンドボックスを有効にする">
    エージェントセッションを分離された Docker コンテナで実行します：

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    まずイメージをビルドします：`scripts/sandbox-setup.sh`

    詳細は [Sandboxing](/gateway/sandboxing)、全オプションは [full reference](/gateway/configuration-reference#sandbox) を参照してください。

  </Accordion>

  <Accordion title="Heartbeat（定期チェックイン）を設定する">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: 期間文字列（`30m`, `2h`）。`0m` で無効化。
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - 詳細は [Heartbeat](/gateway/heartbeat) を参照してください。

  </Accordion>

  <Accordion title="cron ジョブを設定する">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    概要と CLI 例は [Cron jobs](/automation/cron-jobs) を参照してください。

  </Accordion>

  <Accordion title="webhooks（hooks）を設定する">
    Gateway 上で HTTP webhook エンドポイントを有効にします：

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    すべてのマッピングオプションと Gmail 連携は [full reference](/gateway/configuration-reference#hooks) を参照してください。

  </Accordion>

  <Accordion title="マルチエージェントルーティングを設定する">
    個別のワークスペースとセッションを持つ複数の分離エージェントを実行します：

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    バインディングルールやエージェント別アクセスプロファイルは [Multi-Agent](/concepts/multi-agent) および [full reference](/gateway/configuration-reference#multi-agent-routing) を参照してください。

  </Accordion>

  <Accordion title="設定を複数ファイルに分割する（$include）">
    大規模な設定を整理するために `$include` を使用します：

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **単一ファイル**：包含オブジェクトを置換
    - **配列ファイル**：順にディープマージ（後勝ち）
    - **兄弟キー**：include 後にマージ（include を上書き）
    - **ネスト include**：最大 10 階層まで対応
    - **相対パス**：包含元ファイル基準で解決
    - **エラー処理**：未存在ファイル、パースエラー、循環 include を明確に報告

  </Accordion>
</AccordionGroup>

## Config hot reload

Gateway は `~/.openclaw/openclaw.json` を監視し、変更を自動適用します — 多くの設定では手動再起動は不要です。

### Reload modes

| Mode                   | Behavior                                                                                |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (default) | 安全な変更は即時ホット適用。重要な変更は自動再起動。           |
| **`hot`**              | 安全な変更のみホット適用。再起動が必要な場合は警告ログのみ。 |
| **`restart`**          | すべての設定変更で Gateway を再起動。                                 |
| **`off`**              | ファイル監視を無効化。変更は次回手動再起動時に反映。                 |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### What hot-applies vs what needs a restart

ほとんどのフィールドはダウンタイムなしでホット適用されます。`hybrid` モードでは、再起動が必要な変更も自動処理されます。

| Category            | Fields                                                               | Restart needed? |
| ------------------- | -------------------------------------------------------------------- | --------------- |
| Channels            | `channels.*`, `web` (WhatsApp) — すべての組み込み／拡張チャンネル | No              |
| Agent & models      | `agent`, `agents`, `models`, `routing`                               | No              |
| Automation          | `hooks`, `cron`, `agent.heartbeat`                                   | No              |
| Sessions & messages | `session`, `messages`                                                | No              |
| Tools & media       | `tools`, `browser`, `skills`, `audio`, `talk`                        | No              |
| UI & misc           | `ui`, `logging`, `identity`, `bindings`                              | No              |
| Gateway server      | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                 | **Yes**         |
| Infrastructure      | `discovery`, `canvasHost`, `plugins`                                 | **Yes**         |

<Note>
`gateway.reload` と `gateway.remote` の変更は **再起動をトリガーしません**。
</Note>

## Config RPC (programmatic updates)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    設定全体を検証＋書き込みし、1ステップで Gateway を再起動します。

    <Warning>
    `config.apply` は **設定全体を置き換えます**。部分更新には `config.patch`、単一キー変更には `openclaw config set` を使用してください。
    </Warning>

    Params:

    - `raw` (string) — 設定全体の JSON5 ペイロード
    - `baseHash` (optional) — `config.get` からの設定ハッシュ（既存設定がある場合は必須）
    - `sessionKey` (optional) — 再起動後のウェイクアップ ping 用セッションキー
    - `note` (optional) — 再起動センチネル用メモ
    - `restartDelayMs` (optional) — 再起動までの遅延（デフォルト 2000）

    ```bash
    openclaw gateway call config.get --params '{}'  # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```

  </Accordion>

  <Accordion title="config.patch (partial update)">
    既存設定に部分更新をマージします（JSON マージパッチセマンティクス）：

    - オブジェクトは再帰的にマージ
    - `null` はキー削除
    - 配列は置換

    Params:

    - `raw` (string) — 変更対象キーのみ含む JSON5
    - `baseHash` (required) — `config.get` からの設定ハッシュ
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` と同様

    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```

  </Accordion>
</AccordionGroup>

## Environment variables

OpenClaw は親プロセスの環境変数に加えて、以下を読み込みます：

- カレントディレクトリの `.env`（存在する場合）
- `~/.openclaw/.env`（グローバルフォールバック）

どちらも既存の環境変数を上書きしません。設定内でインライン指定も可能です：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  有効時、期待キーが未設定ならログインシェルを実行し、不足キーのみをインポートします：

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var equivalent: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Env var substitution in config values">
  任意の設定文字列内で `${VAR_NAME}` を使用できます：

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Rules:

- 大文字名のみ一致：`[A-Z_][A-Z0-9_]*`
- 未設定／空はロード時エラー
- `$${VAR}` でエスケープ
- `$include` 内でも機能
- インライン例：`"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

完全な優先順位とソースは [Environment](/help/environment) を参照してください。

## Full reference

完全なフィールド別リファレンスは **[Configuration Reference](/gateway/configuration-reference)** を参照してください。

---

_Related: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_

---
summary: "Configurationの概要: よくあるタスク、クイックセットアップ、完全なリファレンスへのリンク"
read_when:
  - OpenClawを初めてセットアップする
  - 一般的な設定パターンを探す
  - 特定の設定セクションに移動する
title: "設定"
---

# 設定 🔧

OpenClaw は、`~/.openclaw/openclaw.json` から任意の **JSON5** 設定を読み込みます（コメントおよび末尾カンマを許可）。

設定されている場合、OpenClaw は（明示的に設定していない場合のみ）次のデフォルトを導出します。 Configを追加する主な理由:

- チャネルを接続し、誰がボットにメッセージを送信できるかを制御する
- モデル、ツール、サンドボックス、または自動化（cron、hooks）を設定する
- セッション、メディア、ネットワーク、またはUIを調整する

利用可能なすべてのフィールドについては、[full reference](/gateway/configuration-reference) を参照してください。

<Tip>
**設定が初めてですか？** 詳細な説明付きの完全な例については、[Configuration Examples](/gateway/configuration-examples) ガイドをご確認ください。
</Tip>

## 最小構成のConfig

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Configの編集

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # full setup wizard
    openclaw configure     # config wizard
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
   
</Tab>
  <Tab title="Control UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) を開き、**Config** タブを使用します。
    Gateway は、UI エディター向けに設定の JSON Schema 表現を `config.schema` 経由で公開します。  
Control UI はこのスキーマからフォームを生成し、エスケープハッチとして **Raw JSON** エディターを提供します。
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/credentials/oauth.json`（または `$OPENCLAW_STATE_DIR/credentials/oauth.json`） Gateway はファイルを監視し、変更を自動的に適用します（[hot reload](#config-hot-reload) を参照）。
  
</Tab>
</Tabs>

## 厳格な設定検証

<Warning>
OpenClawはスキーマと完全に一致する設定のみを受け付けます。 不明なキー、型の不正、または無効な値がある場合、Gateway は**起動を拒否**します。 ルートレベルでの唯一の例外は `$schema`（文字列）で、エディタが JSON Schema メタデータを関連付けられるようにするためのものです。
</Warning>

検証に失敗した場合：

- Gateway は起動しません。
- 診断コマンドのみが許可されます（例：`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`、`openclaw service`、`openclaw help`）。
- 正確な問題点を確認するには `openclaw doctor` を実行してください。
- マイグレーション／修復を適用するには `openclaw doctor --fix`（または `--yes`）を実行してください。

## 一般的なタスク

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    各チャネルには `channels.` の下にそれぞれ独自の設定セクションがあります。<provider>` 配下に置きます。 セットアップ手順については、各チャネルの専用ページを参照してください：

    ```
    33. {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org.com"],
        },
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            GUILD_ID: {
              channels: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          channels: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">
    プライマリモデルとオプションのフォールバックを設定します：

    ````
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
    - モデル参照は `provider/model` 形式を使用します（例：`anthropic/claude-opus-4-6`）。
    - チャットでのモデル切り替えについては [Models CLI](/concepts/models)、認証ローテーションやフォールバック動作については [Model Failover](/concepts/model-failover) を参照してください。
    - カスタム／セルフホスト型プロバイダーについては、リファレンスの [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) を参照してください。
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    DM アクセスは `dmPolicy` によりチャネルごとに制御されます：

    ```
    Groups: `channels.mattermost.groupPolicy="allowlist"` by default (mention-gated). 送信者を制限するには、 `channels.mattermost.groupAllowFrom` を使用してください。
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    グループメッセージはデフォルトで**メンションを必要とします** (メタデータメンションまたは正規表現パターンのいずれか)。 WhatsApp、Telegram、Discord、Googleチャット、iMessageグループチャットに適用されます。 エージェントごとにパターンを設定します：

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">スレッドセッション分離:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // マルチユーザー環境に推奨
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main`（共有） | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - スコープ、ID リンク、送信ポリシーについては [Session Management](/concepts/session) を参照してください。
    - すべてのフィールドについては [full reference](/gateway/configuration-reference#session) を参照してください。
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    分離された Docker コンテナでエージェントセッションを実行します：

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789, auth: { token: "secret" } },
    
      // Common agent defaults
      agents: {
        defaults: {
          sandbox: { mode: "all", scope: "session" },
        },
        // Merge agent lists from all clients
        list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
      },
    
      // Merge broadcast configs
      broadcast: {
        $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
      },
    
      channels: { whatsapp: { groupPolicy: "allowlist" } },
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
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

    ```
    - `every`: 期間文字列（`30m`, `2h`）。無効にするには `0m` を設定します。
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - 詳細は [Heartbeat](/gateway/heartbeat) を参照してください。
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    Cronは、ウェイクアップとスケジュールされたジョブのためのゲートウェイ所有のスケジューラです。 機能の概要と CLI の例については [Cron jobs](/automation/cron-jobs) を参照してください。
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">ゲートウェイHTTPサーバーでシンプルなHTTPWebhookエンドポイントを有効にします。

    ```
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        presets: ["gmail"],
        transformsDir: "~/.openclaw/hooks",
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            wakeMode: "now",
            name: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
            deliver: true,
            channel: "last",
            model: "openai/gpt-5.2-mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">
    個別のワークスペースとセッションで複数の分離されたエージェントを実行します：

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">`$include`ディレクティブを使用して、設定を複数のファイルに分割します。 これは以下の場合に便利です:

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (Config hot reload)

Gateway は `~/.openclaw/openclaw.json` を監視し、変更を自動的に適用します — ほとんどの設定では手動再起動は不要です。

### リロードモード

| モード: | マージ動作                                                  |
| -------------------- | ------------------------------------------------------ |
| **`hybrid`**（デフォルト）  | 安全な変更は即座にホット適用されます。 重要な変更については自動的に再起動します。              |
| …）                   | 安全な変更のみをホット適用します。 再起動が必要な場合は警告をログに出力します — 再起動は手動で行います。 |
| **`restart`**        | 設定変更の内容にかかわらず、Gateway を再起動します。                         |
| `{{Prompt}}`         | ファイル監視を無効にします。 変更は次回の手動再起動時に反映されます。                    |

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### ホット適用されるものと再起動が必要なもの

ほとんどのフィールドはダウンタイムなしでホット適用されます。 `hybrid` モードでは、再起動が必要な変更は自動的に処理されます。

| カテゴリ                                 | フィールド：                                                             | 再起動が必要ですか？   |
| ------------------------------------ | ------------------------------------------------------------------ | ------------ |
| \`channels.<channel> | `channels.*`, `web`（WhatsApp）— すべての組み込みおよび拡張チャネル                   | いいえ          |
| エージェントとモデル                           | `agent`, `agents`, `models`, `routing`                             | いいえ          |
| 自動化                                  | `hooks`, `cron`, `agent.heartbeat`                                 | いいえ          |
| messages                             | `messages.queue`                                                   | いいえ          |
| ツール & メディア       | `tools`, `browser`, `skills`, `audio`, `talk`                      | いいえ          |
| UI & その他         | `ui`, `logging`, `identity`, `bindings`                            | いいえ          |
| Gateway サーバー                         | `gateway` (port/bind/auth/control UI/tailscale) | **ルール：**     |
| インフラストラクチャ                           | `discovery`, `canvasHost`, `plugins`                               | **インライン置換：** |

<Note>
`gateway.reload` と `gateway.remote` は例外です — これらの変更では**再起動はトリガーされません**。
</Note>

## 部分更新（RPC）

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    完全な設定を検証して書き込み、1つのステップで Gateway を再起動します。


    ```
    警告：`config.apply` は **設定全体** を置き換えます。  
    一部のキーのみを変更したい場合は、`config.patch` または `openclaw config set` を使用してください。  
    `~/.openclaw/openclaw.json` のバックアップを保持してください。
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">`config.patch` を使用すると、
関連のないキーをクローブせずに既存の設定に部分的な更新をマージできます。 JSON マージパッチセマンティクスを適用します:

    ````
    - オブジェクトは再帰的にマージされます
    - `null` はキーを削除します
    - 配列は置き換えられます
    
    Params:
    
    - `raw` (string) — 変更するキーのみを含む JSON5
    - `baseHash` (required) — `config.get` から取得した config ハッシュ
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` と同様
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    
    ````

  
</Accordion>
</AccordionGroup>

## 環境変数での指定：

OpenClaw は親プロセスからの環境変数に加えて、次の環境変数を読み取ります:

- カレントワーキングディレクトリにある `.env`（存在する場合）
- `~/.openclaw/.env`（別名 `$OPENCLAW_STATE_DIR/.env`）にあるグローバルフォールバック `.env`

どちらの `.env` ファイルも、既存の環境変数を上書きしません。 また、configにインラインenv varsを指定することもできます。 これらは、
プロセス env がキーを欠落している場合にのみ適用されます (オーバーライドされていないルールと同じ)。

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

<Accordion title="Shell env import (optional)">オプトインの利便性: 有効になっていて、期待されるキーのどれも設定されていない場合 OpenClawはログインシェルを実行し、期待されていないキーのみをインポートします(上書きはありません)。
これにより、シェルプロファイルが効果的に生成されます。

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">
  任意の設定文字列値で `${VAR_NAME}` を使用して環境変数を参照できます:


```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

既定：

- 大文字の環境変数名のみが一致します：`[A-Z_][A-Z0-9_]*`
- 未設定または空の変数は、読み込み時にエラーをスローします
- `$${VAR}` でエスケープすると、リテラルの `${VAR}` を出力します
- `$include` と併用可能（インクルードされたファイルでも置換されます）
- インライン置換: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

優先順位とソースの詳細は [/environment](/help/environment) を参照してください。

## 完全なリファレンス

すべてのフィールドごとの完全なリファレンスについては、**[Configuration Reference](/gateway/configuration-reference)** を参照してください。

---

gateway/configuration.md

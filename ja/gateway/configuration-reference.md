---
title: "構成リファレンス"
description: "~/.openclaw/openclaw.json の完全なフィールド別リファレンス"
---

# 構成リファレンス

`~/.openclaw/openclaw.json` で利用可能なすべてのフィールド。 タスク指向の概要については、[Configuration](/gateway/configuration) を参照してください。

設定形式は **JSON5**（コメントと末尾カンマを許可）です。 すべてのフィールドは任意です — 省略された場合、OpenClaw は安全なデフォルト値を使用します。

---

## チャンネル

各チャンネルは、その設定セクションが存在すると自動的に開始されます（`enabled: false` が指定されていない限り）。

### DM およびグループアクセス

すべてのチャンネルは DM ポリシーとグループポリシーをサポートしています：

| DM ポリシー          | 動作                                       |
| ---------------- | ---------------------------------------- |
| `pairing`（デフォルト） | 不明な送信者には一度限りのペアリングコードが送信され、オーナーの承認が必要です  |
| `allowlist`      | `allowFrom`（またはペアリング済みの許可ストア）に含まれる送信者のみ  |
| `open`           | すべての受信 DM を許可します（`allowFrom: ["*"]` が必要） |
| `disabled`       | すべての受信 DM を無視します                         |

| グループポリシー           | 動作                                      |
| ------------------ | --------------------------------------- |
| `allowlist`（デフォルト） | 設定された許可リストに一致するグループのみ                   |
| `open`             | グループの許可リストをバイパスします（メンションゲートは引き続き適用されます） |
| `disabled`         | すべてのグループ／ルームメッセージをブロックします               |

<Note>
`channels.defaults.groupPolicy` は、プロバイダーの `groupPolicy` が未設定の場合のデフォルトを設定します。
ペアリングコードは 1 時間で有効期限が切れます。 保留中の DM ペアリングリクエストは **各チャンネルあたり最大 3 件** までです。
Slack/Discord には特別なフォールバックがあります：プロバイダーセクションが完全に存在しない場合、実行時のグループポリシーは `open` に解決されることがあります（起動時に警告が表示されます）。
</Note>

### WhatsApp

WhatsApp は Gateway の Web チャンネル（Baileys Web）を通じて動作します。 リンク済みセッションが存在する場合、自動的に開始されます。

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blue ticks (false in self-chat mode)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- 送信コマンドは、`default` アカウントが存在する場合はそれを使用し、存在しない場合は設定済みアカウント ID（ソート順）の最初のものを使用します。
- 従来の単一アカウント Baileys 認証ディレクトリは、`openclaw doctor` によって `whatsapp/default` に移行されます。
- アカウントごとの上書き設定：`channels.whatsapp.accounts.<id>
  `.sendReadReceipts`、`channels.whatsapp.accounts.<id>
  `.dmPolicy`、`channels.whatsapp.accounts.<id>
  `.allowFrom\`。

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Botトークン: `channels.telegram.botToken` または `channels.telegram.tokenFile`。デフォルトアカウントでは `TELEGRAM_BOT_TOKEN` がフォールバックとして使用されます。
- `configWrites: false` は、Telegram から開始される設定の書き込み（スーパーグループIDの移行、`/config set|unset`）をブロックします。
- Telegram のストリームプレビューは `sendMessage` + `editMessageText` を使用します（ダイレクトチャットおよびグループチャットで動作）。
- 再試行ポリシー: [Retry policy](/concepts/retry) を参照してください。

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- トークン: `channels.discord.token`。デフォルトアカウントでは `DISCORD_BOT_TOKEN` がフォールバックとして使用されます。
- 配信先ターゲットには `user:<id>`（DM）または `channel:<id>`（guild チャンネル）を使用してください。数字のみのIDは拒否されます。
- Guild のスラッグは小文字で、スペースは `-` に置き換えます。チャンネルキーにはスラッグ化された名前を使用します（`#` は含めません）。 Guild ID の使用を推奨します。
- Bot が投稿したメッセージはデフォルトで無視されます。 `allowBots: true` で有効化できます（自身のメッセージは引き続き除外されます）。
- `maxLinesPerMessage`（デフォルト 17）は、2000 文字未満でも行数が多いメッセージを分割します。
- `channels.discord.ui.components.accentColor` は、Discord components v2 コンテナのアクセントカラーを設定します。

**リアクション通知モード:** `off`（なし）、`own`（Bot のメッセージ、デフォルト）、`all`（すべてのメッセージ）、`allowlist`（`guilds.<id> .users` に含まれるユーザーのすべてのメッセージ）。

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- サービスアカウント JSON: インライン（`serviceAccount`）またはファイル指定（`serviceAccountFile`）。
- 環境変数のフォールバック: `GOOGLE_CHAT_SERVICE_ACCOUNT` または `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
- 配信先ターゲットには `spaces/<spaceId>` または `users/<userId|email>` を使用してください。

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **Socket モード** では `botToken` と `appToken` の両方が必要です（デフォルトアカウントでは `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` が環境変数フォールバックとして使用されます）。
- **HTTP モード** では `botToken` に加えて `signingSecret`（ルートまたはアカウントごと）が必要です。
- `configWrites: false` は、Slack から開始される設定の書き込みをブロックします。
- 配信先ターゲットには `user:<id>`（DM）または `channel:<id>` を使用してください。

**リアクション通知モード:** `off`、`own`（デフォルト）、`all`、`allowlist`（`reactionAllowlist` から）。

**スレッドセッションの分離:** `thread.historyScope` はスレッド単位（デフォルト）またはチャンネル全体で共有されます。 `thread.inheritParent` は、親チャンネルのトランスクリプトを新しいスレッドにコピーします。

| アクショングループ  | デフォルト | 注記               |
| ---------- | ----- | ---------------- |
| reactions  | 有効    | リアクションの追加 + 一覧取得 |
| messages   | 有効    | 読み取り／送信／編集／削除    |
| ピン         | 有効    | ピン留め／解除／一覧表示     |
| memberInfo | 有効    | メンバー情報           |
| emojiList  | 有効    | カスタム絵文字一覧        |

### Mattermost

Mattermost はプラグインとして提供されています: `openclaw plugins install @openclaw/mattermost`。

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

チャットモード: `oncall`（@メンション時に応答、デフォルト）、`onmessage`（すべてのメッセージに応答）、`onchar`（トリガープレフィックスで始まるメッセージに応答）。

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**リアクション通知モード:** `off`、`own`（デフォルト）、`all`、`allowlist`（`reactionAllowlist` から）。

### iMessage

OpenClaw は `imsg rpc`（stdio 経由の JSON-RPC）を起動します。 デーモンやポートは不要です。

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- Messages の DB へのフルディスクアクセスが必要です。
- `chat_id:<id>` 形式のターゲットを優先してください。 チャット一覧を表示するには `imsg chats --limit 20` を使用します。
- `cliPath` は SSH ラッパーを指定できます。添付ファイルを SCP で取得する場合は `remoteHost` を設定してください。

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### マルチアカウント（全チャネル）

チャネルごとに複数のアカウントを実行できます（それぞれ独自の `accountId` を持ちます）:

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- `accountId` を省略した場合は `default` が使用されます（CLI およびルーティング）。
- 環境変数のトークンは **default** アカウントにのみ適用されます。
- ベースのチャネル設定は、アカウントごとに上書きされない限り、すべてのアカウントに適用されます。
- `bindings[].match.accountId` を使用して、各アカウントを異なるエージェントにルーティングします。

### グループチャットでのメンション制御

グループメッセージはデフォルトで **メンション必須** です（メタデータのメンションまたは正規表現パターン）。 WhatsApp、Telegram、Discord、Google Chat、および iMessage のグループチャットに適用されます。

**メンションの種類:**

- **メタデータメンション**: プラットフォーム標準の @メンション。 WhatsApp のセルフチャットモードでは無視されます。
- **テキストパターン**: `agents.list[].groupChat.mentionPatterns` にある正規表現パターン。 常にチェックされます。
- 検出が可能な場合（ネイティブメンション、または少なくとも1つのパターンがある場合）にのみ、メンション制御が適用されます。

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` はグローバルのデフォルト値を設定します。 チャンネルごとに `channels.<channel> .historyLimit`（またはアカウント単位）で上書きできます。 `0` を設定すると無効になります。

#### DM の履歴制限

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

解決順序: DM ごとの上書き → プロバイダのデフォルト → 制限なし（すべて保持）。

対応: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`。

#### セルフチャットモード

セルフチャットモードを有効にするには、自分の番号を `allowFrom` に含めます（ネイティブの @メンションは無視され、テキストパターンのみに反応します）：

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### コマンド（チャットコマンド処理）

```json5
{
  commands: {
    native: "auto", // サポートされている場合はネイティブコマンドを登録
    text: true, // チャットメッセージ内の /commands を解析
    bash: false, // ! を許可（エイリアス: /bash）
    bashForegroundMs: 2000,
    config: false, // /config を許可
    debug: false, // /debug を許可
    restart: false, // /restart + gateway restart tool を許可
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- テキストコマンドは、先頭が `/` の**単独メッセージ**である必要があります。
- `native: "auto"` は Discord/Telegram でネイティブコマンドを有効にし、Slack では無効のままにします。
- チャンネルごとに上書き: `channels.discord.commands.native`（bool または `"auto"`）。 `false` を設定すると、以前に登録されたコマンドを削除します。
- `channels.telegram.customCommands` は Telegram ボットメニューに追加エントリを加えます。
- `bash: true` で `! <cmd>` をホストシェルで実行できます。 実行には `tools.elevated.enabled` が必要で、送信者が `tools.elevated.allowFrom.<channel>` に含まれている必要があります。
- `config: true` で `/config` を有効化します（`openclaw.json` の読み書き）。
- `channels.<provider> .configWrites` はチャンネルごとに設定変更を制御します（デフォルト: true）。
- `allowFrom` はプロバイダごとの設定です。 設定されている場合、これが**唯一の**認可元となります（チャンネルの allowlist/ペアリングおよび `useAccessGroups` は無視されます）。
- `useAccessGroups: false` を設定すると、`allowFrom` が未設定の場合にコマンドが access-group ポリシーをバイパスできるようになります。

</Accordion>

---

## エージェントのデフォルト設定

### `agents.defaults.workspace`

デフォルト: `~/.openclaw/workspace`。

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

システムプロンプトの Runtime 行に表示されるオプションのリポジトリルート。 未設定の場合、OpenClaw は workspace から上方向にたどって自動検出します。

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

ワークスペースのブートストラップファイル（`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`）の自動作成を無効にします。

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

ワークスペースの各ブートストラップファイルが切り詰められる前の最大文字数。 デフォルト: `20000`。

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

すべてのワークスペースのブートストラップファイルに挿入される合計最大文字数。 デフォルト: `24000`。

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

システムプロンプトのコンテキスト用タイムゾーン（メッセージのタイムスタンプではありません）。 ホストのタイムゾーンにフォールバックします。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

システムプロンプト内で使用される時刻形式。 デフォルト: `auto`（OSの設定に従う）。

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: 形式は `provider/model`（例: `anthropic/claude-opus-4-6`）。 provider を省略した場合、OpenClaw は `anthropic` を前提とします（非推奨）。
- `models`: `/model` 用に設定されたモデルカタログおよび許可リスト。 各エントリには `alias`（ショートカット）および `params`（プロバイダー固有: `temperature`、`maxTokens`）を含めることができます。
- `imageModel`: プライマリモデルが画像入力に対応していない場合のみ使用されます。
- `maxConcurrent`: セッションをまたいで同時に実行できるエージェントの最大数（各セッション内では引き続き直列実行）。 デフォルト: 1。

**組み込みエイリアスのショートハンド**（`agents.defaults.models` にモデルが含まれている場合のみ適用）：

| エイリアス          | モデル                             |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

設定したエイリアスは常にデフォルト設定より優先されます。

Z.AI GLM-4.x モデルは、`--thinking off` を設定するか `agents.defaults.models["zai/<model>"].params.thinking` を自分で定義しない限り、自動的に thinking モードを有効にします。

### `agents.defaults.cliBackends`

テキストのみのフォールバック実行（ツール呼び出しなし）用のオプションの CLI バックエンド。 API プロバイダーが障害を起こした際のバックアップとして有用です。

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- CLI バックエンドはテキスト優先で動作し、ツールは常に無効化されます。
- `sessionArg` が設定されている場合、セッションがサポートされます。
- `imageArg` がファイルパスを受け付ける場合、画像のパススルーがサポートされます。

### `agents.defaults.heartbeat`

定期的なハートビート実行。

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: 期間文字列（ms/s/m/h）。 デフォルト: `30m`。
- エージェントごと: `agents.list[].heartbeat` を設定します。 いずれかのエージェントが `heartbeat` を定義している場合、**そのエージェントのみ** がハートビートを実行します。
- ハートビートはエージェントの完全なターンとして実行されます — 間隔が短いほどより多くのトークンを消費します。

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`: `default` または `safeguard`（長い履歴に対するチャンク分割要約）。 [Compaction](/concepts/compaction) を参照してください。
- `memoryFlush`: 自動 compaction の前に永続的なメモリを保存するためのサイレントなエージェントターン。 ワークスペースが読み取り専用の場合はスキップされます。

### `agents.defaults.contextPruning`

LLM に送信する前に、メモリ内コンテキストから **古いツール結果** を削減します。 ディスク上のセッション履歴は **変更しません**。

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` で pruning パスが有効になります。
- `ttl` は、（最後にキャッシュへアクセスした後）次に pruning を実行できるまでの間隔を制御します。
- Pruning はまずサイズ超過のツール結果をソフトトリムし、必要に応じて古いツール結果をハードクリアします。

**ソフトトリム** は先頭と末尾を保持し、中間に `...` を挿入します。

**ハードクリア** はツール結果全体をプレースホルダーに置き換えます。

注意:

- 画像ブロックはトリム／クリアされることはありません。
- 比率は文字数ベース（概算）であり、正確なトークン数ではありません。
- `keepLastAssistants` 未満の assistant メッセージしか存在しない場合、pruning はスキップされます。

</Accordion>

動作の詳細については [Session Pruning](/concepts/session-pruning) を参照してください。

### ブロックストリーミング

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- Telegram 以外のチャネルでは、ブロック返信を有効にするために明示的に `*.blockStreaming: true` を設定する必要があります。
- チャネルごとの上書き設定: `channels.<channel>``.blockStreamingCoalesce`（およびアカウントごとのバリアント）。 Signal/Slack/Discord/Google Chat のデフォルトは `minChars: 1500`。
- `humanDelay`: ブロック返信間に入るランダムな待機時間。 `natural` = 800～2500ms。 エージェントごとの上書き: `agents.list[].humanDelay`。

動作およびチャンク分割の詳細については [Streaming](/concepts/streaming) を参照してください。

### 入力中インジケーター

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- デフォルト: ダイレクトチャット／メンションありのチャットでは `instant`、メンションのないグループチャットでは `message`。
- セッションごとの上書き: `session.typingMode`, `session.typingIntervalSeconds`。

詳しくは [Typing Indicators](/concepts/typing-indicators) を参照してください。

### `agents.defaults.sandbox`

組み込みエージェント用のオプションの **Docker サンドボックス化**。 完全なガイドは [Sandboxing](/gateway/sandboxing) を参照してください。

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

<Accordion title="Sandbox details">

**ワークスペースアクセス:**

- `none`: `~/.openclaw/sandboxes` 配下にスコープごとのサンドボックス用ワークスペースを作成
- `ro`: サンドボックスのワークスペースを `/workspace` に配置し、エージェントのワークスペースを `/agent` に読み取り専用でマウント
- `rw`: エージェントのワークスペースを `/workspace` に読み書き可能でマウント

**スコープ:**

- `session`: セッションごとにコンテナ + ワークスペースを作成
- `agent`: エージェントごとに 1 つのコンテナ + ワークスペース（デフォルト）
- `shared`: コンテナとワークスペースを共有（セッション間の分離なし）

**`setupCommand`** はコンテナ作成後に一度だけ実行されます（`sh -lc` 経由）。 実行にはネットワークへのアウトバウンド接続、書き込み可能なルート、root ユーザーが必要です。

**コンテナのデフォルトは `network: "none"`** — エージェントにアウトバウンドアクセスが必要な場合は `"bridge"` に設定してください。

**受信添付ファイル** は、アクティブなワークスペース内の `media/inbound/*` に配置されます。

**`docker.binds`** は追加のホストディレクトリをマウントします。グローバル設定とエージェントごとの設定はマージされます。

**サンドボックス化されたブラウザ**（`sandbox.browser.enabled`）: コンテナ内で動作する Chromium + CDP。 noVNC の URL はシステムプロンプトに挿入されます。 メイン設定で `browser.enabled` を有効にする必要はありません。

- `allowHostControl: false`（デフォルト）は、サンドボックス化されたセッションがホストのブラウザを操作対象にすることを防ぎます。
- `sandbox.browser.binds` は追加のホストディレクトリをサンドボックスブラウザコンテナのみにマウントします。 設定された場合（`[]` を含む）、ブラウザコンテナでは `docker.binds` を置き換えます。

</Accordion>

イメージのビルド:

```bash
scripts/sandbox-setup.sh           # メインサンドボックスイメージ
scripts/sandbox-browser-setup.sh   # オプションのブラウザイメージ
```

### `agents.list`（エージェントごとの上書き設定）

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // または { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: 安定したエージェントID（必須）。
- `default`: 複数設定されている場合、最初のものが優先されます（警告がログに記録されます）。 いずれも設定されていない場合、リストの最初のエントリがデフォルトになります。
- `model`: 文字列形式は `primary` のみを上書きします。オブジェクト形式 `{ primary, fallbacks }` は両方を上書きします（`[]` はグローバルなフォールバックを無効化）。
- `identity.avatar`: ワークスペース相対パス、`http(s)` URL、または `data:` URI。
- `identity` はデフォルト値を派生します: `ackReaction` は `emoji` から、`mentionPatterns` は `name`/`emoji` から生成されます。
- `subagents.allowAgents`: `sessions_spawn` に対するエージェントIDの許可リスト（`["*"]` = 任意; デフォルト: 同一エージェントのみ）。

---

## マルチエージェントルーティング

1つの Gateway 内で複数の分離されたエージェントを実行します。 [Multi-Agent](/concepts/multi-agent) を参照してください。

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

### バインディングのマッチフィールド

- `match.channel`（必須）
- `match.accountId`（任意; `*` = 任意のアカウント; 省略時 = デフォルトアカウント）
- `match.peer`（任意; `{ kind: direct|group|channel, id }`）
- `match.guildId` / `match.teamId`（任意; チャネル固有）

**決定的なマッチ順序:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`（完全一致、peer/guild/team なし）
5. `match.accountId: "*"`（チャネル全体）
6. デフォルトエージェント

各ティア内では、最初に一致した `bindings` エントリが優先されます。

### エージェントごとのアクセスプロファイル

<Accordion title="Full access (no sandbox)">

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

</Accordion>

優先順位の詳細については、[Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) を参照してください。

---

## セッション

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // legacy (runtime always uses "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: DM のグルーピング方法。
  - `main`: すべての DM がメインセッションを共有します。
  - `per-peer`: チャネルをまたいで送信者IDごとに分離します。
  - `per-channel-peer`: チャネル + 送信者ごとに分離します（複数ユーザーの受信ボックスに推奨）。
  - `per-account-channel-peer`: アカウント + チャネル + 送信者ごとに分離します（複数アカウントに推奨）。
- **`identityLinks`**: チャネル間でセッションを共有するために、正規IDをプロバイダ接頭辞付きの peer にマッピングします。
- **`reset`**: 主なリセットポリシー。 `daily` はローカル時間の `atHour` にリセットされます。`idle` は `idleMinutes` 経過後にリセットされます。 両方が設定されている場合、先に期限を迎えた方が優先されます。
- **`resetByType`**: タイプ別の上書き設定（`direct`、`group`、`thread`）。 従来の `dm` は `direct` のエイリアスとして受け入れられます。
- **`mainKey`**: 従来のフィールド。 ランタイムは現在、メインのダイレクトチャット用バケットに常に "main" を使用します。
- **`sendPolicy`**: `channel`、`chatType`（`direct|group|channel`、従来の `dm` エイリアスを含む）、`keyPrefix`、または `rawKeyPrefix` によるマッチ。 最初の deny が優先されます。
- **`maintenance`**: `warn` は削除時にアクティブなセッションへ警告します；`enforce` はプルーニングとローテーションを適用します。

</Accordion>

---

Messages
{
messages: {
responsePrefix: "🦞", // または "auto"
ackReaction: "👀",
ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
removeAckAfterReply: false,
queue: {
mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
debounceMs: 1000,
cap: 20,
drop: "summarize", // old | new | summarize
byChannel: {
whatsapp: "collect",
telegram: "collect",
},
},
inbound: {
debounceMs: 2000, // 0 は無効
byChannel: {
whatsapp: 5000,
slack: 1500,
},
},
},
}
-

```json5
レスポンスプレフィックス
```

### チャネル／アカウントごとの上書き: `channels.<channel>
.responsePrefix`, `channels.<channel>
.accounts.<id>
.responsePrefix`.

解決順（最も具体的なものが優先）: account → channel → global。`""` は無効化し、カスケードを停止します。`"auto"` は `[{identity.name}]` を生成します。**テンプレート変数:**

変数 説明 例

`{model}`

| 短いモデル名                                  | `claude-opus-4-6`           | `{modelFull}`                                          |
| --------------------------------------- | --------------------------- | ------------------------------------------------------ |
| 完全なモデル識別子                               | `anthropic/claude-opus-4-6` | `{provider}`                                           |
| プロバイダー名                                 | `anthropic`                 | `{thinkingLevel}`                                      |
| 現在の thinking レベル                        | `high`、`low`、`off`          | `{identity.name}`                                      |
| エージェントの identity 名                      | （`"auto"` と同じ）              | 変数は大文字と小文字を区別しません。                                     |
| `{think}` は `{thinkingLevel}` のエイリアスです。 | Ack リアクション                  | デフォルトではアクティブなエージェントの `identity.emoji`、それ以外の場合は `"👀"`。 |

Variables are case-insensitive. `{think}` is an alias for `{thinkingLevel}`.

### Ack reaction

- Defaults to active agent's `identity.emoji`, otherwise `"👀"`. `""` を設定すると無効になります。
- チャネルごとの上書き: `channels.<channel>`.ackReaction`, `channels.<channel>`.accounts.<id>.ackReaction`.
- 解決順: account → channel → `messages.ackReaction` → identity フォールバック。
- スコープ: `group-mentions`（デフォルト）、`group-all`、`direct`、`all`。
- `removeAckAfterReply`: 返信後に ack を削除します（Slack/Discord/Telegram/Google Chat のみ）。

### インバウンドのデバウンス

同一送信者からのテキストのみの連続メッセージをまとめて、1 回のエージェントターンに統合します。 メディア／添付ファイルは即座にフラッシュされます。 コントロールコマンドはデバウンスをバイパスします。

### TTS（text-to-speech）

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

- `auto` は自動 TTS を制御します。 `/tts off|always|inbound|tagged` はセッションごとに上書きします。
- `summaryModel` は自動サマリー用に `agents.defaults.model.primary` を上書きします。
- API キーは `ELEVENLABS_API_KEY` / `XI_API_KEY` および `OPENAI_API_KEY` にフォールバックします。

---

## Talk

Talk モード（macOS/iOS/Android）のデフォルト設定。

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Voice ID は `ELEVENLABS_VOICE_ID` または `SAG_VOICE_ID` にフォールバックします。
- `apiKey` は `ELEVENLABS_API_KEY` にフォールバックします。
- `voiceAliases` を使用すると、Talk のディレクティブでフレンドリー名を利用できます。

---

## ツール

### ツールプロファイル

`tools.profile` は、`tools.allow` / `tools.deny` の前にベースとなる許可リストを設定します。

| プロファイル      | 含まれるもの                                                                                |
| ----------- | ------------------------------------------------------------------------------------- |
| `minimal`   | `session_status` のみ                                                                   |
| `coding`    | `group:fs`、`group:runtime`、`group:sessions`、`group:memory`、`image`                    |
| `messaging` | `group:messaging`、`sessions_list`、`sessions_history`、`sessions_send`、`session_status` |
| `full`      | 制限なし（未設定と同じ）                                                                          |

### ツールグループ

| グループ               | ツール                                                                                      |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process`（`bash` は `exec` のエイリアスとして使用可能）                                         |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | すべての組み込みツール（プロバイダープラグインを除く）                                                              |

### `tools.allow` / `tools.deny`

グローバルなツール許可／拒否ポリシー（deny が優先）。 大文字・小文字を区別せず、`*` ワイルドカードをサポート。 Docker サンドボックスが無効な場合でも適用されます。

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

特定のプロバイダーやモデルに対してツールをさらに制限します。 順序：ベースプロファイル → プロバイダープロファイル → allow/deny。

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

昇格（ホスト）exec アクセスを制御します：

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- エージェントごとの上書き設定（`agents.list[].tools.elevated`）は、さらに制限する場合にのみ使用できます。
- `/elevated on|off|ask|full` はセッションごとに状態を保存します。インラインディレクティブは単一メッセージにのみ適用されます。
- 昇格された `exec` はホスト上で実行され、サンドボックスをバイパスします。

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // または BRAVE_API_KEY 環境変数
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

受信メディア（画像／音声／動画）の解析を設定します：

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Media model entry fields">

**Provider エントリ**（`type: "provider"` または省略時）：

- `provider`: API プロバイダーID（`openai`、`anthropic`、`google`/`gemini`、`groq` など）
- `model`: モデルIDの上書き指定
- `profile` / `preferredProfile`: 認証プロファイルの選択

**CLI エントリ**（`type: "cli"`）：

- `command`: 実行するコマンド（実行可能ファイル）
- `args`: テンプレート化された引数（`{{MediaPath}}`、`{{Prompt}}`、`{{MaxChars}}` などをサポート）

**共通フィールド：**

- `capabilities`: オプションのリスト（`image`、`audio`、`video`）。 デフォルト: `openai`/`anthropic`/`minimax` → image、`google` → image+audio+video、`groq` → audio。
- `prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`: 各エントリごとの上書き設定。
- 失敗した場合は次のエントリにフォールバックします。

Provider の認証は標準の順序に従います：auth profiles → 環境変数 → `models.providers.*.apiKey`。

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: 生成されるサブエージェントのデフォルトモデル。 省略した場合、サブエージェントは呼び出し元のモデルを継承します。
- サブエージェントごとのツールポリシー: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`。

---

## カスタムプロバイダーとベースURL

OpenClaw は pi-coding-agent のモデルカタログを使用します。 カスタムプロバイダーは、config の `models.providers` または `~/.openclaw/agents/<agentId>/agent/models.json` で追加できます。

```json5
{
  models: {
    mode: "merge", // merge (default) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- カスタム認証が必要な場合は、`authHeader: true` と `headers` を使用します。
- `OPENCLAW_AGENT_DIR`（または `PI_CODING_AGENT_DIR`）でエージェント設定のルートを上書きします。

### プロバイダーの例

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

Cerebras には `cerebras/zai-glm-4.7` を使用し、Z.AI へ直接接続する場合は `zai/glm-4.7` を使用します。

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

`OPENCODE_API_KEY`（または `OPENCODE_ZEN_API_KEY`）を設定します。 ショートカット: `openclaw onboard --auth-choice opencode-zen`。

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

`ZAI_API_KEY` を設定します。 `z.ai/*` および `z-ai/*` はエイリアスとして使用できます。 ショートカット: `openclaw onboard --auth-choice zai-api-key`。

- 一般エンドポイント: `https://api.z.ai/api/paas/v4`
- コーディング用エンドポイント（デフォルト）: `https://api.z.ai/api/coding/paas/v4`
- 一般的なエンドポイントでは、base URL を上書きしてカスタムプロバイダーを定義します。

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

中国向けエンドポイントの場合: `baseUrl: "https://api.moonshot.cn/v1"` または `openclaw onboard --auth-choice moonshot-api-key-cn`。

</Accordion>

<Accordion title="Kimi Coding">

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

Anthropic 互換の組み込みプロバイダー。 ショートカット: `openclaw onboard --auth-choice kimi-code-api-key`。

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Base URL には `/v1` を含めないでください（Anthropic クライアントが自動的に追加します）。 ショートカット: `openclaw onboard --auth-choice synthetic-api-key`。

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

`MINIMAX_API_KEY` を設定します。 ショートカット: `openclaw onboard --auth-choice minimax-api`。

</Accordion>

<Accordion title="Local models (LM Studio)">

[Local Models](/gateway/local-models) を参照してください。 要点: 本格的なハードウェア上で LM Studio Responses API 経由で MiniMax M2.1 を実行し、フォールバック用にホスト型モデルはマージしたままにしておきます。

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: バンドルされたスキルのみに適用される任意の許可リスト（managed/workspace スキルには影響しません）。
- `entries.<skillKey>.enabled: false` は、バンドル済み／インストール済みであってもスキルを無効化します。
- `entries.<skillKey>.apiKey`: 主な環境変数を宣言するスキル向けの簡易設定。

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- `~/.openclaw/extensions`、`<workspace>/.openclaw/extensions`、および `plugins.load.paths` から読み込まれます。
- **設定変更には gateway の再起動が必要です。**
- `allow`: 任意の許可リスト（リストにあるプラグインのみ読み込まれます）。 `deny` が優先されます。

[Plugins](/tools/plugin) を参照してください。

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` は `act:evaluate` と `wait --fn` を無効化します。
- リモートプロファイルはアタッチ専用です（start/stop/reset は無効）。
- 自動検出順: デフォルトブラウザ（Chromium ベースの場合）→ Chrome → Brave → Edge → Chromium → Chrome Canary。
- 制御サービス: ループバックのみ（ポートは `gateway.port` から派生、デフォルトは `18791`）。

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: ネイティブアプリ UI のアクセントカラー（Talk Mode のバブル色など）。
- `assistant`: Control UI の識別情報を上書きします。 アクティブなエージェントの識別情報にフォールバックします。

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local`（gateway を実行）または `remote`（リモート gateway に接続）。 `local` でない限り、Gateway は起動を拒否します。
- `port`: WS + HTTP 用の単一の多重化ポート。 優先順位: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`。
- `bind`: `auto`、`loopback`（デフォルト）、`lan`（`0.0.0.0`）、`tailnet`（Tailscale IP のみ）、または `custom`。
- **Auth**: デフォルトで必須。 loopback 以外にバインドする場合は、共有トークンまたはパスワードが必要です。 オンボーディングウィザードはデフォルトでトークンを生成します。
- `auth.mode: "trusted-proxy"`: 認証を ID 対応リバースプロキシに委任し、`gateway.trustedProxies` からの ID ヘッダーを信頼します（[Trusted Proxy Auth](/gateway/trusted-proxy-auth) を参照）。
- `auth.allowTailscale`: `true` の場合、Tailscale Serve の ID ヘッダーで認証を満たします（`tailscale whois` により検証）。 `tailscale.mode = "serve"` の場合、デフォルトは `true`。
- `auth.rateLimit`: オプションの認証失敗レート制限。 クライアント IP ごと、かつ認証スコープごとに適用されます（shared-secret と device-token は個別に追跡）。 ブロックされた試行は `429` + `Retry-After` を返します。
  - `auth.rateLimit.exemptLoopback` のデフォルトは `true`。localhost のトラフィックにも意図的にレート制限を適用したい場合（テスト環境や厳格なプロキシ構成など）は `false` に設定してください。
- `tailscale.mode`: `serve`（tailnet のみ、loopback バインド）または `funnel`（公開、認証必須）。
- `remote.transport`: `ssh`（デフォルト）または `direct`（ws/wss）。 `direct` の場合、`remote.url` は `ws://` または `wss://` である必要があります。
- `gateway.remote.token` はリモート CLI 呼び出し専用であり、ローカル gateway の認証を有効にはしません。
- `trustedProxies`: TLS を終端するリバースプロキシの IP。 自分で管理しているプロキシのみを指定してください。
- `gateway.tools.deny`: HTTP `POST /tools/invoke` に対して追加でブロックするツール名（デフォルトの deny リストを拡張）。
- `gateway.tools.allow`: デフォルトの HTTP deny リストからツール名を削除します。

</Accordion>

### OpenAI 互換エンドポイント

- Chat Completions: デフォルトでは無効。 `gateway.http.endpoints.chatCompletions.enabled: true` で有効化。
- Responses API: `gateway.http.endpoints.responses.enabled`。
- Responses URL 入力のハードニング:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### マルチインスタンス分離

一意のポートと state ディレクトリを指定して、1 台のホストで複数の gateway を実行できます:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

便利なフラグ: `--dev`（`~/.openclaw-dev` + ポート `19001` を使用）、`--profile <name>`（`~/.openclaw-<name>` を使用）。

[Multiple Gateways](/gateway/multiple-gateways) を参照。

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
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

認証: `Authorization: Bearer <token>` または `x-openclaw-token: <token>`。

**エンドポイント:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - リクエストペイロードの `sessionKey` は、`hooks.allowRequestSessionKey=true` の場合のみ受け付けられます（デフォルト: `false`）。
- `POST /hooks/<name>` → `hooks.mappings` を介して解決されます

<Accordion title="Mapping details">

- `match.path` は `/hooks` 以降のサブパスに一致します（例: `/hooks/gmail` → `gmail`）。
- `match.source` は汎用パスに対してペイロード内のフィールドと一致します。
- `{{messages[0].subject}}` のようなテンプレートはペイロードから値を読み取ります。
- `transform` には、hook アクションを返す JS/TS モジュールを指定できます。
  - `transform.module` は相対パスである必要があり、`hooks.transformsDir` 内に限定されます（絶対パスやディレクトリトラバーサルは拒否されます）。
- `agentId` は特定のエージェントへルーティングします。不明な ID はデフォルトにフォールバックします。
- `allowedAgentIds`: 明示的なルーティングを制限します（`*` または省略 = すべて許可、`[]` = すべて拒否）。
- `defaultSessionKey`: 明示的な `sessionKey` がない hook エージェント実行時に使用される、任意の固定セッションキー。
- `allowRequestSessionKey`: `/hooks/agent` の呼び出し元が `sessionKey` を設定できるようにします（デフォルト: `false`）。
- `allowedSessionKeyPrefixes`: 明示的な `sessionKey` 値（リクエスト + マッピング）に対する任意のプレフィックス許可リスト。例: `["hook:"]`。
- `deliver: true` は最終返信をチャネルに送信します。`channel` のデフォルトは `last` です。
- `model` はこの hook 実行時の LLM を上書きします（モデルカタログが設定されている場合、許可されている必要があります）。

</Accordion>

### Gmail 連携

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- 設定されている場合、Gateway は起動時に `gog gmail watch serve` を自動起動します。 無効にするには `OPENCLAW_SKIP_GMAIL_WATCHER=1` を設定してください。
- Gateway と併せて別途 `gog gmail watch serve` を実行しないでください。

---

## Canvas ホスト

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // または OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Gateway ポート配下で、エージェントが編集可能な HTML/CSS/JS と A2UI を HTTP 経由で提供します:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- ローカル専用: `gateway.bind: "loopback"`（デフォルト）を維持してください。
- loopback 以外にバインドする場合: canvas ルートは、他の Gateway HTTP インターフェースと同様に Gateway 認証（token/password/trusted-proxy）が必要です。
- Node WebView は通常認証ヘッダーを送信しません。ノードがペアリングされ接続されると、Gateway はプライベート IP フォールバックを許可し、URL にシークレットを含めることなくノードが canvas/A2UI を読み込めるようにします。
- 配信される HTML にライブリロードクライアントを挿入します。
- 空の場合、スターター `index.html` を自動作成します。
- `/__openclaw__/a2ui/` で A2UI も提供します。
- 変更を反映するには Gateway の再起動が必要です。
- 大規模ディレクトリや `EMFILE` エラーが発生する場合はライブリロードを無効にしてください。

---

## ディスカバリー

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal`（デフォルト）: TXT レコードから `cliPath` と `sshPort` を省略します。
- `full`: `cliPath` と `sshPort` を含みます。
- Hostname のデフォルトは `openclaw` です。 `OPENCLAW_MDNS_HOSTNAME` で上書きできます。

### ワイドエリア（DNS-SD）

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

`~/.openclaw/dns/` 配下にユニキャスト DNS-SD ゾーンを書き込みます。 クロスネットワーク検出には、DNS サーバー（CoreDNS 推奨）と Tailscale split DNS を組み合わせてください。

セットアップ: `openclaw dns setup --apply`。

---

## 環境

### `env`（インライン環境変数）

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- インライン環境変数は、プロセス環境に同じキーが存在しない場合にのみ適用されます。
- `.env` ファイル: CWD の `.env` と `~/.openclaw/.env`（どちらも既存の変数を上書きしません）。
- `shellEnv`: ログインシェルのプロファイルから、不足している想定キーをインポートします。
- 完全な優先順位については [Environment](/help/environment) を参照してください。

### 環境変数の置換

任意の設定文字列内で `${VAR_NAME}` を使って環境変数を参照できます:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- 一致するのは大文字の名前のみです: `[A-Z_][A-Z0-9_]*`。
- 存在しない、または空の変数は、設定読み込み時にエラーになります。
- リテラルの `${VAR}` を使用するには `$${VAR}` とエスケープしてください。
- `$include` と併用できます。

---

## 認証ストレージ

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- エージェントごとの認証プロファイルは `<agentDir>/auth-profiles.json` に保存されます。
- レガシー OAuth は `~/.openclaw/credentials/oauth.json` からインポートされます。
- [OAuth](/concepts/oauth) を参照してください。

---

## ログ

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- デフォルトのログファイル: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`。
- 安定したパスを使用するには `logging.file` を設定してください。
- `--verbose` 指定時は `consoleLevel` が `debug` に引き上げられます。

---

## ウィザード

CLI ウィザード（`onboard`、`configure`、`doctor`）によって書き込まれるメタデータ:

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## Identity

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

macOS のオンボーディングアシスタントによって書き込まれます。 デフォルト値を導出します:

- `messages.ackReaction` は `identity.emoji` から設定されます（未設定の場合は 👀）。
- `identity.name`/`identity.emoji` から生成される `mentionPatterns`
- `avatar` で指定可能な形式: ワークスペース相対パス、`http(s)` URL、または `data:` URI

---

## Bridge（レガシー、削除済み）

現在のビルドには TCP ブリッジは含まれていません。 ノードは Gateway WebSocket 経由で接続します。 `bridge.*` キーは設定スキーマの一部ではなくなりました（削除するまでバリデーションは失敗します。`openclaw doctor --fix` で不明なキーを削除できます）。

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`: 完了した cron セッションを削除するまで保持する期間。 デフォルト: `24h`。

[Cron Jobs](/automation/cron-jobs) を参照してください。

---

## メディアモデルのテンプレート変数

`tools.media.*.models[].args` 内で展開されるテンプレートプレースホルダー:

| 変数                 | 説明                                      |
| ------------------ | --------------------------------------- |
| `{{Body}}`         | 受信メッセージ本文全体                             |
| `{{RawBody}}`      | 生の本文（履歴や送信者ラッパーなし）                      |
| `{{BodyStripped}}` | グループメンションを除去した本文                        |
| `{{From}}`         | 送信者識別子                                  |
| `{{To}}`           | 宛先識別子                                   |
| `{{MessageSid}}`   | チャネルメッセージ ID                            |
| `{{SessionId}}`    | 現在のセッション UUID                           |
| `{{IsNewSession}}` | 新しいセッションが作成された場合は `"true"`              |
| `{{MediaUrl}}`     | 受信メディアの疑似 URL                           |
| `{{MediaPath}}`    | ローカルのメディアパス                             |
| `{{MediaType}}`    | メディアタイプ（image/audio/document/…）         |
| `{{Transcript}}`   | 音声文字起こし                                 |
| `{{Prompt}}`       | CLIエントリのメディアプロンプトを解決しました                |
| `{{MaxChars}}`     | CLIエントリの最大出力文字数を解決しました                  |
| `{{ChatType}}`     | "direct" or "group"                     |
| `{{GroupSubject}}` | グループの件名（可能な範囲で）                         |
| `{{GroupMembers}}` | グループメンバーのプレビュー（可能な範囲で）                  |
| `{{SenderName}}`   | 送信者の表示名（可能な範囲で）                         |
| `{{SenderE164}}`   | 送信者の電話番号（可能な範囲で）                        |
| `{{Provider}}`     | プロバイダのヒント（whatsapp、telegram、discord など） |

---

## Configに含める（`$include`）

Configを複数ファイルに分割する:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**マージの挙動:**

- 単一ファイル: 含まれているオブジェクトを置き換えます。
- ファイル配列: 順番にディープマージされます（後のものが前のものを上書き）。
- 同階層のキー: includeの後にマージされます（includeされた値を上書き）。
- ネストされたinclude: 最大10階層まで。
- パス: 相対パス（include元ファイル基準）、絶対パス、または `../` による親参照。
- エラー: ファイル未検出、パースエラー、循環includeに対して明確なメッセージを表示します。

---

_関連: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_


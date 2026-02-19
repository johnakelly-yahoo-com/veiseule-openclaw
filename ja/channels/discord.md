---
summary: "Discord ボットのサポート状況、機能、および設定"
read_when:
  - Discord チャンネル機能に取り組むとき
title: "Discord"
---

# Discord（Bot API）

ステータス: 公式 Discord ボットゲートウェイ経由で、DM およびギルドのテキストチャンネルに対応済みです。

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DM はデフォルトでペアリングモードになります。
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    ネイティブコマンドの動作およびコマンドカタログ。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    クロスチャネル診断および修復フロー。
  
</Card>
</CardGroup>

## クイックセットアップ（初心者向け）

<Steps>
  <Step title="Create a Discord bot and enable intents">Discord アプリ + ボットユーザーを作成

    ```
    **Server Members Intent**（推奨。一部のメンバー/ユーザー検索や、ギルド内の許可リスト照合に必要）
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    デフォルトアカウントの Env フォールバック:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">使用したい場所でメッセージを読み書きできる権限を付与して、ボットをサーバーに招待します。

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    ペアリングコードの有効期限は1時間です。
    ```

  
</Step>
</Steps>

<Note>
トークン解決はアカウント単位で行われます。 設定ファイルのトークン値は env フォールバックよりも優先されます。 `DISCORD_BOT_TOKEN` はデフォルトアカウントにのみ使用されます。
</Note>

## モデレーション

- Gateway が Discord 接続を管理します。
- 返信のルーティングは決定的です：Discord からの受信返信は Discord に返されます。
- エージェントは `discord` を呼び出して、次のようなアクションを実行できます:
- ダイレクトチャットはエージェントのメインセッション（デフォルト `agent:main:main`）に集約され、ギルドチャンネルは `agent:<agentId>:discord:channel:<channelId>` として分離されます（表示名は `discord:<guildSlug>#<channelSlug>` を使用）。
- グループ DM はデフォルトで無視されます。`channels.discord.dm.groupEnabled` で有効化し、必要に応じて `channels.discord.dm.groupChannels` で制限します。
- ネイティブコマンドは、共有の `main` セッションではなく、分離されたセッションキー（`agent:<agentId>:discord:slash:<userId>`）を使用します。

## アクセス制御とルーティング

<Tabs>
  <Tab title="DM policy">`dm.policy`: DM のアクセス制御(`ペアリング`を推奨)。 `dm.policy`: DM アクセス制御（`pairing` 推奨）。`"open"` には `dm.allowFrom=["*"]` が必要です。

    ```
    厳格な許可リストにする場合: `channels.discord.dm.policy="allowlist"` を設定し、`channels.discord.dm.allowFrom` に送信者を列挙します。
    ```

  
</Tab>

  <Tab title="Guild policy">挙動は `channels.discord.replyToMode` で制御されます:

    ```
    ネイティブコマンドは、DM/ギルドメッセージと同じ許可リスト（`channels.discord.dm.allowFrom`、`channels.discord.guilds`、チャンネル別ルール）を尊重します。
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    `DISCORD_BOT_TOKEN` のみを設定し、`channels.discord` セクションを作成しない場合、ランタイムは
    `groupPolicy` を `open` にデフォルト設定します。`channels.discord.groupPolicy`、
    `channels.defaults.groupPolicy`、またはギルド/チャンネル許可リストを追加して制限してください。 `channels.discord.groupPolicy` 、
    `channels.defaults.groupPolicy` を追加するか、guild/channel allowlistでロックダウンします。
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild メッセージはデフォルトでメンションが必要です。

    ```
    メンション検出には以下が含まれます:
    
    - 明示的なボットへのメンション
    - 設定されたメンションパターン（`agents.list[].groupChat.mentionPatterns`、フォールバック: `messages.groupChat.mentionPatterns`）
    - サポートされている場合の、ボットへの返信による暗黙的な挙動
    
    `requireMention` はギルド／チャンネルごとに設定します（`channels.discord.guilds...`）。
    
    グループ DM:
    
    - デフォルト: 無視されます（`dm.groupEnabled=false`）
    - `dm.groupChannels`（チャンネル ID またはスラッグ）によるオプションの許可リスト
    ```

  
</Tab>
</Tabs>

### ロールベースのエージェントルーティング

`bindings[].match.roles` を使用して、Discord ギルドメンバーをロール ID に基づいて異なるエージェントへルーティングします。 ロールベースのバインディングはロール ID のみを受け付け、peer または parent-peer バインディングの後、guild-only バインディングの前に評価されます。 バインディングで他の match フィールド（例: `peer` + `guildId` + `roles`）も設定している場合、設定されたすべてのフィールドが一致する必要があります。

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Developer Portal のセットアップ

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord アプリケーション → Bot を作成し、必要なインテント（DM、ギルドメッセージ、メッセージ内容）を有効化して、ボットトークンを取得します。
    ```

  
</Accordion>

  <Accordion title="Privileged intents">**Bot** → **Privileged Gateway Intents** で次を有効化します:

    ```
    - Message Content Intent
    - Server Members Intent（推奨）
    
    Presence intent は任意で、プレゼンス更新を受信したい場合にのみ必要です。ボットのプレゼンス設定（`setPresence`）自体には、メンバーのプレゼンス更新を有効にする必要はありません。
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">アプリ内: **OAuth2** → **URL Generator**

    ```
    - scopes: `bot`, `applications.commands`
    
    一般的な基本権限:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions（任意）
    
    明示的に必要な場合を除き、`Administrator` は避けてください。
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Discord の Developer Mode を有効にし、次をコピーします:

    ```
    - server ID
    - channel ID
    - user ID
    
    信頼性の高い監査およびプローブのために、OpenClaw の設定では数値 ID の使用を推奨します。
    ```

  
</Accordion>
</AccordionGroup>

## ネイティブコマンドとコマンド認可

- `commands.native` のデフォルトは "auto" で、Discord では有効になっています。
- 設定内の `channels.discord.execApprovals.enabled: true`。
- `commands.native=false` を設定すると、以前に登録された Discord ネイティブコマンドが明示的に削除されます。
- ネイティブコマンドの認可は、通常のメッセージ処理と同じ Discord の許可リスト／ポリシーを使用します。
- スラッシュコマンドは、許可リストに含まれないユーザーにも Discord UI 上で表示される場合がありますが、OpenClaw は実行時に許可リストを適用し、「not authorized」と返信します。

詳細は [Exec approvals](/tools/exec-approvals) および [Slash commands](/tools/slash-commands) を参照してください。

## 機能詳細

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord はエージェント出力内の返信タグをサポートしています:

    ```
    `[[reply_to:<id>]]` — コンテキスト/履歴内の特定のメッセージ ID に返信します。現在のメッセージ ID は `[message_id: …]` としてプロンプトに付加されます。履歴エントリには既に ID が含まれています。
      現在のメッセージ ID は `[message_id: …]` としてプロンプトするために追加されます; 履歴エントリはすでにIDを含みます。
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    ギルド履歴コンテキスト:

    ```
    - `channels.discord.historyLimit` のデフォルトは `20`
    - フォールバック: `messages.groupChat.historyLimit`
    - `0` で無効化
    
    DM 履歴の制御:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    スレッドの動作:
    
    - Discord スレッドはチャンネルセッションとしてルーティングされます
    - 親スレッドのメタデータは親セッションとのリンクに使用できます
    - スレッド固有のエントリが存在しない限り、スレッド設定は親チャンネル設定を継承します
    
    チャンネルトピックは **信頼されていない** コンテキストとして挿入されます（system プロンプトとしてではありません）。
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications` を使用します:

    ```
    `guilds.<id> .reactionNotifications`: リアクションシステムのイベントモード（`off`、`own`、`all`、`allowlist`）。
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` は、OpenClaw が受信メッセージを処理中に確認用の絵文字を送信します。

    ```
    解決順序:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - エージェントの identity 絵文字へのフォールバック（`agents.list[].identity.emoji`、なければ "👀"）
    
    注意:
    
    - Discord は Unicode 絵文字またはカスタム絵文字名を受け付けます。
    - チャンネルまたはアカウントでリアクションを無効にするには `""` を使用します。
    ```

  
</Accordion>

  <Accordion title="Config writes">
    チャンネルから開始される設定書き込みはデフォルトで有効です。

    ```
    これは `/config set|unset` フロー（コマンド機能が有効な場合）に影響します。
    
    無効化:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    `channels.discord.proxy` を使用して、Discord gateway WebSocket トラフィックを HTTP(S) プロキシ経由でルーティングします。

```json5
参照は **元の** Discord メッセージ ID（プロキシ前）を使用するため、PK API では 30 分のウィンドウ内でのみ解決されます。
```

    ```
    アカウントごとの上書き設定:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    PluralKit 解決を有効にして、プロキシされたメッセージをシステムメンバーのアイデンティティにマッピングします:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    注意:
    
    - 許可リストでは `pk:<memberId>` を使用できます
    - メンバーの表示名は名前／スラッグで照合されます
    - 参照は元のメッセージ ID を使用し、時間ウィンドウ内に制限されます
    - 参照に失敗した場合、プロキシメッセージはボットメッセージとして扱われ、`allowBots=true` でない限り破棄されます
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    ステータスまたはアクティビティフィールドを設定した場合にのみ、プレゼンス更新が適用されます。

    ```
    ステータスのみの例:
    ```

```json5
または設定: `channels.discord.token: "..."`。
```

    ```
    アクティビティの例（カスタムステータスがデフォルトのアクティビティタイプです）:
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    ストリーミングの例:
    ```

```json5
OpenClaw を `channels.discord.token` で設定します（フォールバックとして `DISCORD_BOT_TOKEN` を使用可能）。
```

    ```
    アクティビティタイプ一覧:
    
    - 0: Playing
    - 1: Streaming（`activityUrl` が必要）
    - 2: Listening
    - 3: Watching
    - 4: Custom（アクティビティテキストをステータス状態として使用；絵文字は任意）
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    DiscordはDM内でボタンベースのexec承認をサポートし、必要に応じて元のチャンネルに承認プロンプトを投稿できます。

    ```
    `execApprovals`: Discord 専用の実行承認 DM（ボタン UI）。`enabled`、`approvers`、`agentFilter`、`sessionFilter` をサポートします。 `enabled` 、 `approvers` 、 `agentFilter` 、 `sessionFilter` に対応しています。
    ```

  
</Accordion>
</AccordionGroup>

## ツールアクション

Discordのメッセージアクションには、メッセージ送信、チャンネル管理、モデレーション、プレゼンス、およびメタデータ操作が含まれます。

主な例：

- `readMessages`、`sendMessage`、`editMessage`、`deleteMessage`
- reactions: `react`, `reactions`, `emojiList`
- `timeout`、`kick`、`ban`
- presence: `setPresence`

アクションゲートは `channels.discord.actions.*` の下にあります。

デフォルトのゲート動作：

| アクショングループ                                                                                                                                                                | デフォルト   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled |
| roleInfo                                                                                                                                                                 | 無効      |
| polls                                                                                                                                                                    | 無効      |
| presence                                                                                                                                                                 | 無効      |

## Components v2 UI

OpenClawは、exec承認およびコンテキスト間マーカーにDiscord components v2を使用します。 Discordのメッセージアクションは、カスタムUI用に `components` も受け付けます（上級者向け；Carbonコンポーネントインスタンスが必要）。従来の `embeds` も引き続き利用可能ですが、推奨されていません。

- `channels.discord.ui.components.accentColor` は、Discordコンポーネントコンテナで使用されるアクセントカラー（hex）を設定します。
- アカウントごとに `channels.discord.accounts.<id> .ui.components.accentColor` で設定します。
- components v2が存在する場合、`embeds` は無視されます。

例：

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
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

## messages

Discordのボイスメッセージは波形プレビューを表示し、OGG/Opus音声とメタデータが必要です。 OpenClawは波形を自動生成しますが、音声ファイルを検査および変換するために、gatewayホスト上で `ffmpeg` と `ffprobe` が利用可能である必要があります。

要件と制約：

- **ローカルファイルパス** を指定してください（URLは拒否されます）。
- テキストコンテンツは省略してください（Discordでは同一ペイロード内でテキストとボイスメッセージを同時に送信できません）。
- すべての音声形式が利用可能です。必要に応じてOpenClawがOGG/Opusに変換します。

例：

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## トラブルシューティング

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    **「Used disallowed intents」**: Developer Portal で **Message Content Intent**（および多くの場合 **Server Members Intent**）を有効化し、ゲートウェイを再起動します。
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    **チャンネルを一切許可しない** 場合は `channels.discord.groupPolicy: "disabled"` を設定します（または空の許可リストを維持します）。
    ```

```bash
まず、`openclaw doctor` と `openclaw channels status --probe` を実行します（対処可能な警告と簡易監査）。
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    一般的な原因：

    ```
    `groupPolicy`: ギルドチャンネルの扱いを制御（`open|disabled|allowlist`）。`allowlist` にはチャンネル許可リストが必要です。
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` の権限チェックは、数値のチャンネルIDでのみ機能します。

    ```
    スラッグキーを使用している場合、実行時のマッチングは機能することがありますが、probeでは権限を完全に検証できません。
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DM が動作しない**: `channels.discord.dm.enabled=false`、`channels.discord.dm.policy="disabled"`、またはまだ承認されていません（`channels.discord.dm.policy="pairing"`）。
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">ボット自身が送信したメッセージはデフォルトで無視されます。許可するには `channels.discord.allowBots=true` を設定します（自分自身のメッセージは引き続き除外されます）。

    ```
    `channels.discord.allowBots=true` を設定する場合は、ループ動作を防ぐために厳密なメンションおよび許可リストルールを使用してください。
    ```

  
</Accordion>
</AccordionGroup>

## 設定リファレンスポインタ

主要リファレンス：

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

重要なDiscordフィールド：

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- `guilds.<id> .channels.<channel> .allow`: `groupPolicy="allowlist"` の場合にチャンネルを許可/拒否。
- command: `commands.native`, `commands.useAccessGroups`, `configWrites`
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## 安全性と運用

- ボットトークンはシークレットとして扱ってください（監視付き環境では `DISCORD_BOT_TOKEN` の使用を推奨）。
- Discord の権限は最小権限の原則で付与してください。
- コマンドのデプロイ／状態が古い場合は、gateway を再起動し、`openclaw channels status --probe` で再確認してください。

## 関連

- [Pairing](/channels/pairing)
- `channels`（チャンネル/カテゴリ/権限の作成・編集・削除）
- [Troubleshooting](/channels/troubleshooting)
- コマンド一覧と設定: [Slash commands](/tools/slash-commands)

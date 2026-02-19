---
summary: "Socket モードまたは HTTP Webhook モード向けの Slack セットアップ"
read_when:
  - Slack をセットアップする場合、または Slack の Socket / HTTP モードをデバッグする場合
title: "Slack"
---

# Slack

ステータス: Slack アプリ連携を通じた DM + チャンネルに対応した本番対応済み。 HTTP モード（Events API）

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">    Slack の DM はデフォルトでペアリングモードになります。
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">    ネイティブコマンドの動作およびコマンドカタログ。
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    クロスチャネル診断および修復プレイブック。
</Card>
</CardGroup>

## クイックセットアップ（初心者向け）

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        Slack アプリ設定で:

        ```
        **ソケットモード** → オンに切り替え **Socket Mode** → 有効化。次に **Basic Information** → **App-Level Tokens** → スコープ `connections:write` を指定して **Generate Token and Scopes** を実行します。**App Token**（`xapp-...`）をコピーします。 **App Token** (`xapp-...`) をコピーします。
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            環境変数フォールバック（デフォルトアカウントのみ）:
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="アプリイベントを購読">
          以下のボットイベントを購読します:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          さらに、DM 用に App Home の **Messages Tab** を有効にします。
        
</Step>
      
        <Step title="gateway を起動">
      

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        Gateway（ゲートウェイ）が HTTPS 経由で Slack から到達可能な場合（サーバー配備が一般的）に HTTP Webhook モードを使用します。HTTP モードは、Events API + Interactivity + Slash Commands を共通のリクエスト URL で使用します。
        HTTP モードでは、イベント API + Interactivity + Slash コマンドと共有リクエストの URL を使用します。
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      マルチアカウントの HTTP モード: `channels.slack.accounts.<id> .mode = "http"` を設定し、アカウントごとに一意の `webhookPath` を指定して、各 Slack アプリが固有の URL を指すようにします。

  
</Tab>
</Tabs>

## トークンモデル

- Socket Mode には `botToken` と `appToken` が必要です。
- HTTP モードには `botToken` と `signingSecret` が必要です。
- 設定済みトークンは環境変数フォールバックより優先されます。
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` の環境変数フォールバックはデフォルトアカウントにのみ適用されます。
- userTokenReadOnly を明示的に設定した例（ユーザートークンでの書き込みを許可）:
- 任意: 送信メッセージでアクティブなエージェントのアイデンティティ（カスタム `username` とアイコン）を使用したい場合は、`chat:write.customize` を追加してください。 `icon_emoji` は `:emoji_name:` 構文を使用します。

<Tip>
アクションやディレクトリ読み取りでは、設定されている場合はユーザートークンが優先されることがあります。 `userTokenReadOnly: false` の場合でも、利用可能であれば書き込みにはボットトークンが優先されます。
</Tip>

## アクセス制御とルーティング

<Tabs>
  <Tab title="DM policy">誰でも許可する場合: `channels.slack.dm.policy="open"` と `channels.slack.dm.allowFrom=["*"]` を設定します。

    ```
    `allowlist` は、チャンネルが `channels.slack.channels` に列挙されていることを要求します。
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` はチャンネルの扱い（`open|disabled|allowlist`）を制御します。

    ```
    `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` のみを設定し、`channels.slack` セクションを作成しない場合、実行時の既定で `groupPolicy` は `open` になります。制限するには `channels.slack.groupPolicy`、`channels.defaults.groupPolicy`、またはチャンネル許可リストを追加してください。 `channels.slack.groupPolicy` 、
    `channels.defaults.groupPolicy` を追加するか、チャンネルをロックするための許可リストを追加します。
    ```

  
</Tab>

  <Tab title="Mentions and channel users">    チャンネルメッセージはデフォルトでメンションによるゲート制御が行われます。

    ```
    メンション制御は `channels.slack.channels` で管理されます（`requireMention` を `true` に設定）。`agents.list[].groupChat.mentionPatterns`（または `messages.groupChat.mentionPatterns`）もメンションとして扱われます。
    ```

  
</Tab>
</Tabs>

## コマンドとスラッシュコマンドの動作

- ネイティブコマンド登録は `commands.native` を使用します（グローバル既定 `"auto"` → Slack はオフ）。`channels.slack.commands.native` によりワークスペース単位で上書きできます。テキストコマンドは単独の `/...` メッセージを必要とし、`commands.text: false` で無効化できます。Slack の Slash コマンドは Slack アプリ側で管理され、自動削除されません。コマンドのアクセスグループチェックを回避するには `commands.useAccessGroups: false` を使用します。 テキストコマンドはスタンドアロンの `/...` メッセージを必要とし、`commands.text: false` で無効にできます。 SlackスラッシュコマンドはSlackアプリで管理され、自動的には削除されません。 アクセスグループのコマンドチェックを回避するには、`commands.useAccessGroups: false` を使用します。
- スラッシュコマンド → `channels.slack.slashCommand` を使用する場合は、 `/openclaw` を作成します。 ネイティブコマンドを有効にする場合は、組み込みコマンド(`/help`と同じ名前)ごとにスラッシュコマンドを1つ追加してください。 Slash Commands → `channels.slack.slashCommand` を使用する場合は `/openclaw` を作成します。ネイティブコマンドを有効にする場合、組み込みコマンドごとに 1 つの Slash コマンドを追加します（`/help` と同名）。Slack では、`channels.slack.commands.native: true` を設定しない限りネイティブはデフォルトでオフです（グローバル `commands.native` の既定値は `"auto"` で、Slack はオフのままです）。
- ネイティブコマンドが有効な場合、Slack に対応するスラッシュコマンド（`/<command>` 名）を登録してください。
- ネイティブコマンドが有効でない場合、`channels.slack.slashCommand` を介して設定された単一のスラッシュコマンドを実行できます。

デフォルトのスラッシュコマンド設定:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

スラッシュセッションは分離されたキーを使用します:

- Slash コマンドは `agent:<agentId>:slack:slash:<userId>` セッションを使用します（プレフィックスは `channels.slack.slashCommand.sessionPrefix` で設定可能）。

また、コマンド実行は対象の会話セッション（`CommandTargetSessionKey`）に対してルーティングされます。

## スレッド、セッション、および返信タグ

- グループ DM はスレッド化し、チャンネルはルートに保持:
- デフォルトの `session.dmScope=main` では、Slack DM はエージェントのメインセッションに統合されます。
- チャンネルは `agent:<agentId>:slack:channel:<channelId>` セッションにマップされます。
- スレッド返信は、該当する場合にスレッドセッションのサフィックス（`:thread:<threadTs>`）を作成できます。
- `channels.slack.thread.historyScope` のデフォルトは `thread`、`thread.inheritParent` のデフォルトは `false` です。
- `channels.slack.thread.initialHistoryLimit` は、新しいスレッドセッション開始時に取得する既存スレッドメッセージ数を制御します（デフォルトは `20`。無効にするには `0` に設定）。

返信のスレッド化

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- ダイレクトチャット用のレガシーフォールバック: `channels.slack.dm.replyToMode`

手動返信タグがサポートされています:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — 特定のメッセージ ID に返信します。

注意: `replyToMode="off"` は暗黙的な返信スレッドを無効にします。 明示的な `[[reply_to_*]]` タグは引き続き有効です。

## メディア、チャンク分割、および配信

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack のファイル添付は、Slack がホストするプライベート URL（トークン認証リクエストフロー）からダウンロードされ、取得に成功しサイズ制限内であればメディアストアに保存されます。

    ```
    メディアアップロードは `channels.slack.mediaMaxMb` で上限が設定されます（デフォルト 20）。
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">送信テキストは `channels.slack.textChunkLimit` まで分割されます（デフォルト 4000）。
</Accordion>

  <Accordion title="Delivery targets">
    推奨される明示的なターゲット:

    ```
    - DM には `user:<id>`
    - チャンネルには `channel:<id>`
    
    ユーザー宛に送信する場合、Slack conversation APIs を使用して Slack DM が開始されます。
    ```

  
</Accordion>
</AccordionGroup>

## アクションとゲート

Slack のツールアクションは `channels.slack.actions.*` で制御できます。

現在の Slack ツールで利用可能なアクショングループ:

| グループポリシー   | デフォルト   |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

## イベントおよび動作仕様

- `message.*`（編集／削除／スレッドブロードキャストを含む）
- リアクションの追加／削除イベントはシステムイベントにマッピングされます。
- メンバーの参加／退出、チャンネルの作成／名称変更、ピンの追加／削除イベントはシステムイベントにマッピングされます。
- `channel_id_changed` は、`configWrites` が有効な場合にチャンネル設定キーを移行できます。
- チャンネルのトピック／目的メタデータは信頼されていないコンテキストとして扱われ、ルーティングコンテキストに挿入される可能性があります。

## 確認用リアクション

`ackReaction` は、OpenClaw が受信メッセージを処理中に確認用の絵文字を送信します。

解決順序:

- `または`channels.slack.channels.<name>`.ackReaction`
- `channels.slack.ackReaction`
- `replyToMode`
- エージェント識別絵文字のフォールバック（`agents.list[].identity.emoji`、未設定の場合は "👀"）

注記

- Slack はショートコード（例: `"eyes"`）を想定しています。
- チャンネルまたはアカウントでリアクションを無効にするには `""` を使用します。

## マニフェストおよびスコープのチェックリスト

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    `channels.slack.userToken` を設定する場合、一般的な読み取りスコープは次のとおりです:

    ```
    `channels:history`、`groups:history`、`im:history`、`mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## トラブルシューティング

<AccordionGroup>
  <Accordion title="No replies in channels">
    次の順で確認してください:

    ```
    `users`: 任意のチャンネル単位ユーザー許可リスト。
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    確認:

    ```
    **チャンネルを一切許可しない** 場合は `channels.slack.groupPolicy: "disabled"` を設定します（または空の許可リストを維持します）。
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">Slack アプリを作成し、**Socket Mode** を有効にします。
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    検証:

    ```
    **Event Subscriptions** → イベントを有効化し、**Request URL** にゲートウェイの Webhook パス（デフォルト `/slack/events`）を設定します。
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    意図した設定かどうかを確認してください:

    ```
    ネイティブコマンドを有効にする場合、公開したいコマンドごとに `slash_commands` エントリを 1 つ追加します（`/help` の一覧と一致させます）。`channels.slack.commands.native` で上書きできます。 `channels.slack.commands.native` で上書きします。
    ```

  
</Accordion>
</AccordionGroup>

## 設定リファレンスへの参照

優先順位:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  重要な Slack フィールド:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM アクセス: `dm.enabled`, `dmPolicy`, `allowFrom`（旧: `dm.policy`, `dm.allowFrom`）, `dm.groupEnabled`, `dm.groupChannels`
  - `allow`: `groupPolicy="allowlist"` の場合にチャンネルを許可／拒否します。
  - スレッド／履歴: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - 配信: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - 運用／機能: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## 関連

- [Pairing](/channels/pairing)
- `channel`: 通常チャンネル（公開／非公開）
- 切り分けフロー: [/channels/troubleshooting](/channels/troubleshooting)。
- [Configuration](/gateway/configuration)
- 完全なコマンド一覧と設定: [Slash commands](/tools/slash-commands)

---
summary: "Telegram ボットのサポート状況、機能、および設定"
read_when:
  - Telegram 機能や webhook に取り組むとき
title: "Telegram"
---

# Telegram（Bot API）

状態:gramY経由でボットDM+グループのプロダクション準備ができています。 デフォルトでロングポーリング。webhookは任意。 ロングポーリングがデフォルトモードです。Webhook モードは任意です。

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">    Telegram のデフォルト DM ポリシーは pairing です。
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    クロスチャネルの診断および修復プレイブック。
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">    完全なチャネル設定パターンと例。
</Card>
</CardGroup>

## クイックセットアップ（初心者向け）

<Steps>
  <Step title="Create the bot token in BotFather">Telegram を開き、**@BotFather**（[直リンク](https://t.me/BotFather)）とチャットします。ハンドルが正確に `@BotFather` であることを確認します。 ハンドルが正確に `@BotFather` であることを確認します。

    ```
    `/newbot` を実行し、指示に従います（名前 + `bot` で終わるユーザー名）。
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Env フォールバック: `TELEGRAM_BOT_TOKEN=...`（デフォルトアカウントのみ）。
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    ペアリングコードは 1 時間で期限切れになります。
    ```

  
</Step>

  <Step title="Add the bot to a group">    ボットをグループに追加し、その後 `channels.telegram.groups` と `groupPolicy` をアクセスモデルに合わせて設定します。
</Step>
</Steps>

<Note>
トークンの解決順はアカウント対応です。 実際には、config の値が env フォールバックより優先され、`TELEGRAM_BOT_TOKEN` はデフォルトアカウントにのみ適用されます。
</Note>

## Telegram 側の設定

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Telegram ボットはデフォルトで **プライバシーモード** が有効で、受信できるグループメッセージが制限されます。
ボットが _すべて_ のグループメッセージを見る必要がある場合、次の 2 つの方法があります。
Botが_all_グループメッセージを表示する必要がある場合は、2つのオプションがあります。

    ```
    `/setprivacy` — ボットがすべてのグループメッセージを見るかどうかを制御します。
    ```

  
</Accordion>

  <Accordion title="Group permissions">管理ステータスはグループ内(テレグラムUI)に設定されています。 管理者ステータスはグループ内（Telegram UI）で設定します。管理者ボットは常にすべての
グループメッセージを受信するため、完全な可視性が必要な場合は管理者を使用してください。

    ```
    ボットをグループ **管理者** として追加（管理者ボットはすべてのメッセージを受信します）。
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — ボットをグループに追加できるかを許可／拒否します。
    ```

  
</Accordion>
</AccordionGroup>

## アクセス制御とアクティベーション

<Tabs>
  <Tab title="DM policy">    `channels.telegram.dmPolicy` はダイレクトメッセージのアクセスを制御します。

    ```
    `channels.telegram.allowFrom` は数値のユーザー ID（推奨）または `@username` エントリを受け付けます。ボットのユーザー名ではありません。人間の送信者の ID を使用してください。ウィザードは `@username` を受け付け、可能な場合は数値 ID に解決します。 ボットのユーザー名ではありません。人間の送信者のIDを使用してください。 ウィザードは `@username` を受け取り、可能な場合は数値IDで解決します。
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    サードパーティ方式（プライバシーはやや低い）: `@userinfobot` または `@getidsbot`。
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">2 つの独立した制御があります。

    ```
    `channels.telegram.groupAllowFrom`: グループ送信者許可リスト（ID／ユーザー名）。
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">    グループ返信はデフォルトでメンションが必要です。

メンションは以下のいずれかから行えます:

- ネイティブの `@botusername` メンション、または
- 次のメンションパターン:
  - `agents.list[].groupChat.mentionPatterns`
  - `messages.groupChat.mentionPatterns`

セッションレベルのコマンド切り替え:

- `/activation always`
- `/activation mention`

これらはセッション状態のみを更新します。永続化するには config を使用してください。

永続的な設定例:

    ```
      
</Tab>
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }, // all groups, always respond
      },
    },
  },
}
```

    ```
    グループ内の任意のメッセージを `@userinfobot` または `@getidsbot` に転送すると、チャット ID（`-1001234567890` のような負の数）が表示されます。
    ```

  
</Tab>
</Tabs>

## 動作の仕組み（挙動）

- Gateway（ゲートウェイ）が所有する Telegram Bot API チャンネルです。
- 決定的ルーティング: 返信は必ず Telegram に戻り、モデルがチャンネルを選択することはありません。
- 受信メッセージは、返信コンテキストとメディアプレースホルダーを含む共有チャンネルエンベロープに正規化されます。
- グループチャット ID の取得 各トピックを分離するため、Telegram グループのセッションキーに `:topic:<threadId>` を付加します。
- プライベートチャットには、 `message_thread_id` をいくつかのエッジケースに含めることができます。 プライベートチャットでも、まれに `message_thread_id` が含まれる場合があります。OpenClaw は DM セッションキーを変更しませんが、存在する場合は返信／ドラフトストリーミングにスレッド ID を使用します。
- ロングポーリングは grammY runner を使用し、チャットごとのシーケンシングを行います。全体の同時実行数は `agents.defaults.maxConcurrent` で上限が設定されます。 機能リファレンス
- Telegram Bot API は既読通知をサポートしていないため、`sendReadReceipts` オプションはありません。

## 要件:- `channels.telegram.streamMode` が `"off"` ではないこと（デフォルト: `"partial"`）モード:- `off`: ライブプレビューなし
- `partial`: 部分テキストから頻繁にプレビュー更新
- `block`: `channels.telegram.draftChunk` を使用したチャンク単位のプレビュー更新`streamMode: "block"` の場合の `draftChunk` デフォルト値:- `minChars: 200`
- `maxChars: 800`
- `breakPreference: "paragraph"``maxChars` は `channels.telegram.textChunkLimit` によって制限されます。これはダイレクトチャットおよびグループ／トピックで動作します。テキストのみの返信では、OpenClaw は同じプレビューメッセージを維持し、最終的にその場で編集します（2通目のメッセージは送信しません）。複雑な返信（例: メディアペイロード）の場合、OpenClaw は通常の最終配信にフォールバックし、その後プレビューメッセージをクリーンアップします。`streamMode` はブロックストリーミングとは別です。Telegram 用にブロックストリーミングが明示的に有効な場合、OpenClaw は二重ストリーミングを避けるためにプレビューストリームをスキップします。Telegram 専用の推論ストリーム:- `/reasoning stream` は生成中の推論をライブプレビューに送信します
- 最終回答は推論テキストなしで送信されます

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw は `sendMessageDraft` を使用して Telegram DM で部分的な返信をストリーミングできます。

    ```
    - Markdown 風テキストは Telegram で安全な HTML にレンダリングされます。
    - 生のモデル HTML は Telegram の解析失敗を減らすためにエスケープされます。
    - Telegram が解析済み HTML を拒否した場合、OpenClaw はプレーンテキストとして再試行します。
    
    リンクプレビューはデフォルトで有効で、`channels.telegram.linkPreview: false` で無効化できます。
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">送信される Telegram テキストは `parse_mode: "HTML"`（Telegram がサポートするタグのサブセット）を使用します。

    ```
        Telegram コマンドメニューの登録は起動時に `setMyCommands` で処理されます。
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">ルール:

- 名前は正規化されます（先頭の `/` を削除し、小文字化）
- 有効なパターン: `a-z`、`0-9`、`_`、長さ `1..32`
- カスタムコマンドはネイティブコマンドを上書きできません
- 競合／重複はスキップされ、ログに記録されます

注意:

- カスタムコマンドはメニュー項目のみであり、動作は自動実装されません
- プラグイン／Skill のコマンドは、Telegram メニューに表示されていなくても入力すれば動作します

ネイティブコマンドが無効化されている場合、ビルトインは削除されます。設定されていれば、カスタム／プラグインコマンドは引き続き登録できます。

よくあるセットアップ失敗:

- `setMyCommands failed` は通常、`api.telegram.org` への外向き DNS/HTTPS がブロックされていることを意味します。

### デバイスペアリングコマンド（`device-pair` プラグイン）

`device-pair` プラグインがインストールされている場合:

1. `/pair` でセットアップコードを生成
2. iOS アプリにコードを貼り付け
3. `/pair approve` で最新の保留リクエストを承認

詳細: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios)。

    ```
    `commands.native`（デフォルトは `"auto"` → Telegram／Discord でオン、Slack でオフ）、`commands.text`、`commands.useAccessGroups`（コマンド挙動）。`channels.telegram.commands.native` で上書き可能。 `channels.telegram.commands.native` で上書きします。
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
        インラインキーボードのスコープを設定:
    ```

  
</Accordion>

  <Accordion title="Inline buttons">    Telegram のツールアクションには以下が含まれます:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    アカウントごとの設定:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    デフォルト: `allowlist`
    デフォルト: `allowlist`。
    レガシー: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`。
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    ユーザーがボタンをクリックすると、コールバックデータが次の形式のメッセージとしてエージェントに送信されます。
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">    ### 音声メッセージ

    ```
    ツール: `telegram` の `deleteMessage` アクション（`chatId`、`messageId`）。
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram はタグによる任意のスレッド返信をサポートしています。

    ```
    `[[reply_to_current]]` -- トリガーとなったメッセージに返信。
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">トピック（フォーラムスーパーグループ）

    ```
    **フォーラムグループ:** フォーラムグループ内のリアクションには `message_thread_id` が含まれ、`agent:main:telegram:group:{chatId}:topic:{threadId}` のようなセッションキーが使用されます。これにより、同一トピック内のリアクションとメッセージが一緒に扱われます。 これにより、同じトピックのリアクションとメッセージが一緒に保たれます。
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">ビデオノートはキャプションをサポートしていません。提供されたメッセージテキストは別途送信されます。

### ステッカー

受信ステッカーの処理:

- 静的 WEBP: ダウンロードして処理（プレースホルダー `<media:sticker>`）
- アニメーション TGS: スキップ
- 動画 WEBM: スキップ

ステッカーのコンテキストフィールド:

- `Sticker.emoji`
- `Sticker.setName`
- `Sticker.fileId`
- `Sticker.fileUniqueId`
- `Sticker.cachedDescription`

ステッカーキャッシュファイル:

- `~/.openclaw/telegram/sticker-cache.json`

ステッカーは（可能な場合）一度だけ説明が生成され、繰り返しのビジョン呼び出しを減らすためにキャッシュされます。

ステッカーアクションを有効化:

    ```
    `[[audio_as_voice]]` — ファイルではなくボイスノートとして音声を送信します。
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    動画メッセージ（動画 vs ビデオノート）
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    有効にすると、OpenClaw は次のようなシステムイベントをキューに追加します:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    設定:
    
    - `channels.telegram.reactionNotifications`: `off | own | all`（デフォルト: `own`）
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive`（デフォルト: `minimal`）
    
    注意:
    
    - `own` はボット送信メッセージに対するユーザーのリアクションのみを意味します（送信メッセージキャッシュによるベストエフォート）。
    - Telegram はリアクション更新にスレッド ID を提供しません。
      - 非フォーラムグループはグループチャットセッションにルーティングされます
      - フォーラムグループは特定の元トピックではなく、グループの一般トピックセッション（`:topic:1`）にルーティングされます
    
    ポーリング／Webhook 用の `allowed_updates` には `message_reaction` が自動的に含まれます。
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    ステッカーの送信
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    ステッカーキャッシュ
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Telegram API から `message_reaction` 更新を受信

    ```
    解決順序:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - エージェントのアイデンティティ絵文字フォールバック（`agents.list[].identity.emoji`、未設定の場合は "👀"）
    
    注意:
    
    - Telegram は Unicode 絵文字（例: "👀"）を想定しています。
    - チャネルまたはアカウントでリアクションを無効化するには `""` を使用します。
    ```

  
</Accordion>

  <Accordion title="Ack reactions">**リアクションの仕組み:**
Telegram のリアクションは、メッセージペイロードのプロパティではなく、**個別の `message_reaction` イベント**として届きます。ユーザーがリアクションを追加すると、OpenClaw は次を行います。 ユーザーがリアクションを追加すると、OpenClawは次のようになります。

    ```
        チャネル設定の書き込みはデフォルトで有効です（`configWrites !== false`）。
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">- `channels.telegram.timeoutSeconds` は Telegram API クライアントのタイムアウトを上書きします（未設定の場合は grammY のデフォルトが適用されます）。

    ```
    デフォルトでは、チャンネルイベントまたは `/config set|unset` によってトリガーされた設定更新の書き込みが Telegram に許可されています。
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">デフォルト: ロングポーリング（公開 URL 不要）。

    ```
    公開 URL が異なる場合は、リバースプロキシを使用し、`channels.telegram.webhookUrl` を公開エンドポイントに向けます。
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    送信テキストは `channels.telegram.textChunkLimit` まで分割されます（デフォルト 4000）。
    任意の改行分割: `channels.telegram.chunkMode="newline"` を設定すると、長さ分割の前に空行（段落境界）で分割します。
    メディアのダウンロード／アップロードは `channels.telegram.mediaMaxMb` までに制限されます（デフォルト 5）。
    "].historyLimit`
    - 送信側 Telegram API のリトライは `channels.telegram.retry` で設定可能です。
    グループ履歴コンテキストは `channels.telegram.historyLimit`（または `channels.telegram.accounts.*.historyLimit`）を使用し、`messages.groupChat.historyLimit` にフォールバックします。無効化するには `0` を設定します（デフォルト 50）。 `0`を無効にします（デフォルトは50）。
    DM 履歴は `channels.telegram.dmHistoryLimit`（ユーザーターン）で制限できます。ユーザーごとの上書き: `channels.telegram.dms["<user_id>"].historyLimit`。 `channels.imessage.dmHistoryLimit`: ユーザー ターン数での DM 履歴上限。ユーザーごとの上書き: `channels.telegram.dms["<user_id>"].historyLimit`。<user_id>CLI の送信先は数値のチャット ID またはユーザー名を指定できます:

    ```
      
</Accordion>
    ```

```bash
例: `openclaw message send --channel telegram --target 123456789 --message "hi"`。
```

  
</Accordion>
</AccordionGroup>

## トラブルシューティング

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    `channels.telegram.groups.*.requireMention=false` を設定している場合、Telegram Bot API の **プライバシーモード** を無効にする必要があります。
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    `channels.telegram.groups` が設定されている場合、グループは一覧に含まれるか `"*"` を使用する必要があります。
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    ログに `setMyCommands failed` が表示される場合、`api.telegram.org` への送信 HTTPS／DNS がブロックされていることが多いです。
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    dig +short api.telegram.org A
    dig +short api.telegram.org AAAA
    ```

```bash
  
</Accordion>
```

  
</Accordion>
</AccordionGroup>

詳細: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios)。

## 設定リファレンス（Telegram）

Primary reference:

- `channels.telegram.enabled`: チャンネル起動の有効／無効。

- `channels.telegram.botToken`: ボットトークン（BotFather）。

- `channels.telegram.tokenFile`: ファイルパスからトークンを読み込み。

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: ペアリング）。

- `channels.telegram.allowFrom`: DM 許可リスト（ID／ユーザー名）。`open` には `"*"` が必要です。 `open`には`"*"`が必要です。 `open`には`"*"`が必要です。 `openclaw doctor --fix` は、従来の `@username` エントリを ID に解決できます。

- `channels.telegram.groupPolicy`: `open | allowlist | disabled`（デフォルト: 許可リスト）。

- 許可されるグループ\*\*（`channels.telegram.groups` によるグループ許可リスト）: `openclaw doctor --fix` は、従来の `@username` エントリを ID に解決できます。

- `channels.telegram.groups`: グループ別の既定値 + 許可リスト（グローバル既定は `"*"` を使用）。
  - `channels.telegram.groups.<id>.groupPolicy`: グループポリシー（`open | allowlist | disabled`）のグループ別上書き。
  - `channels.telegram.groups.<id>.requireMention`: メンションゲーティングの既定。
  - `channels.telegram.groups.<id>.skills`: skill フィルタ（省略 = すべての Skills、空 = なし）。
  - `channels.telegram.groups.<id>.allowFrom`: グループ送信者許可リストの上書き。
  - `channels.telegram.groups.<id>.systemPrompt`: グループ用の追加システムプロンプト。
  - `channels.telegram.groups.<id>.enabled`: `false` の場合にグループを無効化。
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: トピック別上書き（グループと同じフィールド）。
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: トピック別のグループポリシー上書き（`open | allowlist | disabled`）。
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: トピック別のメンションゲーティング上書き。

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist`（デフォルト: 許可リスト）。

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: アカウント別上書き。

- `channels.telegram.replyToMode`: `off | first | all`（デフォルト: `first`）。

- `channels.telegram.textChunkLimit`: 送信チャンクサイズ（文字数）。

- `channels.telegram.chunkMode`: `length`（デフォルト）または `newline` を使用して、長さ分割の前に空行（段落境界）で分割。

- `channels.telegram.linkPreview`: 送信メッセージのリンクプレビュー切り替え（デフォルト: true）。

- `channels.telegram.streamMode`: `off | partial | block`（ドラフトストリーミング）。

- `channels.telegram.mediaMaxMb`: 送受信メディア上限（MB）。

- `channels.telegram.retry`: Telegram API 送信のリトライポリシー（回数、minDelayMs、maxDelayMs、jitter）。

- `channels.telegram.network.autoSelectFamily`: Node の autoSelectFamily を上書き（true=有効、false=無効）。Happy Eyeballs のタイムアウト回避のため、Node 22 ではデフォルト無効。 デフォルトはハッピーアイボールタイムアウトを避けるため、ノード22で無効になります。 デフォルトはハッピーアイボールタイムアウトを避けるため、ノード22で無効になります。

- `channels.telegram.proxy`: Bot API 呼び出し用のプロキシ URL（SOCKS／HTTP）。

- `channels.telegram.webhookUrl`: webhook モードを有効化（`channels.telegram.webhookSecret` が必要）。

- `channels.telegram.webhookSecret`: webhook シークレット（webhookUrl 設定時に必須）。

- `channels.telegram.webhookPath`: ローカル webhook パス（デフォルト `/telegram-webhook`）。

- ローカルリスナーは `0.0.0.0:8787` にバインドされ、デフォルトで `POST /telegram-webhook` を提供します。

- `channels.telegram.actions.reactions`: Telegram ツールのリアクションをゲート。

- `channels.telegram.actions.sendMessage`: Telegram ツールのメッセージ送信をゲート。

- `channels.telegram.actions.deleteMessage`: Telegram ツールのメッセージ削除をゲート。

- `channels.telegram.actions.sticker`: Telegram ステッカーアクション（送信／検索）をゲート（デフォルト: false）。

- `channels.telegram.reactionNotifications`: `off | own | all` — システムイベントをトリガーするリアクションを制御（未設定時のデフォルト: `own`）。

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — エージェントのリアクション能力を制御（未設定時のデフォルト: `minimal`）。

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

Telegram 固有の重要フィールド：

- 起動／認証: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- アクセス制御: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- コマンド／メニュー: `commands.native`, `customCommands`
- スレッド／返信: `replyToMode`
- 任意（`streamMode: "block"` のみ）:
- フォーマット／配信: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- メディア／ネットワーク: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook モード: `channels.telegram.webhookUrl` と `channels.telegram.webhookSecret`（任意で `channels.telegram.webhookPath`）を設定します。
- アクション／機能: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- リアクション: `reactionNotifications`, `reactionLevel`
- 書き込み／履歴: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## 関連

- [Pairing](/channels/pairing)
- 配信先（CLI／cron）
- 詳細: [チャンネルトラブルシューティング](/channels/troubleshooting)。


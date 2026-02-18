---
title: "Telegram"
---

# Telegram（Bot API）

状態: grammY 経由でボット DM + グループに本番対応。デフォルトはロングポーリング、webhook モードは任意です。

<CardGroup cols={3}>
  <Card title="ペアリング" icon="link" href="/channels/pairing">
    Telegram のデフォルト DM ポリシーはペアリングです。
  </Card>
  <Card title="チャンネルトラブルシューティング" icon="wrench" href="/channels/troubleshooting">
    クロスチャネル診断と修復プレイブック。
  </Card>
  <Card title="ゲートウェイ設定" icon="settings" href="/gateway/configuration">
    チャンネル設定パターンと完全な例。
  </Card>
</CardGroup>

## クイックセットアップ

<Steps>
  <Step title="BotFather でボットトークンを作成">
    Telegram を開き、**@BotFather** とチャットします（ハンドルが正確に `@BotFather` であることを確認）。

    `/newbot` を実行し、指示に従ってトークンを保存します。

  </Step>

  <Step title="トークンと DM ポリシーを設定">

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

    環境変数フォールバック: `TELEGRAM_BOT_TOKEN=...`（デフォルトアカウントのみ）。

  </Step>

  <Step title="ゲートウェイを起動し、最初の DM を承認">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ペアリングコードは 1 時間で失効します。

  </Step>

  <Step title="ボットをグループに追加">
    ボットをグループに追加し、アクセスモデルに合わせて `channels.telegram.groups` と `groupPolicy` を設定します。
  </Step>
</Steps>

<Note>
トークン解決順はアカウント対応です。実際には config の値が env フォールバックより優先され、`TELEGRAM_BOT_TOKEN` はデフォルトアカウントにのみ適用されます。
</Note>

## Telegram 側の設定

<AccordionGroup>
  <Accordion title="プライバシーモードとグループ可視性">
    Telegram ボットはデフォルトで **プライバシーモード** が有効で、グループメッセージの受信が制限されます。

    ボットがすべてのグループメッセージを見る必要がある場合、次のいずれかを行います。

    - `/setprivacy` でプライバシーモードを無効化する、または
    - ボットをグループ管理者にする。

    プライバシーモードを切り替えた場合は、各グループからボットを削除して再追加し、変更を適用させてください。

  </Accordion>

  <Accordion title="グループ権限">
    管理者ステータスは Telegram グループ設定で制御されます。

    管理者ボットはすべてのグループメッセージを受信できるため、常時有効なグループ動作に便利です。

  </Accordion>

  <Accordion title="便利な BotFather トグル">

    - `/setjoingroups` — グループ追加の許可／拒否
    - `/setprivacy` — グループ可視性の制御

  </Accordion>
</AccordionGroup>

## アクセス制御とアクティベーション

<Tabs>
  <Tab title="DM ポリシー">
    `channels.telegram.dmPolicy` はダイレクトメッセージのアクセスを制御します。

    - `pairing`（デフォルト）
    - `allowlist`
    - `open`（`allowFrom` に `"*"` を含める必要あり）
    - `disabled`

    `channels.telegram.allowFrom` は数値の Telegram ユーザー ID を受け付けます。`telegram:` / `tg:` プレフィックスも受け付けられ、正規化されます。
    オンボーディングウィザードは `@username` 入力を受け付け、数値 ID に解決します。
    以前の設定に `@username` の allowlist エントリが含まれている場合は、`openclaw doctor --fix` を実行して解決してください（ベストエフォート。Telegram ボットトークンが必要）。

    ### Telegram ユーザー ID の確認方法

    より安全（サードパーティ不要）:

    1. ボットに DM を送信。
    2. `openclaw logs --follow` を実行。
    3. `from.id` を確認。

    公式 Bot API 方法:

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    サードパーティ（プライバシー低）: `@userinfobot` または `@getidsbot`。

  </Tab>

  <Tab title="グループポリシーと allowlist">
    2 つの独立した制御があります。

    1. **許可されるグループ**（`channels.telegram.groups`）
       - `groups` 設定なし: すべてのグループを許可
       - `groups` 設定あり: allowlist として機能（明示的な ID または `"*"`）

    2. **グループ内で許可される送信者**（`channels.telegram.groupPolicy`）
       - `open`
       - `allowlist`（デフォルト）
       - `disabled`

    `groupAllowFrom` はグループ送信者フィルタリングに使用されます。未設定の場合、Telegram は `allowFrom` にフォールバックします。
    `groupAllowFrom` のエントリは数値の Telegram ユーザー ID である必要があります。

    例: 特定の 1 グループで全メンバーを許可:

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

  <Tab title="メンション動作">
    グループ返信はデフォルトでメンションが必要です。

    メンションは次のいずれかで認識されます。

    - ネイティブ `@botusername` メンション、または
    - 次に定義されたパターン:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`

    セッションレベルのコマンド切り替え:

    - `/activation always`
    - `/activation mention`

    これらはセッション状態のみを更新します。永続化には設定を使用してください。

    永続設定例:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    グループチャット ID の取得方法:

    - グループメッセージを `@userinfobot` / `@getidsbot` に転送
    - または `openclaw logs --follow` で `chat.id` を確認
    - または Bot API `getUpdates` を確認

  </Tab>
</Tabs>

## 実行時の動作

- Telegram は gateway プロセスによって管理されます。
- ルーティングは決定的です。Telegram からの受信は Telegram に返信されます（モデルがチャンネルを選択することはありません）。
- 受信メッセージは、返信メタデータとメディアプレースホルダーを含む共有チャンネルエンベロープに正規化されます。
- グループセッションはグループ ID ごとに分離されます。フォーラムトピックでは `:topic:<threadId>` が付加され、トピックを分離します。
- DM メッセージは `message_thread_id` を含む場合があり、OpenClaw はスレッド対応セッションキーでルーティングし、返信時に thread ID を保持します。
- ロングポーリングは grammY runner を使用し、チャット／スレッド単位で順序制御されます。全体の同時実行数は `agents.defaults.maxConcurrent` を使用します。
- Telegram Bot API は既読通知をサポートしません（`sendReadReceipts` は適用されません）。

## 機能リファレンス

<AccordionGroup>
  <Accordion title="ライブストリームプレビュー（メッセージ編集）">
    OpenClaw は一時的な Telegram メッセージを送信し、テキスト到着に応じて編集することで部分返信をストリーミングできます。

    要件:

    - `channels.telegram.streamMode` が `"off"` ではない（デフォルト: `"partial"`）

    モード:

    - `off`: ライブプレビューなし
    - `partial`: 部分テキストで頻繁に更新
    - `block`: `channels.telegram.draftChunk` を使用したチャンク更新

    `streamMode: "block"` の `draftChunk` デフォルト:

    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`

    `maxChars` は `channels.telegram.textChunkLimit` によって制限されます。

    これは DM とグループ／トピックで動作します。

    テキストのみの返信では、同じプレビューメッセージを保持し、最終的にその場で編集します（2 通目は送信されません）。

    複雑な返信（例: メディアを含む場合）は通常の最終配信にフォールバックし、その後プレビューメッセージを削除します。

    `streamMode` はブロックストリーミングとは別です。Telegram でブロックストリーミングが有効な場合、二重ストリーミングを避けるためプレビューはスキップされます。

    Telegram 専用推論ストリーム:

    - `/reasoning stream` は生成中に推論をライブプレビューへ送信
    - 最終回答は推論テキストなしで送信

  </Accordion>
</AccordionGroup>

More help: [Channel troubleshooting](/channels/troubleshooting).

## 関連

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
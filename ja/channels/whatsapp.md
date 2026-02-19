---
summary: "WhatsApp チャネルのサポート、アクセス制御、配信動作、および運用"
read_when:
  - WhatsApp/Web チャンネルの挙動や受信トレイのルーティングに取り組むとき
title: "WhatsApp"
---

# WhatsApp（Web チャンネル）

ステータス: WhatsApp Web（Baileys）経由で本番対応。 Gateway がリンクされたセッションを管理します。

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    デフォルトの DM ポリシーは、不明な送信者に対してはペアリングです。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    チャネル横断の診断および修復プレイブック。
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    完全なチャネル設定パターンと例。
  
</Card>
</CardGroup>

## クイックセットアップ（初心者向け）

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
ログインコマンド：`openclaw channels login`（リンク済みデバイス経由の QR）。
```

    ```
    特定のアカウントの場合：
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
次で承認します：`openclaw pairing approve whatsapp <code>`（一覧は `openclaw pairing list whatsapp`）。
```

    ```
    コードは 1 時間で失効し、保留リクエストはチャンネルごとに最大 3 件です。
    ```

  
</Step>
</Steps>

<Note>
**個人の WhatsApp 番号** で OpenClaw を実行する場合は、`channels.whatsapp.selfChatMode` を有効化してください（上記サンプル参照）。 （チャネルのメタデータおよびオンボーディングフローはそのセットアップ向けに最適化されていますが、個人番号でのセットアップにも対応しています。）
</Note>

## デプロイパターン

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    これは最もクリーンな運用モードです：

    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    オンボーディングは個人番号モードをサポートし、セルフチャットに適したベースラインを書き込みます：

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    現在の OpenClaw チャネルアーキテクチャでは、メッセージングプラットフォームのチャネルは WhatsApp Web ベース（`Baileys`）です。

    ```
    組み込みのチャットチャネルレジストリには、Twilio WhatsApp メッセージング用の別個のチャネルはありません。
    ```

  
</Accordion>
</AccordionGroup>

## ランタイムモデル

- Gateway が WhatsApp ソケットおよび再接続ループを管理します。
- Outbound 送信には、対象アカウントに対して有効な WhatsApp リスナーが必要です。
- ステータス／ブロードキャストチャットは無視されます。
- ダイレクトチャットは DM セッションルール（`session.dmScope`、デフォルトの `main` では DM はエージェントのメインセッションに統合されます）を使用します。
- グループは `agent:<agentId>:whatsapp:group:<jid>` セッションにマッピングされます。

## アクセス制御とアクティベーション

<Tabs>
  <Tab title="DM policy">**DM ポリシー**：`channels.whatsapp.dmPolicy` がダイレクトチャットのアクセスを制御します（デフォルト：`pairing`）。

    ```
    `channels.whatsapp.dmPolicy`（DM ポリシー：pairing/allowlist/open/disabled）。
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    グループアクセスには 2 つのレイヤーがあります：

    ```
    グループポリシー：`channels.whatsapp.groupPolicy = open|disabled|allowlist`（デフォルト `allowlist`）。
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    グループ返信はデフォルトでメンションが必要です。

メンション検出には以下が含まれます：

- ボット ID への明示的な WhatsApp メンション
- 設定されたメンション用正規表現パターン（`agents.list[].groupChat.mentionPatterns`、フォールバックは `messages.groupChat.mentionPatterns`）
- ボットへの返信の暗黙的検出（返信送信者がボット ID と一致）

セッションレベルのアクティベーションコマンド：

- `/activation mention`
- `/activation always`

`activation` はグローバル設定ではなく、セッション状態を更新します。オーナーのみ実行可能です。

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

## リンクされた自分の番号が `allowFrom` にも含まれている場合、WhatsApp のセルフチャット保護機能が有効になります：

セルフチャットのターンでは既読受信通知をスキップする

- 自分自身に通知してしまう mention-JID の自動トリガー動作を無効化する
- メッセージの正規化とコンテキスト
- 自己チャットの返信は、設定されている場合はデフォルトで `[{identity.name}]` を使用します（それ以外は `[openclaw]`）。  
  `messages.responsePrefix` が未設定の場合です。明示的に設定してカスタマイズまたは無効化してください  
  （削除するには `""` を使用）。 7.

## ```
受信した WhatsApp メッセージは、共通の inbound エンベロープでラップされます。
```

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
返信が引用を含む場合、コンテキストは次の形式で追加されます：

```text
[Replying to <sender> id:<stanzaId>]
<quoted body or media placeholder>
[/Replying]
```

利用可能な場合、返信メタデータフィールド（`ReplyToId`、`ReplyToBody`、`ReplyToSender`、送信者 JID/E.164）も設定されます。

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    位置情報および連絡先ペイロードは、ルーティング前にテキストコンテキストへ正規化されます。
    ```

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">メディアのみの受信メッセージはプレースホルダーを使用します：

    ```
    
        グループでは、未処理のメッセージをバッファリングし、ボットが最終的にトリガーされた際にコンテキストとして挿入できます。
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">  
</Accordion>

    ```
    最近の _未処理_ メッセージ（デフォルト 50 件）を次に挿入：
    `[Chat messages since your last reply - for context]`（すでにセッション内にあるメッセージは再挿入されません）
    ```

  
</Accordion>

  <Accordion title="Read receipts">デフォルトでは、受信した WhatsApp メッセージは受理されると既読（青いチェック）に設定されます。

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

```
- 画像、動画、音声（PTT ボイスノート）、およびドキュメントのペイロードをサポート
- `audio/ogg` はボイスノート互換性のため `audio/ogg; codecs=opus` に書き換えられます
- 動画送信時に `gifPlayback: true` を指定することでアニメーション GIF の再生をサポート
- 複数メディアの返信ペイロード送信時、キャプションは最初のメディア項目に適用されます
- メディアソースは HTTP(S)、`file://`、またはローカルパスを指定可能
```
---

<AccordionGroup>
  <Accordion title="Text chunking">改行による分割（任意）：`channels.whatsapp.chunkMode="newline"` を設定すると、長さ分割の前に空行（段落境界）で分割します。
</Accordion>

  <Accordion title="Outbound media behavior">
    - 受信メディア保存上限：`channels.whatsapp.mediaMaxMb`（デフォルト `50`）
    - 自動返信における送信メディア上限：`agents.defaults.mediaMaxMb`（デフォルト `5MB`）
    - 画像は制限内に収めるため自動最適化（リサイズ／画質調整）されます
    - メディア送信に失敗した場合、最初の項目のフォールバックとして、応答を無言で破棄せずテキスト警告を送信します
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">確認リアクション
</Accordion>
</AccordionGroup>

## 受信が受け付けられた直後（返信前）に送信されます

`channels.whatsapp.ackReaction`（受信時の自動リアクション：`{emoji, direct, group}`）。

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

挙動：

- 失敗はログに記録されますが、通常の返信配信を妨げることはありません
- グループモード `mentions` ではメンショントリガー時のターンでリアクションします；グループアクティベーション `always` はこのチェックをバイパスします
- マルチアカウントおよび認証情報
- WhatsApp は `messages.ackReaction` を無視するため、代わりに `channels.whatsapp.ackReaction` を使用してください。

/creds.json`     - バックアップファイル：`creds.json.bak`    -`~/.openclaw/credentials/\` 内のレガシーデフォルト認証も、デフォルトアカウントフローに対して引き続き認識／移行されます
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*`（アカウント別設定 + 任意の `authDir`）。
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">`~/.openclaw/openclaw.json` で WhatsApp を設定します。<accountId>]` はそのアカウントの WhatsApp 認証状態をクリアします。
</Accordion>

  <Accordion title="Logout behavior">`channels.whatsapp.accounts.<accountId><id>レガシー認証ディレクトリでは、`oauth.json` は保持され、Baileys の認証ファイルのみが削除されます。

    ```
      
</Accordion>
    ```

  
</Accordion>
</AccordionGroup>

## エージェントのツールサポートには、WhatsApp のリアクションアクション（`react`）が含まれます。

- アクションゲート：
- ```
  症状：リンクされたアカウントで切断や再接続の試行が繰り返される。
  ```
  - `channels.whatsapp.actions.reactions`（WhatsApp ツールのリアクションをゲート）。
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## トラブルシューティング（簡易）

<AccordionGroup>
  <Accordion title="Not linked (QR required)">症状：`channels status` に `linked: false` が表示される、または「Not linked」と警告される。

    ```
    ログアウト：`openclaw channels logout`（または `--account <id>`）は WhatsApp の認証状態を削除します（共有の `oauth.json` は保持）。
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    対象アカウントに有効な gateway リスナーが存在しない場合、Outbound 送信は即座に失敗します。

    ```
    修正: `openclaw doctor` (またはゲートウェイを再起動) それが続く場合は、 `channels login` 経由でrelinkし、 `openclawログ --follow` を調べてください。
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">gateway が実行中であり、アカウントがリンクされていることを確認してください。

    ```
    
        次の順序で確認してください：
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">- `groupPolicy`
- `groupAllowFrom` / `allowFrom`
- `groups` の allowlist エントリ
- メンションゲート（`requireMention` + メンションパターン）

    ```
    WhatsApp gateway ランタイムには Node を使用してください。
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Bun は安定した WhatsApp/Telegram gateway 運用には非互換とされています。 Bun is flagged as incompatible for stable WhatsApp/Telegram gateway operation.
  
</Accordion>
</AccordionGroup>

## 設定リファレンスへの参照ポインタ

主要リファレンス:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

重要度の高い WhatsApp フィールド:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- マルチアカウントログイン：`openclaw channels login --account <id>`（`<id>` = `accountId`）。.enabled`, `accounts.<id>`.authDir`, アカウントレベルのオーバーライド
- operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- セッション動作: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`。

## 関連項目

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- トラブルシューティングガイド：[Gateway troubleshooting](/gateway/troubleshooting)。


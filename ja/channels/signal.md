---
summary: "signal-cli（JSON-RPC + SSE）による Signal サポート、セットアップ、および番号モデル"
read_when:
  - Signal サポートのセットアップ時
  - Signal の送受信をデバッグする場合
title: "Signal"
---

# Signal（signal-cli）

ステータス: 外部 CLI 統合。 ステータス: 外部 CLI 連携。Gateway（ゲートウェイ）は HTTP JSON-RPC + SSE 経由で `signal-cli` と通信します。

## 前提条件

- サーバーにOpenClawがインストールされていること（以下のLinux手順はUbuntu 24でテスト済み）。
- gatewayが実行されているホストで`signal-cli`が利用可能であること。
- （SMS登録手順用に）1回の認証SMSを受信できる電話番号。
- 登録中にSignalのcaptcha（`signalcaptchas.org`）へアクセスできるブラウザ。

## クイックスタート（初心者向け）

1. ボット用に **別の Signal 番号** を使用してください（推奨）。
2. `signal-cli` をインストールします（Java が必要です）。
3. いずれかのセットアップパスを選択してください:
   - `signal-cli link -n "OpenClaw"`
   - **パスB（SMS登録）:** 専用番号をcaptcha＋SMS認証で登録します。
4. OpenClaw を設定し、ゲートウェイを起動します。
5. 最初のDMを送信し、ペアリングを承認します（`openclaw pairing approve signal <CODE>`）。

最小構成:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

フィールド参照：

| フィールド        | 説明                                                                                                                                                                                                                      |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `account`    | E.164形式のボット電話番号（`+15551234567`）                                                                                                                                                                         |
| セットアップ（高速パス） | ステータス: 外部 CLI 統合。 ステータス: 外部 CLI 連携。Gateway（ゲートウェイ）は HTTP JSON-RPC + SSE 経由で `signal-cli` と通信します。                                                                                        |
| `dmPolicy`   | DMアクセスポリシー（`pairing`を推奨）                                                                                                                                                                                                |
| `allowFrom`  | `channels.signal.allowFrom`: DM 許可リスト（E.164 または `uuid:&lt;id&gt;`）。`open` には `"*"` が必要です。Signal にはユーザー名がないため、電話番号/UUID ID を使用してください。 `open`には`"*"`が必要です。 シグナルにはユーザー名がありません。電話/UUID を使用してください。 |

## これは何か

- `signal-cli` を介した Signal チャンネル（libsignal の組み込みではありません）。
- 決定論的ルーティング: 返信は常に Signal に戻ります。
- ダイレクトメッセージはエージェントのメインセッションを共有し、グループは分離されます（`agent:<agentId>:signal:group:<groupId>`）。

## 設定の書き込み

デフォルトでは、Signal は `/config set|unset` によってトリガーされる設定更新を書き込むことが許可されています（`commands.config: true` が必要です）。

無効化するには次を使用します:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## 番号モデル（重要）

- ゲートウェイは **Signal デバイス**（`signal-cli` アカウント）に接続します。
- **個人の Signal アカウント** でボットを実行した場合、自分自身のメッセージは無視されます（ループ防止）。
- 「自分がボットに送信し、ボットが返信する」動作には、**別のボット番号** を使用してください。

## セットアップ方法A：既存のSignalアカウントをリンク（QR）

1. `signal-cli` をインストールします（Java が必要です）。
2. ボットアカウントをリンクします:
   - `signal-cli link -n "OpenClaw"` を実行し、Signal で QR コードをスキャンします。
3. Signal を設定し、ゲートウェイを起動します。

例:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

マルチアカウントサポート: アカウントごとの設定とオプションの `name` で `channels.signal.accounts` を使用します。 マルチアカウント対応: アカウントごとの設定と任意の `name` を使用して `channels.signal.accounts` を指定します。共通パターンについては [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。

## セットアップ方法B：専用ボット番号を登録（SMS、Linux）

既存のSignalアプリアカウントをリンクせず、専用のボット番号を使用したい場合に利用します。

1. SMS（または固定電話の場合は音声認証）を受信できる番号を用意します。
   - アカウントやセッションの競合を避けるため、専用のボット番号を使用してください。
2. Gatewayホストに`signal-cli`をインストールします：

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

JVMビルド（`signal-cli-${VERSION}.tar.gz`）を使用する場合は、先にJRE 25以上をインストールしてください。
`signal-cli`は常に最新の状態に保ってください。SignalサーバーのAPI変更により、古いリリースは動作しなくなる場合があるとupstreamで注意されています。

3. 番号を登録して確認します：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

captchaが必要な場合：

1. `https://signalcaptchas.org/registration/generate.html`を開きます。
2. captchaを完了し、「Open Signal」から`signalcaptcha://...`リンクのリンク先をコピーします。
3. 可能であれば、ブラウザセッションと同じ外部IPアドレスから実行してください。
4. 登録をすぐに再実行します（captchaトークンはすぐに期限切れになります）：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. これにより、OpenClaw 内での自動起動と起動待ちがスキップされます。自動起動時の起動が遅い場合は、`channels.signal.startupTimeoutMs` を設定してください。 自動スポーン時にゆっくり起動するには、 `channels.signal.startupTimeoutMs` を設定してください。

```bash
# Gatewayをユーザーsystemdサービスとして実行している場合：
systemctl --user restart openclaw-gateway

# その後確認：
openclaw doctor
openclaw channels status --probe
```

5. ペアリングは Signal の DM におけるデフォルトのトークン交換方式です。詳細: [Pairing](/channels/pairing) 詳細: [Pairing](/channels/pairing)
   - ボットデバイスをリンクし、デーモンを起動します:
   - サーバー上でコードを承認します：`openclaw pairing approve signal <PAIRING_CODE>`。
   - 「不明な連絡先」と表示されないよう、ボット番号をスマートフォンの連絡先に保存してください。

重要：`signal-cli`で電話番号アカウントを登録すると、その番号のメインSignalアプリのセッションが認証解除される可能性があります。 既存のスマートフォンアプリ設定を維持する必要がある場合は、専用のボット番号を使用するか、QRリンクモードを利用してください。

Upstream参照：

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Captchaフロー：`https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- リンクフロー：`https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## 外部デーモンモード（httpUrl）

`signal-cli` を自分で管理したい場合（JVM のコールドスタートが遅い、コンテナ初期化、CPU 共有など）、デーモンを別途実行し、OpenClaw からそれを参照するようにします:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

これにより、OpenClaw 内での自動起動と起動待ちがスキップされます。自動起動時の起動が遅い場合は、`channels.signal.startupTimeoutMs` を設定してください。 自動スポーン時にゆっくり起動するには、 `channels.signal.startupTimeoutMs` を設定してください。

## アクセス制御（DM + グループ）

DM:

- デフォルト: `channels.signal.dmPolicy = "pairing"`。
- 未知の送信者にはペアリングコードが送信され、承認されるまでメッセージは無視されます（コードは 1 時間で失効します）。
- 承認方法:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- ペアリングは Signal の DM におけるデフォルトのトークン交換方式です。詳細: [Pairing](/channels/pairing) 詳細: [Pairing](/channels/pairing)
- `sourceUuid` からの UUID のみの送信者は、`channels.signal.allowFrom` 内で `uuid:<id>` として保存されます。

グループ:

- `channels.signal.groupPolicy = open | allowlist | disabled`。
- `allowlist` が設定されている場合、`channels.signal.groupAllowFrom` がグループ内でトリガー可能なユーザーを制御します。

## 動作の仕組み（挙動）

- `signal-cli` はデーモンとして実行され、ゲートウェイは SSE 経由でイベントを読み取ります。
- 受信メッセージは共通チャンネルエンベロープに正規化されます。
- 返信は常に同じ番号またはグループにルーティングされます。

## メディア + 制限

- 送信テキストは `channels.signal.textChunkLimit` まで分割されます（デフォルト 4000）。
- 任意の改行分割: `channels.signal.chunkMode="newline"` を設定すると、長さ分割の前に空行（段落境界）で分割されます。
- 添付ファイルをサポートします（`signal-cli` から取得した base64）。
- デフォルトのメディア上限: `channels.signal.mediaMaxMb`（デフォルト 8）。
- `channels.signal.ignoreAttachments` を使用するとメディアのダウンロードをスキップします。
- グループ履歴コンテキストは `channels.signal.historyLimit`（または `channels.signal.accounts.*.historyLimit`）を使用し、`messages.groupChat.historyLimit` にフォールバックします。無効化するには `0` を設定してください（デフォルト 50）。 `0`を無効にします（デフォルトは50）。 `0`を無効にします（デフォルトは50）。

## 入力+開封通知

- **入力中インジケーター**: OpenClaw は `signal-cli sendTyping` を介して入力中シグナルを送信し、返信の実行中に更新します。
- **既読通知**: `channels.signal.sendReadReceipts` が true の場合、OpenClaw は許可された DM の既読通知を転送します。
- signal-cli はグループの既読通知を公開しません。

## リアクション（message ツール）

- `message action=react` を `channel=signal` とともに使用します。
- 対象: 送信者の E.164 または UUID（ペアリング出力の `uuid:<id>` を使用してください。UUID のみでも動作します）。
- `messageId` はリアクション対象メッセージの Signal タイムスタンプです。
- グループのリアクションには `targetAuthor` または `targetAuthorUuid` が必要です。

例:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

設定:

- `channels.signal.actions.reactions`: リアクション操作の有効化/無効化（デフォルト true）。
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`。
  - `off`/`ack` はエージェントのリアクションを無効化します（message ツール `react` はエラーになります）。
  - `minimal`/`extensive` はエージェントのリアクションを有効化し、ガイダンスレベルを設定します。
- アカウントごとの上書き: `channels.signal.accounts.<id>.actions.reactions`、`channels.signal.accounts.<id>.reactionLevel`。

## 配信ターゲット（CLI/cron）

- DM: `signal:+15551234567`（またはプレーンな E.164）。
- UUID DM: `uuid:<id>`（または UUID のみ）。
- グループ: `signal:group:<groupId>`。
- ユーザー名: `username:<name>`（お使いの Signal アカウントが対応している場合）。

## トラブルシューティング

まず次の手順を順に実行してください:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

必要に応じて DM のペアリング状態を確認します:

```bash
openclaw pairing list signal
```

一般的な失敗例:

- デーモンには到達できるが返信がない: アカウント/デーモン設定（`httpUrl`、`account`）と受信モードを確認してください。
- DM が無視される: 送信者がペアリング承認待ちです。
- グループメッセージが無視される: グループの送信者/メンション制御により配信がブロックされています。
- 編集後に設定検証エラーが発生した場合：`openclaw doctor --fix`を実行してください。
- マルチアカウントサポート: アカウントごとの設定とオプションの `name` で `channels.signal.accounts` を使用します。 マルチアカウント対応: アカウントごとの設定と任意の `name` を使用して `channels.signal.accounts` を指定します。共通パターンについては [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。

追加チェック：

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

切り分けフローについては [/channels/troubleshooting](/channels/troubleshooting) を参照してください。

## セキュリティに関する注意

- `signal-cli`はアカウントキーをローカル（通常は`~/.local/share/signal-cli/data/`）に保存します。
- サーバーの移行や再構築の前に、Signal アカウントの状態をバックアップしてください。
- `channels.signal.dmHistoryLimit`: ユーザターンのDM履歴制限。 `channels.signal.dmHistoryLimit`: DM 履歴のユーザーターン上限。ユーザーごとの上書き: `channels.signal.dms["<phone_or_uuid>"].historyLimit`。
- SMS 認証は登録または復旧フローでのみ必要ですが、番号やアカウントの管理権を失うと再登録が複雑になる可能性があります。

## 設定リファレンス（Signal）

完全な設定: [Configuration](/gateway/configuration)

プロバイダーオプション:

- `channels.signal.enabled`: チャンネル起動の有効化/無効化。
- `channels.signal.account`: ボットアカウントの E.164。
- `channels.signal.cliPath`: `signal-cli` へのパス。
- `channels.signal.httpUrl`: 完全なデーモン URL（host/port を上書き）。
- `channels.signal.httpHost`、`channels.signal.httpPort`: デーモンのバインド（デフォルト 127.0.0.1:8080）。
- `channels.signal.autoStart`: デーモンの自動起動（`httpUrl` 未設定時のデフォルトは true）。
- `channels.signal.startupTimeoutMs`: 起動待ちタイムアウト（ミリ秒、上限 120000）。
- `channels.signal.receiveMode`: `on-start | manual`。
- `channels.signal.ignoreAttachments`: 添付ファイルのダウンロードをスキップ。
- `channels.signal.ignoreStories`: デーモンからのストーリーを無視。
- `channels.signal.sendReadReceipts`: 既読通知を転送。
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: ペアリング）。
- `channels.signal.allowFrom`: DM 許可リスト（E.164 または `uuid:<id>`）。`open` には `"*"` が必要です。Signal にはユーザー名がないため、電話番号/UUID ID を使用してください。 `open`には`"*"`が必要です。 シグナルにはユーザー名がありません。電話/UUID を使用してください。
- `channels.signal.groupPolicy`: `open | allowlist | disabled`（デフォルト: 許可リスト）。
- `channels.signal.groupAllowFrom`: グループ送信者の許可リスト。
- `channels.signal.historyLimit`: コンテキストに含めるグループメッセージの最大数（0 で無効）。
- `channels.signal.dmHistoryLimit`: ユーザターンのDM履歴制限。 `channels.signal.dmHistoryLimit`: DM 履歴のユーザーターン上限。ユーザーごとの上書き: `channels.signal.dms["<phone_or_uuid>"].historyLimit`。
- `channels.signal.textChunkLimit`: 送信チャンクサイズ（文字数）。
- `channels.signal.chunkMode`: `length`（デフォルト）または `newline` を使用して、長さ分割の前に空行（段落境界）で分割します。
- `channels.signal.mediaMaxMb`: 受信/送信メディアの上限（MB）。

関連するグローバルオプション:

- `agents.list[].groupChat.mentionPatterns`（Signal はネイティブのメンションをサポートしません）。
- `messages.groupChat.mentionPatterns`（グローバルフォールバック）。
- `messages.responsePrefix`。

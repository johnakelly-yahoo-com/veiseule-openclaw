---
summary: "imsg（stdio 上の JSON-RPC）によるレガシー iMessage サポート。新規セットアップでは BlueBubbles の使用を推奨します。 新しいセットアップはBlueBubblesを使用する必要があります。 新しいセットアップはBlueBubblesを使用する必要があります。"
read_when:
  - iMessage サポートのセットアップ
  - iMessage の送受信のデバッグ
title: "iMessage"
---

# iMessage（レガシー: imsg）

<Warning>
**推奨:** 新しい iMessage セットアップでは [BlueBubbles](/channels/bluebubbles) を使用してください。

`imsg` チャンネルはレガシーな外部 CLI 統合であり、将来のリリースで削除される可能性があります。 
</Warning>

ステータス:レガシーの外部CLI統合。 Gateway は `imsg rpc` を起動し、標準入出力上で JSON-RPC を使用して通信します（個別のデーモン／ポートは不要）。

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    新規セットアップでは iMessage パスを推奨します。
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage の DM はデフォルトでペアリングモードになります。
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">設定リファレンス（iMessage）
</Card>
</CardGroup>

## クイックセットアップ（初心者）

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="OpenClaw を設定">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="gateway を起動">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="最初の DM ペアリングを承認（デフォルト dmPolicy）">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            ペアリングリクエストは 1 時間で期限切れになります。
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">別の Mac で iMessage を使用したい場合は、SSH 経由でリモート macOS ホスト上の `imsg` を実行するラッパーを指すよう、`channels.imessage.cliPath` を設定します。OpenClaw は stdio のみを必要とします。 OpenClawはstdioしか必要ありません。

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    添付ファイルを有効にする場合の推奨設定:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    `remoteHost` が設定されていない場合、OpenClaw はラッパースクリプト内の SSH コマンドを解析して自動検出を試みます。信頼性のため、明示的な設定を推奨します。 信頼性のために明示的な構成が推奨されます。
    ```

  
</Tab>
</Tabs>

## macOS の要件と権限

- Messages にサインイン済みの macOS。
- OpenClaw と `imsg` に対するフルディスクアクセス（Messages DB へのアクセス）。
- Messages.app を通じてメッセージを送信するには、オートメーション権限が必要です。

<Tip>
権限はプロセスコンテキストごとに付与されます。 gateway をヘッドレス（LaunchAgent/SSH）で実行する場合は、同じコンテキストで一度だけ対話型コマンドを実行し、権限ダイアログを表示させてください:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## アクセス制御とルーティング

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` はダイレクトメッセージを制御します:

    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled`（デフォルト: 許可リスト）。
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom`: グループ送信者許可リスト。

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DM はダイレクトルーティングを使用し、グループはグループルーティングを使用します。
    ダイレクトメッセージはエージェントのメイン セッションを共有し、グループは分離されます（`agent:<agentId>:imessage:group:<chat_id>`）。
    - グループセッションは分離されています（`agent:`<agentId>グループ:<chat_id>`）。
    - 返信は、送信元のチャネル／ターゲットのメタデータを使用して iMessage にルーティングされます。

    ```
    複数参加者のスレッドが `is_group=false` で届いた場合でも、`channels.imessage.groups` を使用して `chat_id` することで分離できます（下記「Group-ish threads」を参照）。
    ```

  
</Tab>
</Tabs>

## デプロイパターン

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">ボットを **別の iMessage アイデンティティ** から送信させ（個人の Messages をクリーンに保つ）たい場合は、専用の Apple ID と専用の macOS ユーザーを使用します。

    ```
    ボットユーザーとして `imsg` を実行する SSH ラッパーを指すよう、`channels.imessage.accounts.bot.cliPath` を設定します。
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    一般的なトポロジー:

    ```
    Gateway が Linux ホスト／VM 上で動作し、iMessage は Mac 上で動作させる必要がある場合、Tailscale が最も簡単なブリッジです。Gateway は tailnet 経由で Mac と通信し、SSH で `imsg` を実行し、添付ファイルを SCP で取得します。
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    SSH と SCP の両方が非対話で実行できるように、SSH キーを使用してください。
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">macOS 上の `imsg` によって支えられた iMessage チャンネルです。

    ```
    各アカウントは、`cliPath`、`dbPath`、`allowFrom`、`groupPolicy`、`mediaMaxMb`、および履歴設定などのフィールドを上書きできます。
    ```

  
</Accordion>
</AccordionGroup>

## メディア、チャンク分割、および配信先

<AccordionGroup>
  <Accordion title="Attachments and media">メディアのアップロードは `channels.imessage.mediaMaxMb`（デフォルト 16）で制限されます。
</Accordion>

  <Accordion title="Outbound chunking">`channels.imessage.chunkMode`: 長さ分割の前に空行（段落境界）で分割する `newline`、または `length`（デフォルト）。
</Accordion>

  <Accordion title="Addressing formats">
    明示的なターゲットの指定を推奨します:

    ```
    直接ハンドル: `imessage:+1555` / `sms:+1555` / `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Config の書き込み

デフォルトでは、iMessage は `/config set|unset` によってトリガーされる Config 更新の書き込みが許可されています（`commands.config: true` が必要）。

無効化するには:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## トラブルシューティング

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    バイナリとRPCサポートを確認します:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    probeでRPC未対応と表示された場合は、`imsg`を更新してください。
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">注記:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: ペアリング）。
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">チェックリスト:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`。
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">DM:

    ```
    `channels.imessage.remoteHost`: `cliPath` がリモート Mac を指す場合の、SCP 添付転送用 SSH ホスト（例: `user@gateway-host`）。未設定時は SSH ラッパーから自動検出されます。 設定されていない場合、SSHラッパーから自動検出されました。
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    同じユーザー／セッションコンテキストの対話型GUIターミナルで再実行し、表示されるプロンプトを承認します:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    OpenClaw/`imsg`を実行しているプロセスコンテキストに対して、フルディスクアクセスとオートメーションの許可が付与されていることを確認してください。
    ```

  
</Accordion>
</AccordionGroup>

## 設定リファレンスへのリンク

- iMessage を設定し、ゲートウェイを起動します。
- 完全な設定: [設定](/gateway/configuration)
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

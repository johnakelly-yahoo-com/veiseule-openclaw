---
title: IRC
description: OpenClaw を IRC チャンネルおよびダイレクトメッセージに接続します。
---

クラシックなチャンネル（`#room`）やダイレクトメッセージで OpenClaw を使いたい場合は IRC を使用します。
IRC は拡張プラグインとして提供されますが、設定はメイン設定ファイルの `channels.irc` で行います。

## クイックスタート

1. `~/.openclaw/openclaw.json` で IRC 設定を有効にします。
2. 少なくとも次を設定します:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. gateway を開始／再起動します:

```bash
openclaw gateway run
```

## セキュリティのデフォルト設定

- `channels.irc.dmPolicy` のデフォルトは `"pairing"` です。
- `channels.irc.groupPolicy` のデフォルトは `"allowlist"` です。
- `groupPolicy="allowlist"` の場合、許可するチャンネルを定義するために `channels.irc.groups` を設定します。
- 意図的に平文通信を許可する場合を除き、TLS（`channels.irc.tls=true`）を使用してください。

## アクセス制御

IRC チャンネルには、2 つの独立した「ゲート」があります:

1. **チャンネルアクセス**（`groupPolicy` + `groups`）: ボットがそのチャンネルからのメッセージをそもそも受け付けるかどうか。
2. **送信者アクセス**（`groupAllowFrom` / チャンネルごとの `groups["#channel"].allowFrom`）: そのチャンネル内で誰がボットをトリガーできるか。

設定キー:

- DM の許可リスト（DM 送信者アクセス）: `channels.irc.allowFrom`
- グループ送信者の許可リスト（チャンネル送信者アクセス）: `channels.irc.groupAllowFrom`
- チャンネルごとの制御（チャンネル + 送信者 + メンションルール）: `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` は未設定のチャンネルを許可します（**デフォルトでは引き続きメンションが必要です**）

許可リストのエントリには、nick または `nick!user@host` 形式を使用できます。

### よくある注意点: `allowFrom` は DM 用であり、チャンネル用ではありません

次のようなログが表示される場合:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…これは **グループ／チャンネル** メッセージに対して送信者が許可されていないことを意味します。 次のいずれかで修正できます:

- `channels.irc.groupAllowFrom` を設定する（全チャンネル共通のグローバル設定）、または
- チャンネルごとに送信者の許可リストを設定する: `channels.irc.groups["#channel"].allowFrom`

例（`#tuirc-dev` 内の誰でもボットに話しかけられるようにする）:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## 返信のトリガー（メンション）

チャンネルが許可されており（`groupPolicy` + `groups`）、かつ送信者も許可されている場合でも、OpenClaw はグループコンテキストではデフォルトで **メンション必須** です。

そのため、`drop channel …` のようなログが表示されることがあります （missing-mention）メッセージにボットに一致するメンションパターンが含まれていない場合に発生します。

IRC チャンネルで **メンションなしで** ボットに返信させたい場合は、そのチャンネルのメンション必須設定を無効にします:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

**すべて**のIRCチャンネルを許可し（チャンネルごとの許可リストなし）、メンションなしでも返信を許可する場合：

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## セキュリティに関する注意（公開チャンネルでは推奨）

公開チャンネルで `allowFrom: ["*"]` を許可すると、誰でもボットにプロンプトを送ることができます。
リスクを軽減するために、そのチャンネルで使用できるツールを制限してください。

### チャンネル内の全員に同じツールを適用する場合

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### 送信者ごとに異なるツールを設定する場合（オーナーにより多くの権限を付与）

`toolsBySender` を使用して、`"*"` にはより厳しいポリシーを、自分のニックネームにはより緩やかなポリシーを適用します：

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

注意：

- `toolsBySender` のキーには、ニックネーム（例: `"eigen"`）またはより強力な本人確認のための完全なホストマスク（`"eigen!~eigen@174.127.248.171"`）を指定できます。
- 最初に一致した送信者ポリシーが適用されます。`"*"` はワイルドカードのフォールバックです。

グループアクセスとメンション必須設定（およびそれらの相互作用）の詳細については、こちらを参照してください：[/channels/groups](/channels/groups)。

## NickServ

接続後にNickServで認証するには：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

接続時に一度だけ登録するオプション：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

ニックネームの登録完了後は、REGISTERの繰り返し試行を避けるために `register` を無効にしてください。

## 環境変数

デフォルトアカウントでサポートされている項目：

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS`（カンマ区切り）
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## トラブルシューティング

- ボットが接続はするもののチャンネルでまったく返信しない場合は、`channels.irc.groups` を確認し、さらにメンション必須設定によってメッセージが破棄されていないか（`missing-mention`）も確認してください。 ピングなしで返信させたい場合は、そのチャンネルで `requireMention:false` を設定してください。
- ログインに失敗する場合は、ニックネームの利用可否とサーバーパスワードを確認してください。
- カスタムネットワークでTLSが失敗する場合は、ホスト／ポートおよび証明書の設定を確認してください。

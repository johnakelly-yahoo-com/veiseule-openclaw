---
title: "system"
---

# `openclaw system`

Gateway（ゲートウェイ）向けのシステムレベルのヘルパーです。システムイベントのキュー投入、ハートビートの制御、プレゼンスの表示を行います。

## 一般的なコマンド

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## `system event`

**main** セッションでシステムイベントをキューに入れます。 11. 次のハートビートで、プロンプト内に `System:` 行として挿入されます。 **main** セッションにシステムイベントをキューに入れます。次回のハートビートで、プロンプト内に `System:` 行として注入されます。`--mode now` を使用するとハートビートを即時にトリガーします。`next-heartbeat` は次のスケジュールされたティックまで待機します。

フラグ:

- `--text <text>`: 必須のシステムイベントテキストです。
- `--mode <mode>`: `now` または `next-heartbeat`（デフォルト）です。
- `--json`: 機械可読な出力です。

## `system heartbeat last|enable|disable`

ハートビートの制御:

- `last`: 最後のハートビートイベントを表示します。
- `enable`: ハートビートを再び有効にします（無効化されていた場合に使用してください）。
- `disable`: ハートビートを一時停止します。

フラグ:

- `--json`: 機械可読な出力です。

## `system presence`

Gateway（ゲートウェイ）が把握している現在のシステムプレゼンスエントリー（ノード、インスタンス、その他のステータス行）を一覧表示します。

フラグ:

- `--json`: 機械可読な出力です。

## 注意事項

- 現在の設定（ローカルまたはリモート）から到達可能な、稼働中の Gateway（ゲートウェイ）が必要です。
- システムイベントは一時的なものであり、再起動後も保持されません。

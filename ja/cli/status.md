---
title: "status"
---

# `openclaw status`

チャンネルとセッションの診断。

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

注記:

- `--deep` はライブプローブ（WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal）を実行します。
- 複数のエージェントが設定されている場合、出力にはエージェントごとのセッションストアが含まれます。
- 利用可能な場合、概要には Gateway（ゲートウェイ）およびノードホストサービスのインストール／実行時ステータスが含まれます。
- 概要には更新チャンネルと git SHA（ソースチェックアウトの場合）が含まれます。
- 更新情報は概要に表示されます。更新が利用可能な場合、ステータスは `openclaw update` を実行するためのヒントを表示します（[Updating](/install/updating) を参照）。


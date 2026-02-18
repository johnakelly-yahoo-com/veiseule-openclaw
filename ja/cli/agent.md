---
title: "agent"
---

# `openclaw agent`

Gateway（ゲートウェイ）経由でエージェントターンを実行します（埋め込みの場合は `--local` を使用してください）。
設定済みのエージェントを直接指定するには `--agent <id>` を使用します。
設定されたエージェントを直接ターゲットにするには、 `--agent <id>` を使用します。

関連項目：

- エージェント送信ツール：[Agent send](/tools/agent-send)

## 例

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


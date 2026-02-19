---
summary: "`openclaw webhooks`（Webhook 辅助工具 + Gmail Pub/Sub）的 CLI 参考"
read_when:
  - 你想将 Gmail Pub/Sub 事件接入 OpenClaw
  - 你需要 Webhook 辅助命令
title: "webhooks"
---

# `openclaw webhooks`

Webhook 辅助工具和集成（Gmail Pub/Sub、Webhook 辅助工具）。

相关内容：

- Webhook：[Webhook](/automation/webhook)
- Gmail Pub/Sub：[Gmail Pub/Sub](/automation/gmail-pubsub)

## Gmail

```bash
openclaw webhooks gmail setup --account you@example.com
openclaw webhooks gmail run
```

详情请参阅 [Gmail Pub/Sub 文档](/automation/gmail-pubsub)。

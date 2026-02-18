---
title: "agent"
---

# `openclaw agent`

Voer één agentbeurt uit via de Gateway (gebruik `--local` voor embedded).
Gebruik `--agent <id>` om direct een geconfigureerde agent aan te spreken.

Gerelateerd:

- Agent verzendtool: [Agent verzenden](/tools/agent-send)

## Voorbeelden

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```



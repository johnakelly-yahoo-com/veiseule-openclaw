---
title: "agent"
---

# `openclaw agent`

Gateway မှတစ်ဆင့် agent turn တစ်ကြိမ် chạy ပါ (`--local` ကို embedded အတွက် အသုံးပြုပါ)။
`--agent <id>` ကို အသုံးပြုပြီး configure လုပ်ထားသော agent ကို တိုက်ရိုက် target လုပ်ပါ။

ဆက်စပ်အရာများ:

- Agent ပို့ရန် ကိရိယာ: [Agent ပို့ရန်](/tools/agent-send)

## ဥပမာများ

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


---
title: "Skills ဖွဲ့စည်းမှု"
---

# Skills ဖွဲ့စည်းမှု

Skills နှင့် သက်ဆိုင်သော ဖွဲ့စည်းပြင်ဆင်မှုအားလုံးသည် `~/.openclaw/openclaw.json` ထဲရှိ `skills` အောက်တွင် တည်ရှိသည်။

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## အကွက်များ

- `allowBundled`: **bundled** skills များအတွက်သာ အသုံးပြုနိုင်သော optional allowlist ဖြစ်သည်။ သတ်မှတ်ထားပါက၊ သာ
စာရင်းထဲရှိ bundled skills များသာ အသုံးပြုခွင့်ရှိသည် (managed/workspace skills များကို မထိခိုက်ပါ)။
- `load.extraDirs`: စကင်လုပ်ရန် ထည့်သွင်းပေးထားသော skill ဒိုင်ရက်ထရီများ (အနိမ့်ဆုံး ဦးစားပေးအဆင့်)။
- `load.watch`: skill ဖိုလ်ဒါများကို စောင့်ကြည့်ပြီး skills snapshot ကို ပြန်လည်အသစ်ပြုလုပ်ခြင်း (မူလသတ်မှတ်ချက်: true)။
- `load.watchDebounceMs`: skill watcher ဖြစ်ရပ်များအတွက် debounce ကို မီလီစက္ကန့်ဖြင့် သတ်မှတ်ခြင်း (မူလသတ်မှတ်ချက်: 250)။
- `install.preferBrew`: ရနိုင်ပါက brew installer များကို ဦးစားပေးအသုံးပြုခြင်း (မူလသတ်မှတ်ချက်: true)။
- `install.nodeManager`: node installer အတွက် ဦးစားပေးရွေးချယ်မှု (`npm` | `pnpm` | `yarn` | `bun`, default: npm).
ဤအရာသည် **skill installs** များအပေါ်သာ သက်ရောက်မှုရှိသည်။ Gateway runtime သည် Node ဖြစ်ရမည်
  (Bun not recommended for WhatsApp/Telegram).
- `entries.<skillKey>`: skill တစ်ခုချင်းစီအလိုက် override များ။

Skill တစ်ခုချင်းစီအတွက် fields များ:

- `enabled`: bundled/installed ဖြစ်နေသော်လည်း skill တစ်ခုကို ပိတ်ရန် `false` ကို သတ်မှတ်နိုင်သည်။
- `env`: agent ကို run လုပ်စဉ် ထည့်သွင်းပေးမည့် environment variables (မတိုင်မီ သတ်မှတ်ထားပြီးသား မဟုတ်ပါကသာ)။
- `apiKey`: primary env var ကို ကြေညာထားသော skills များအတွက် optional အဆင်ပြေမှု။

## မှတ်ချက်များ

- မူလအနေဖြင့် `entries` အောက်ရှိ keys များသည် skill name သို့ ချိတ်ဆက်ထားသည်။ အကယ်၍ skill တစ်ခုက သတ်မှတ်ထားပါက
  `metadata.openclaw.skillKey`, use that key instead.
- watcher ကို ဖွင့်ထားပါက skills အပြောင်းအလဲများကို နောက်လာမည့် agent turn တွင် သိရှိမည်ဖြစ်သည်။

### Sandboxed skills + env vars

When a session is **sandboxed**, skill processes run inside Docker. The sandbox
does **not** inherit the host `process.env`.

အောက်ပါအနက် တစ်ခုကို အသုံးပြုပါ–

- `agents.defaults.sandbox.docker.env` (သို့မဟုတ် per-agent `agents.list[].sandbox.docker.env`)
- သင်၏ custom sandbox image ထဲတွင် env ကို bake လုပ်ထားခြင်း

Global `env` and `skills.entries.<skill>.env/apiKey` apply to **host** runs only.


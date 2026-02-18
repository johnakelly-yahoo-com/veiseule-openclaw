---
title: "config"
---

# `openclaw config`

Config helpers: path အလိုက် values များကို get/set/unset လုပ်နိုင်သည်။ subcommand မပါဘဲ chạy ပါက configure wizard ကို ဖွင့်ပေးမည် (`openclaw configure` နှင့် အတူတူဖြစ်သည်)။

## ဥပမာများ

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```

## လမ်းကြောင်းများ

Paths များတွင် dot သို့မဟုတ် bracket notation ကို အသုံးပြုသည်—

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

အထူး အေးဂျင့်တစ်ခုကို ရည်ရွယ်ရန် agent စာရင်း၏ index ကို အသုံးပြုပါ—

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## တန်ဖိုးများ

values များကို ဖြစ်နိုင်ပါက JSON5 အဖြစ် parse လုပ်သည်၊ မဖြစ်ပါက strings အဖြစ် ဆက်ဆံသည်။
JSON5 parsing ကို မဖြစ်မနေ လိုအပ်စေရန် `--json` ကို အသုံးပြုပါ။

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

တည်းဖြတ်ပြီးနောက် Gateway（ဂိတ်ဝေး）ကို ပြန်လည်စတင်ပါ။


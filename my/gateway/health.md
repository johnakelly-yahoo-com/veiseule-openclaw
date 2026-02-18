---
title: "စနစ်အခြေအနေ စစ်ဆေးမှုများ"
---

# စနစ်အခြေအနေ စစ်ဆေးမှုများ (CLI)

ခန့်မှန်းခြင်းမလုပ်ဘဲ ချန်နယ် ချိတ်ဆက်နိုင်မှုကို စစ်ဆေးရန် အတိုချုံး လမ်းညွှန်။

## အမြန် စစ်ဆေးမှုများ

- `openclaw status` — local အကျဉ်းချုပ်: Gateway（ဂိတ်ဝေး） ချိတ်ဆက်နိုင်မှု/မုဒ်၊ အပ်ဒိတ် အကြံပြုချက်၊ ချိတ်ဆက်ထားသော ချန်နယ်၏ အတည်ပြုချက် အသက်ကာလ၊ ဆက်ရှင်များ + လတ်တလော လှုပ်ရှားမှု။
- `openclaw status --all` — local အပြည့်အစုံ ချို့ယွင်းချက် စစ်ဆေးမှု (read-only, အရောင်ပါ, debugging အတွက် paste လုပ်လို့ အဆင်ပြေ)။
- `openclaw status --deep` — လည်ပတ်နေသော Gateway（ဂိတ်ဝေး）ကိုပါ စမ်းသပ်စစ်ဆေးသည် (ထောက်ပံ့ထားပါက ချန်နယ်တစ်ခုချင်းစီအလိုက် probes)။
- `openclaw health --json` — လည်ပတ်နေသော Gateway（ဂိတ်ဝေး）ထံမှ Health snapshot အပြည့်အစုံ တောင်းခံသည် (WS-only; Baileys socket ကို တိုက်ရိုက် မသုံးပါ)။
- Agent ကို မခေါ်ဘဲ အခြေအနေ အဖြေပြန်ရရန် WhatsApp/WebChat တွင် `/status` ကို သီးသန့် မက်ဆေ့ချ်အဖြစ် ပို့ပါ။
- Logs: `/tmp/openclaw/openclaw-*.log` ကို tail လုပ်ပြီး `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound` များဖြင့် filter လုပ်ပါ။

## အသေးစိတ် ချို့ယွင်းချက် စစ်ဆေးမှုများ

- Disk ပေါ်ရှိ Creds: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (mtime သည် လတ်တလော ဖြစ်သင့်သည်)။
- 9. Session store: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (path ကို config တွင် override ပြုလုပ်နိုင်သည်)။ 10. Count နှင့် မကြာသေးမီ recipient များကို `status` မှတဆင့် ပြသပါသည်။
- 11. Relink flow: log တွင် status code 409–515 သို့မဟုတ် `loggedOut` ပေါ်လာပါက `openclaw channels logout && openclaw channels login --verbose` ကို အသုံးပြုပါ။ (မှတ်ချက် - pairing ပြီးနောက် status 515 ဖြစ်လာပါက QR login flow သည် အလိုအလျောက် တစ်ကြိမ် ပြန်လည်စတင်ပါသည်။)

## တစ်ခုခု ပျက်ကွက်သောအခါ

- `logged out` သို့မဟုတ် status 409–515 → `openclaw channels logout` ဖြင့် relink လုပ်ပြီး ထို့နောက် `openclaw channels login` ကို လုပ်ဆောင်ပါ။
- Gateway（ဂိတ်ဝေး） မရောက်နိုင်ပါက → စတင်ပါ: `openclaw gateway --port 18789` (port အလုပ်ရှုပ်နေပါက `--force` ကို အသုံးပြုပါ)။
- အဝင် မက်ဆေ့ချ်များ မရှိပါက → ချိတ်ဆက်ထားသော ဖုန်း အွန်လိုင်း ဖြစ်နေကြောင်းနှင့် ပို့သူသည် ခွင့်ပြုထားသူ ဖြစ်ကြောင်း အတည်ပြုပါ (`channels.whatsapp.allowFrom`)။ အုပ်စုချတ်များအတွက် allowlist + mention စည်းမျဉ်းများ ကိုက်ညီနေကြောင်း စစ်ဆေးပါ (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`)။

## သီးသန့် "health" command

13. `openclaw health --json` သည် လည်ပတ်နေသော Gateway ထံမှ health snapshot ကို တောင်းခံပါသည် (CLI မှ channel socket ကို တိုက်ရိုက် မချိတ်ဆက်ပါ)။ 14. ရနိုင်ပါက linked creds/auth age၊ channel တစ်ခုချင်းစီ၏ probe summary၊ session-store summary နှင့် probe duration ကို အစီရင်ခံပါသည်။ 15. Gateway ကို မရောက်ရှိနိုင်ပါက သို့မဟုတ် probe မအောင်မြင်/timeout ဖြစ်ပါက non-zero ဖြင့် exit ပြုလုပ်ပါသည်။ 16. 10s default ကို override ပြုလုပ်ရန် `--timeout <ms>` ကို အသုံးပြုပါ။


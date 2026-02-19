---
read_when:
  - Onboarding Wizard ကို လည်ပတ်စဉ် သို့မဟုတ် ဆက်တင်ပြင်ဆင်စဉ်
  - စက်အသစ်တစ်လုံးကို setup လုပ်စဉ်
sidebarTitle: Wizard (CLI)
summary: CLI onboarding wizard သည် Gateway၊ workspace၊ ချန်နယ်များနှင့် Skills များကို အပြန်အလှန်မေးမြန်းပုံစံဖြင့် setup ပြုလုပ်ရန် အသုံးပြုသည်။
title: Onboarding Wizard (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Onboarding Wizard (CLI)

CLI onboarding wizard သည် macOS၊ Linux နှင့် Windows (WSL2 မှတစ်ဆင့်) တွင် OpenClaw ကို setup လုပ်ရန် အကြံပြုထားသော နည်းလမ်းဖြစ်သည်။ ၎င်းသည် local Gateway သို့မဟုတ် remote Gateway ချိတ်ဆက်မှုအပြင် workspace ၏ default ဆက်တင်များ၊ ချန်နယ်များနှင့် Skills များကိုလည်း ဖွဲ့စည်းပေးသည်။

```bash
openclaw onboard
```

<Info>
ပထမဆုံး chat ကို အမြန်ဆုံး စတင်ရန် နည်းလမ်း：Control UI ကို ဖွင့်ပါ (ချန်နယ် setup မလိုအပ်ပါ)။ `openclaw dashboard` ကို လည်ပတ်ပြီး browser မှ chat ပြုလုပ်နိုင်သည်။ စာရွက်စာတမ်း：[Dashboard](/web/dashboard)。
</Info>

## Quick Start နှင့် Advanced Setup

Wizard သည် **Quick Start** (default ဆက်တင်များ) သို့မဟုတ် **Advanced Setup** (အပြည့်အစုံ ထိန်းချုပ်နိုင်မှု) ကို ရွေးချယ်ခြင်းဖြင့် စတင်သည်။

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - loopback ပေါ်ရှိ local Gateway
    - ရှိပြီးသား workspace သို့မဟုတ် default workspace
    - Gateway port `18789`
    - Gateway authentication token ကို အလိုအလျောက် ထုတ်လုပ်ပေးသည် (loopback ပေါ်တွင်လည်း ထုတ်လုပ်သည်)
    - Tailscale public access ကို ပိတ်ထားသည်
    - Telegram နှင့် WhatsApp DM များကို default အနေဖြင့် allowlist ဖြင့် သတ်မှတ်ထားသည် (ဖုန်းနံပါတ် ထည့်ရန် တောင်းဆိုနိုင်သည်)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - mode၊ workspace၊ Gateway၊ ချန်နယ်များ၊ daemon နှင့် Skills များအတွက် ပြည့်စုံသော prompt flow ကို ပြသသည်
  
</Tab>
</Tabs>

## CLI onboarding အသေးစိတ်

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    local နှင့် remote flow များ၏ ပြည့်စုံသော ရှင်းလင်းချက်၊ authentication နှင့် model matrix၊ configuration output၊ wizard RPC နှင့် signal-cli ၏ လုပ်ဆောင်ပုံ။
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    non-interactive onboarding အတွက် recipe များနှင့် အလိုအလျောက် `agents add` ဥပမာများ။
  
</Card>
</Columns>

## မကြာခဏ အသုံးပြုသော နောက်ဆက်တွဲ command များ

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` သည် non-interactive mode ကို မဆိုလိုပါ။ script များတွင် `--non-interactive` ကို အသုံးပြုပါ။
</Note>

<Tip>
အကြံပြုချက်：agent များသည် `web_search` ကို အသုံးပြုနိုင်ရန် Brave Search API key ကို သတ်မှတ်ပါ (`web_fetch` သည် key မလိုအပ်ဘဲ လုပ်ဆောင်နိုင်သည်)။ အလွယ်ဆုံးနည်းလမ်းမှာ `openclaw configure --section web` ကို လည်ပတ်ခြင်းဖြစ်ပြီး `tools.web.search.apiKey` ကို သိမ်းဆည်းပေးမည်ဖြစ်သည်။ စာရွက်စာတမ်း：[Web Tools](/tools/web)。
</Tip>

## ဆက်စပ်စာရွက်စာတမ်းများ

- CLI ကွန်မန် ရည်ညွှန်းချက်：[`openclaw onboard`](/cli/onboard)
- macOS အက်ပ် အွန်ဘော်ဒင်း：[အွန်ဘော်ဒင်း](/start/onboarding)
- အေးဂျင့်ကို ပထမဆုံး စတင်လုပ်ဆောင်ရန် လုပ်ထုံးလုပ်နည်း：[အေးဂျင့် ဘူ့တ်စထရပ်](/start/bootstrapping)

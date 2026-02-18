---
title: "ပလပ်ဂင်များ"
---

# `openclaw plugins`

Gateway（ဂိတ်ဝေး） ပလပ်ဂင်/တိုးချဲ့မှုများကို စီမံခန့်ခွဲရန် (in-process အဖြစ် တင်သွင်းထားသည်)။

ဆက်စပ်အကြောင်းအရာများ—

- ပလပ်အင် စနစ်: [Plugins](/tools/plugin)
- ပလပ်အင် manifest + schema: [Plugin manifest](/plugins/manifest)
- လုံခြုံရေး ခိုင်မာစေခြင်း: [Security](/gateway/security)

## Commands

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Bundled plugin များသည် OpenClaw နှင့်အတူ ပါဝင်လာသော်လည်း စတင်ချိန်တွင် ပိတ်ထားပါသည်။ ၎င်းတို့ကို အသက်သွင်းရန် `plugins enable` ကို အသုံးပြုပါ။

Plugin အားလုံးတွင် inline JSON Schema (`configSchema` — မရှိလည်းဖြစ်နိုင်) ပါဝင်သော `openclaw.plugin.json` ဖိုင် တစ်ခု ပါရမည်ဖြစ်ပါသည်။ Manifest သို့မဟုတ် schema များ မရှိခြင်း/မမှန်ကန်ခြင်းသည် plugin ကို load မလုပ်နိုင်စေပြီး config validation ကိုလည်း မအောင်မြင်စေပါသည်။

### Install

```bash
openclaw plugins install <path-or-spec>
```

Security မှတ်ချက်: plugin install လုပ်ခြင်းကို code chạy သလိုပဲ ဆက်ဆံပါ။ Pinned version များကို ဦးစားပေး အသုံးပြုပါ။

ပံ့ပိုးထားသော archive များ—`.zip`, `.tgz`, `.tar.gz`, `.tar`။

local directory ကို ကူးယူခြင်း မလုပ်ဘဲ ရှောင်ရှားရန် `--link` ကို အသုံးပြုပါ (`plugins.load.paths` ထဲသို့ ထည့်ပေါင်းသည်)—

```bash
openclaw plugins install -l ./my-plugin
```

### Update

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Update များသည် npm မှ ထည့်သွင်းထားသော ပလပ်ဂင်များအတွက်သာ သက်ရောက်သည် (`plugins.installs` တွင် ခြေရာခံထားသည်)။

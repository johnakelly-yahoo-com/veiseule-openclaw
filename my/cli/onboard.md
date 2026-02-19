---
summary: "`openclaw onboard` အတွက် CLI ကိုးကားချက် (အပြန်အလှန် ဆက်သွယ်သည့် onboarding wizard)"
read_when:
  - Gateway၊ workspace၊ auth၊ ချန်နယ်များနှင့် Skills များကို လမ်းညွှန်ပေးထားသည့် တပ်ဆင်ခြင်း လိုအပ်သောအခါ
title: "onboard"
---

# `openclaw onboard`

အပြန်အလှန် ဆက်သွယ်သည့် onboarding wizard (local သို့မဟုတ် remote Gateway တပ်ဆင်ခြင်း)။

## ဆက်စပ် လမ်းညွှန်များ

- CLI စတင်အသုံးပြုမှု အချက်အလက်စင်တာ: [စတင်အသုံးပြုမှု လမ်းညွှန် (CLI)](/start/wizard)
- CLI onboarding ကိုးကားချက်: [CLI Onboarding Reference](/start/wizard-cli-reference)
- CLI onboarding ကိုးကားချက်: [CLI Onboarding Reference](/start/wizard-cli-reference)
- CLI automation: [CLI Automation](/start/wizard-cli-automation)
- macOS onboarding: [Onboarding (macOS App)](/start/onboarding)

## ဥပမာများ

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

လုပ်ဆောင်မှု လမ်းကြောင်း မှတ်ချက်များ—

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

non-interactive mode တွင် `--custom-api-key` သည် မဖြစ်မနေ မလိုအပ်ပါ။ မထည့်ထားပါက onboarding သည် `CUSTOM_API_KEY` ကို စစ်ဆေးမည်။

Non-interactive Z.AI endpoint ရွေးချယ်မှုများ:

မှတ်ချက် - `--auth-choice zai-api-key` သည် ယခုအခါ သင့် key အတွက် အကောင်းဆုံး Z.AI endpoint ကို အလိုအလျောက် ရွေးချယ်ပေးသည် (`zai/glm-5` ပါဝင်သော general API ကို ဦးစားပေးသည်)။
GLM Coding Plan endpoint များကို သီးသန့် အသုံးပြုလိုပါက `zai-coding-global` သို့မဟုတ် `zai-coding-cn` ကို ရွေးချယ်ပါ။

```bash
# Promptless endpoint selection
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# အခြား Z.AI endpoint ရွေးချယ်မှုများ:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

လုပ်ဆောင်မှု လမ်းကြောင်း မှတ်ချက်များ—

- `quickstart`: မေးခွန်းအနည်းဆုံးသာ မေးပြီး gateway token ကို အလိုအလျောက် ဖန်တီးပေးသည်။
- `manual`: port/bind/auth အတွက် မေးခွန်းများ အပြည့်အစုံ ( `advanced` ၏ alias)။
- ပထမဆုံး ချတ်ကို အမြန်ဆုံး စတင်နိုင်ခြင်း: `openclaw dashboard` (Control UI၊ ချန်နယ် တပ်ဆင်ခြင်း မလိုအပ်)။
- Custom Provider: စာရင်းတွင် မပါဝင်သော hosted provider များအပါအဝင် OpenAI သို့မဟုတ် Anthropic နှင့် ကိုက်ညီသော endpoint မည်သည့်အရာကိုမဆို ချိတ်ဆက်နိုင်သည်။ အလိုအလျောက် ရှာဖွေသတ်မှတ်ရန် Unknown ကို အသုံးပြုပါ။

## နောက်တစ်ဆင့် အသုံးများသော အမိန့်များ

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` သည် non-interactive mode ကို အလိုအလျောက် မဆိုလိုပါ။ Script များအတွက် `--non-interactive` ကို အသုံးပြုပါ။
</Note>


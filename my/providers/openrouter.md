---
title: "OpenRouter"
---

# OpenRouter

OpenRouter သည် တစ်ခုတည်းသော အင်တာဖေ့စ်တစ်ခု၏ နောက်ကွယ်တွင် မော်ဒယ်များစွာသို့ တောင်းဆိုချက်များကို လမ်းကြောင်းပြောင်းပေးသော **unified API** ကို ပံ့ပိုးပေးသည်
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## CLI စနစ်ပြင်ဆင်ခြင်း

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## ဖွဲ့စည်းမှု နမူနာအပိုင်း

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## မှတ်ချက်များ

- မော်ဒယ် ရည်ညွှန်းချက်များသည် `openrouter/<provider>/<model>` ဖြစ်ပါသည်။
- မော်ဒယ်/ပံ့ပိုးသူ ရွေးချယ်စရာများ ပိုမိုကြည့်ရှုရန် [/concepts/model-providers](/concepts/model-providers) ကို ကြည့်ပါ။
- OpenRouter သည် အတွင်းပိုင်းတွင် သင်၏ API key ကို Bearer token အဖြစ် အသုံးပြုပါသည်။



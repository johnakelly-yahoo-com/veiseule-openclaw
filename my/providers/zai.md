---
title: "Z.AI"
---

# Z.AI

Z.AI သည် **GLM** မော်ဒယ်များအတွက် API ပလက်ဖောင်းဖြစ်သည်။ ၎င်းသည် GLM အတွက် REST API များကို ပံ့ပိုးပေးပြီး API keys ကို အသုံးပြုသည်။
for authentication. Create your API key in the Z.AI console. OpenClaw uses the `zai` provider
with a Z.AI API key.

## CLI စတင်ပြင်ဆင်ခြင်း

```bash
openclaw onboard --auth-choice zai-api-key
# or non-interactive
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Config နမူနာ

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } },
}
```

## မှတ်စုများ

- GLM မော်ဒယ်များကို `zai/<model>` အဖြစ် ရရှိနိုင်သည် (ဥပမာ: `zai/glm-4.7`)။
- မော်ဒယ် မိသားစု အကျဉ်းချုပ်အတွက် [/providers/glm](/providers/glm) ကို ကြည့်ပါ။
- Z.AI သည် သင့် API key ဖြင့် Bearer auth ကို အသုံးပြုသည်။


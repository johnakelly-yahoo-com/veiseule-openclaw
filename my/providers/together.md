---
summary: "Together AI setup (auth + model selection)"
read_when:
  - သင်သည် OpenClaw နှင့်အတူ Together AI ကို အသုံးပြုလိုပါသည်
  - API key env var သို့မဟုတ် CLI auth choice လိုအပ်ပါသည်
---

# Together AI

[Together AI](https://together.ai) သည် Llama, DeepSeek, Kimi စသည့် ဦးဆောင် open-source မော်ဒယ်များအပါအဝင် အခြားများစွာကို တစ်စုတစ်စည်းတည်း API မှတဆင့် ဝင်ရောက်အသုံးပြုနိုင်ရန် ပံ့ပိုးပေးပါသည်။

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-compatible

## အမြန်စတင်ရန်

1. API key ကို သတ်မှတ်ပါ (အကြံပြုချက်: Gateway အတွက် သိမ်းဆည်းထားပါ):

```bash
openclaw onboard --auth-choice together-api-key
```

2. မူလ (default) မော်ဒယ်ကို သတ်မှတ်ပါ:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## အပြန်အလှန်မလိုအပ်သော ဥပမာ

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

ဤအရာသည် `together/moonshotai/Kimi-K2.5` ကို မူလ (default) မော်ဒယ်အဖြစ် သတ်မှတ်ပေးပါမည်။

## ပတ်ဝန်းကျင် မှတ်ချက်

Gateway ကို daemon (launchd/systemd) အဖြစ် လည်ပတ်နေပါက `TOGETHER_API_KEY` ကို
ထို process မှ အသုံးပြုနိုင်ရန် သေချာစေပါ (ဥပမာ `~/.clawdbot/.env` တွင် ထည့်သွင်းခြင်း သို့မဟုတ်
`env.shellEnv` မှတဆင့် သတ်မှတ်ခြင်း)။

## ရရှိနိုင်သော မော်ဒယ်များ

Together AI သည် လူကြိုက်များသော open-source မော်ဒယ်များ အများအပြားကို အသုံးပြုခွင့်ပေးပါသည်:

- **GLM 4.7 Fp8** - 200K context window ပါဝင်သော မူလ မော်ဒယ်
- **Llama 3.3 70B Instruct Turbo** - မြန်ဆန်ပြီး ထိရောက်သော ညွှန်ကြားချက်လိုက်နာမှု
- **Llama 4 Scout** - ပုံရိပ် နားလည်နိုင်သော Vision မော်ဒယ်
- **Llama 4 Maverick** - အဆင့်မြင့် Vision နှင့် ဆင်ခြင်သုံးသပ်နိုင်မှု
- **DeepSeek V3.1** - အားကောင်းသော coding နှင့် ဆင်ခြင်သုံးသပ်မှု မော်ဒယ်
- **DeepSeek R1** - အဆင့်မြင့် ဆင်ခြင်သုံးသပ်မှု မော်ဒယ်
- **Kimi K2 Instruct** - 262K context window ပါဝင်သော စွမ်းဆောင်ရည်မြင့် မော်ဒယ်

မော်ဒယ်အားလုံးသည် စံချိန်မီ chat completions ကို ထောက်ပံ့ပေးပြီး OpenAI API နှင့် ကိုက်ညီပါသည်။

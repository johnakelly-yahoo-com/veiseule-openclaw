---
summary: "OpenClaw တွင် NVIDIA ၏ OpenAI-နှင့် ကိုက်ညီသော API ကို အသုံးပြုပါ"
read_when:
  - သင်သည် OpenClaw တွင် NVIDIA မော်ဒယ်များကို အသုံးပြုလိုပါသည်
  - NVIDIA_API_KEY ကို စနစ်တကျ သတ်မှတ်ထားရန် လိုအပ်ပါသည်
title: "NVIDIA"
---

# NVIDIA

NVIDIA သည် Nemotron နှင့် NeMo မော်ဒယ်များအတွက် OpenAI-နှင့် ကိုက်ညီသော API ကို `https://integrate.api.nvidia.com/v1` တွင် ပံ့ပိုးပေးသည်။ [NVIDIA NGC](https://catalog.ngc.nvidia.com/) မှ API key ဖြင့် အတည်ပြုပါ။

## CLI setup

key ကို တစ်ကြိမ် export လုပ်ပြီးနောက် onboarding ကို run ပြုလုပ်ကာ NVIDIA မော်ဒယ်ကို သတ်မှတ်ပါ:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

`--token` ကို ဆက်လက်အသုံးပြုပါက ၎င်းသည် shell history နှင့် `ps` output တွင် မှတ်တမ်းတင်သွားမည်ဖြစ်သောကြောင့် ဖြစ်နိုင်ပါက env var ကို အသုံးပြုရန် အကြံပြုပါသည်။

## Config snippet

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## Model IDs

- `nvidia/llama-3.1-nemotron-70b-instruct` (မူလသတ်မှတ်ချက်)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## မှတ်ချက်များ

- OpenAI-နှင့် ကိုက်ညီသော `/v1` endpoint ကို အသုံးပြုပါ; NVIDIA NGC မှ API key ကို အသုံးပြုပါ။
- `NVIDIA_API_KEY` ကို သတ်မှတ်ထားပါက provider သည် အလိုအလျောက် ဖွင့်လှစ်မည်ဖြစ်ပြီး static defaults (131,072-token context window, 4,096 max tokens) ကို အသုံးပြုမည်ဖြစ်သည်။

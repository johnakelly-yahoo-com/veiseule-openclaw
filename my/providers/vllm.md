---
summary: "vLLM ဖြင့် OpenClaw ကို လည်ပတ်ရန် (OpenAI-compatible local server)"
read_when:
  - OpenClaw ကို local vLLM server နှင့် ချိတ်ဆက်၍ လည်ပတ်လိုပါက
  - သင့်ကိုယ်ပိုင် မော်ဒယ်များဖြင့် OpenAI-compatible `/v1` endpoints များကို အသုံးပြုလိုပါက
title: "vLLM"
---

# vLLM

vLLM သည် open-source (နှင့် အချို့ custom) မော်ဒယ်များကို **OpenAI-compatible** HTTP API မှတဆင့် ဝန်ဆောင်မှုပေးနိုင်ပါသည်။ OpenClaw သည် `openai-completions` API ကို အသုံးပြု၍ vLLM နှင့် ချိတ်ဆက်နိုင်ပါသည်။

`VLLM_API_KEY` (server မှ auth မစစ်ဆေးပါက မည်သည့်တန်ဖိုးမဆို အသုံးပြုနိုင်သည်) ကို သတ်မှတ်ပြီး `models.providers.vllm` entry ကို တိတိကျကျ မသတ်မှတ်ထားပါက OpenClaw သည် vLLM မှ ရရှိနိုင်သော မော်ဒယ်များကို **auto-discover** လုပ်နိုင်ပါသည်။

## အမြန်စတင်ရန်

1. OpenAI-compatible server ဖြင့် vLLM ကို စတင်ပါ။

သင့် base URL တွင် `/v1` endpoints များ (ဥပမာ `/v1/models`, `/v1/chat/completions`) ပါဝင်ရပါမည်။ vLLM သည် ပုံမှန်အားဖြင့် အောက်ပါလိပ်စာတွင် လည်ပတ်ပါသည်:

- `http://127.0.0.1:8000/v1`

2. Opt in (auth မသတ်မှတ်ထားပါက မည်သည့်တန်ဖိုးမဆို အသုံးပြုနိုင်သည်):

```bash
export VLLM_API_KEY="vllm-local"
```

3. မော်ဒယ်တစ်ခုကို ရွေးချယ်ပါ (သင့် vLLM model ID များထဲမှ တစ်ခုဖြင့် အစားထိုးပါ):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Model discovery (implicit provider)

When `VLLM_API_KEY` is set (or an auth profile exists) and you **do not** define `models.providers.vllm`, OpenClaw will query:

- `GET http://127.0.0.1:8000/v1/models`

…ပြီးနောက် ပြန်လာသော ID များကို model entry များအဖြစ် ပြောင်းလဲပါမည်။

`models.providers.vllm` ကို တိတိကျကျ သတ်မှတ်ထားပါက auto-discovery ကို ကျော်လွှားမည်ဖြစ်ပြီး models များကို လက်ဖြင့် သတ်မှတ်ရပါမည်။

## တိတိကျကျ configuration (manual models)

အောက်ပါအခြေအနေများတွင် explicit config ကို အသုံးပြုပါ—

- vLLM သည် အခြား host/port ပေါ်တွင် လည်ပတ်နေပါက။
- `contextWindow`/`maxTokens` တန်ဖိုးများကို တိတိကျကျ သတ်မှတ်လိုပါက။
- သင့် server သည် အမှန်တကယ် API key လိုအပ်ပါက (သို့မဟုတ် headers များကို သင်ကိုယ်တိုင် ထိန်းချုပ်လိုပါက)။

```json5
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Local vLLM Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## ပြဿနာ ဖြေရှင်းခြင်း

- server ကို ဆက်သွယ်နိုင်မှု ရှိ/မရှိ စစ်ဆေးပါ—

```bash
curl http://127.0.0.1:8000/v1/models
```

- request များတွင် auth error ဖြစ်ပါက၊ သင့် server configuration နှင့် ကိုက်ညီသော အမှန်တကယ် `VLLM_API_KEY` ကို သတ်မှတ်ပါ၊ သို့မဟုတ် `models.providers.vllm` အောက်တွင် provider ကို တိတိကျကျ configure လုပ်ပါ။

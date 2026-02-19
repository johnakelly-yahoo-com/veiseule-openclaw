---
summary: "တစ်စုတစ်စည်းတည်း မော်ဒယ် အသုံးပြုခွင့်နှင့် ကုန်ကျစရိတ် ခြေရာခံရန် LiteLLM Proxy မှတဆင့် OpenClaw ကို run လုပ်ပါ"
read_when:
  - OpenClaw ကို LiteLLM proxy မှတဆင့် လမ်းကြောင်းပြောင်းလိုပါသည်
  - LiteLLM မှတဆင့် ကုန်ကျစရိတ် ခြေရာခံခြင်း၊ logging သို့မဟုတ် မော်ဒယ် လမ်းကြောင်းပြောင်းခြင်း လိုအပ်ပါသည်
---

# LiteLLM

[LiteLLM](https://litellm.ai) သည် open-source LLM gateway တစ်ခုဖြစ်ပြီး မော်ဒယ် provider ၁၀၀ ကျော်အတွက် တစ်စုတစ်စည်းတည်းသော API ကို ပေးပါသည်။ OpenClaw config ကို မပြောင်းလဲဘဲ backend များကို လွယ်ကူစွာ ပြောင်းလဲနိုင်ရန်နှင့် တစ်နေရာတည်းတွင် ကုန်ကျစရိတ် ခြေရာခံခြင်း၊ logging ပြုလုပ်နိုင်ရန် OpenClaw ကို LiteLLM မှတဆင့် လမ်းကြောင်းပြောင်းပါ။

## OpenClaw နှင့်အတူ LiteLLM ကို ဘာကြောင့် အသုံးပြုသင့်သနည်း?

- **ကုန်ကျစရိတ် ခြေရာခံခြင်း** — OpenClaw က မော်ဒယ်အားလုံးအပေါ် ဘယ်လောက် သုံးစွဲနေသည်ကို တိတိကျကျ ကြည့်ရှုနိုင်သည်
- **မော်ဒယ် လမ်းကြောင်းပြောင်းခြင်း** — Config မပြောင်းဘဲ Claude, GPT-4, Gemini, Bedrock တို့အကြား ပြောင်းလဲနိုင်သည်
- **Virtual keys** — OpenClaw အတွက် သုံးစွဲမှု ကန့်သတ်ချက်များပါသော key များ ဖန်တီးနိုင်သည်
- **Logging** — Debugging အတွက် request/response log များ အပြည့်အစုံ
- **Fallbacks** — Primary provider မရရှိပါက အလိုအလျောက် failover ပြုလုပ်ပေးသည်

## အမြန်စတင်ရန်

### Onboarding မှတဆင့်

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Manual setup

1. LiteLLM Proxy ကို စတင်ပါ:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. OpenClaw ကို LiteLLM သို့ ချိတ်ဆက်ပါ:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

ဒါပါပဲ။ ယခု OpenClaw သည် LiteLLM မှတဆင့် လမ်းကြောင်းပြောင်းပြီး လုပ်ဆောင်နေပါသည်။

## Configuration

### Environment variables

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Config ဖိုင်

```json5
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Virtual keys

OpenClaw အတွက် သုံးစွဲမှု ကန့်သတ်ချက်ပါသော သီးသန့် key တစ်ခု ဖန်တီးပါ:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

ဖန်တီးထားသော key ကို `LITELLM_API_KEY` အဖြစ် အသုံးပြုပါ။

## မော်ဒယ် လမ်းကြောင်းပြောင်းခြင်း

LiteLLM သည် မော်ဒယ် request များကို backend မျိုးစုံသို့ လမ်းကြောင်းပြောင်းနိုင်သည်။ သင့် LiteLLM `config.yaml` တွင် ပြင်ဆင်သတ်မှတ်ပါ:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw သည် `claude-opus-4-6` ကိုသာ ဆက်လက် request လုပ်နေမည်ဖြစ်ပြီး — လမ်းကြောင်းပြောင်းခြင်းကို LiteLLM က ကိုင်တွယ်ပေးပါသည်။

## အသုံးပြုမှု ကြည့်ရှုခြင်း

LiteLLM ၏ dashboard သို့မဟုတ် API ကို စစ်ဆေးပါ:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## မှတ်ချက်များ

- LiteLLM သည် မူလအတိုင်း `http://localhost:4000` တွင် အလုပ်လုပ်သည်
- OpenClaw သည် OpenAI-နှင့် ကိုက်ညီသော `/v1/chat/completions` endpoint မှတဆင့် ချိတ်ဆက်သည်
- OpenClaw ၏ လုပ်ဆောင်ချက်အားလုံးသည် LiteLLM မှတဆင့် အလုပ်လုပ်ပြီး — ကန့်သတ်ချက်များ မရှိပါ

## ထပ်မံကြည့်ရှုရန်

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)

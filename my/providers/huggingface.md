---
summary: "Hugging Face Inference စနစ်တကျသတ်မှတ်ခြင်း (auth + model ရွေးချယ်မှု)"
read_when:
  - OpenClaw နှင့်အတူ Hugging Face Inference ကို အသုံးပြုလိုပါသည်။
  - HF token environment variable သို့မဟုတ် CLI auth ရွေးချယ်မှု လိုအပ်ပါသည်။
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) သည် router API တစ်ခုတည်းမှတစ်ဆင့် OpenAI-compatible chat completions များကို ပံ့ပိုးပေးပါသည်။ Token တစ်ခုတည်းဖြင့် မော်ဒယ်များစွာ (DeepSeek, Llama နှင့် အခြားများ) ကို အသုံးပြုနိုင်ပါသည်။ OpenClaw သည် **OpenAI-compatible endpoint** (chat completions အတွက်သာ) ကို အသုံးပြုပါသည်။ text-to-image, embeddings သို့မဟုတ် speech အတွက်တော့ [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) ကို တိုက်ရိုက်အသုံးပြုပါ။

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` သို့မဟုတ် `HF_TOKEN` (**Make calls to Inference Providers** ခွင့်ပြုချက်ပါရှိသော fine-grained token)
- API: OpenAI-compatible (`https://router.huggingface.co/v1`)
- Billing: HF token တစ်ခုတည်း; [pricing](https://huggingface.co/docs/inference-providers/pricing) သည် provider နှုန်းထားများအတိုင်း လိုက်နာပြီး free tier ပါရှိသည်။

## အမြန်စတင်ရန်

1. **Make calls to Inference Providers** ခွင့်ပြုချက်ပါရှိသော fine-grained token တစ်ခုကို [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) တွင် ဖန်တီးပါ။
2. onboarding ကို run လုပ်ပြီး provider dropdown မှာ **Hugging Face** ကိုရွေးချယ်ပါ၊ ထို့နောက် prompt ပေါ်လာသောအခါ သင့် API key ကိုထည့်သွင်းပါ:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. **Default Hugging Face model** dropdown မှာ သင်အသုံးပြုလိုသော model ကိုရွေးချယ်ပါ (မှန်ကန်သော token ရှိပါက Inference API မှ စာရင်းကို load လုပ်မည်၊ မရှိပါက built-in စာရင်းကို ပြသမည်)။ သင်ရွေးချယ်ထားသောအရာကို default model အဖြစ် သိမ်းဆည်းထားပါသည်။
4. နောက်ပိုင်းတွင်လည်း config ထဲတွင် default model ကို သတ်မှတ်ရန် သို့မဟုတ် ပြောင်းလဲရန် လုပ်ဆောင်နိုင်ပါသည်:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Non-interactive ဥပမာ

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

ဤလုပ်ဆောင်မှုသည် `huggingface/deepseek-ai/DeepSeek-R1` ကို default model အဖြစ် သတ်မှတ်ပေးမည်ဖြစ်သည်။

## Environment မှတ်ချက်

Gateway ကို daemon (launchd/systemd) အဖြစ် run လုပ်နေပါက `HUGGINGFACE_HUB_TOKEN` သို့မဟုတ် `HF_TOKEN` ကို
အဆိုပါ process မှ အသုံးပြုနိုင်အောင် သေချာစေပါ (ဥပမာ `~/.openclaw/.env` ထဲတွင် သို့မဟုတ်
`env.shellEnv` မှတဆင့်)။

## Model ရှာဖွေခြင်းနှင့် onboarding dropdown

OpenClaw သည် **Inference endpoint ကို တိုက်ရိုက်ခေါ်ဆိုခြင်းဖြင့်** model များကို ရှာဖွေပါသည်:

```bash
GET https://router.huggingface.co/v1/models
```

(ရွေးချယ်နိုင်သည်: စာရင်းအပြည့်အစုံရရှိရန် `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` သို့မဟုတ် `$HF_TOKEN` ပေးပို့ပါ; အချို့ endpoint များသည် auth မပါဘဲ အချို့သောစာရင်းသာ ပြန်လည်ပေးနိုင်သည်။) ပြန်လာသော response သည် OpenAI-style `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }` ဖြစ်သည်။

Hugging Face API key ကို (onboarding, `HUGGINGFACE_HUB_TOKEN`, သို့မဟုတ် `HF_TOKEN` မှတဆင့်) သတ်မှတ်ပြီးပါက OpenClaw သည် ရရှိနိုင်သော chat-completion model များကို ရှာဖွေရန် ဤ GET ကို အသုံးပြုပါသည်။ **interactive onboarding** အတွင်း token ကိုထည့်သွင်းပြီးနောက် **Default Hugging Face model** dropdown ကို တွေ့ရမည်ဖြစ်ပြီး ထိုစာရင်းမှ populate လုပ်ထားပါသည် (သို့မဟုတ် request မအောင်မြင်ပါက built-in catalog ကို ပြသမည်)။ Runtime (ဥပမာ Gateway စတင်ချိန်) တွင် key ရှိပါက OpenClaw သည် catalog ကို refresh လုပ်ရန် **GET** `https://router.huggingface.co/v1/models` ကို ထပ်မံခေါ်ဆိုပါသည်။ ဤစာရင်းကို built-in catalog နှင့် ပေါင်းစည်းထားပြီး (context window နှင့် cost ကဲ့သို့သော metadata အတွက်) အသုံးပြုပါသည်။ Request မအောင်မြင်ပါက သို့မဟုတ် key မသတ်မှတ်ထားပါက built-in catalog ကိုသာ အသုံးပြုမည်ဖြစ်သည်။

## Model အမည်များနှင့် ပြင်ဆင်နိုင်သော ရွေးချယ်စရာများ

- **API မှ အမည်:** API သည် `name`, `title`, သို့မဟုတ် `display_name` ကို ပြန်ပေးပါက model display name ကို **GET /v1/models မှ hydrate လုပ်ပါသည်**; မဟုတ်ပါက model id မှ ဆင်းသက်ဖန်တီးပါသည် (ဥပမာ `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”)။
- **Display name ကို override လုပ်ခြင်း:** CLI နှင့် UI တွင် သင်လိုချင်သည့်ပုံစံအတိုင်း ပြသရန် model တစ်ခုချင်းစီအတွက် custom label ကို config ထဲတွင် သတ်မှတ်နိုင်ပါသည်:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Provider / policy ရွေးချယ်ခြင်း:** router က backend ကို မည်သို့ရွေးချယ်မည်ကို သတ်မှတ်ရန် **model id** ၏နောက်တွင် suffix ကို ပေါင်းထည့်ပါ:

  - **`:fastest`** — throughput အမြင့်ဆုံး (router မှ ရွေးချယ်မည်; provider ရွေးချယ်မှုကို **lock လုပ်ထားသည်** — interactive backend picker မပြသပါ)။
  - **`:cheapest`** — output token တစ်ခုချင်းအလိုက် ကုန်ကျစရိတ် အနည်းဆုံး (router မှ ရွေးချယ်မည်; provider ရွေးချယ်မှုကို **lock လုပ်ထားသည်**)
  - **`:provider`** — သတ်မှတ်ထားသော backend ကို အတင်းအကျပ် သုံးစွဲရန် (ဥပမာ `:sambanova`, `:together`)။

  **:cheapest** သို့မဟုတ် **:fastest** ကို (ဥပမာ onboarding model dropdown တွင်) ရွေးချယ်ပါက provider ကို lock လုပ်ထားမည်ဖြစ်သည် — router သည် cost သို့မဟုတ် speed အပေါ်မူတည်၍ ဆုံးဖြတ်မည်ဖြစ်ပြီး “prefer specific backend” ဆိုသော ရွေးချယ်နိုင်သည့်အဆင့်ကို မပြသပါ။ ဤအရာများကို `models.providers.huggingface.models` ထဲတွင် သီးခြား entry များအဖြစ် ထည့်နိုင်သည် သို့မဟုတ် `model.primary` ကို suffix ဖြင့် သတ်မှတ်နိုင်ပါသည်။ [Inference Provider settings](https://hf.co/settings/inference-providers) တွင်လည်း သင့် default order ကို သတ်မှတ်နိုင်ပါသည် (suffix မပါလျှင် ထိုအစီအစဉ်ကို အသုံးပြုမည်)။

- **Config merge:** config ကို merge လုပ်သောအခါ `models.providers.huggingface.models` (ဥပမာ `models.json` ထဲရှိ) ရှိပြီးသား entry များကို ဆက်လက် ထိန်းသိမ်းထားပါသည်။ ထို့ကြောင့် ထိုနေရာတွင် သင်သတ်မှတ်ထားသော custom `name`, `alias`, သို့မဟုတ် model options များကို မပျက်မစီး ထိန်းသိမ်းထားမည်ဖြစ်သည်။

## Model ID များနှင့် configuration ဥပမာများ

Model reference များသည် `huggingface/<org>/<model>` ပုံစံကို အသုံးပြုပါသည် (Hub-style ID များ)။ အောက်ပါစာရင်းသည် **GET** `https://router.huggingface.co/v1/models` မှ ရရှိထားခြင်းဖြစ်ပြီး သင့် catalog တွင် ထိုထက်ပိုမိုပါဝင်နိုင်ပါသည်။

**ဥပမာ ID များ (inference endpoint မှ):**

| မော်ဒယ်                                | Ref (`huggingface/` ဖြင့် prefix ထည့်ပါ) |
| -------------------------------------- | ----------------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                                   |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                                 |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                             |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                                  |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                            |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                         |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                          |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                       |
| GLM 4.7                | `zai-org/GLM-4.7`                                           |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                      |

မော်ဒယ် ID ၏ နောက်တွင် `:fastest`, `:cheapest`, သို့မဟုတ် `:provider` (ဥပမာ `:together`, `:sambanova`) ကို ထပ်ထည့်နိုင်သည်။ သင်၏ မူလအစီအစဉ်ကို [Inference Provider settings](https://hf.co/settings/inference-providers) တွင် သတ်မှတ်ပါ။ အပြည့်အစုံစာရင်းအတွက် [Inference Providers](https://huggingface.co/docs/inference-providers) နှင့် **GET** `https://router.huggingface.co/v1/models` ကို ကြည့်ပါ။

### ပြည့်စုံသော configuration ဥပမာများ

**Primary DeepSeek R1 နှင့် Qwen fallback:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen ကို မူလအဖြစ် သတ်မှတ်ပြီး :cheapest နှင့် :fastest အမျိုးအစားများ အသုံးပြုခြင်း:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**Alias များဖြင့် DeepSeek + Llama + GPT-OSS:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**:provider ဖြင့် သတ်မှတ်ထားသော backend တစ်ခုကို အတင်းအကြပ် အသုံးပြုခြင်း:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Policy suffix များပါဝင်သော Qwen နှင့် DeepSeek မော်ဒယ်များစွာ:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```


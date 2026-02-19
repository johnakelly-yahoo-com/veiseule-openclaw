---
summary: "Ollama (local LLM runtime) ဖြင့် OpenClaw ကို လည်ပတ်ရန်"
read_when:
  - Ollama မှတစ်ဆင့် local မော်ဒယ်များဖြင့် OpenClaw ကို လည်ပတ်လိုပါက
  - Ollama ကို တပ်ဆင်ခြင်းနှင့် ဖွဲ့စည်းပြင်ဆင်ခြင်းအတွက် လမ်းညွှန်ချက်များ လိုအပ်ပါက
title: "Ollama"
---

# Ollama

Ollama is a local LLM runtime that makes it easy to run open-source models on your machine. OpenClaw သည် Ollama ၏ native API (`/api/chat`) နှင့် ပေါင်းစည်းထားပြီး streaming နှင့် tool calling ကို ပံ့ပိုးပါသည်။ ထို့ပြင် `OLLAMA_API_KEY` (သို့မဟုတ် auth profile) ဖြင့် opt in လုပ်ပြီး `models.providers.ollama` ကို တိတိကျကျ မသတ်မှတ်ထားပါက **tool-capable models များကို အလိုအလျောက် ရှာဖွေတွေ့ရှိနိုင်သည်**။

## အမြန်စတင်ခြင်း

1. Ollama ကို ထည့်သွင်းတပ်ဆင်ပါ: [https://ollama.ai](https://ollama.ai)

2. မော်ဒယ်တစ်ခုကို pull လုပ်ပါ:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. OpenClaw အတွက် Ollama ကို ဖွင့်ပါ (တန်ဖိုးမည်သည့်အရာမဆို ရပါသည်; Ollama သည် အမှန်တကယ် API key မလိုအပ်ပါ):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Ollama မော်ဒယ်များကို အသုံးပြုပါ:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## မော်ဒယ် ရှာဖွေခြင်း (implicit provider)

`OLLAMA_API_KEY` (သို့မဟုတ် auth profile) ကို သတ်မှတ်ထားပြီး **`models.providers.ollama` ကို မသတ်မှတ်ထားပါက**, OpenClaw သည် `http://127.0.0.1:11434` ရှိ local Ollama instance မှ မော်ဒယ်များကို ရှာဖွေတွေ့ရှိပါသည်-

- `/api/tags` နှင့် `/api/show` ကို query လုပ်သည်
- `tools` စွမ်းဆောင်ရည်ကို အစီရင်ခံသော မော်ဒယ်များကိုသာ ထားရှိသည်
- မော်ဒယ်က `thinking` ကို အစီရင်ခံပါက `reasoning` ဟု မှတ်သားသည်
- ရရှိနိုင်ပါက `model_info["<arch>.context_length"]` မှ `contextWindow` ကို ဖတ်ယူသည်
- `maxTokens` ကို context window ၏ 10× အဖြစ် သတ်မှတ်သည်
- ကုန်ကျစရိတ်အားလုံးကို `0` အဖြစ် သတ်မှတ်သည်

ဤနည်းလမ်းသည် မော်ဒယ်များကို လက်ဖြင့် ထည့်သွင်းရန် မလိုအပ်ဘဲ Ollama ၏ စွမ်းဆောင်ရည်များနှင့် ကိုက်ညီသော catalog ကို ထိန်းသိမ်းနိုင်စေသည်။

ရရှိနိုင်သော မော်ဒယ်များကို ကြည့်ရန်-

```bash
ollama list
openclaw models list
```

မော်ဒယ်အသစ်တစ်ခု ထည့်ရန် Ollama ဖြင့် pull လုပ်ရုံသာ လိုအပ်ပါသည်-

```bash
ollama pull mistral
```

မော်ဒယ်အသစ်သည် အလိုအလျောက် ရှာဖွေတွေ့ရှိပြီး အသုံးပြုနိုင်လာမည် ဖြစ်သည်။

`models.providers.ollama` ကို တိတိကျကျ သတ်မှတ်ထားပါက auto-discovery ကို ကျော်လွှားပြီး မော်ဒယ်များကို လက်ဖြင့် သတ်မှတ်ရပါမည် (အောက်တွင် ကြည့်ပါ)။

## ဖွဲ့စည်းပြင်ဆင်ခြင်း

### အခြေခံ သတ်မှတ်ခြင်း (implicit discovery)

Ollama ကို ဖွင့်ရန် အလွယ်ဆုံးနည်းမှာ environment variable ကို အသုံးပြုခြင်း ဖြစ်သည်-

```bash
export OLLAMA_API_KEY="ollama-local"
```

### အတိအကျ သတ်မှတ်ခြင်း (manual models)

အောက်ပါအခြေအနေများတွင် explicit config ကို အသုံးပြုပါ-

- Ollama သည် အခြား ဟို့စ်/port တွင် လည်ပတ်နေပါက
- သီးခြား context window များ သို့မဟုတ် မော်ဒယ်စာရင်းများကို အတင်းအကျပ် သတ်မှတ်လိုပါက
- tool support ကို အစီရင်မခံသော မော်ဒယ်များကို ထည့်သွင်းလိုပါက

```json5
{
  models: {
    providers: {
      ollama: {
        // Use a host that includes /v1 for OpenAI-compatible APIs
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

`OLLAMA_API_KEY` ကို သတ်မှတ်ထားပါက provider entry ထဲတွင် `apiKey` ကို ချန်လှပ်နိုင်ပြီး OpenClaw က availability စစ်ဆေးရန် အလိုအလျောက် ဖြည့်ပေးမည် ဖြစ်သည်။

### စိတ်ကြိုက် base URL (explicit config)

Ollama ကို မတူညီသော ဟို့စ် သို့မဟုတ် port တွင် လည်ပတ်နေပါက (explicit config သည် auto-discovery ကို ပိတ်ထားသောကြောင့် မော်ဒယ်များကို လက်ဖြင့် သတ်မှတ်ရပါမည်)-

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1",
      },
    },
  },
}
```

### မော်ဒယ် ရွေးချယ်ခြင်း

ဖွဲ့စည်းပြင်ဆင်ပြီးပါက သင့် Ollama မော်ဒယ်များအားလုံးကို အသုံးပြုနိုင်ပါသည်-

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## အဆင့်မြင့်

### Reasoning models

Ollama က `/api/show` အတွင်း `thinking` ကို အစီရင်ခံပါက OpenClaw သည် မော်ဒယ်များကို reasoning-capable ဟု မှတ်သားပါသည်-

```bash
ollama pull deepseek-r1:32b
```

### Model Costs

Ollama သည် အခမဲ့ဖြစ်ပြီး local တွင် လည်ပတ်သောကြောင့် မော်ဒယ်ကုန်ကျစရိတ်အားလုံးကို $0 အဖြစ် သတ်မှတ်ထားပါသည်။

### Streaming Configuration

OpenClaw ၏ Ollama ပေါင်းစည်းမှုသည် မူလအတိုင်း **native Ollama API** (`/api/chat`) ကို အသုံးပြုပြီး streaming နှင့် tool calling ကို တပြိုင်နက်တည်း အပြည့်အဝ ပံ့ပိုးပေးပါသည်။ အထူး configuration မလိုအပ်ပါ။

#### Legacy OpenAI-Compatible Mode

OpenAI-နှင့် ကိုက်ညီသော endpoint ကို အစားထိုးအသုံးပြုရန် လိုအပ်ပါက (ဥပမာ OpenAI format ကိုသာ ပံ့ပိုးသည့် proxy နောက်ကွယ်တွင်ရှိပါက) `api: "openai-completions"` ကို တိတိကျကျ သတ်မှတ်ပါ:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

မှတ်ချက် - OpenAI-နှင့် ကိုက်ညီသော endpoint သည် streaming နှင့် tool calling ကို တပြိုင်နက်တည်း မပံ့ပိုးနိုင်နိုင်ပါ။ model config တွင် `params: { streaming: false }` ဖြင့် streaming ကို ပိတ်ရန် လိုအပ်နိုင်ပါသည်။

### Context windows

For auto-discovered models, OpenClaw uses the context window reported by Ollama when available, otherwise it defaults to `8192`. You can override `contextWindow` and `maxTokens` in explicit provider config.

## Troubleshooting

### Ollama ကို မတွေ့ရှိနိုင်ခြင်း

Ollama လည်ပတ်နေကြောင်းနှင့် `OLLAMA_API_KEY` (သို့မဟုတ် auth profile) ကို သတ်မှတ်ထားကြောင်း၊ ထို့အပြင် တိတိကျကျ `models.providers.ollama` ကို မသတ်မှတ်ထားကြောင်း သေချာစစ်ဆေးပါ-

```bash
ollama serve
```

API ကို ဝင်ရောက်အသုံးပြုနိုင်ကြောင်းလည်း စစ်ဆေးပါ-

```bash
curl http://localhost:11434/api/tags
```

### မော်ဒယ်များ မရရှိနိုင်ခြင်း

OpenClaw only auto-discovers models that report tool support. If your model isn't listed, either:

- tool-capable မော်ဒယ်တစ်ခုကို pull လုပ်ပါ၊ သို့မဟုတ်
- `models.providers.ollama` တွင် မော်ဒယ်ကို လက်ဖြင့် သတ်မှတ်ပါ။

မော်ဒယ်များ ထည့်ရန်-

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Connection refused

မော်ဒယ်များ ထည့်ရန်-

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## See Also

- [Model Providers](/concepts/model-providers) - provider အားလုံး၏ အနှစ်ချုပ်
- [Model Selection](/concepts/models) - မော်ဒယ်များကို မည်သို့ ရွေးချယ်ရမည်
- [Configuration](/gateway/configuration) - ဖွဲ့စည်းပြင်ဆင်ခြင်း အပြည့်အစုံ


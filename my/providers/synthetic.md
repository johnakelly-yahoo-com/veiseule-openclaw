---
title: "Synthetic"
---

# Synthetic

Synthetic သည် Anthropic-compatible endpoints များကို ဖော်ပြပေးသည်။ OpenClaw သည် ၎င်းကို အဖြစ် မှတ်ပုံတင်သည်
`synthetic` provider and uses the Anthropic Messages API.

## အမြန် စနစ်တကျ ပြင်ဆင်ခြင်း

1. `SYNTHETIC_API_KEY` ကို သတ်မှတ်ပါ (သို့မဟုတ် အောက်ပါ wizard ကို ပြေးဆွဲပါ)။
2. onboarding ကို ပြေးဆွဲပါ—

```bash
openclaw onboard --auth-choice synthetic-api-key
```

မူလ မော်ဒယ်ကို အောက်ပါအတိုင်း သတ်မှတ်ထားပါသည်—

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

## ဖွဲ့စည်းပုံ ဥပမာ

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

မှတ်ချက် - OpenClaw ၏ Anthropic client သည် base URL သို့ `/v1` ကို ပေါင်းထည့်သည်၊ ထို့ကြောင့် အသုံးပြုပါ
`https://api.synthetic.new/anthropic` (not `/anthropic/v1`). If Synthetic changes
its base URL, override `models.providers.synthetic.baseUrl`.

## မော်ဒယ် စာရင်း

အောက်ပါ မော်ဒယ်များအားလုံးသည် ကုန်ကျစရိတ် `0` (input/output/cache) ကို အသုံးပြုပါသည်။

| မော်ဒယ် ID                                               | Context window အရွယ်အစား | အများဆုံး တိုကင်များ | Reasoning | Input        |
| ------------------------------------------------------ | -------------- | ---------- | --------- | ------------ |
| `hf:MiniMaxAI/MiniMax-M2.1`                            | 192000         | 65536      | false     | text         |
| `hf:moonshotai/Kimi-K2-Thinking`                       | 256000         | 8192       | true      | text         |
| `hf:zai-org/GLM-4.7`                                   | 198000         | 128000     | false     | text         |
| `hf:deepseek-ai/DeepSeek-R1-0528`                      | 128000         | 8192       | false     | text         |
| `hf:deepseek-ai/DeepSeek-V3-0324`                      | 128000         | 8192       | false     | text         |
| `hf:deepseek-ai/DeepSeek-V3.1`                         | 128000         | 8192       | false     | text         |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus`                | 128000         | 8192       | false     | text         |
| `hf:deepseek-ai/DeepSeek-V3.2`                         | 159000         | 8192       | false     | text         |
| `hf:meta-llama/Llama-3.3-70B-Instruct`                 | 128000         | 8192       | false     | text         |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000         | 8192       | false     | text         |
| `hf:moonshotai/Kimi-K2-Instruct-0905`                  | 256000         | 8192       | false     | text         |
| `hf:openai/gpt-oss-120b`                               | 128000         | 8192       | false     | text         |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507`                | 256000         | 8192       | false     | text         |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct`               | 256000         | 8192       | false     | text         |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct`                  | 250000         | 8192       | false     | text + image |
| `hf:zai-org/GLM-4.5`                                   | 128000         | 128000     | false     | text         |
| `hf:zai-org/GLM-4.6`                                   | 198000         | 128000     | false     | text         |
| `hf:deepseek-ai/DeepSeek-V3`                           | 128000         | 8192       | false     | text         |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507`                | 256000         | 8192       | true      | text         |

## Notes

- မော်ဒယ် reference များသည် `synthetic/<modelId>` ကို အသုံးပြုပါသည်။
- မော်ဒယ် allowlist (`agents.defaults.models`) ကို ဖွင့်ထားပါက သင်အသုံးပြုရန် စီစဉ်ထားသော မော်ဒယ်အားလုံးကို ထည့်သွင်းပါ။
- ပံ့ပိုးသူ စည်းမျဉ်းများအတွက် [Model providers](/concepts/model-providers) ကို ကြည့်ပါ။


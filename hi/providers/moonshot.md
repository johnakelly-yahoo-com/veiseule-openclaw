---
title: "Moonshot AI"
---

# Moonshot AI (Kimi)

Moonshot Kimi API को OpenAI-compatible endpoints के साथ प्रदान करता है। कॉन्फ़िगर करें
provider and set the default model to `moonshot/kimi-k2.5`, or use
Kimi Coding with `kimi-coding/k2p5`.

वर्तमान Kimi K2 मॉडल आईडी:

{/_moonshot-kimi-k2-ids:start_/ && null}

- `kimi-k2.5`
- `kimi-k2-0905-preview`
- `kimi-k2-turbo-preview`
- `kimi-k2-thinking`
- `kimi-k2-thinking-turbo`
  {/_moonshot-kimi-k2-ids:end_/ && null}

```bash
openclaw onboard --auth-choice moonshot-api-key
```

Kimi Coding:

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

नोट: Moonshot और Kimi Coding अलग-अलग प्रदाता हैं। कुंजियाँ परस्पर विनिमेय नहीं हैं, एंडपॉइंट्स अलग हैं, और मॉडल रेफरेंस भी अलग हैं (Moonshot `moonshot/...` का उपयोग करता है, Kimi Coding `kimi-coding/...` का उपयोग करता है)।

## विन्यास स्निपेट (Moonshot API)

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: {
        // moonshot-kimi-k2-aliases:start
        "moonshot/kimi-k2.5": { alias: "Kimi K2.5" },
        "moonshot/kimi-k2-0905-preview": { alias: "Kimi K2" },
        "moonshot/kimi-k2-turbo-preview": { alias: "Kimi K2 Turbo" },
        "moonshot/kimi-k2-thinking": { alias: "Kimi K2 Thinking" },
        "moonshot/kimi-k2-thinking-turbo": { alias: "Kimi K2 Thinking Turbo" },
        // moonshot-kimi-k2-aliases:end
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          // moonshot-kimi-k2-models:start
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-0905-preview",
            name: "Kimi K2 0905 Preview",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-turbo-preview",
            name: "Kimi K2 Turbo",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking",
            name: "Kimi K2 Thinking",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking-turbo",
            name: "Kimi K2 Thinking Turbo",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          // moonshot-kimi-k2-models:end
        ],
      },
    },
  },
}
```

## Kimi Coding

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: {
        "kimi-coding/k2p5": { alias: "Kimi K2.5" },
      },
    },
  },
}
```

## नोट्स

- Moonshot मॉडल रेफरेंस `moonshot/<modelId>` का उपयोग करते हैं। Kimi Coding मॉडल रेफरेंस `kimi-coding/<modelId>` का उपयोग करते हैं।
- आवश्यकता होने पर `models.providers` में मूल्य निर्धारण और संदर्भ मेटाडेटा को ओवरराइड करें।
- यदि Moonshot किसी मॉडल के लिए अलग संदर्भ सीमाएँ प्रकाशित करता है, तो
  `contextWindow` को तदनुसार समायोजित करें।
- अंतरराष्ट्रीय एंडपॉइंट के लिए `https://api.moonshot.ai/v1` का उपयोग करें, और चीन एंडपॉइंट के लिए `https://api.moonshot.cn/v1` का उपयोग करें।


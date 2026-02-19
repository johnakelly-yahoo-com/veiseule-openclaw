---
summary: "vLLM (OpenAI-compatible लोकल सर्वर) के साथ OpenClaw चलाएँ"
read_when:
  - आप OpenClaw को एक लोकल vLLM सर्वर के साथ चलाना चाहते हैं
  - आप अपने स्वयं के मॉडलों के साथ OpenAI-compatible /v1 endpoints चाहते हैं
title: "vLLM"
---

# vLLM

vLLM एक **OpenAI-compatible** HTTP API के माध्यम से ओपन-सोर्स (और कुछ कस्टम) मॉडलों को सर्व कर सकता है। OpenClaw `openai-completions` API का उपयोग करके vLLM से कनेक्ट कर सकता है।

जब आप `VLLM_API_KEY` के साथ ऑप्ट-इन करते हैं (यदि आपका सर्वर auth लागू नहीं करता है तो कोई भी मान काम करेगा) और `models.providers.vllm` की स्पष्ट एंट्री परिभाषित नहीं करते हैं, तब OpenClaw vLLM से उपलब्ध मॉडलों को **auto-discover** भी कर सकता है।

## त्वरित प्रारंभ

1. OpenAI-compatible सर्वर के साथ vLLM प्रारंभ करें।

आपका base URL `/v1` endpoints (जैसे `/v1/models`, `/v1/chat/completions`) एक्सपोज़ करना चाहिए। vLLM आमतौर पर यहाँ चलता है:

- `http://127.0.0.1:8000/v1`

2. ऑप्ट-इन करें (यदि auth कॉन्फ़िगर नहीं है तो कोई भी मान काम करेगा):

```bash
export VLLM_API_KEY="vllm-local"
```

3. एक मॉडल चुनें (अपने vLLM मॉडल IDs में से किसी एक से बदलें):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## मॉडल खोज (implicit provider)

जब `VLLM_API_KEY` सेट होता है (या कोई auth प्रोफ़ाइल मौजूद होती है) और आप `models.providers.vllm` परिभाषित **नहीं** करते हैं, तो OpenClaw यह क्वेरी करेगा:

- `GET http://127.0.0.1:8000/v1/models`

…और लौटाए गए IDs को मॉडल एंट्रीज़ में परिवर्तित कर देगा।

यदि आप `models.providers.vllm` को स्पष्ट रूप से सेट करते हैं, तो auto-discovery छोड़ दिया जाएगा और आपको मॉडल मैन्युअली परिभाषित करने होंगे।

## स्पष्ट कॉन्फ़िगरेशन (मैन्युअल मॉडल)

स्पष्ट कॉन्फ़िगरेशन का उपयोग करें जब:

- vLLM किसी भिन्न host/port पर चल रहा हो।
- आप `contextWindow`/`maxTokens` मानों को निर्धारित (pin) करना चाहते हों।
- आपके सर्वर को एक वास्तविक API key की आवश्यकता हो (या आप headers को नियंत्रित करना चाहते हों)।

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

## समस्या निवारण

- जाँचें कि सर्वर पहुँचा जा सकता है:

```bash
curl http://127.0.0.1:8000/v1/models
```

- यदि अनुरोध auth त्रुटियों के साथ विफल होते हैं, तो अपने सर्वर कॉन्फ़िगरेशन से मेल खाने वाली एक वास्तविक `VLLM_API_KEY` सेट करें, या `models.providers.vllm` के अंतर्गत provider को स्पष्ट रूप से कॉन्फ़िगर करें।


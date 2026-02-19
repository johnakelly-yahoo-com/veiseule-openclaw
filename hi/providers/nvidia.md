---
summary: "OpenClaw में NVIDIA का OpenAI-संगत API उपयोग करें"
read_when:
  - आप OpenClaw में NVIDIA मॉडल्स का उपयोग करना चाहते हैं
  - आपको NVIDIA_API_KEY सेटअप करना होगा
title: "NVIDIA"
---

# NVIDIA

NVIDIA, Nemotron और NeMo मॉडल्स के लिए `https://integrate.api.nvidia.com/v1` पर OpenAI-संगत API प्रदान करता है। [NVIDIA NGC](https://catalog.ngc.nvidia.com/) से प्राप्त API key के साथ प्रमाणित करें।

## CLI सेटअप

key को एक बार export करें, फिर onboarding चलाएँ और एक NVIDIA मॉडल सेट करें:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

यदि आप अभी भी `--token` पास करते हैं, तो ध्यान रखें कि यह shell history और `ps` आउटपुट में दर्ज हो जाता है; संभव हो तो env var का उपयोग करना बेहतर है।

## कॉन्फ़िग स्निपेट

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

## मॉडल आईडी

- `nvidia/llama-3.1-nemotron-70b-instruct` (डिफ़ॉल्ट)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## नोट्स

- OpenAI-संगत `/v1` एंडपॉइंट; NVIDIA NGC से एक API key का उपयोग करें।
- जब `NVIDIA_API_KEY` सेट किया जाता है तो प्रोवाइडर स्वतः सक्षम हो जाता है; यह स्थिर डिफ़ॉल्ट्स का उपयोग करता है (131,072-टोकन कॉन्टेक्स्ट विंडो, 4,096 अधिकतम टोकन)।

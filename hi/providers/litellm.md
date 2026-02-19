---
summary: "एकीकृत मॉडल एक्सेस और लागत ट्रैकिंग के लिए OpenClaw को LiteLLM Proxy के माध्यम से चलाएँ"
read_when:
  - आप OpenClaw को LiteLLM proxy के माध्यम से रूट करना चाहते हैं
  - आपको LiteLLM के माध्यम से लागत ट्रैकिंग, लॉगिंग या मॉडल रूटिंग की आवश्यकता है
---

# LiteLLM

[LiteLLM](https://litellm.ai) एक ओपन-सोर्स LLM gateway है जो 100+ मॉडल प्रदाताओं के लिए एकीकृत API प्रदान करता है। केंद्रीकृत लागत ट्रैकिंग, लॉगिंग और अपने OpenClaw config को बदले बिना backend बदलने की सुविधा के लिए OpenClaw को LiteLLM के माध्यम से रूट करें।

## OpenClaw के साथ LiteLLM क्यों उपयोग करें?

- **लागत ट्रैकिंग** — देखें कि OpenClaw सभी मॉडलों पर ठीक-ठीक कितना खर्च करता है
- **मॉडल रूटिंग** — बिना config बदलाव के Claude, GPT-4, Gemini, Bedrock के बीच स्विच करें
- **Virtual keys** — OpenClaw के लिए खर्च सीमा के साथ keys बनाएँ
- **लॉगिंग** — डिबगिंग के लिए पूर्ण request/response logs
- **Fallbacks** — यदि आपका प्राथमिक provider डाउन हो जाए तो स्वचालित failover

## त्वरित प्रारंभ

### Onboarding के माध्यम से

```bash
openclaw onboard --auth-choice litellm-api-key
```

### मैनुअल सेटअप

1. LiteLLM Proxy प्रारंभ करें:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. OpenClaw को LiteLLM की ओर निर्देशित करें:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

बस इतना ही। OpenClaw अब LiteLLM के माध्यम से रूट करता है।

## कॉन्फ़िगरेशन

### एनवायरनमेंट वेरिएबल्स

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### कॉन्फ़िग फ़ाइल

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

## वर्चुअल कीज़

खर्च सीमा के साथ OpenClaw के लिए एक समर्पित key बनाएँ:

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

जनरेट की गई key को `LITELLM_API_KEY` के रूप में उपयोग करें।

## मॉडल रूटिंग

LiteLLM मॉडल अनुरोधों को अलग-अलग बैकएंड्स की ओर रूट कर सकता है। अपने LiteLLM `config.yaml` में कॉन्फ़िगर करें:

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

OpenClaw `claude-opus-4-6` का अनुरोध करता रहता है — रूटिंग LiteLLM संभालता है।

## उपयोग देखना

LiteLLM का डैशबोर्ड या API जाँचें:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## नोट्स

- LiteLLM डिफ़ॉल्ट रूप से `http://localhost:4000` पर चलता है
- OpenClaw, OpenAI-संगत `/v1/chat/completions` endpoint के माध्यम से कनेक्ट करता है
- सभी OpenClaw फीचर्स LiteLLM के माध्यम से काम करते हैं — कोई सीमाएँ नहीं

## यह भी देखें

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)


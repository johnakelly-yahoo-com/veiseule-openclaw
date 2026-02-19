---
summary: "Hugging Face Inference सेटअप (auth + मॉडल चयन)"
read_when:
  - आप OpenClaw के साथ Hugging Face Inference का उपयोग करना चाहते हैं
  - आपको HF token env var या CLI auth विकल्प की आवश्यकता है
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) एकल router API के माध्यम से OpenAI-संगत chat completions प्रदान करते हैं। आपको एक ही token के साथ कई मॉडल्स (DeepSeek, Llama, और अन्य) का एक्सेस मिलता है। OpenClaw **OpenAI-compatible endpoint** (केवल chat completions) का उपयोग करता है; text-to-image, embeddings, या speech के लिए सीधे [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) का उपयोग करें।

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` या `HF_TOKEN` (**Make calls to Inference Providers** अनुमति वाला fine-grained token)
- API: OpenAI-compatible (`https://router.huggingface.co/v1`)
- Billing: एकल HF token; [pricing](https://huggingface.co/docs/inference-providers/pricing) provider दरों का पालन करती है और एक free tier शामिल है।

## त्वरित शुरुआत

1. [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) पर एक fine-grained token बनाएँ, जिसमें **Make calls to Inference Providers** अनुमति हो।
2. Onboarding चलाएँ और provider dropdown में **Hugging Face** चुनें, फिर संकेत मिलने पर अपनी API key दर्ज करें:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. **Default Hugging Face model** dropdown में, अपनी इच्छित मॉडल चुनें (यदि आपके पास मान्य token है तो सूची Inference API से लोड होती है; अन्यथा एक अंतर्निहित सूची दिखाई जाती है)। आपका चयन डिफ़ॉल्ट मॉडल के रूप में सहेजा जाता है।
4. आप बाद में config में डिफ़ॉल्ट मॉडल सेट या बदल भी सकते हैं:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Non-interactive उदाहरण

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

यह `huggingface/deepseek-ai/DeepSeek-R1` को डिफ़ॉल्ट मॉडल के रूप में सेट करेगा।

## Environment नोट

यदि Gateway एक daemon (launchd/systemd) के रूप में चल रहा है, तो सुनिश्चित करें कि `HUGGINGFACE_HUB_TOKEN` या `HF_TOKEN`
उस प्रक्रिया के लिए उपलब्ध हो (उदाहरण के लिए, `~/.openclaw/.env` में या
`env.shellEnv` के माध्यम से)।

## Model discovery और onboarding dropdown

OpenClaw मॉडल्स का पता लगाने के लिए सीधे **Inference endpoint** को कॉल करता है:

```bash
GET https://router.huggingface.co/v1/models
```

(वैकल्पिक: पूरी सूची के लिए `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` या `$HF_TOKEN` भेजें; कुछ endpoints बिना auth के आंशिक सूची लौटाते हैं।) प्रतिक्रिया OpenAI-स्टाइल `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

जब आप Hugging Face API key कॉन्फ़िगर करते हैं (onboarding, `HUGGINGFACE_HUB_TOKEN`, या `HF_TOKEN` के माध्यम से), OpenClaw उपलब्ध chat-completion models खोजने के लिए इस GET का उपयोग करता है। **interactive onboarding** के दौरान, टोकन दर्ज करने के बाद आपको एक **Default Hugging Face model** ड्रॉपडाउन दिखाई देता है, जो उस सूची से भरा होता है (या यदि अनुरोध विफल हो जाए तो बिल्ट-इन कैटलॉग से)। रनटाइम पर (उदाहरण के लिए, Gateway स्टार्टअप के समय), जब कोई key मौजूद होती है, OpenClaw कैटलॉग को रिफ्रेश करने के लिए फिर से **GET** `https://router.huggingface.co/v1/models` कॉल करता है। इस सूची को एक बिल्ट-इन कैटलॉग के साथ मर्ज किया जाता है (मेटाडेटा जैसे context window और लागत के लिए)। यदि अनुरोध विफल हो जाता है या कोई key सेट नहीं है, तो केवल बिल्ट-इन कैटलॉग का उपयोग किया जाता है।

## मॉडल नाम और संपादन योग्य विकल्प

- **API से नाम:** यदि API `name`, `title`, या `display_name` लौटाता है तो मॉडल का डिस्प्ले नाम **GET /v1/models** से **hydrated** किया जाता है; अन्यथा यह मॉडल id से निकाला जाता है (उदाहरण: `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”)।
- **डिस्प्ले नाम ओवरराइड करें:** आप config में प्रति मॉडल एक कस्टम लेबल सेट कर सकते हैं ताकि वह CLI और UI में आपकी पसंद के अनुसार दिखाई दे:

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

- **Provider / policy चयन:** राउटर बैकएंड कैसे चुने, यह निर्धारित करने के लिए **model id** में एक suffix जोड़ें:

  - **`:fastest`** — सबसे अधिक throughput (राउटर चयन करता है; provider चयन **लॉक** रहता है — कोई इंटरैक्टिव बैकएंड पिकर नहीं)।
  - **`:cheapest`** — प्रति आउटपुट टोकन सबसे कम लागत (राउटर चयन करता है; provider चयन **लॉक** रहता है)।
  - **`:provider`** — किसी विशिष्ट बैकएंड को बाध्य करें (उदाहरण: `:sambanova`, `:together`)।

  जब आप **:cheapest** या **:fastest** चुनते हैं (उदाहरण के लिए, onboarding मॉडल ड्रॉपडाउन में), तो provider लॉक हो जाता है: राउटर लागत या गति के आधार पर निर्णय लेता है और कोई वैकल्पिक “prefer specific backend” चरण प्रदर्शित नहीं होता। आप इन्हें `models.providers.huggingface.models` में अलग प्रविष्टियों के रूप में जोड़ सकते हैं या suffix के साथ `model.primary` सेट कर सकते हैं। आप [Inference Provider settings](https://hf.co/settings/inference-providers) में अपना डिफ़ॉल्ट क्रम भी सेट कर सकते हैं (कोई suffix नहीं = उसी क्रम का उपयोग होगा)।

- **Config मर्ज:** जब config मर्ज किया जाता है, तो `models.providers.huggingface.models` (उदाहरण: `models.json` में) की मौजूदा प्रविष्टियाँ बरकरार रहती हैं। इसलिए वहाँ सेट किया गया कोई भी कस्टम `name`, `alias`, या मॉडल विकल्प सुरक्षित रहता है।

## मॉडल IDs और कॉन्फ़िगरेशन उदाहरण

मॉडल refs का प्रारूप `huggingface/<org>/<model>` होता है (Hub-स्टाइल IDs)। नीचे दी गई सूची **GET** `https://router.huggingface.co/v1/models` से है; आपके कैटलॉग में इससे अधिक मॉडल शामिल हो सकते हैं।

**उदाहरण IDs (inference endpoint से):**

| मॉडल                                   | Ref (के पहले `huggingface/` जोड़ें) |
| -------------------------------------- | ------------------------------------------------------ |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                              |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                            |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                        |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                             |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                       |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                    |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                     |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                  |
| GLM 4.7                | `zai-org/GLM-4.7`                                      |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                 |

आप मॉडल id के साथ `:fastest`, `:cheapest`, या `:provider` (जैसे `:together`, `:sambanova`) जोड़ सकते हैं। अपना डिफ़ॉल्ट क्रम [Inference Provider settings](https://hf.co/settings/inference-providers) में सेट करें; पूरी सूची के लिए [Inference Providers](https://huggingface.co/docs/inference-providers) और **GET** `https://router.huggingface.co/v1/models` देखें।

### पूर्ण कॉन्फ़िगरेशन उदाहरण

**Qwen fallback के साथ प्राथमिक DeepSeek R1:**

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

**डिफ़ॉल्ट के रूप में Qwen, साथ में :cheapest और :fastest वेरिएंट:**

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

**उपनामों के साथ DeepSeek + Llama + GPT-OSS:**

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

**:provider के साथ किसी विशेष backend को अनिवार्य करें:**

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

**पॉलिसी suffix के साथ कई Qwen और DeepSeek मॉडल:**

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

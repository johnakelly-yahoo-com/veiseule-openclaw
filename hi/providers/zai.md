---
title: "Z.AI"
---

# Z.AI

Z.AI **GLM** मॉडलों के लिए API प्लेटफ़ॉर्म है। यह GLM के लिए REST APIs प्रदान करता है और API keys का उपयोग करता है।
for authentication. Create your API key in the Z.AI console. OpenClaw uses the `zai` provider
with a Z.AI API key.

## CLI सेटअप

```bash
openclaw onboard --auth-choice zai-api-key
# or non-interactive
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## विन्यास स्निपेट

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } },
}
```

## टिप्पणियाँ

- GLM मॉडल `zai/<model>` के रूप में उपलब्ध हैं (उदाहरण: `zai/glm-4.7`)।
- मॉडल परिवार के अवलोकन के लिए [/providers/glm](/providers/glm) देखें।
- Z.AI आपकी API कुंजी के साथ Bearer auth का उपयोग करता है।

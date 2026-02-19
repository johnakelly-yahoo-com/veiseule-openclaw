---
summary: "OpenClaw میں NVIDIA کا OpenAI سے مطابقت رکھنے والا API استعمال کریں"
read_when:
  - آپ OpenClaw میں NVIDIA ماڈلز استعمال کرنا چاہتے ہیں
  - آپ کو NVIDIA_API_KEY سیٹ اپ کرنا ہوگا
title: "NVIDIA"
---

# NVIDIA

NVIDIA، Nemotron اور NeMo ماڈلز کے لیے `https://integrate.api.nvidia.com/v1` پر OpenAI سے مطابقت رکھنے والا API فراہم کرتا ہے۔ [NVIDIA NGC](https://catalog.ngc.nvidia.com/) سے API key کے ذریعے تصدیق کریں۔

## CLI سیٹ اپ

کلید کو ایک بار ایکسپورٹ کریں، پھر آن بورڈنگ چلائیں اور ایک NVIDIA ماڈل سیٹ کریں:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

اگر آپ اب بھی `--token` پاس کرتے ہیں تو یاد رکھیں کہ یہ شیل ہسٹری اور `ps` آؤٹ پٹ میں محفوظ ہو جاتا ہے؛ ممکن ہو تو env ویری ایبل کو ترجیح دیں۔

## کنفیگ اسنیپٹ

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

## ماڈل IDs

- `nvidia/llama-3.1-nemotron-70b-instruct` (ڈیفالٹ)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## نوٹس

- OpenAI سے مطابقت رکھنے والا `/v1` اینڈ پوائنٹ؛ NVIDIA NGC سے حاصل کردہ API کلید استعمال کریں۔
- جب `NVIDIA_API_KEY` سیٹ ہو تو پرووائیڈر خودکار طور پر فعال ہو جاتا ہے؛ یہ جامد ڈیفالٹس استعمال کرتا ہے (131,072-ٹوکن کانٹیکسٹ ونڈو، 4,096 زیادہ سے زیادہ ٹوکنز)۔


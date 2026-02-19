---
summary: "OpenClaw’da NVIDIA’ning OpenAI-compatible API’sidan foydalanish"
read_when:
  - Siz OpenClaw’da NVIDIA modellaridan foydalanmoqchisiz
  - Sizda NVIDIA_API_KEY sozlangan bo‘lishi kerak
title: "NVIDIA"
---

# NVIDIA

NVIDIA Nemotron va NeMo modellari uchun `https://integrate.api.nvidia.com/v1` manzilida OpenAI-compatible API taqdim etadi. [NVIDIA NGC](https://catalog.ngc.nvidia.com/) orqali olingan API kaliti bilan autentifikatsiya qiling.

## CLI sozlamasi

Kalitni bir marta export qiling, so‘ng onboarding’ni ishga tushiring va NVIDIA modelini o‘rnating:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Agar hali ham `--token` uzatayotgan bo‘lsangiz, u shell history va `ps` chiqishida saqlanishini unutmang; imkon qadar env o‘zgaruvchisidan foydalaning.

## Config namunasi

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

## Model ID’lari

- `nvidia/llama-3.1-nemotron-70b-instruct` (sukut bo‘yicha)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Eslatmalar

- OpenAI-compatible `/v1` endpoint; NVIDIA NGC’dan olingan API kalitidan foydalaning.
- `NVIDIA_API_KEY` o‘rnatilganda provider avtomatik yoqiladi; statik sukut sozlamalaridan foydalanadi (131,072 tokenli kontekst oynasi, 4,096 maksimal token).

---
summary: "OpenClaw'da NVIDIA'nın OpenAI uyumlu API'sini kullanın"
read_when:
  - OpenClaw'da NVIDIA modellerini kullanmak istiyorsunuz
  - NVIDIA_API_KEY ayarının yapılmış olması gerekir
title: "NVIDIA"
---

# NVIDIA

NVIDIA, Nemotron ve NeMo modelleri için `https://integrate.api.nvidia.com/v1` adresinde OpenAI uyumlu bir API sunar. [NVIDIA NGC](https://catalog.ngc.nvidia.com/) üzerinden bir API anahtarı ile kimlik doğrulaması yapın.

## CLI kurulumu

Anahtarı bir kez dışa aktarın, ardından onboarding işlemini çalıştırın ve bir NVIDIA modeli ayarlayın:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Hâlâ `--token` kullanıyorsanız, bunun shell geçmişine ve `ps` çıktısına kaydedileceğini unutmayın; mümkün olduğunda env değişkenini tercih edin.

## Yapılandırma örneği

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

## Model Kimlikleri

- `nvidia/llama-3.1-nemotron-70b-instruct` (varsayılan)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Notlar

- OpenAI uyumlu `/v1` endpoint; NVIDIA NGC’den bir API anahtarı kullanın.
- `NVIDIA_API_KEY` ayarlandığında sağlayıcı otomatik olarak etkinleşir; statik varsayılanlar kullanır (131.072 token bağlam penceresi, 4.096 maksimum token).


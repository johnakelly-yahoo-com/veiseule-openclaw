---
summary: "OpenClaw’ı vLLM ile çalıştırın (OpenAI uyumlu yerel sunucu)"
read_when:
  - OpenClaw’ı yerel bir vLLM sunucusuna karşı çalıştırmak istiyorsunuz
  - Kendi modellerinizle OpenAI uyumlu /v1 endpoint’lerini kullanmak istiyorsunuz
title: "vLLM"
---

# vLLM

vLLM, **OpenAI uyumlu** bir HTTP API aracılığıyla açık kaynaklı (ve bazı özel) modelleri sunabilir. OpenClaw, `openai-completions` API’sini kullanarak vLLM’e bağlanabilir.

`VLLM_API_KEY` ile katılım sağladığınızda (sunucunuz kimlik doğrulama zorunlu kılmıyorsa herhangi bir değer çalışır) ve açık bir `models.providers.vllm` girdisi tanımlamadığınızda, OpenClaw vLLM’den mevcut modelleri **otomatik olarak keşfedebilir**.

## Hızlı başlangıç

1. vLLM’i OpenAI uyumlu bir sunucuyla başlatın.

Temel URL’niz `/v1` endpoint’lerini (ör. `/v1/models`, `/v1/chat/completions`) sunmalıdır. vLLM genellikle şu adreste çalışır:

- `http://127.0.0.1:8000/v1`

2. Katılım sağlayın (kimlik doğrulama yapılandırılmadıysa herhangi bir değer çalışır):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Bir model seçin (vLLM model kimliklerinizden biriyle değiştirin):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Model keşfi (örtük sağlayıcı)

`VLLM_API_KEY` ayarlandığında (veya bir kimlik doğrulama profili mevcutsa) ve `models.providers.vllm` tanımlamadığınızda, OpenClaw şu sorguyu yapar:

- `GET http://127.0.0.1:8000/v1/models`

…ve döndürülen kimlikleri model girdilerine dönüştürür.

`models.providers.vllm` açıkça ayarlanırsa, otomatik keşif atlanır ve modelleri manuel olarak tanımlamanız gerekir.

## Açık yapılandırma (manuel modeller)

Şu durumlarda açık yapılandırma kullanın:

- vLLM farklı bir host/port üzerinde çalışıyorsa.
- `contextWindow`/`maxTokens` değerlerini sabitlemek istiyorsanız.
- Sunucunuz gerçek bir API anahtarı gerektiriyorsa (veya header’ları kontrol etmek istiyorsanız).

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
            name: "Yerel vLLM Modeli",
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

## Sorun giderme

- Sunucunun erişilebilir olduğunu kontrol edin:

```bash
curl http://127.0.0.1:8000/v1/models
```

- İstekler kimlik doğrulama hatalarıyla başarısız olursa, sunucu yapılandırmanızla eşleşen gerçek bir `VLLM_API_KEY` ayarlayın veya sağlayıcıyı `models.providers.vllm` altında açıkça yapılandırın.

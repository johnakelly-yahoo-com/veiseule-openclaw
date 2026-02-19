---
summary: "Birleşik model erişimi ve maliyet takibi için OpenClaw’u LiteLLM Proxy üzerinden çalıştırın"
read_when:
  - OpenClaw’u bir LiteLLM proxy üzerinden yönlendirmek istiyorsunuz
  - LiteLLM üzerinden maliyet takibi, günlükleme veya model yönlendirme ihtiyacınız var
---

# LiteLLM

[LiteLLM](https://litellm.ai), 100’den fazla model sağlayıcısına birleşik bir API sunan açık kaynaklı bir LLM gateway’idir. Merkezi maliyet takibi, günlükleme ve OpenClaw yapılandırmanızı değiştirmeden backend’ler arasında geçiş esnekliği elde etmek için OpenClaw’u LiteLLM üzerinden yönlendirin.

## OpenClaw ile LiteLLM neden kullanılmalı?

- **Maliyet takibi** — OpenClaw’un tüm modellerde tam olarak ne kadar harcadığını görün
- **Model yönlendirme** — Yapılandırma değişikliği yapmadan Claude, GPT-4, Gemini, Bedrock arasında geçiş yapın
- **Sanal anahtarlar** — OpenClaw için harcama limitli anahtarlar oluşturun
- **Günlükleme** — Hata ayıklama için tam istek/yanıt günlükleri
- **Yedekler** — Birincil sağlayıcınız devre dışı kalırsa otomatik geçiş

## Hızlı başlangıç

### Onboarding aracılığıyla

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Manuel kurulum

1. LiteLLM Proxy'yi başlatın:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. OpenClaw'ı LiteLLM'e yönlendirin:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Hepsi bu kadar. OpenClaw artık LiteLLM üzerinden yönlendirme yapıyor.

## Yapılandırma

### Ortam değişkenleri

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Yapılandırma dosyası

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

## Sanal anahtarlar

Harcama limitleri olan, OpenClaw için özel bir anahtar oluşturun:

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

Oluşturulan anahtarı `LITELLM_API_KEY` olarak kullanın.

## Model yönlendirme

LiteLLM, model isteklerini farklı arka uçlara yönlendirebilir. LiteLLM `config.yaml` dosyanızda yapılandırın:

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

OpenClaw `claude-opus-4-6` modelini istemeye devam eder — yönlendirmeyi LiteLLM gerçekleştirir.

## Kullanımı görüntüleme

LiteLLM'in kontrol panelini veya API'sini kontrol edin:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Notlar

- LiteLLM varsayılan olarak `http://localhost:4000` üzerinde çalışır
- OpenClaw, OpenAI uyumlu `/v1/chat/completions` endpoint'i üzerinden bağlanır
- Tüm OpenClaw özellikleri LiteLLM üzerinden çalışır — herhangi bir kısıtlama yoktur

## Ayrıca bakın

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)

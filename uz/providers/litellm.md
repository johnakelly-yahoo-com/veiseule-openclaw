---
summary: "Yagona model kirishi va xarajatlarni kuzatish uchun OpenClaw’ni LiteLLM Proxy orqali ishga tushiring"
read_when:
  - OpenClaw’ni LiteLLM proxy orqali yo‘naltirmoqchisiz
  - Sizga LiteLLM orqali xarajatlarni kuzatish, loglash yoki model yo‘naltirish kerak
---

# LiteLLM

[LiteLLM](https://litellm.ai) — bu 100+ model provayderlariga yagona API taqdim etuvchi ochiq manbali LLM gateway. Markazlashgan xarajat kuzatuvi, loglash va OpenClaw konfiguratsiyasini o‘zgartirmasdan backendlarni almashtirish imkoniyati uchun OpenClaw’ni LiteLLM orqali yo‘naltiring.

## Nega OpenClaw bilan LiteLLM’dan foydalanish kerak?

- **Xarajatlarni kuzatish** — OpenClaw barcha modellar bo‘yicha qancha sarflayotganini aniq ko‘ring
- **Model yo‘naltirish** — Konfiguratsiyani o‘zgartirmasdan Claude, GPT-4, Gemini, Bedrock o‘rtasida almashing
- **Virtual kalitlar** — OpenClaw uchun xarajat limiti bilan kalitlar yarating
- **Loglash** — Nosozliklarni aniqlash uchun to‘liq so‘rov/javob loglari
- **Fallbacklar** — Asosiy provayder ishlamay qolganda avtomatik failover

## Tezkor boshlash

### Onboarding orqali

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Qo‘lda sozlash

1. LiteLLM Proxy’ni ishga tushiring:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. OpenClaw’ni LiteLLM’ga yo‘naltiring:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Hammasi shu. Endi OpenClaw LiteLLM orqali yo‘naltiriladi.

## Konfiguratsiya

### Muhit o‘zgaruvchilari

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Konfiguratsiya fayli

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

## Virtual kalitlar

Xarajat limitlari bilan OpenClaw uchun alohida kalit yarating:

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

Yaratilgan kalitni `LITELLM_API_KEY` sifatida ishlating.

## Model marshrutlash

LiteLLM model so‘rovlarini turli backend’larga yo‘naltira oladi. LiteLLM `config.yaml` faylingizda sozlang:

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

OpenClaw har doim `claude-opus-4-6` modelini so‘raydi — marshrutlashni LiteLLM bajaradi.

## Foydalanishni ko‘rish

LiteLLM boshqaruv paneli yoki API orqali tekshiring:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Eslatmalar

- LiteLLM sukut bo‘yicha `http://localhost:4000` manzilida ishlaydi
- OpenClaw OpenAI-compatible `/v1/chat/completions` endpoint orqali ulanadi
- Barcha OpenClaw funksiyalari LiteLLM orqali ishlaydi — hech qanday cheklovlarsiz

## Shuningdek qarang

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)

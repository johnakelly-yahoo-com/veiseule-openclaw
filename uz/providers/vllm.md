---
summary: "OpenClaw’ni vLLM bilan ishga tushirish (OpenAI-compatible lokal server)"
read_when:
  - Siz OpenClaw’ni lokal vLLM serverga ulab ishlatmoqchisiz
  - Siz o‘z modellaringiz bilan OpenAI-compatible /v1 endpoint’lardan foydalanmoqchisiz
title: "vLLM"
---

# vLLM

vLLM ochiq manbali (va ba’zi maxsus) modellarni **OpenAI-compatible** HTTP API orqali taqdim eta oladi. OpenClaw vLLM’ga `openai-completions` API orqali ulana oladi.

OpenClaw `VLLM_API_KEY` bilan rozilik bildirganingizda (agar serveringiz autentifikatsiyani majburlamasa, istalgan qiymat ishlaydi) va `models.providers.vllm` uchun aniq yozuv kiritmagan bo‘lsangiz, vLLM’dan mavjud modellarni **auto-discover** qila oladi.

## Tezkor boshlash

1. OpenAI-compatible server bilan vLLM’ni ishga tushiring.

Sizning base URL manzilingiz `/v1` endpointlarini (masalan, `/v1/models`, `/v1/chat/completions`) taqdim etishi kerak. vLLM odatda quyidagida ishlaydi:

- `http://127.0.0.1:8000/v1`

2. Rozilik bildiring (agar autentifikatsiya sozlanmagan bo‘lsa, istalgan qiymat ishlaydi):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Modelni tanlang (vLLM model IDlaringizdan biriga almashtiring):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Modelni aniqlash (yashirin provider)

`VLLM_API_KEY` o‘rnatilgan bo‘lsa (yoki autentifikatsiya profili mavjud bo‘lsa) va siz `models.providers.vllm` ni **aniq ko‘rsatmagan** bo‘lsangiz, OpenClaw quyidagiga so‘rov yuboradi:

- `GET http://127.0.0.1:8000/v1/models`

…va qaytarilgan IDlarni model yozuvlariga aylantiradi.

Agar siz `models.providers.vllm` ni aniq sozlasangiz, auto-discovery o‘tkazib yuboriladi va modellarni qo‘lda belgilashingiz kerak bo‘ladi.

## Aniq konfiguratsiya (qo‘lda modellar)

Quyidagi holatlarda aniq konfiguratsiyadan foydalaning:

- vLLM boshqa host/portda ishlaydi.
- Siz `contextWindow`/`maxTokens` qiymatlarini qat’iy belgilamoqchisiz.
- Serveringiz haqiqiy API kalitini talab qiladi (yoki headerlarni boshqarishni xohlaysiz).

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
            name: "Local vLLM Model",
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

## Nosozliklarni bartaraf etish

- Serverga ulanish mumkinligini tekshiring:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Agar so‘rovlar autentifikatsiya xatolari bilan muvaffaqiyatsiz tugasa, server konfiguratsiyangizga mos haqiqiy `VLLM_API_KEY` ni o‘rnating yoki providerni `models.providers.vllm` ostida aniq sozlang.


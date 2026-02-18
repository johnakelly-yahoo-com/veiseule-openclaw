---
title: "Lokal modellar"
---

# Lokal modellar

Lokal ishga tushirish mumkin, ammo OpenClaw katta kontekst va prompt injection’ga qarshi kuchli himoyani talab qiladi. Kichik GPU kartalar kontekstni qisqartiradi va xavfsizlikni zaiflashtiradi. Yuqori darajani maqsad qiling: **kamida 2 ta maksimal konfiguratsiyadagi Mac Studio yoki shunga teng GPU rig (~$30k+)**. Bitta **24 GB** GPU faqat yengilroq prompt’lar uchun va yuqori kechikish bilan ishlaydi. Imkon qadar **eng katta / to‘liq o‘lchamli model variantini ishlating**; kuchli kvantlangan yoki “kichik” checkpoint’lar prompt injection xavfini oshiradi (qarang: [Security](/gateway/security)).

## Tavsiya etiladi: LM Studio + MiniMax M2.1 (Responses API, to‘liq o‘lcham)

Hozirgi eng yaxshi lokal stek. MiniMax M2.1’ni LM Studio’da yuklang, lokal serverni yoqing (standart `http://127.0.0.1:1234`), va fikrlashni yakuniy matndan ajratish uchun Responses API’dan foydalaning.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**Sozlash bo‘yicha tekshiruv ro‘yxati**

- LM Studio’ni o‘rnating: [https://lmstudio.ai](https://lmstudio.ai)
- LM Studio ichida **mavjud eng katta MiniMax M2.1 build’ini yuklab oling** (“small”/kuchli kvantlangan variantlardan saqlaning), serverni ishga tushiring va `http://127.0.0.1:1234/v1/models` orqali model ro‘yxatda borligini tekshiring.
- Model yuklangan holda tursin; sovuq yuklash ishga tushish kechikishini oshiradi.
- Agar LM Studio build’ingiz boshqacha bo‘lsa, `contextWindow`/`maxTokens` qiymatlarini moslang.
- WhatsApp uchun faqat Responses API’dan foydalaning, shunda faqat yakuniy matn yuboriladi.

Lokal ishlatayotganda ham hosted modellarni sozlangan holda qoldiring; fallback’lar mavjud bo‘lishi uchun `models.mode: "merge"` dan foydalaning.

### Gibrid konfiguratsiya: hosted asosiy, lokal fallback

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### Lokal-birinchi, hosted xavfsizlik tarmog‘i bilan

Asosiy va fallback tartibini almashtiring; xuddi shu providers blokini va `models.mode: "merge"` ni saqlang, shunda lokal server ishlamay qolganda Sonnet yoki Opus’ga o‘ta olasiz.

### Mintaqaviy hosting / ma’lumot yo‘naltirish

- Hosted MiniMax/Kimi/GLM variantlari OpenRouter’da ham mavjud va mintaqaga bog‘langan endpoint’larga ega (masalan, US-hosted). Trafikni tanlagan yurisdiksiyangizda saqlash uchun u yerda mintaqaviy variantni tanlang va shu bilan birga Anthropic/OpenAI fallback’lari uchun `models.mode: "merge"` dan foydalaning.
- Faqat-lokal yechim maxfiylik uchun eng kuchli yo‘l; hosted mintaqaviy yo‘naltirish esa provayder funksiyalari kerak bo‘lganda va ma’lumot oqimini nazorat qilishni istaganda o‘rtacha variantdir.

## Boshqa OpenAI-compatible lokal proksilar

vLLM, LiteLLM, OAI-proxy yoki maxsus gateway’lar OpenAI uslubidagi `/v1` endpoint’ni taqdim etsa, ishlaydi. Yuqoridagi provider blokini o‘z endpoint’ingiz va model ID bilan almashtiring:

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Hosted modellar fallback sifatida mavjud bo‘lib qolishi uchun `models.mode: "merge"` ni saqlang.

## Muammolarni bartaraf etish

- Gateway proksiga ulanganmi? `curl http://127.0.0.1:1234/v1/models`.
- LM Studio modeli yuklanmaganmi? Qayta yuklang; sovuq ishga tushish ko‘pincha “osilib qolish” sababi bo‘ladi.
- Kontekst xatolari? `contextWindow` ni kamaytiring yoki server limitini oshiring.
- Xavfsizlik: lokal modellar provayder tomondagi filtrlarsiz ishlaydi; prompt injection ta’sir doirasini cheklash uchun agentlarni tor doirada saqlang va compaction’ni yoqilgan holda qoldiring.

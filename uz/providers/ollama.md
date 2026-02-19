---
summary: "OpenClaw’ni Ollama (mahalliy LLM runtime) bilan ishga tushirish"
read_when:
  - Siz OpenClaw’ni Ollama orqali mahalliy modellarda ishga tushirmoqchisiz
  - Sizga Ollama’ni o‘rnatish va sozlash bo‘yicha qo‘llanma kerak
title: "Ollama"
---

# Ollama

Ollama — bu ochiq manbali modellarni kompyuteringizda oson ishga tushirish imkonini beruvchi mahalliy LLM runtime. OpenClaw Ollama’ning mahalliy API (`/api/chat`) bilan integratsiyalashgan bo‘lib, streaming va tool calling’ni qo‘llab-quvvatlaydi hamda `OLLAMA_API_KEY` (yoki auth profile) orqali ruxsat berilgan va `models.providers.ollama` bo‘limi aniq ko‘rsatilmagan holatda **tool-capable modellarni avtomatik aniqlay oladi**.

## Tezkor boshlash

1. Ollama’ni o‘rnating: [https://ollama.ai](https://ollama.ai)

2. Modelni yuklab oling:

```bash
ollama pull gpt-oss:20b
# yoki
ollama pull llama3.3
# yoki
ollama pull qwen2.5-coder:32b
# yoki
ollama pull deepseek-r1:32b
```

3. OpenClaw uchun Ollama’ni yoqing (istalgan qiymat mos keladi; Ollama haqiqiy kalitni talab qilmaydi):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Ollama modellaridan foydalaning:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Modelni aniqlash (implicit provider)

`OLLAMA_API_KEY` (yoki autentifikatsiya profili) o‘rnatilganda va `models.providers.ollama` **aniqlanmagan** bo‘lsa, OpenClaw `http://127.0.0.1:11434` manzilidagi mahalliy Ollama instansiyasidan modellarni avtomatik aniqlaydi:

- `/api/tags` va `/api/show` so‘rovlarini yuboradi
- Faqat `tools` imkoniyatini bildirgan modellarni saqlab qoladi
- Model `thinking` haqida xabar berganda `reasoning` sifatini belgilaydi
- Mavjud bo‘lsa, `contextWindow` ni `model_info["<arch>.context_length"]` dan o‘qiydi
- `maxTokens` ni kontekst oynasining 10 baravariga teng qilib belgilaydi
- Barcha xarajatlarni `0` ga o‘rnatadi

Bu qo‘lda model kiritishdan qochishga va katalogni Ollama imkoniyatlari bilan mos holda saqlashga yordam beradi.

Qaysi modellar mavjudligini ko‘rish uchun:

```bash
ollama list
openclaw models list
```

Yangi model qo‘shish uchun uni Ollama orqali shunchaki yuklab oling:

```bash
ollama pull mistral
```

Yangi model avtomatik aniqlanadi va foydalanish uchun mavjud bo‘ladi.

Agar `models.providers.ollama` ni aniq belgilasangiz, avtomatik aniqlash o‘chiriladi va modellarni qo‘lda belgilashingiz kerak bo‘ladi (quyida qarang).

## Konfiguratsiya

### Asosiy sozlash (implicit aniqlash)

Ollama’ni yoqishning eng oddiy usuli — muhit o‘zgaruvchisi orqali:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Aniq sozlash (qo‘lda modellar)

Quyidagi holatlarda aniq konfiguratsiyadan foydalaning:

- Ollama boshqa host yoki portda ishlayotgan bo‘lsa.
- Muayyan kontekst oynalari yoki model ro‘yxatlarini majburan belgilamoqchi bo‘lsangiz.
- `tools` qo‘llab-quvvatlashini bildirmaydigan modellarni qo‘shmoqchi bo‘lsangiz.

```json5
{
  models: {
    providers: {
      ollama: {
        // Use a host that includes /v1 for OpenAI-compatible APIs
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Agar `OLLAMA_API_KEY` o‘rnatilgan bo‘lsa, provider yozuvida `apiKey` ni ko‘rsatmasangiz ham bo‘ladi — OpenClaw uni mavjudlik tekshiruvlari uchun avtomatik to‘ldiradi.

### Maxsus base URL (aniq konfiguratsiya)

Agar Ollama boshqa host yoki portda ishlayotgan bo‘lsa (aniq konfiguratsiya avtomatik aniqlashni o‘chiradi, shuning uchun modellarni qo‘lda belgilang):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1",
      },
    },
  },
}
```

### Model tanlash

Sozlangandan so‘ng, barcha Ollama modellaringiz mavjud bo‘ladi:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## Kengaytirilgan

### Reasoning modellari

Ollama `/api/show` da `thinking` haqida xabar berganda, OpenClaw modellarni reasoning imkoniyatiga ega deb belgilaydi:

```bash
ollama pull deepseek-r1:32b
```

### Model xarajatlari

Ollama bepul va mahalliy ishlaydi, shuning uchun barcha model xarajatlari $0 ga teng.

### Streaming konfiguratsiyasi

OpenClaw’ning Ollama integratsiyasi sukut bo‘yicha **mahalliy Ollama API** (`/api/chat`) dan foydalanadi, u streaming va tool calling’ni bir vaqtda to‘liq qo‘llab-quvvatlaydi. Hech qanday maxsus sozlash talab qilinmaydi.

#### Legacy OpenAI-Compatible Mode

Agar buning o‘rniga OpenAI-compatible endpoint’dan foydalanishingiz kerak bo‘lsa (masalan, faqat OpenAI formatini qo‘llab-quvvatlaydigan proxy ortida), `api: "openai-completions"` ni aniq belgilang:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Eslatma: OpenAI-compatible endpoint streaming va tool calling’ni bir vaqtda qo‘llab-quvvatlamasligi mumkin. Model konfiguratsiyasida `params: { streaming: false }` orqali streaming’ni o‘chirishingiz kerak bo‘lishi mumkin.

### Context windows

Avtomatik aniqlangan modellar uchun OpenClaw mavjud bo‘lsa, Ollama xabar qilgan kontekst oynasidan foydalanadi, aks holda `8192` ga sukut bo‘yicha o‘rnatadi. Aniq provayder konfiguratsiyasida `contextWindow` va `maxTokens` ni o‘zgartirishingiz mumkin.

## Troubleshooting

### Ollama not detected

Make sure Ollama is running and that you set `OLLAMA_API_KEY` (or an auth profile), and that you did **not** define an explicit `models.providers.ollama` entry:

```bash
ollama serve
```

And that the API is accessible:

```bash
curl http://localhost:11434/api/tags
```

### Hech qanday model mavjud emas

OpenClaw only auto-discovers models that report tool support. Agar modelingiz ro‘yxatda bo‘lmasa, quyidagilardan biri:

- Pull a tool-capable model, or
- Define the model explicitly in `models.providers.ollama`.

Modellarni qo‘shish uchun:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Ulanish rad etildi

Modellarni qo‘shish uchun:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## Ulanish rad etildi

- [Model Providers](/concepts/model-providers) - Overview of all providers
- [Model Selection](/concepts/models) - How to choose models
- [Configuration](/gateway/configuration) - Full config reference


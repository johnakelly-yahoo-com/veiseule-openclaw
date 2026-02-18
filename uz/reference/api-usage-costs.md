---
title: "API’dan foydalanish va xarajatlar"
---

# API’dan foydalanish va xarajatlar

Ushbu hujjat **API kalitlarini ishga tushirishi mumkin bo‘lgan funksiyalarni** va ularning xarajatlari qayerda ko‘rinishini sanab o‘tadi. U
provayderdan foydalanish yoki pullik API chaqiruvlarini keltirib chiqarishi mumkin bo‘lgan OpenClaw funksiyalariga qaratilgan.

## Xarajatlar qayerda ko‘rinadi (chat + CLI)

**Sessiya bo‘yicha xarajat ko‘rinishi**

- `/status` joriy sessiya modeli, kontekstdan foydalanish va oxirgi javob tokenlarini ko‘rsatadi.
- Agar model **API-key auth** dan foydalansa, `/status` oxirgi javob uchun **taxminiy xarajatni** ham ko‘rsatadi.

**Har bir xabar uchun xarajat izohi**

- `/usage full` har bir javob oxiriga foydalanish izohini qo‘shadi, jumladan **taxminiy xarajat** (faqat API-key).
- `/usage tokens` faqat tokenlarni ko‘rsatadi; OAuth oqimlarida dollar qiymati yashiriladi.

**CLI foydalanish oynalari (provayder kvotalari)**

- `openclaw status --usage` va `openclaw channels list` provayderning **foydalanish oynalarini**
  ko‘rsatadi (har bir xabar xarajati emas, kvota ko‘rinishi).

Batafsil ma’lumot va misollar uchun [Token use & costs](/reference/token-use) ga qarang.

## Kalitlar qanday aniqlanadi

OpenClaw hisob ma’lumotlarini quyidagilardan olishi mumkin:

- **Auth profillari** (har bir agent uchun, `auth-profiles.json` da saqlanadi).
- **Muhit o‘zgaruvchilari** (masalan, `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
- **Konfiguratsiya** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
- **Skills** (`skills.entries.<name>.apiKey`) — kalitlarni skill jarayoni muhitiga eksport qilishi mumkin.

## Kalitlardan mablag‘ sarflashi mumkin bo‘lgan funksiyalar

### 1) Asosiy model javoblari (chat + vositalar)

Har bir javob yoki vosita chaqiruvi **joriy model provayderidan** (OpenAI, Anthropic va boshqalar) foydalanadi. Bu
foydalanish va xarajatning asosiy manbai hisoblanadi.

Narx sozlamalari uchun [Models](/providers/models) va ko‘rsatish tafsilotlari uchun [Token use & costs](/reference/token-use) ga qarang.

### 2) Media tushunish (audio/rasm/video)

Kirish mediasi javobdan oldin qisqacha bayon qilinishi yoki transkripsiya qilinishi mumkin. Bu model/provayder API’laridan foydalanadi.

- Audio: OpenAI / Groq / Deepgram (endi **kalit mavjud bo‘lsa avtomatik yoqiladi**).
- Rasm: OpenAI / Anthropic / Google.
- Video: Google.

Qarang: [Media understanding](/nodes/media-understanding).

### 3) Xotira embeddinglari + semantik qidiruv

Semantik xotira qidiruvi masofaviy provayderlar sozlangan bo‘lsa **embedding API’lari** dan foydalanadi:

- `memorySearch.provider = "openai"` → OpenAI embeddinglari
- `memorySearch.provider = "gemini"` → Gemini embeddinglari
- `memorySearch.provider = "voyage"` → Voyage embeddinglari
- Mahalliy embeddinglar ishlamasa, ixtiyoriy masofaviy provayderga o‘tish

`memorySearch.provider = "local"` bilan uni mahalliy saqlashingiz mumkin (API ishlatilmaydi).

Qarang: [Memory](/concepts/memory).

### 4) Web qidiruv vositasi (Brave / Perplexity via OpenRouter)

`web_search` API kalitlaridan foydalanadi va foydalanish uchun to‘lov olinishi mumkin:

- **Brave Search API**: `BRAVE_API_KEY` yoki `tools.web.search.apiKey`
- **Perplexity** (via OpenRouter): `PERPLEXITY_API_KEY` yoki `OPENROUTER_API_KEY`

**Brave bepul tarifi (keng imkoniyatli):**

- **Oyiga 2 000 so‘rov**
- **Soniyasiga 1 so‘rov**
- Tasdiqlash uchun **bank kartasi talab qilinadi** (yangilamaguningizcha to‘lov olinmaydi)

Qarang: [Web tools](/tools/web).

### 5) Web fetch vositasi (Firecrawl)

`web_fetch` API kalit mavjud bo‘lsa **Firecrawl** ni chaqirishi mumkin:

- `FIRECRAWL_API_KEY` yoki `tools.web.fetch.firecrawl.apiKey`

Agar Firecrawl sozlanmagan bo‘lsa, vosita to‘g‘ridan-to‘g‘ri fetch + readability usuliga o‘tadi (pullik API yo‘q).

Qarang: [Web tools](/tools/web).

### 6) Provayder foydalanish ko‘rinishlari (status/health)

Ba’zi status buyruqlari **provayder foydalanish endpointlari** ni chaqirib, kvota oynalari yoki autentifikatsiya holatini ko‘rsatadi.
Bu odatda kam hajmli chaqiruvlar, ammo baribir provayder API’lariga murojaat qiladi:

- `openclaw status --usage`
- `openclaw models status --json`

Qarang: [Models CLI](/cli/models).

### 7) Kompaktlash himoyasi orqali qisqacha bayon

Kompaktlash himoyasi sessiya tarixini **joriy model** yordamida qisqacha bayon qilishi mumkin,
bu ishga tushganda provayder API’larini chaqiradi.

Qarang: [Session management + compaction](/reference/session-management-compaction).

### 8) Model skanerlash / sinov

`openclaw models scan` OpenRouter modellarini sinovdan o‘tkazishi mumkin va
sinov yoqilgan bo‘lsa `OPENROUTER_API_KEY` dan foydalanadi.

Qarang: [Models CLI](/cli/models).

### 9) Talk (nutq)

Talk rejimi sozlangan bo‘lsa **ElevenLabs** ni chaqirishi mumkin:

- `ELEVENLABS_API_KEY` yoki `talk.apiKey`

Qarang: [Talk mode](/nodes/talk).

### 10) Skills (uchinchi tomon API’lari)

Skills `skills.entries.<name>.apiKey` da `apiKey` saqlashi mumkin. Agar skill ushbu kalitdan tashqi
API’lar uchun foydalansa, skill provayderi shartlariga ko‘ra xarajatlar yuzaga kelishi mumkin.

Qarang: [Skills](/tools/skills).



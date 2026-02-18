---
title: "Plugin manifesti"
---

# Plugin manifesti (openclaw.plugin.json)

Har bir plagin **plugin root** papkasida `openclaw.plugin.json` faylini taqdim etishi **shart**.
OpenClaw ushbu manifestdan plagin kodini **ishga tushirmasdan** konfiguratsiyani tekshirish uchun foydalanadi. Yetishmayotgan yoki noto‘g‘ri manifest plagin xatosi sifatida qabul qilinadi va konfiguratsiya tekshiruvini bloklaydi.

To‘liq plagin tizimi qo‘llanmasini ko‘ring: [Plugins](/tools/plugin).

## Majburiy maydonlar

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Majburiy kalitlar:

- `id` (string): plaginining kanonik identifikatori.
- `configSchema` (object): plagin konfiguratsiyasi uchun JSON Schema (inline).

Ixtiyoriy kalitlar:

- `kind` (string): plagin turi (masalan: `"memory"`).
- `channels` (array): ushbu plagin ro‘yxatdan o‘tkazadigan channel identifikatorlari (masalan: `["matrix"]`).
- `providers` (array): ushbu plagin ro‘yxatdan o‘tkazadigan provider identifikatorlari.
- `skills` (array): yuklanadigan skill kataloglari (plugin root ga nisbatan).
- `name` (string): plagin uchun ko‘rsatiladigan nom.
- `description` (string): plagin haqida qisqa tavsif.
- `uiHints` (object): UI’da ko‘rsatish uchun konfiguratsiya maydonlari yorliqlari/placeholderlari/maxfiy belgilari.
- `version` (string): plagin versiyasi (ma’lumot uchun).

## JSON Schema talablari

- **Har bir plagin JSON Schema taqdim etishi shart**, hatto konfiguratsiya qabul qilmasa ham.
- Bo‘sh schema qabul qilinadi (masalan, `{ "type": "object", "additionalProperties": false }`).
- Sxemalar konfiguratsiyani o‘qish/yozish vaqtida tekshiriladi, runtime jarayonida emas.

## Tekshiruv xatti-harakati

- Noma’lum `channels.*` kalitlari **xato** hisoblanadi, agar channel identifikatori
  plagin manifestida e’lon qilinmagan bo‘lsa.
- `plugins.entries.<id>`, `plugins.allow`, `plugins.deny`, va `plugins.slots.*`
  **topilishi mumkin bo‘lgan** plagin identifikatorlariga murojaat qilishi kerak. Noma’lum identifikatorlar **xato** hisoblanadi.
- Agar plagin o‘rnatilgan bo‘lsa, lekin manifesti yoki sxemasi buzilgan yoki mavjud bo‘lmasa,
  tekshiruv muvaffaqiyatsiz yakunlanadi va Doctor plagin xatosini ko‘rsatadi.
- Agar plagin konfiguratsiyasi mavjud bo‘lsa, lekin plagin **o‘chirilgan** bo‘lsa,
  konfiguratsiya saqlanadi va Doctor + loglarda **ogohlantirish** ko‘rsatiladi.

## Eslatmalar

- Manifest **barcha plaginlar uchun majburiy**, jumladan lokal fayl tizimidan yuklanadiganlar uchun ham.
- Runtime baribir plagin modulini alohida yuklaydi; manifest faqat
  aniqlash + tekshirish uchun xizmat qiladi.
- Agar plaginingiz native modullarga bog‘liq bo‘lsa, build bosqichlarini va
  paket menejeri allowlist talablari (masalan, pnpm `allow-build-scripts`
  - `pnpm rebuild <package>`) haqida hujjatlashtiring.

---
title: "Zalo"
---

# Zalo (Bot API)

Holati: eksperimental. Faqat shaxsiy xabarlar; guruhlar Zalo hujjatlariga ko‘ra tez orada qo‘shiladi.

## Plagin talab qilinadi

Zalo plagin sifatida taqdim etiladi va asosiy o‘rnatish paketiga kiritilmagan.

- CLI orqali o‘rnating: `openclaw plugins install @openclaw/zalo`
- Yoki boshlang‘ich sozlash vaqtida **Zalo** ni tanlang va o‘rnatishni tasdiqlang
- Tafsilotlar: [Plugins](/tools/plugin)

## Tezkor sozlash (boshlovchilar uchun)

1. Zalo plaginini o‘rnating:
   - Manba kodidan: `openclaw plugins install ./extensions/zalo`
   - npm orqali (agar chop etilgan bo‘lsa): `openclaw plugins install @openclaw/zalo`
   - Yoki boshlang‘ich sozlashda **Zalo** ni tanlang va o‘rnatishni tasdiqlang
2. Tokenni sozlang:
   - Muhit o‘zgaruvchisi: `ZALO_BOT_TOKEN=...`
   - Yoki konfiguratsiyada: `channels.zalo.botToken: "..."`.
3. Gateway’ni qayta ishga tushiring (yoki boshlang‘ich sozlashni yakunlang).
4. Shaxsiy xabarlarga kirish sukut bo‘yicha pairing orqali amalga oshiriladi; birinchi aloqa vaqtida pairing kodini tasdiqlang.

Minimal konfiguratsiya:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## Bu nima?

Zalo — Vyetnamga yo‘naltirilgan messenjer ilovasi; uning Bot API Gateway’ga 1:1 suhbatlar uchun bot ishlatish imkonini beradi.  
Agar siz Zalo’ga aniq va ishonchli yo‘naltirishni xohlasangiz (masalan, qo‘llab-quvvatlash yoki bildirishnomalar uchun), bu yaxshi tanlov.

- Gateway tomonidan boshqariladigan Zalo Bot API kanali.
- Deterministik marshrutlash: javoblar Zalo’ga qaytadi; model kanallarni tanlamaydi.
- Shaxsiy xabarlar agentning asosiy sessiyasi bilan bo‘lishiladi.
- Guruhlar hozircha qo‘llab-quvvatlanmaydi (Zalo hujjatlarida "tez orada" deb ko‘rsatilgan).

## Sozlash (tezkor usul)

### 1) Bot tokenini yarating (Zalo Bot Platform)

1. [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) sahifasiga o‘ting va tizimga kiring.
2. Yangi bot yarating va sozlamalarini moslang.
3. Bot tokenini nusxalang (format: `12345689:abc-xyz`).

### 2) Tokenni sozlang (muhit yoki konfiguratsiya orqali)

Misol:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

Muhit o‘zgaruvchisi varianti: `ZALO_BOT_TOKEN=...` (faqat sukut bo‘yicha akkaunt uchun ishlaydi).

Ko‘p akkauntli qo‘llab-quvvatlash: har bir akkaunt uchun token va ixtiyoriy `name` bilan `channels.zalo.accounts` dan foydalaning.

3. Gateway’ni qayta ishga tushiring. Token (muhit yoki konfiguratsiya orqali) aniqlanganda Zalo ishga tushadi.
4. Shaxsiy xabarlar uchun kirish sukut bo‘yicha pairing orqali amalga oshiriladi. Bot bilan birinchi aloqa vaqtida kodni tasdiqlang.

## Qanday ishlaydi (xulq-atvor)

- Kiruvchi xabarlar media o‘rinbosarlari bilan umumiy kanal formatiga normallashtiriladi.
- Javoblar har doim o‘sha Zalo chatiga qaytariladi.
- Sukut bo‘yicha long-polling; `channels.zalo.webhookUrl` bilan webhook rejimi mavjud.

## Cheklovlar

- Chiquvchi matn 2000 belgiga bo‘linadi (Zalo API cheklovi).
- Media yuklab olish/yuklash `channels.zalo.mediaMaxMb` bilan cheklangan (sukut bo‘yicha 5 MB).
- 2000 belgi cheklovi sababli streaming sukut bo‘yicha bloklangan.

## Kirishni boshqarish (DM)

### Shaxsiy xabarlarga kirish

- Sukut bo‘yicha: `channels.zalo.dmPolicy = "pairing"`. Noma’lum yuboruvchilar pairing kodini oladi; tasdiqlanmaguncha xabarlar e’tiborsiz qoldiriladi (kodlar 1 soatdan keyin muddati tugaydi).
- Tasdiqlash:
  - `openclaw pairing list zalo`
  - `openclaw pairing approve zalo <CODE>`
- Pairing — sukut bo‘yicha token almashish usuli. Tafsilotlar: [Pairing](/channels/pairing)
- `channels.zalo.allowFrom` raqamli foydalanuvchi ID’larini qabul qiladi (username orqali qidirish mavjud emas).

## Long-polling va webhook

- Sukut bo‘yicha: long-polling (ommaviy URL talab qilinmaydi).
- Webhook rejimi: `channels.zalo.webhookUrl` va `channels.zalo.webhookSecret` ni sozlang.
  - Webhook siri 8–256 belgidan iborat bo‘lishi kerak.
  - Webhook URL HTTPS’dan foydalanishi shart.
  - Zalo tasdiqlash uchun `X-Bot-Api-Secret-Token` sarlavhasi bilan hodisalarni yuboradi.
  - Gateway HTTP webhook so‘rovlarini `channels.zalo.webhookPath` da qabul qiladi (sukut bo‘yicha webhook URL yo‘li).

**Eslatma:** Zalo API hujjatlariga ko‘ra getUpdates (polling) va webhook bir vaqtda ishlamaydi.

## Qo‘llab-quvvatlanadigan xabar turlari

- **Matnli xabarlar**: 2000 belgi bo‘yicha bo‘lish bilan to‘liq qo‘llab-quvvatlanadi.
- **Rasmli xabarlar**: kiruvchi rasmlar yuklab olinadi va qayta ishlanadi; rasmlar `sendPhoto` orqali yuboriladi.
- **Stikerlar**: qayd etiladi, ammo to‘liq qayta ishlanmaydi (agent javobi yo‘q).
- **Qo‘llab-quvvatlanmaydigan turlar**: qayd etiladi (masalan, himoyalangan foydalanuvchilardan xabarlar).

## Imkoniyatlar

| Funksiya        | Holati                         |
| --------------- | ------------------------------ |
| Shaxsiy xabarlar | ✅ Qo‘llab-quvvatlanadi        |
| Guruhlar        | ❌ Tez orada (Zalo hujjatlariga ko‘ra) |
| Media (rasmlar) | ✅ Qo‘llab-quvvatlanadi        |
| Reaksiyalar     | ❌ Qo‘llab-quvvatlanmaydi      |
| Tarmoqlar (threads) | ❌ Qo‘llab-quvvatlanmaydi  |
| So‘rovnomalar   | ❌ Qo‘llab-quvvatlanmaydi      |
| Native buyruqlar | ❌ Qo‘llab-quvvatlanmaydi     |
| Streaming       | ⚠️ Bloklangan (2000 belgi cheklovi) |

## Yetkazib berish manzillari (CLI/cron)

- Maqsad sifatida chat ID’dan foydalaning.
- Misol: `openclaw message send --channel zalo --target 123456789 --message "hi"`.

## Muammolarni bartaraf etish

**Bot javob bermayapti:**

- Token to‘g‘ri ekanini tekshiring: `openclaw channels status --probe`
- Yuboruvchi tasdiqlanganini tekshiring (pairing yoki allowFrom)
- Gateway loglarini tekshiring: `openclaw logs --follow`

**Webhook hodisalarni qabul qilmayapti:**

- Webhook URL HTTPS’dan foydalanayotganini tekshiring
- Secret token 8–256 belgidan iborat ekanini tekshiring
- Gateway HTTP endpoint sozlangan yo‘lda ochiq ekanini tasdiqlang
- getUpdates polling ishlamayotganini tekshiring (ular bir vaqtda ishlamaydi)

## Konfiguratsiya ma’lumotnomasi (Zalo)

To‘liq konfiguratsiya: [Configuration](/gateway/configuration)

Provayder sozlamalari:

- `channels.zalo.enabled`: kanalni ishga tushirishni yoqish/o‘chirish.
- `channels.zalo.botToken`: Zalo Bot Platform’dan olingan bot tokeni.
- `channels.zalo.tokenFile`: tokenni fayldan o‘qish yo‘li.
- `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (sukut bo‘yicha: pairing).
- `channels.zalo.allowFrom`: DM allowlist (foydalanuvchi ID’lari). `open` uchun `"*"` talab qilinadi. Sozlash ustasi raqamli ID’larni so‘raydi.
- `channels.zalo.mediaMaxMb`: kiruvchi/chiquvchi media cheklovi (MB, sukut bo‘yicha 5).
- `channels.zalo.webhookUrl`: webhook rejimini yoqish (HTTPS talab qilinadi).
- `channels.zalo.webhookSecret`: webhook siri (8–256 belgi).
- `channels.zalo.webhookPath`: gateway HTTP serveridagi webhook yo‘li.
- `channels.zalo.proxy`: API so‘rovlari uchun proxy URL.

Ko‘p akkauntli sozlamalar:

- `channels.zalo.accounts.<id>.botToken`: har bir akkaunt uchun token.
- `channels.zalo.accounts.<id>.tokenFile`: har bir akkaunt uchun token fayli.
- `channels.zalo.accounts.<id>.name`: ko‘rsatiladigan nom.
- `channels.zalo.accounts.<id>.enabled`: akkauntni yoqish/o‘chirish.
- `channels.zalo.accounts.<id>.dmPolicy`: har bir akkaunt uchun DM siyosati.
- `channels.zalo.accounts.<id>.allowFrom`: har bir akkaunt uchun allowlist.
- `channels.zalo.accounts.<id>.webhookUrl`: har bir akkaunt uchun webhook URL.
- `channels.zalo.accounts.<id>.webhookSecret`: har bir akkaunt uchun webhook siri.
- `channels.zalo.accounts.<id>.webhookPath`: har bir akkaunt uchun webhook yo‘li.
- `channels.zalo.accounts.<id>.proxy`: har bir akkaunt uchun proxy URL.



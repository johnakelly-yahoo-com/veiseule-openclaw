---
title: "Mattermost"
---

# Mattermost (plagin)

Holati: plagin orqali qo‘llab-quvvatlanadi (bot tokeni + WebSocket hodisalari). Kanallar, guruhlar va DMlar qo‘llab-quvvatlanadi.
Mattermost — o‘zingiz joylashtirishingiz mumkin bo‘lgan jamoaviy xabar almashish platformasi; mahsulot tafsilotlari va yuklab olish uchun rasmiy saytga qarang:
[mattermost.com](https://mattermost.com).

## Plagin talab qilinadi

Mattermost plagin sifatida taqdim etiladi va asosiy o‘rnatish tarkibiga kiritilmagan.

CLI orqali o‘rnating (npm registry):

```bash
openclaw plugins install @openclaw/mattermost
```

Mahalliy checkout (git repodan ishga tushirilganda):

```bash
openclaw plugins install ./extensions/mattermost
```

Agar sozlash/ilk ishga tushirish vaqtida Mattermost tanlansa va git checkout aniqlansa,
OpenClaw avtomatik ravishda mahalliy o‘rnatish yo‘lini taklif qiladi.

Batafsil: [Plugins](/tools/plugin)

## Tezkor sozlash

1. Mattermost plaginini o‘rnating.
2. Mattermost bot akkauntini yarating va **bot token**ni nusxa oling.
3. Mattermost **base URL** manzilini nusxa oling (masalan, `https://chat.example.com`).
4. OpenClaw’ni sozlang va gateway’ni ishga tushiring.

Minimal konfiguratsiya:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Environment o‘zgaruvchilari (standart akkaunt)

Agar environment o‘zgaruvchilaridan foydalanmoqchi bo‘lsangiz, ularni gateway joylashgan hostda sozlang:

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

Environment o‘zgaruvchilari faqat **default** akkauntga (`default`) qo‘llaniladi. Boshqa akkauntlar konfiguratsiya qiymatlaridan foydalanishi kerak.

## Chat rejimlari

Mattermost DMlarga avtomatik javob beradi. Kanal xatti-harakati `chatmode` orqali boshqariladi:

- `oncall` (standart): kanallarda faqat @mention qilinganda javob beradi.
- `onmessage`: kanaldagi har bir xabarga javob beradi.
- `onchar`: xabar belgilangan trigger prefiksi bilan boshlanganda javob beradi.

Konfiguratsiya namunasi:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Eslatmalar:

- `onchar` aniq @mentionlarga ham javob beradi.
- `channels.mattermost.requireMention` eski konfiguratsiyalar uchun amal qiladi, biroq `chatmode` tavsiya etiladi.

## Kirish nazorati (DMlar)

- Standart: `channels.mattermost.dmPolicy = "pairing"` (noma’lum jo‘natuvchilarga pairing kodi beriladi).
- Tasdiqlash:
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- Ochiq DMlar: `channels.mattermost.dmPolicy="open"` va `channels.mattermost.allowFrom=["*"]`.

## Kanallar (guruhlar)

- Standart: `channels.mattermost.groupPolicy = "allowlist"` (@mention orqali cheklangan).
- `channels.mattermost.groupAllowFrom` yordamida jo‘natuvchilarni allowlist’ga qo‘shing (foydalanuvchi IDlari yoki `@username`).
- Ochiq kanallar: `channels.mattermost.groupPolicy="open"` (@mention orqali cheklangan).

## Chiquvchi xabarlar uchun target formatlari

Quyidagi target formatlaridan `openclaw message send` yoki cron/webhooks bilan foydalaning:

- `channel:<id>` — kanal uchun
- `user:<id>` — DM uchun
- `@username` — DM uchun (Mattermost API orqali aniqlanadi)

Oddiy IDlar kanal sifatida qabul qilinadi.

## Ko‘p akkauntli rejim

Mattermost `channels.mattermost.accounts` ostida bir nechta akkauntni qo‘llab-quvvatlaydi:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Nosozliklarni bartaraf etish

- Kanallarda javob yo‘q: bot kanalga qo‘shilganini tekshiring va uni @mention qiling (`oncall`), trigger prefiksidan foydalaning (`onchar`) yoki `chatmode: "onmessage"` ni sozlang.
- Autentifikatsiya xatolari: bot token, base URL va akkaunt yoqilganini tekshiring.
- Ko‘p akkaunt bilan bog‘liq muammolar: environment o‘zgaruvchilari faqat `default` akkauntga qo‘llaniladi.
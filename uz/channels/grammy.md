---
summary: "grammY orqali Telegram Bot API integratsiyasi va sozlash eslatmalari"
read_when:
  - Telegram yoki grammY yo‘llari ustida ishlash
title: grammY
---

# grammY integratsiyasi (Telegram Bot API)

# Nima uchun grammY

- TS-first Bot API mijoz — ichki long-poll + webhook yordamchilari, middleware, xatolarni qayta ishlash va tezlik cheklovchiga ega.
- Qo‘lda fetch + FormData yozishga qaraganda tozaroq media yordamchilari; Bot API’ning barcha usullarini qo‘llab-quvvatlaydi.
- Kengaytiriladigan: maxsus fetch orqali proksi qo‘llab-quvvatlashi, sessiya middleware’i (ixtiyoriy), turga xavfsiz kontekst.

# Nimani chiqardik

- **Yagona mijoz yo‘li:** fetch asosidagi implementatsiya olib tashlandi; endi grammY yagona Telegram mijozi (yuborish + gateway) bo‘lib, grammY throttler sukut bo‘yicha yoqilgan.
- **Gateway:** `monitorTelegramProvider` builds a grammY `Bot`, wires mention/allowlist gating, media download via `getFile`/`download`, and delivers replies with `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument`. Supports long-poll or webhook via `webhookCallback`.
- **Proxy:** ixtiyoriy `channels.telegram.proxy` grammY’ning `client.baseFetch` orqali `undici.ProxyAgent`dan foydalanadi.
- **Webhook support:** `webhook-set.ts` wraps `setWebhook/deleteWebhook`; `webhook.ts` hosts the callback with health + graceful shutdown. Gateway enables webhook mode when `channels.telegram.webhookUrl` + `channels.telegram.webhookSecret` are set (otherwise it long-polls).
- **Sessiyalar:** to‘g‘ridan-to‘g‘ri chatlar agentning asosiy sessiyasiga birlashtiriladi (`agent:<agentId>:<mainKey>`); guruhlar `agent:<agentId>:telegram:group:<chatId>`dan foydalanadi; javoblar o‘sha kanalga qayta yo‘naltiriladi.
- **Config knobs:** `channels.telegram.botToken`, `channels.telegram.dmPolicy`, `channels.telegram.groups` (allowlist + mention defaults), `channels.telegram.allowFrom`, `channels.telegram.groupAllowFrom`, `channels.telegram.groupPolicy`, `channels.telegram.mediaMaxMb`, `channels.telegram.linkPreview`, `channels.telegram.proxy`, `channels.telegram.webhookSecret`, `channels.telegram.webhookUrl`.
- **Jonli translyatsiya oldindan ko‘rish:** ixtiyoriy `channels.telegram.streamMode` vaqtinchalik xabar yuboradi va uni `editMessageText` bilan yangilaydi. This is separate from channel block streaming.
- **Testlar:** grammy mock’lari DM + guruh mention filtrlashi va chiqish yuborishni qamrab oladi; qo‘shimcha media/webhook fixture’lari mamnuniyat bilan qabul qilinadi.

Ochiq savollar

- Agar Bot API 429 xatolariga duch kelsak, ixtiyoriy grammY plaginlari (throttler).
- Add more structured media tests (stickers, voice notes).
- Make webhook listen port configurable (currently fixed to 8787 unless wired through the gateway).

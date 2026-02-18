---
title: "RPC Adapterlar"
---

# RPC adapterlar

OpenClaw tashqi CLI’larni JSON-RPC orqali integratsiya qiladi. Hozirda ikki naqsh qo‘llaniladi.

## Naqsh A: HTTP demon (signal-cli)

- `signal-cli` JSON-RPC’ni HTTP orqali taqdim etuvchi demon sifatida ishlaydi.
- Hodisa oqimi SSE (`/api/v1/events`).
- Sog‘liqni tekshirish: `/api/v1/check`.
- `channels.signal.autoStart=true` bo‘lganda hayotiy sikl OpenClaw tomonidan boshqariladi.

Sozlash va endpointlar uchun [Signal](/channels/signal) ga qarang.

## Naqsh B: stdio farzand jarayon (legacy: imsg)

> **Eslatma:** Yangi iMessage sozlamalari uchun buning o‘rniga [BlueBubbles](/channels/bluebubbles) dan foydalaning.

- OpenClaw `imsg rpc` ni farzand jarayon sifatida ishga tushiradi (legacy iMessage integratsiyasi).
- JSON-RPC stdin/stdout orqali satrlar bo‘yicha uzatiladi (har bir satrda bitta JSON obyekt).
- TCP port yo‘q, demon talab qilinmaydi.

Ishlatiladigan asosiy metodlar:

- `watch.subscribe` → notifications (`method: "message"`)
- `watch.unsubscribe`
- `send`
- `chats.list` (sinov/diagnostika)

Legacy sozlash va manzillash (`chat_id` afzal) uchun [iMessage](/channels/imessage) ga qarang.

## Adapter bo‘yicha ko‘rsatmalar

- Gateway jarayonga egalik qiladi (ishga tushirish/to‘xtatish provayder hayotiy sikliga bog‘langan).
- RPC mijozlarini barqaror qiling: vaqt cheklovlari, chiqishda qayta ishga tushirish.
- Ko‘rsatish satrlaridan ko‘ra barqaror IDlarni (masalan, `chat_id`) afzal ko‘ring.



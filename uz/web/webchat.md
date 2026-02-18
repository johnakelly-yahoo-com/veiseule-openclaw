---
summary: "Chat UI uchun Loopback WebChat statik xosti va Gateway WS’dan foydalanish"
read_when:
  - WebChat kirishini sozlash yoki nosozliklarni bartaraf etishda
title: "WebChat"
---

# WebChat (Gateway WebSocket foydalanuvchi interfeysi)

Holati: macOS/iOS SwiftUI chat UI to‘g‘ridan-to‘g‘ri Gateway WebSocket’ga ulanadi.

## Bu nima

- Gateway uchun mahalliy (native) chat UI (o‘rnatilgan brauzersiz va lokal statik serversiz).
- Boshqa kanallar bilan bir xil sessiyalar va marshrutlash qoidalaridan foydalanadi.
- Deterministik marshrutlash: javoblar har doim WebChat’ga qaytadi.

## Tez boshlash

1. Gateway’ni ishga tushiring.
2. WebChat UI’ni (macOS/iOS ilovasi) yoki Control UI’dagi chat yorlig‘ini oching.
3. Gateway autentifikatsiyasi sozlanganligiga ishonch hosil qiling (hatto loopback’da ham sukut bo‘yicha talab qilinadi).

## Qanday ishlaydi (xulq-atvori)

- UI Gateway WebSocket’ga ulanadi va `chat.history`, `chat.send`, hamda `chat.inject` dan foydalanadi.
- `chat.inject` assistant izohini to‘g‘ridan-to‘g‘ri transkriptga qo‘shadi va uni UI’ga uzatadi (agent ishga tushirilmaydi).
- Tarix har doim gateway’dan olinadi (lokal fayllarni kuzatish yo‘q).
- Agar gateway’ga ulanib bo‘lmasa, WebChat faqat o‘qish rejimida ishlaydi.

## Masofaviy foydalanish

- Masofaviy rejim gateway WebSocket’ni SSH/Tailscale orqali tunnel qiladi.
- Alohida WebChat serverini ishga tushirish talab qilinmaydi.

## Konfiguratsiya ma’lumotnomasi (WebChat)

To‘liq konfiguratsiya: [Configuration](/gateway/configuration)

Kanal opsiyalari:

- Alohida `webchat.*` bo‘limi yo‘q. WebChat quyidagi gateway endpoint + auth sozlamalaridan foydalanadi.

Tegishli global opsiyalar:

- `gateway.port`, `gateway.bind`: WebSocket xost/port.
- `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket autentifikatsiyasi (token/parol).
- `gateway.auth.mode: "trusted-proxy"`: brauzer mijozlari uchun reverse-proxy autentifikatsiyasi (qarang [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: masofaviy gateway manzili.
- `session.*`: sessiya saqlash va asosiy kalit (main key) uchun sukut bo‘yicha sozlamalar.

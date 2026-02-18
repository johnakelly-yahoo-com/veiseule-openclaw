---
summary: "Bridge protokoli (eski tugunlar): TCP JSONL, juftlash, chegaralangan RPC"
read_when:
  - Tugun mijozlarini (iOS/Android/macOS tugun rejimi) yaratishda yoki nosozliklarni tuzatishda
  - Juftlash yoki bridge autentifikatsiya xatolarini tekshirishda
  - Gateway tomonidan ochilgan tugun sirtini audit qilishda
title: "Bridge protokoli"
---

# Bridge protokoli (eski tugun transporti)

Bridge protokoli — bu **eski** tugun transporti (TCP JSONL). Yangi tugun mijozlari
uning o‘rniga yagona Gateway WebSocket protokolidan foydalanishi kerak.

Agar siz operator yoki tugun mijozini yaratayotgan bo‘lsangiz,  
[Gateway protocol](/gateway/protocol) dan foydalaning.

**Eslatma:** Hozirgi OpenClaw versiyalari endi TCP bridge listener bilan yetkazilmaydi; ushbu hujjat tarixiy ma’lumot uchun saqlangan.  
Eski `bridge.*` konfiguratsiya kalitlari endi konfiguratsiya sxemasining bir qismi emas.

## Nega ikkisi ham mavjud

- **Xavfsizlik chegarasi**: bridge to‘liq gateway API sirtini emas, balki kichik ruxsat etilgan ro‘yxatni (allowlist) ochadi.
- **Juftlash + tugun identifikatsiyasi**: tugunni qabul qilish gateway tomonidan boshqariladi va har bir tugun uchun alohida token bilan bog‘langan.
- **Aniqlash UX**: tugunlar LAN orqali Bonjour yordamida gateway’larni topishi yoki tailnet orqali to‘g‘ridan-to‘g‘ri ulanashi mumkin.
- **Loopback WS**: to‘liq WS boshqaruv tekisligi SSH orqali tunnellamasdan turib lokal qoladi.

## Transport

- TCP, har bir qatorda bitta JSON obyekt (JSONL).
- Ixtiyoriy TLS (agar `bridge.tls.enabled` true bo‘lsa).
- Eski sukut bo‘yicha tinglovchi port `18790` edi (joriy versiyalar TCP bridge’ni ishga tushirmaydi).

TLS yoqilganda, discovery TXT yozuvlari `bridgeTls=1` hamda maxfiy bo‘lmagan ishora sifatida  
`bridgeTlsSha256` ni o‘z ichiga oladi. E’tibor bering, Bonjour/mDNS TXT yozuvlari autentifikatsiyalanmagan; mijozlar e’lon qilingan fingerprint’ni foydalanuvchining aniq roziligisiz yoki boshqa tashqi tasdiqsiz ishonchli pin sifatida qabul qilmasliklari kerak.

## Handshake + juftlash

1. Mijoz tugun metama’lumotlari va token (agar allaqachon juftlangan bo‘lsa) bilan `hello` yuboradi.
2. Agar juftlanmagan bo‘lsa, gateway `error` (`NOT_PAIRED`/`UNAUTHORIZED`) bilan javob beradi.
3. Mijoz `pair-request` yuboradi.
4. Gateway tasdiqlashni kutadi, so‘ng `pair-ok` va `hello-ok` yuboradi.

`hello-ok` `serverName` ni qaytaradi va `canvasHostUrl` ni ham o‘z ichiga olishi mumkin.

## Freymlar

Mijoz → Gateway:

- `req` / `res`: chegaralangan gateway RPC (chat, sessions, config, health, voicewake, skills.bins)
- `event`: tugun signallari (ovoz transkripti, agent so‘rovi, chat obunasi, exec hayot sikli)

Gateway → Mijoz:

- `invoke` / `invoke-res`: tugun buyruqlari (`canvas.*`, `camera.*`, `screen.record`,
  `location.get`, `sms.send`)
- `event`: obuna qilingan sessiyalar uchun chat yangilanishlari
- `ping` / `pong`: aloqani saqlab turish (keepalive)

Eski allowlist nazorati `src/gateway/server-bridge.ts` da joylashgan edi (olib tashlangan).

## Exec hayot sikli voqealari

Tugunlar system.run faoliyatini ko‘rsatish uchun `exec.finished` yoki `exec.denied` voqealarini yuborishi mumkin.  
Ular gateway’da system voqealariga moslanadi. (Eski tugunlar hali ham `exec.started` yuborishi mumkin.)

Yuklama (payload) maydonlari (aks belgilanmagan bo‘lsa, barchasi ixtiyoriy):

- `sessionKey` (majburiy): system voqeasini qabul qiladigan agent sessiyasi.
- `runId`: guruhlash uchun yagona exec identifikatori.
- `command`: xom yoki formatlangan buyruq satri.
- `exitCode`, `timedOut`, `success`, `output`: yakuniy tafsilotlar (faqat finished uchun).
- `reason`: rad etish sababi (faqat denied uchun).

## Tailnet’dan foydalanish

- Bridge’ni tailnet IP’ga bog‘lash: `bridge.bind: "tailnet"`  
  `~/.openclaw/openclaw.json` ichida.
- Mijozlar MagicDNS nomi yoki tailnet IP orqali ulanadi.
- Bonjour tarmoqlar orasidan o‘tmaydi; zarur bo‘lsa qo‘lda host/port yoki keng hududli DNS‑SD’dan foydalaning.

## Versiyalash

Bridge hozirda **implicit v1** (min/max kelishuvisiz). Orqaga moslik (backward‑compat) kutiladi; har qanday buzuvchi o‘zgarishdan oldin bridge protokoli versiya maydonini qo‘shing.
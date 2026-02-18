---
title: "Holat tekshiruvlari"
---

# Holat tekshiruvlari (CLI)

Taxmin qilmasdan kanal ulanishini tekshirish uchun qisqa qo‘llanma.

## Tezkor tekshiruvlar

- `openclaw status` — lokal xulosa: gateway mavjudligi/rejimi, yangilanish eslatmasi, ulangan kanal autentifikatsiya yoshi, sessiyalar va so‘nggi faollik.
- `openclaw status --all` — to‘liq lokal diagnostika (faqat o‘qish rejimida, rangli, nosozliklarni tahlil qilish uchun ulashishga xavfsiz).
- `openclaw status --deep` — ishlayotgan Gateway’ni ham tekshiradi (qo‘llab-quvvatlansa, har bir kanal bo‘yicha tekshiruvlar).
- `openclaw health --json` — ishlayotgan Gateway’dan to‘liq holat hisobotini so‘raydi (faqat WS; to‘g‘ridan-to‘g‘ri Baileys socket ishlatilmaydi).
- WhatsApp/WebChat’da agentni chaqirmasdan holat javobini olish uchun `/status` ni alohida xabar sifatida yuboring.
- Loglar: `/tmp/openclaw/openclaw-*.log` ni kuzating va `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound` bo‘yicha filtrlang.

## Chuqur diagnostika

- Diskdagi creds: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (mtime yaqinda yangilangan bo‘lishi kerak).
- Sessiya ombori: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (yo‘l konfiguratsiyada o‘zgartirilishi mumkin). Soni va so‘nggi qabul qiluvchilar `status` orqali ko‘rsatiladi.
- Qayta ulash jarayoni: loglarda 409–515 status kodlari yoki `loggedOut` paydo bo‘lsa, `openclaw channels logout && openclaw channels login --verbose` ni ishga tushiring. (Eslatma: QR orqali kirish jarayoni 515 statusidan so‘ng bir marta avtomatik qayta boshlanadi.)

## Muammo yuzaga kelganda

- `logged out` yoki 409–515 status → `openclaw channels logout` so‘ng `openclaw channels login` bilan qayta ulang.
- Gateway mavjud emas → uni ishga tushiring: `openclaw gateway --port 18789` (agar port band bo‘lsa, `--force` dan foydalaning).
- Kiruvchi xabarlar yo‘q → ulangan telefon onlayn ekanini va jo‘natuvchi ruxsat etilganini tekshiring (`channels.whatsapp.allowFrom`); guruh chatlari uchun allowlist va mention qoidalari mos ekanini tasdiqlang (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

## Maxsus "health" buyrug‘i

`openclaw health --json` ishlayotgan Gateway’dan uning holat hisobotini so‘raydi (CLI’dan to‘g‘ridan-to‘g‘ri kanal socketlari ishlatilmaydi). Mavjud bo‘lsa, ulangan creds/auth yoshi, har bir kanal bo‘yicha tekshiruv xulosalari, sessiya ombori xulosasi va tekshiruv davomiyligini ko‘rsatadi. Agar Gateway mavjud bo‘lmasa yoki tekshiruv muvaffaqiyatsiz/taym-aut bo‘lsa, nol bo‘lmagan kod bilan yakunlanadi. 10 soniyalik standart vaqtni o‘zgartirish uchun `--timeout <ms>` dan foydalaning.


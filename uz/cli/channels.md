---
summary: "`openclaw channels` uchun CLI ma’lumotnomasi (akkauntlar, holat, kirish/chiqish, loglar)"
read_when:
  - Siz kanal akkauntlarini qo‘shmoqchi/olib tashlamoqchisiz (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage)
  - Siz kanal holatini tekshirmoqchisiz yoki kanal loglarini real vaqt rejimida ko‘rmoqchisiz
title: "channels"
---

# `openclaw channels`

Gateway’da chat kanallari akkauntlari va ularning ish holatini boshqaring.

Tegishli hujjatlar:

- Kanal qo‘llanmalari: [Channels](/channels/index)
- Gateway sozlamalari: [Configuration](/gateway/configuration)

## Keng tarqalgan buyruqlar

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## Akkaunt qo‘shish / olib tashlash

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Maslahat: `openclaw channels add --help` har bir kanal uchun mavjud flaglarni (token, app token, signal-cli yo‘llari va boshqalar) ko‘rsatadi.

## Kirish / chiqish (interaktiv)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## Muammolarni bartaraf etish

- Keng qamrovli tekshiruv uchun `openclaw status --deep` ni ishga tushiring.
- Yo‘naltirilgan tuzatishlar uchun `openclaw doctor` dan foydalaning.
- `openclaw channels list` da `Claude: HTTP 403 ... user:profile` chiqsa → foydalanish holati uchun `user:profile` scope talab qilinadi. `--no-usage` dan foydalaning yoki claude.ai sessiya kalitini taqdim eting (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), yoki Claude Code CLI orqali qayta autentifikatsiya qiling.

## Imkoniyatlarni tekshirish (Capabilities probe)

Provayder imkoniyatlari bo‘yicha ma’lumotlarni (mavjud joylarda intents/scopes) hamda statik funksional qo‘llab-quvvatlashni oling:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Eslatmalar:

- `--channel` majburiy emas; barcha kanallarni (shu jumladan kengaytmalarni) ko‘rish uchun uni ko‘rsatmasangiz ham bo‘ladi.
- `--target` `channel:<id>` yoki oddiy raqamli kanal id qabul qiladi va faqat Discord uchun amal qiladi.
- Tekshiruvlar provayderga xos: Discord intents + ixtiyoriy kanal ruxsatlari; Slack bot + user scopes; Telegram bot flaglari + webhook; Signal daemon versiyasi; MS Teams app token + Graph rollari/scopes (ma’lum joylarda izoh bilan). Tekshiruv mavjud bo‘lmagan kanallar `Probe: unavailable` deb ko‘rsatadi.

## Nomlarni ID ga o‘girish

Provayder katalogidan foydalanib kanal/foydalanuvchi nomlarini ID ga o‘giring:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Eslatmalar:

- Maqsad turini majburan belgilash uchun `--kind user|group|auto` dan foydalaning.
- Bir xil nomli bir nechta yozuv bo‘lsa, tizim faol (active) moslikni ustun qo‘yadi.
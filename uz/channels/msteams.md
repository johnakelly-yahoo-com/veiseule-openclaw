---
title: "35. Microsoft Teams"
---

# 36. Microsoft Teams (plagin)

> "Bu yerga kirgan barcha umidlaringni tashla."

Yangilandi: 2026-01-21

Holat: matn + DM biriktirmalari qo‘llab-quvvatlanadi; kanal/guruhga fayl yuborish uchun `sharePointSiteId` + Graph ruxsatlari talab etiladi (qarang: [Sending files in group chats](#sending-files-in-group-chats)). So‘rovnomalar Adaptive Cards orqali yuboriladi.

## 41. Plagin talab qilinadi

Microsoft Teams plagin sifatida yetkaziladi va asosiy o‘rnatmaga kiritilmagan.

**Breaking change (2026.1.15):** MS Teams moved out of core. Agar undan foydalansangiz, plaginni o‘rnatishingiz kerak.

Tushuntirish: bu yadro o‘rnatmalarini yengillashtiradi va MS Teams bog‘liqliklarini mustaqil yangilashga imkon beradi.

CLI orqali o‘rnatish (npm registri):

```bash
openclaw plugins install @openclaw/msteams
```

Mahalliy checkout (git repozitoriydan ishga tushirilganda):

```bash
openclaw plugins install ./extensions/msteams
```

Agar sozlash/onboarding vaqtida Teams’ni tanlasangiz va git checkout aniqlansa, OpenClaw mahalliy o‘rnatish yo‘lini avtomatik taklif qiladi.

Details: [Plugins](/tools/plugin)

## Quick setup (beginner)

1. Microsoft Teams plaginini o‘rnating.
2. **Azure Bot** yarating (App ID + client secret + tenant ID).
3. OpenClaw’ni ushbu hisob ma’lumotlari bilan sozlang.
4. `/api/messages` (standart bo‘yicha 3978-port) ni ommaviy URL yoki tunnel orqali oching.
5. Teams ilova paketini o‘rnating va gateway’ni ishga tushiring.

Minimal sozlama:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

Eslatma: guruh chatlari standart bo‘yicha bloklangan (`channels.msteams.groupPolicy: "allowlist"`). Guruh javoblariga ruxsat berish uchun `channels.msteams.groupAllowFrom` ni sozlang (yoki `groupPolicy: "open"` dan foydalaning — istalgan a’zo, mention orqali).

## 12. Maqsadlar

- Teams DM’lari, guruh chatlari yoki kanallar orqali OpenClaw bilan muloqot qilish.
- Marshrutlashni deterministik saqlash: javoblar har doim kelgan kanaliga qaytadi.
- Standart bo‘yicha xavfsiz kanal xatti-harakati (aks holda sozlanmasa, @mention talab qilinadi).

## 16. Konfiguratsiya yozuvlari

Standart bo‘yicha Microsoft Teams `/config set|unset` orqali ishga tushirilgan konfiguratsiya yangilanishlarini yozishga ruxsat etiladi (`commands.config: true` talab etiladi).

O‘chirish:

```json5
{
  channels: { msteams: { configWrites: false } },
}
```

## 20. Kirish nazorati (DM’lar + guruhlar)

**DM kirish**

- Standart: `channels.msteams.dmPolicy = "pairing"`. Noma’lum jo‘natuvchilar tasdiqlanmaguncha e’tiborsiz qoldiriladi.
- `channels.msteams.allowFrom` AAD obyekt ID’lari, UPN’lar yoki ko‘rinadigan nomlarni qabul qiladi. Usta (wizard) ruxsatlar mavjud bo‘lsa, Microsoft Graph orqali nomlarni ID’larga moslaydi.

**Group access**

- Standart: `channels.msteams.groupPolicy = "allowlist"` (`groupAllowFrom` qo‘shilmaguncha bloklangan). Sozlanmagan holatda standartni bekor qilish uchun `channels.defaults.groupPolicy` dan foydalaning.
- `channels.msteams.groupAllowFrom` guruh chatlari/kanallarida qaysi jo‘natuvchilar ishga tushira olishini boshqaradi (`channels.msteams.allowFrom` ga qaytadi).
- `groupPolicy: "open"` ni o‘rnating — istalgan a’zo (standart bo‘yicha baribir mention talab qilinadi).
- **Hech qanday kanalga** ruxsat bermaslik uchun `channels.msteams.groupPolicy: "disabled"` ni o‘rnating.

Misol:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Teams + kanal allowlist**

- `channels.msteams.teams` ostida jamoalar va kanallarni sanab, guruh/kanal javoblarini cheklang.
- Kalitlar jamoa ID’lari yoki nomlari bo‘lishi mumkin; kanal kalitlari esa suhbat ID’lari yoki nomlari bo‘lishi mumkin.
- `groupPolicy="allowlist"` va jamoalar allowlist’i mavjud bo‘lsa, faqat ko‘rsatilgan jamoalar/kanallar qabul qilinadi (mention talab qilinadi).
- Sozlash ustasi `Team/Channel` yozuvlarini qabul qiladi va ularni siz uchun saqlaydi.
- Ishga tushishda OpenClaw jamoa/kanal va foydalanuvchi allowlist nomlarini ID’larga moslaydi (Graph ruxsatlari mavjud bo‘lsa) va moslikni jurnalga yozadi; moslanmagan yozuvlar kiritilgandek saqlanadi.

Misol:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```
---
title: IRC
description: OpenClaw’ni IRC kanallari va shaxsiy xabarlarga ulang.
---

OpenClaw’ni klassik kanallarda (`#room`) va shaxsiy xabarlarda ishlatmoqchi bo‘lsangiz, IRC’dan foydalaning.
IRC kengaytma plagini sifatida yetkaziladi, ammo u asosiy konfiguratsiyada `channels.irc` ostida sozlanadi.

## Tezkor boshlash

1. IRC konfiguratsiyasini `~/.openclaw/openclaw.json` ichida yoqing.
2. Hech bo‘lmaganda quyidagilarni sozlang:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Gateway’ni ishga tushiring/qayta ishga tushiring:

```bash
openclaw gateway run
```

## Xavfsizlik bo‘yicha standart sozlamalar

- `channels.irc.dmPolicy` sukut bo‘yicha `"pairing"`.
- `channels.irc.groupPolicy` sukut bo‘yicha `"allowlist"`.
- Agar `groupPolicy="allowlist"` bo‘lsa, ruxsat etilgan kanallarni belgilash uchun `channels.irc.groups` ni sozlang.
- Agar ataylab ochiq matnli uzatishni qabul qilmasangiz, TLS’dan foydalaning (`channels.irc.tls=true`).

## Kirish nazorati

IRC kanallari uchun ikkita alohida “to‘siq” mavjud:

1. **Kanalga kirish** (`groupPolicy` + `groups`): bot umuman kanaldan xabarlarni qabul qiladimi-yo‘qmi.
2. **Yuboruvchi kirishi** (`groupAllowFrom` / har bir kanal uchun `groups["#channel"].allowFrom`): ushbu kanal ichida botni kim ishga tushira oladi.

Konfiguratsiya kalitlari:

- DM allowlist (DM yuboruvchi kirishi): `channels.irc.allowFrom`
- Guruh yuboruvchi allowlist (kanal yuboruvchi kirishi): `channels.irc.groupAllowFrom`
- Har bir kanal uchun boshqaruv (kanal + yuboruvchi + mention qoidalari): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` sozlanmagan kanallarga ruxsat beradi (**baribir sukut bo‘yicha mention orqali cheklanadi**)

Allowlist yozuvlari nick yoki `nick!user@host` shaklida bo‘lishi mumkin.

### Keng tarqalgan xato: `allowFrom` kanallar uchun emas, DM uchun

Agar loglarda quyidagiga o‘xshash yozuvni ko‘rsangiz:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…bu **guruh/kanal** xabarlari uchun yuboruvchiga ruxsat berilmaganini anglatadi. Buni quyidagilardan biri orqali tuzating:

- `channels.irc.groupAllowFrom` ni sozlash (barcha kanallar uchun global), yoki
- har bir kanal uchun yuboruvchi allowlist’ni sozlash: `channels.irc.groups["#channel"].allowFrom`

Misol (`#tuirc-dev` kanalida istalgan foydalanuvchi bot bilan gaplasha olishi uchun):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Javobni ishga tushirish (mentionlar)

Hatto kanal ruxsat etilgan bo‘lsa ham (`groupPolicy` + `groups` orqali) va yuboruvchi ruxsat etilgan bo‘lsa ham, OpenClaw guruh kontekstida sukut bo‘yicha **mention-gating**ni qo‘llaydi.

Bu shuni anglatadiki, botga mos keladigan mention namunasi xabarda bo‘lmasa, `drop channel …` kabi loglarni ko‘rishingiz mumkin. (missing-mention)\`

Bot IRC kanalida **mention talab qilmasdan** javob berishi uchun, o‘sha kanal uchun mention-gating’ni o‘chirib qo‘ying:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Yoki **barcha** IRC kanallariga ruxsat berish (har bir kanal uchun alohida allowlist’siz) va eslatmasiz ham javob berish uchun:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Xavfsizlik eslatmasi (ommaviy kanallar uchun tavsiya etiladi)

Agar ommaviy kanalda `allowFrom: ["*"]` ga ruxsat bersangiz, istalgan kishi botga so‘rov yuborishi mumkin.
Xavfni kamaytirish uchun ushbu kanal uchun vositalarni cheklang.

### Kanaldagi hamma uchun bir xil vositalar

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Har bir jo‘natuvchi uchun turli vositalar (egasi ko‘proq huquqqa ega)

`toolsBySender` dan foydalanib, `"*"` uchun qat’iyroq siyosatni va o‘z nick’ingiz uchun yumshoqroq siyosatni qo‘llang:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Eslatmalar:

- `toolsBySender` kalitlari nick (masalan, `"eigen"`) yoki yanada kuchli identifikatsiya uchun to‘liq hostmask (`"eigen!~eigen@174.127.248.171"`) bo‘lishi mumkin.
- Birinchi mos kelgan jo‘natuvchi siyosati qo‘llanadi; `"*"` — bu umumiy (wildcard) zaxira varianti.

Guruhga kirish va mention-gating (va ularning o‘zaro ishlashi) haqida batafsil: [/channels/groups](/channels/groups).

## NickServ

Ulanishdan so‘ng NickServ orqali identifikatsiya qilish uchun:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Ulanish vaqtida ixtiyoriy bir martalik ro‘yxatdan o‘tish:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Nick ro‘yxatdan o‘tgandan so‘ng takroriy REGISTER urinishlarining oldini olish uchun `register` ni o‘chirib qo‘ying.

## Muhit o‘zgaruvchilari

Standart akkaunt quyidagilarni qo‘llab-quvvatlaydi:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (vergul bilan ajratilgan)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Nosozliklarni bartaraf etish

- Agar bot ulanib, lekin kanallarda hech qachon javob bermasa, `channels.irc.groups` ni **va** mention-gating xabarlarni bloklayotganini (`missing-mention`) tekshiring. Agar u pinglarsiz javob berishini istasangiz, kanal uchun `requireMention:false` ni o‘rnating.
- Agar kirish amalga oshmasa, nick mavjudligini va server parolini tekshiring.
- Agar maxsus tarmoqda TLS ishlamasa, host/port va sertifikat sozlamalarini tekshiring.

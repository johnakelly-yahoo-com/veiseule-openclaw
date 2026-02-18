---
title: "Telegram"
---

# Telegram (Bot API)

Status: grammY orqali bot DM + guruhlar uchun production-ready. Long polling sukut bo‘yicha; webhook ixtiyoriy.

<CardGroup cols={3}>
  <Card title="Juftlash (Pairing)" icon="link" href="/channels/pairing">
    Telegram uchun sukut bo‘yicha DM siyosati — juftlash (pairing).
  </Card>
  <Card title="Kanal nosozliklari" icon="wrench" href="/channels/troubleshooting">
    Kanallar bo‘yicha diagnostika va tuzatish qo‘llanmalari.
  </Card>
  <Card title="Gateway konfiguratsiyasi" icon="settings" href="/gateway/configuration">
    Kanal konfiguratsiyasi naqshlari va to‘liq misollar.
  </Card>
</CardGroup>

## Tezkor sozlash

<Steps>
  <Step title="BotFather’da bot tokenini yarating">
    Telegram’ni oching va **@BotFather** bilan chat qiling (handle aynan `@BotFather` ekanini tasdiqlang).

    `/newbot` buyrug‘ini bajaring, ko‘rsatmalarga amal qiling va tokenni saqlang.

  </Step>

  <Step title="Token va DM siyosatini sozlang">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    Muhit varianti: `TELEGRAM_BOT_TOKEN=...` (faqat sukut bo‘yicha akkaunt uchun).

  </Step>

  <Step title="Gateway’ni ishga tushiring va birinchi DM’ni tasdiqlang">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    Juftlash kodlari 1 soatdan keyin muddati tugaydi.

  </Step>

  <Step title="Botni guruhga qo‘shing">
    Botni guruhingizga qo‘shing, so‘ng `channels.telegram.groups` va `groupPolicy` ni kirish modelingizga moslab sozlang.
  </Step>
</Steps>

<Note>
Token aniqlash tartibi akkauntga bog‘liq. Amalda konfiguratsiya qiymatlari muhitdan ustun turadi va `TELEGRAM_BOT_TOKEN` faqat sukut bo‘yicha akkauntga qo‘llanadi.
</Note>

## Telegram tomoni sozlamalari

<AccordionGroup>
  <Accordion title="Privacy mode va guruh ko‘rinishi">
    Telegram botlari sukut bo‘yicha **Privacy Mode** rejimida bo‘ladi, bu esa guruhdagi qaysi xabarlarni qabul qilishini cheklaydi.

    Agar bot barcha guruh xabarlarini ko‘rishi kerak bo‘lsa, quyidagilardan birini bajaring:

    - `/setprivacy` orqali privacy mode’ni o‘chiring, yoki
    - botni guruhga admin sifatida qo‘shing.

    Privacy mode’ni o‘zgartirgandan so‘ng, Telegram o‘zgarishni qo‘llashi uchun botni har bir guruhdan olib tashlab, qayta qo‘shing.

  </Accordion>

  <Accordion title="Guruh ruxsatlari">
    Admin maqomi Telegram guruh sozlamalarida boshqariladi.

    Admin botlar barcha guruh xabarlarini oladi, bu doimiy faol guruh xatti-harakati uchun foydali.

  </Accordion>

  <Accordion title="Foydali BotFather sozlamalari">

    - `/setjoingroups` — guruhlarga qo‘shishni ruxsat berish/taqiqlash
    - `/setprivacy` — guruh ko‘rinish xatti-harakatini boshqarish

  </Accordion>
</AccordionGroup>

## Kirish nazorati va faollashtirish

<Tabs>
  <Tab title="DM siyosati">
    `channels.telegram.dmPolicy` to‘g‘ridan-to‘g‘ri xabar (DM) kirishini boshqaradi:

    - `pairing` (standart)
    - `allowlist`
    - `open` (`allowFrom` ichida `"*"` bo‘lishi kerak)
    - `disabled`

    `channels.telegram.allowFrom` raqamli Telegram foydalanuvchi ID’larini qabul qiladi. `telegram:` / `tg:` prefikslari qabul qilinadi va normallashtiriladi.
    Onboarding wizard `@username` kiritishni qabul qiladi va uni raqamli ID’ga o‘giradi.
    Agar yangilangan bo‘lsangiz va konfiguratsiyada `@username` ko‘rinishidagi allowlist yozuvlari bo‘lsa, ularni ID’ga o‘girish uchun `openclaw doctor --fix` ni ishga tushiring (best-effort; Telegram bot tokeni talab qilinadi).

    ### Telegram user ID’ingizni topish

    Xavfsiz usul (uchinchi tomon botsiz):

    1. Botga DM yuboring.
    2. `openclaw logs --follow` ni ishga tushiring.
    3. `from.id` ni o‘qing.

    Rasmiy Bot API usuli:

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    Uchinchi tomon (kamroq maxfiy): `@userinfobot` yoki `@getidsbot`.

  </Tab>

  <Tab title="Guruh siyosati va allowlist’lar">
    Ikkita mustaqil boshqaruv mavjud:

    1. **Qaysi guruhlarga ruxsat beriladi** (`channels.telegram.groups`)
       - `groups` yo‘q: barcha guruhlarga ruxsat
       - `groups` sozlangan: allowlist sifatida ishlaydi (aniq ID’lar yoki `"*"`)

    2. **Guruhlarda qaysi yuboruvchilarga ruxsat beriladi** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (standart)
       - `disabled`

    `groupAllowFrom` guruh yuboruvchilarini filtrlash uchun ishlatiladi. Agar o‘rnatilmagan bo‘lsa, Telegram `allowFrom` ga qaytadi.
    `groupAllowFrom` yozuvlari raqamli Telegram user ID bo‘lishi kerak.

    Misol: ma’lum bir guruhda istalgan a’zoga ruxsat berish:

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  </Tab>

  <Tab title="Mention xatti-harakati">
    Guruh javoblari sukut bo‘yicha mention talab qiladi.

    Mention quyidagilardan kelishi mumkin:

    - mahalliy `@botusername` mention, yoki
    - quyidagi joylardagi mention naqshlari:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`

    Sessiya darajasidagi buyruqlar:

    - `/activation always`
    - `/activation mention`

    Bular faqat sessiya holatini yangilaydi. Doimiylik uchun konfiguratsiyadan foydalaning.

    Doimiy konfiguratsiya misoli:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    Guruh chat ID’sini olish:

    - guruh xabarini `@userinfobot` / `@getidsbot` ga yo‘naltiring
    - yoki `openclaw logs --follow` dan `chat.id` ni o‘qing
    - yoki Bot API `getUpdates` ni tekshiring

  </Tab>
</Tabs>

## Ishlash xatti-harakati

- Telegram gateway jarayoniga tegishli.
- Marshrutlash deterministik: Telegram’dan kirgan xabarlar Telegram’ga javob qaytaradi (model kanallarni tanlamaydi).
- Kiruvchi xabarlar umumiy kanal konvertiga reply metadata va media placeholder’lar bilan normallashtiriladi.
- Guruh sessiyalari guruh ID bo‘yicha ajratiladi. Forum mavzulari izolyatsiya uchun `:topic:<threadId>` qo‘shimchasini qo‘shadi.
- DM xabarlari `message_thread_id` ni o‘z ichiga olishi mumkin; OpenClaw thread-aware sessiya kalitlari bilan marshrutlaydi va javoblarda thread ID’ni saqlaydi.
- Long polling grammY runner’dan har-chat/har-thread ketma-ketlik bilan foydalanadi. Umumiy parallelizm `agents.defaults.maxConcurrent` orqali boshqariladi.
- Telegram Bot API o‘qilganlik kvitansiyalarini qo‘llab-quvvatlamaydi (`sendReadReceipts` qo‘llanilmaydi).

## Tegishli

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
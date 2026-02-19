---
summary: "11. Telegram (Bot API)"
read_when:
  - "12. Holat: grammY orqali bot DMlari + guruhlar uchun ishlab chiqarishga tayyor."
title: "13. Sukut bo‘yicha long-polling; webhook ixtiyoriy."
---

# 14. Tezkor sozlash (boshlovchilar uchun)

15. **@BotFather** bilan bot yarating ([to‘g‘ridan-to‘g‘ri havola](https://t.me/BotFather)). Long polling — standart rejim; webhook rejimi ixtiyoriy.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Telegram uchun standart DM siyosati — pairing.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallararo diagnostika va tiklash bo‘yicha qo‘llanmalar.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Kanal konfiguratsiyasi uchun to‘liq namunalar va misollar.
  
</Card>
</CardGroup>

## Tezkor sozlash

<Steps>
  <Step title="Create the bot token in BotFather">
    Telegram’ni oching va **@BotFather** bilan yozishing (username aynan `@BotFather` ekanini tasdiqlang).
  

    ```
    27. Deterministik marshrutlash: javoblar Telegram’ga qaytadi; model kanallarni tanlamaydi.
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
43. Ko‘p akkauntli qo‘llab-quvvatlash: akkauntlar bo‘yicha tokenlar va ixtiyoriy `name` bilan `channels.telegram.accounts` dan foydalaning.
```

    ```
    Env fallback: `TELEGRAM_BOT_TOKEN=...` (faqat default account uchun).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Pairing kodlari 1 soatdan keyin muddati tugaydi.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Botni guruhingizga qo‘shing, so‘ng `channels.telegram.groups` va `groupPolicy` ni kirish modelingizga moslab sozlang.
  
</Step>
</Steps>

<Note>
Token aniqlash tartibi account-aware. Amalda, config qiymatlari env fallback’dan ustun turadi va `TELEGRAM_BOT_TOKEN` faqat default account’ga qo‘llanadi.
</Note>

## Token + privacy + permissions (Telegram side)

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">
    Telegram botlari odatda **Privacy Mode** rejimida bo‘ladi, bu esa ular qabul qiladigan guruh xabarlarini cheklaydi.
  

    ```
    Agar bot barcha guruh xabarlarini ko‘rishi kerak bo‘lsa, quyidagilardan birini bajaring:
    
    - `/setprivacy` orqali privacy mode’ni o‘chiring, yoki
    - botni guruh administratori qiling.
    
    Privacy mode’ni o‘zgartirganda, Telegram o‘zgarishni qo‘llashi uchun har bir guruhdan botni olib tashlab, qayta qo‘shing.
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    Administrator maqomi Telegram guruh sozlamalarida boshqariladi.
  

    ```
    Admin botlar barcha guruh xabarlarini qabul qiladi, bu doimiy faol guruh xatti-harakatlari uchun foydali.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - Guruhlarga qo‘shishga ruxsat berish/taqiqlash uchun `/setjoingroups`
    - Guruh ko‘rinish xatti-harakati uchun `/setprivacy`
    ```

  
</Accordion>
</AccordionGroup>

## Kirish nazorati va faollashtirish

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` to‘g‘ridan-to‘g‘ri xabarlarga kirishni boshqaradi:
  

    ```
    - `pairing` (standart)
    - `allowlist`
    - `open` (`allowFrom` ichida `"*"` bo‘lishi shart)
    - `disabled`
    
    `channels.telegram.allowFrom` raqamli Telegram user ID’larni qabul qiladi. `telegram:` / `tg:` prefikslari qabul qilinadi va normallashtiriladi.
    Onboarding wizard `@username` kiritishni qabul qiladi va uni raqamli ID’ga aylantiradi.
    Agar yangilagandan so‘ng config’da `@username` allowlist yozuvlari bo‘lsa, ularni aniqlashtirish uchun `openclaw doctor --fix` ni ishga tushiring (imkon qadar; Telegram bot token talab qilinadi).
    
    ### Telegram user ID’ingizni topish
    
    Xavfsizroq usul (uchinchi tomon botsiz):
    
    1. Botingizga DM yuboring.
    2. `openclaw logs --follow` ni ishga tushiring.
    3. `from.id` ni o‘qing.
    
    Rasmiy Bot API usuli:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Uchinchi tomon usuli (kamroq maxfiy): `@userinfobot` yoki `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    Ikki mustaqil nazorat mavjud:
  

    ```
    1. **Qaysi guruhlarga ruxsat beriladi** (`channels.telegram.groups`)
       - `groups` config yo‘q: barcha guruhlarga ruxsat
       - `groups` sozlangan: allowlist sifatida ishlaydi (aniq ID’lar yoki `"*"`)
    
    2. **Guruhlarda qaysi yuboruvchilarga ruxsat beriladi** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (standart)
       - `disabled`
    
    `groupAllowFrom` guruh yuboruvchilarini filtrlash uchun ishlatiladi. Agar sozlanmagan bo‘lsa, Telegram `allowFrom` ga qaytadi.
    `groupAllowFrom` yozuvlari raqamli Telegram user ID bo‘lishi kerak.
    
    Misol: bitta aniq guruhda istalgan a’zoga ruxsat berish:
    ```

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

  <Tab title="Mention behavior">
    Guruh javoblari odatda mention talab qiladi.
  

    ```
    Mention quyidagilardan biri orqali bo‘lishi mumkin:
    
    - native `@botusername` mention, yoki
    - quyidagilardagi mention patterns:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Sessiya darajasidagi buyruq o‘zgartirishlari:
    
    - `/activation always`
    - `/activation mention`
    
    Bular faqat sessiya holatini yangilaydi. Doimiy saqlash uchun config’dan foydalaning.
    
    Doimiy config misoli:
    ```

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

    ```
    Guruh chat ID’sini olish:
    
    - guruh xabarini `@userinfobot` / `@getidsbot` ga forward qiling
    - yoki `openclaw logs --follow` dan `chat.id` ni o‘qing
    - yoki Bot API `getUpdates` ni tekshiring
    ```

  
</Tab>
</Tabs>

## Ish vaqtidagi xatti-harakat

- Telegram gateway jarayoniga tegishli.
- Yo‘naltirish deterministik: Telegram’dan kirgan javoblar yana Telegram’ga qaytadi (model kanallarni tanlamaydi).
- Kirish xabarlari umumiy kanal envelope’iga javob metadata’si va media placeholder’lari bilan normallashtiriladi.
- Guruh seanslari guruh ID bo‘yicha izolyatsiya qilinadi. Forum mavzulari mavzularni alohida saqlash uchun `:topic:<threadId>` qo‘shimchasini qo‘shadi.
- DM xabarlari `message_thread_id` ni o‘z ichiga olishi mumkin; OpenClaw ularni thread-aware seans kalitlari bilan yo‘naltiradi va javoblar uchun thread ID ni saqlab qoladi.
- Long polling grammY runner yordamida har bir chat/har bir thread bo‘yicha ketma-ketlikni ta’minlaydi. Umumiy runner sink concurrency `agents.defaults.maxConcurrent` orqali boshqariladi.
- Telegram Bot API o‘qilganlik haqidagi bildirishnomalarni qo‘llab-quvvatlamaydi (`sendReadReceipts` amal qilmaydi).

## Funksiya ma’lumotnomasi

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">
    OpenClaw vaqtinchalik Telegram xabarini yuborib va matn kelishi bilan uni tahrirlab, qisman javoblarni oqim tarzida uzata oladi.


    ```
    Talab:
    
    - `channels.telegram.streamMode` "off" emas (standart: "partial")
    
    Rejimlar:
    
    - `off`: jonli oldindan ko‘rish yo‘q
    - `partial`: qisman matndan tez-tez oldindan ko‘rish yangilanishlari
    - `block`: `channels.telegram.draftChunk` yordamida bo‘laklab oldindan ko‘rish yangilanishlari
    
    `streamMode: "block"` uchun `draftChunk` standart qiymatlari:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` qiymati `channels.telegram.textChunkLimit` bilan cheklanadi.
    
    Bu to‘g‘ridan-to‘g‘ri chatlarda va guruh/mavzularda ishlaydi.
    
    Faqat matndan iborat javoblar uchun OpenClaw bir xil oldindan ko‘rish xabarini saqlab qoladi va oxirida shu xabarni joyida yakuniy tahrir qiladi (ikkinchi xabar yuborilmaydi).
    
    Murakkab javoblar (masalan, media payloadlar) uchun OpenClaw odatiy yakuniy yuborishga o‘tadi va keyin oldindan ko‘rish xabarini tozalaydi.
    
    `streamMode` block streaming’dan alohida. Agar Telegram uchun block streaming aniq yoqilgan bo‘lsa, OpenClaw ikki martalik oqimni oldini olish uchun preview stream’ni o‘tkazib yuboradi.
    
    Faqat Telegram uchun reasoning stream:
    
    - `/reasoning stream` generatsiya vaqtida reasoning’ni jonli preview’ga yuboradi
    - yakuniy javob reasoning matnisiz yuboriladi
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">
    Chiquvchi matn Telegram `parse_mode: "HTML"` dan foydalanadi.


    ```
    - Markdown uslubidagi matn Telegram uchun xavfsiz HTML ga aylantiriladi.
    - Modeldan kelgan xom HTML Telegram parse xatolarini kamaytirish uchun escapelanadi.
    - Agar Telegram parse qilingan HTML ni rad etsa, OpenClaw oddiy matn sifatida qayta urinadi.
    
    Havola preview’lari standart bo‘yicha yoqilgan va `channels.telegram.linkPreview: false` bilan o‘chirib qo‘yilishi mumkin.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Telegram buyruqlar menyusini ro‘yxatdan o‘tkazish ishga tushishda `setMyCommands` orqali amalga oshiriladi.


    ```
    Standart native buyruqlar:
    
    - `commands.native: "auto"` Telegram uchun native buyruqlarni yoqadi
    
    Maxsus buyruq menyusi yozuvlarini qo‘shish:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Qoidalar:
    
    - nomlar normalizatsiya qilinadi (boshidagi `/` olib tashlanadi, kichik harfga o‘tkaziladi)
    - ruxsat etilgan andoza: `a-z`, `0-9`, `_`, uzunligi `1..32`
    - maxsus buyruqlar native buyruqlarni bekor qila olmaydi
    - ziddiyatlar/dublikatlar o‘tkazib yuboriladi va log qilinadi
    
    Eslatma:
    
    - maxsus buyruqlar faqat menyu yozuvlari; ular avtomatik ravishda xulq-atvorni amalga oshirmaydi
    - plugin/skill buyruqlari Telegram menyusida ko‘rsatilmagan bo‘lsa ham, yozib yuborilganda ishlashi mumkin
    
    Agar native buyruqlar o‘chirilgan bo‘lsa, built-in buyruqlar olib tashlanadi. Mos ravishda sozlangan bo‘lsa, maxsus/plugin buyruqlari baribir ro‘yxatdan o‘tishi mumkin.
    
    Keng tarqalgan sozlash xatosi:
    
    - `setMyCommands failed` odatda `api.telegram.org` ga chiqish DNS/HTTPS bloklanganini anglatadi.
    
    ### Qurilmani juftlash buyruqlari (`device-pair` plugin)
    
    `device-pair` plugin o‘rnatilganda:
    
    1. `/pair` sozlash kodini yaratadi
    2. kodni iOS ilovasiga joylashtiring
    3. `/pair approve` oxirgi kutilayotgan so‘rovni tasdiqlaydi
    
    Batafsil: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    Inline klaviatura doirasini sozlash:


```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Har bir akkaunt uchun alohida sozlama:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    Doiralar:
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (standart)
    
    Eski `capabilities: ["inlineButtons"]` `inlineButtons: "all"` ga moslanadi.
    
    Xabar harakati misoli:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Callback bosishlari agentga matn sifatida uzatiladi:
    `callback_data: <value>`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram tool harakatlari quyidagilarni o‘z ichiga oladi:


    ```
    - `sendMessage` (`to`, `content`, ixtiyoriy `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Kanal xabar harakatlari qulay aliaslarni taqdim etadi (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Cheklash boshqaruvlari:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (standart: o‘chirilgan)
    
    Reaksiyani olib tashlash semantikasi: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">
    Telegram generatsiya qilingan chiqishda aniq reply threading teglarini qo‘llab-quvvatlaydi:


    ```
    - `[[reply_to_current]]` ishga tushirgan xabarga javob beradi
    - `[[reply_to:<id>]]` ma’lum bir Telegram xabar ID siga javob beradi
    
    `channels.telegram.replyToMode` qayta ishlashni boshqaradi:
    
    - `off` (standart)
    - `first`
    - `all`
    
    Eslatma: `off` implicit reply threading’ni o‘chiradi. Aniq `[[reply_to_*]]` teglari baribir hisobga olinadi.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    Forum superguruhlar:


    ```
    - mavzu seans kalitlari `:topic:<threadId>` qo‘shimchasini qo‘shadi
    - javoblar va typing amallari mavzu thread’iga yo‘naltiriladi
    - mavzu konfiguratsiya yo‘li:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    Umumiy mavzu (`threadId=1`) maxsus holat:
    
    - xabar yuborishda `message_thread_id` ko‘rsatilmaydi (Telegram `sendMessage(...thread_id=1)` ni rad etadi)
    - typing amallari baribir `message_thread_id` ni o‘z ichiga oladi
    
    Mavzu merosxo‘rligi: mavzu yozuvlari guruh sozlamalarini meros qilib oladi, agar ustiga yozilmagan bo‘lsa (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).
    
    Shablon konteksti quyidagilarni o‘z ichiga oladi:
    
    - `MessageThreadId`
    - `IsForum`
    
    DM thread xatti-harakati:
    
    - `message_thread_id` ga ega private chatlar DM marshrutini saqlab qoladi, lekin thread-aware seans kalitlari/javob maqsadlaridan foydalanadi.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Audio xabarlar


    ```
    Telegram voice note va audio fayllarni farqlaydi.
    
    - standart: audio fayl xatti-harakati
    - agent javobida `[[audio_as_voice]]` tegi voice-note sifatida yuborishni majburlaydi
    
    Xabar harakati misoli:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### Video xabarlar
    
    Telegram video fayllar va video note’larni farqlaydi.
    
    Xabar harakati misoli:
    ```

```json5
36. (Eslatma: Video eslatmalar sarlavhani qo‘llab-quvvatlamaydi.
```

    ```
    Video note’lar caption’ni qo‘llab-quvvatlamaydi; berilgan xabar matni alohida yuboriladi.
    
    ### Stikerlar
    
    Kiruvchi stikerlarni qayta ishlash:
    
    - statik WEBP: yuklab olinadi va qayta ishlanadi (placeholder `<media:sticker>`)
    - animatsiyalangan TGS: o‘tkazib yuboriladi
    - video WEBM: o‘tkazib yuboriladi
    
    Stiker konteksti maydonlari:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Stiker kesh fayli:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Stikerlar (imkon bo‘lsa) bir marta tavsiflanadi va takroriy vision chaqiruvlarini kamaytirish uchun kesh qilinadi.
    
    Stiker harakatlarini yoqing:
    ```

```json5
18. {
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Stiker yuborish harakati:
    ```

```json5
20. {
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Kesh qilingan stikerlarni qidirish:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">
    Telegram reaksiyalari `message_reaction` yangilanishlari sifatida keladi (xabar payloadlaridan alohida).


    ```
    Yoqilganda, OpenClaw quyidagi kabi tizim hodisalarini navbatga qo‘shadi:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Sozlama:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (standart: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (standart: `minimal`)
    
    Eslatma:
    
    - `own` faqat bot yuborgan xabarlarga foydalanuvchi reaksiyalarini anglatadi (yuborilgan xabarlar keshi orqali best-effort).
    - Telegram reaksiya yangilanishlarida thread ID bermaydi.
      - forum bo‘lmagan guruhlar guruh chat seansiga yo‘naltiriladi
      - forum guruhlari aniq kelib chiqqan mavzuga emas, balki guruhning umumiy mavzu seansiga (`:topic:1`) yo‘naltiriladi
    
    Polling/webhook uchun `allowed_updates` avtomatik ravishda `message_reaction` ni o‘z ichiga oladi.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` OpenClaw kiruvchi xabarni qayta ishlayotgan paytda tasdiqlovchi emoji yuboradi.


    ```
    Hal qilish tartibi:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, aks holda "👀")
    
    Eslatma:
    
    - Telegram unicode emoji kutadi (masalan "👀").
    - Kanal yoki akkaunt uchun reaksiyani o‘chirish uchun `""` dan foydalaning.
    
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Kanal konfiguratsiya yozuvlari standart bo‘yicha yoqilgan (`configWrites !== false`).


    ```
    Telegram orqali ishga tushiriladigan yozuvlar quyidagilarni o‘z ichiga oladi:
    
    - guruh migratsiya hodisalari (`migrate_to_chat_id`) `channels.telegram.groups` ni yangilash uchun
    - `/config set` va `/config unset` (buyruq yoqilgan bo‘lishi kerak)
    
    O‘chirish:
    ```

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    Standart: long polling.


    ```
    Webhook rejimi:
    
    - `channels.telegram.webhookUrl` ni o‘rnating
    - `channels.telegram.webhookSecret` ni o‘rnating (webhook URL o‘rnatilganda majburiy)
    - ixtiyoriy `channels.telegram.webhookPath` (standart `/telegram-webhook`)
    - ixtiyoriy `channels.telegram.webhookHost` (standart `127.0.0.1`)
    
    Webhook rejimi uchun standart lokal tinglovchi `127.0.0.1:8787` ga ulanadi.
    
    Agar ommaviy endpoint boshqacha bo‘lsa, oldiga reverse proxy qo‘ying va `webhookUrl` ni ommaviy URL ga yo‘naltiring.
    Tashqi kirish ataylab kerak bo‘lsa, `webhookHost` ni (masalan `0.0.0.0`) o‘rnating.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - `channels.telegram.textChunkLimit` standart qiymati 4000.
    - `channels.telegram.chunkMode="newline"` uzunlik bo‘yicha bo‘lishdan oldin paragraf chegaralarini (bo‘sh qatorlar) afzal ko‘radi.
    - `channels.telegram.mediaMaxMb` (standart 5) kiruvchi Telegram media fayllarini yuklab olish/qayta ishlash hajmini cheklaydi.
    - `channels.telegram.timeoutSeconds` Telegram API mijozining kutish vaqtini bekor qiladi (agar o‘rnatilmagan bo‘lsa, grammY standarti qo‘llanadi).
    - guruh konteksti tarixi `channels.telegram.historyLimit` yoki `messages.groupChat.historyLimit` (standart 50) dan foydalanadi; `0` o‘chiradi.
    - DM tarixi boshqaruvlari:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - chiquvchi Telegram API qayta urinishlari `channels.telegram.retry` orqali sozlanadi.

    ```
    CLI yuborish manzili raqamli chat ID yoki username bo‘lishi mumkin:
    ```

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  
</Accordion>
</AccordionGroup>

## 45. Nosozliklarni bartaraf etish

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - Agar `requireMention=false` bo‘lsa, Telegram maxfiylik rejimi to‘liq ko‘rinishga ruxsat berishi kerak.
      - BotFather: `/setprivacy` -> Disable
      - so‘ng botni guruhdan olib tashlab, qayta qo‘shing
    - `openclaw channels status` konfiguratsiya eslatmasiz guruh xabarlarini kutayotganida ogohlantiradi.
    - `openclaw channels status --probe` aniq raqamli guruh IDlarini tekshirishi mumkin; wildcard `"*"` uchun a’zolikni tekshirib bo‘lmaydi.
    - tezkor sessiya testi: `/activation always`.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - agar `channels.telegram.groups` mavjud bo‘lsa, guruh ro‘yxatda bo‘lishi kerak (yoki `"*"` ni o‘z ichiga olishi kerak)
    - botning guruhdagi a’zoligini tekshiring
    - o‘tkazib yuborish sabablarini ko‘rish uchun loglarni tekshiring: `openclaw logs --follow`
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - yuboruvchi identifikatoringizni avtorizatsiya qiling (juftlash va/yoki raqamli `allowFrom`)
    - guruh siyosati `open` bo‘lsa ham buyruq avtorizatsiyasi amal qiladi
    - `setMyCommands failed` odatda `api.telegram.org` ga DNS/HTTPS ulanish muammolarini bildiradi
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + maxsus fetch/proxy AbortSignal turlari mos kelmasa, darhol bekor qilish xatti-harakatini keltirib chiqarishi mumkin.
    - Ba’zi xostlar `api.telegram.org` ni avval IPv6 ga yo‘naltiradi; ishlamaydigan IPv6 chiqishi Telegram API’da vaqti-vaqti bilan nosozliklarga sabab bo‘lishi mumkin.
    - DNS javoblarini tekshiring:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Qo‘shimcha yordam: [Channel troubleshooting](/channels/troubleshooting).

## Telegram konfiguratsiya bo‘yicha ma’lumot ko‘rsatkichlari

Xabar yuborish vositalari uchun, ovozga mos audio `media` URL bilan `asVoice: true` ni o‘rnating
(`message` media mavjud bo‘lsa ixtiyoriy):

- `channels.telegram.enabled`: kanal ishga tushishini yoqish/o‘chirish.

- `channels.telegram.botToken`: bot tokeni (BotFather).

- `channels.telegram.tokenFile`: tokenni fayl yo‘lidan o‘qish.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (standart: pairing).

- `channels.telegram.allowFrom`: DM allowlist (raqamli Telegram foydalanuvchi IDlari). `open` uchun `"*"` talab qilinadi. `openclaw doctor --fix` eski `@username` yozuvlarini IDlarga aylantirishi mumkin.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (standart: allowlist).

- `channels.telegram.groupAllowFrom`: guruh yuboruvchilari uchun allowlist (raqamli Telegram foydalanuvchi IDlari). `openclaw doctor --fix` eski `@username` yozuvlarini IDlarga aylantirishi mumkin.

- `channels.telegram.groups`: har bir guruh uchun standartlar + ruxsat ro‘yxati (`"*"` global standartlar uchun).
  - 39. **Bir dona WhatsApp raqamida bir nechta OpenClaw instansiyasidan foydalanish mumkinmi?**  
        Ha, har bir jo‘natuvchini `bindings` orqali turli agentlarga yo‘naltirish orqali (peer `kind: "direct"`, jo‘natuvchi E.164 masalan `+15551234567`)..groupPolicy`: groupPolicy uchun guruh bo‘yicha override (`open | allowlist | disabled\`).
  - `channels.telegram.groups.<id>.requireMention`: eslatma talabini boshqarish uchun standart.
  - `channels.telegram.groups.<id>.skills`: skill filtri (qoldirilsa = barcha skilllar, bo‘sh = hech biri).
  - `channels.telegram.groups.<id>.allowFrom`: guruh bo‘yicha jo‘natuvchilar ruxsat ro‘yxatini override qilish.
  - `channels.telegram.groups.<id>.systemPrompt`: guruh uchun qo‘shimcha system prompt.
  - `channels.telegram.groups.<id>.enabled`: `false` bo‘lsa, guruhni o‘chirish.
  - `channels.telegram.groups.<id>46. .topics.<threadId>.*`: mavzu bo‘yicha override’lar (guruh bilan bir xil maydonlar).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: groupPolicy uchun mavzu bo‘yicha override (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic mention gating override.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (default: allowlist).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account override.

- `channels.telegram.replyToMode`: `off | first | all` (standart: `off`).

- `channels.telegram.textChunkLimit`: outbound chunk size (chars).

- `channels.telegram.chunkMode`: `length` (default) or `newline` to split on blank lines (paragraph boundaries) before length chunking.

- `channels.telegram.linkPreview`: toggle link previews for outbound messages (default: true).

- `channels.telegram.streamMode`: `off | partial | block` (jonli oqim preview).

- `channels.telegram.mediaMaxMb`: inbound/outbound media cap (MB).

- `channels.telegram.retry`: retry policy for outbound Telegram API calls (attempts, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`: proxy URL for Bot API calls (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: enable webhook mode (requires `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: webhook secret (required when webhookUrl is set).

- `channels.telegram.webhookPath`: local webhook path (default `/telegram-webhook`).

- `channels.telegram.webhookHost`: lokal webhook bind xosti (standart `127.0.0.1`).

- `channels.telegram.actions.reactions`: gate Telegram tool reactions.

- `channels.telegram.actions.sendMessage`: gate Telegram tool message sends.

- `channels.telegram.actions.deleteMessage`: gate Telegram tool message deletes.

- `channels.telegram.actions.sticker`: gate Telegram sticker actions — send and search (default: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — control which reactions trigger system events (default: `own` when not set).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — control agent's reaction capability (default: `minimal` when not set).

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

Telegram’ga xos muhim maydonlar:

- OpenClaw sukut bo‘yicha video fayllardan foydalanadi.
- Xabar yuborish vositasi orqali jo‘natishda video `media` URL bilan `asVideoNote: true` ni o‘rnating:
- {
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
  }
- threading/replies: `replyToMode`
- streaming: `streamMode` (preview), `draftChunk`, `blockStreaming`
- formatlash/yetkazib berish: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/tarmoq: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- harakatlar/imkoniyatlar: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- reaksiyalar: `reactionNotifications`, `reactionLevel`
- yozishlar/tarix: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Bog‘liq

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)


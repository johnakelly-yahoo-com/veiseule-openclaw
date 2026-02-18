---
title: "13. Sukut bo‘yicha long-polling; webhook ixtiyoriy."
---

# 14. Tezkor sozlash (boshlovchilar uchun)

15. **@BotFather** bilan bot yarating ([to‘g‘ridan-to‘g‘ri havola](https://t.me/BotFather)). Long-polling by default; webhook optional.

## 17. Tokenni o‘rnating:

1. 18. Muhit: `TELEGRAM_BOT_TOKEN=...` 19. Yoki konfiguratsiya: `channels.telegram.botToken: "..."`.
2. 20. Ikkalasi ham o‘rnatilgan bo‘lsa, konfiguratsiya ustuvor (muhit — faqat sukut bo‘yicha akkaunt uchun zaxira).
   - 21. Gateway’ni ishga tushiring.
   - 22. DM kirishi sukut bo‘yicha juftlash orqali; birinchi aloqada juftlash kodini tasdiqlang.
   - 23. Minimal konfiguratsiya:
3. Start the gateway.
4. 25. Bu nima

26) Gateway’ga tegishli Telegram Bot API kanali.

```json5
27. Deterministik marshrutlash: javoblar Telegram’ga qaytadi; model kanallarni tanlamaydi.
```

## 28. DMlar agentning asosiy sessiyasini bo‘lishadi; guruhlar alohida qoladi (`agent:<agentId>:telegram:group:<chatId>`).

- 29. Sozlash (tez yo‘l)
- 30. 1. Bot tokenini yarating (BotFather)
- 31. Telegram’ni oching va **@BotFather** bilan chat qiling ([to‘g‘ridan-to‘g‘ri havola](https://t.me/BotFather)).

## 32. Handle aniq `@BotFather` ekanini tasdiqlang.

### 33. `/newbot` buyrug‘ini bajaring, so‘ng ko‘rsatmalarga amal qiling (nom + `bot` bilan tugaydigan foydalanuvchi nomi).

1. 34. Tokenni nusxalang va uni xavfsiz saqlang. 35. Ixtiyoriy BotFather sozlamalari:
2. 36. `/setjoingroups` — botni guruhlarga qo‘shishga ruxsat berish/taqiqlash.
3. 37. `/setprivacy` — bot barcha guruh xabarlarini ko‘rishini boshqarish.

38) 2) Tokenni sozlang (muhit yoki konfiguratsiya)

- 39. Misol:
- 40. {
      channels: {
      telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "\*": { requireMention: true } },
      },
      },
      }

### 41. Muhit varianti: `TELEGRAM_BOT_TOKEN=...` (sukut bo‘yicha akkaunt uchun ishlaydi).

42. Muhit va konfiguratsiya ikkalasi ham o‘rnatilgan bo‘lsa, konfiguratsiya ustuvor.

```json5
43. Ko‘p akkauntli qo‘llab-quvvatlash: akkauntlar bo‘yicha tokenlar va ixtiyoriy `name` bilan `channels.telegram.accounts` dan foydalaning.
```

44. Umumiy naqsh uchun [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) ga qarang.
45. Gateway’ni ishga tushiring.

46. Token aniqlanganda Telegram ishga tushadi (avval konfiguratsiya, muhit — zaxira). 47. DM kirishi sukut bo‘yicha juftlash.

3. 48. Bot bilan birinchi marta bog‘langanda kodni tasdiqlang. 49. Guruhlar uchun: botni qo‘shing, maxfiylik/administrator xatti-harakatlarini tanlang (quyida), so‘ng eslatma shlyuzi va ruxsat ro‘yxatlarini boshqarish uchun `channels.telegram.groups` ni sozlang.
4. 50. Token + maxfiylik + ruxsatlar (Telegram tomoni) Approve the code when the bot is first contacted.
5. For groups: add the bot, decide privacy/admin behavior (below), then set `channels.telegram.groups` to control mention gating + allowlists.

## Token + privacy + permissions (Telegram side)

### Token yaratish (BotFather)

- `/newbot` botni yaratadi va tokenni qaytaradi (uni maxfiy saqlang).
- Agar token sizib chiqsa, uni @BotFather orqali bekor qiling/qayta yarating va konfiguratsiyangizni yangilang.

### Guruh xabarlarining ko‘rinishi (Privacy Mode)

Telegram botlari sukut bo‘yicha **Privacy Mode** rejimida bo‘ladi, bu esa guruhdagi qaysi xabarlarni qabul qilishini cheklaydi.
Agar botingiz guruhdagi _barcha_ xabarlarni ko‘rishi kerak bo‘lsa, ikkita variant bor:

- `/setprivacy` orqali privacy mode’ni o‘chiring **yoki**
- Botni guruhga **admin** sifatida qo‘shing (admin botlar barcha xabarlarni oladi).

**Eslatma:** Privacy mode’ni o‘zgartirganda, o‘zgarish kuchga kirishi uchun Telegram botni har bir guruhdan olib tashlab, qayta qo‘shishni talab qiladi.

### Guruh ruxsatlari (admin huquqlari)

Admin maqomi guruh ichida (Telegram UI orqali) o‘rnatiladi. Admin botlar har doim barcha guruh xabarlarini oladi, shuning uchun to‘liq ko‘rinish kerak bo‘lsa admin’dan foydalaning.

## Qanday ishlaydi (xatti-harakat)

- Kiruvchi xabarlar javob konteksti va media placeholder’lari bilan umumiy kanal konvertiga normallashtiriladi.
- Guruh javoblari sukut bo‘yicha mention talab qiladi (mahalliy @mention yoki `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
- Multi-agent override: har bir agent uchun `agents.list[].groupChat.mentionPatterns` da alohida pattern’larni belgilang.
- Javoblar har doim o‘sha Telegram chat’iga qaytadi.
- Long-polling grammY runner’dan har-chat ketma-ketligi bilan foydalanadi; umumiy parallelizm `agents.defaults.maxConcurrent` bilan cheklanadi.
- Telegram Bot API o‘qilganlik kvitansiyalarini qo‘llab-quvvatlamaydi; `sendReadReceipts` opsiyasi yo‘q.

## Draft streaming

OpenClaw Telegram DM’larda `sendMessageDraft` orqali qisman javoblarni stream qilishi mumkin.

Talablar:

- @BotFather’da bot uchun Threaded Mode yoqilgan bo‘lishi (forum topic mode).
- Faqat private chat thread’lari (Telegram kiruvchi xabarlarga `message_thread_id` ni qo‘shadi).
- `channels.telegram.streamMode` `"off"` ga o‘rnatilmagan bo‘lishi (sukut bo‘yicha: `"partial"`, `"block"` esa bo‘laklab draft yangilanishlarini yoqadi).

Draft streaming faqat DM’lar uchun; Telegram guruhlar yoki kanallarda buni qo‘llab-quvvatlamaydi.

## Formatlash (Telegram HTML)

- Chiquvchi Telegram matni `parse_mode: "HTML"` dan foydalanadi (Telegram qo‘llab-quvvatlaydigan teglar to‘plami).
- Markdown’ga o‘xshash kirish **Telegram’ga xavfsiz HTML** ga render qilinadi (qalin/kursiv/chizilgan/kod/havolalar); blok elementlar yangi qatorlar/bullet’lar bilan matnga tekislanadi.
- Raw HTML from models is escaped to avoid Telegram parse errors.
- Agar Telegram HTML payload’ni rad etsa, OpenClaw xuddi shu xabarni oddiy matn sifatida qayta yuboradi.

## Buyruqlar (native + custom)

OpenClaw registers native commands (like `/status`, `/reset`, `/model`) with Telegram’s bot menu on startup.
Menyu’ga custom buyruqlarni konfiguratsiya orqali qo‘shishingiz mumkin:

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

## Sozlashdagi muammolarni bartaraf etish (buyruqlar)

- Loglarda `setMyCommands failed` odatda `api.telegram.org` ga chiquvchi HTTPS/DNS bloklanganini anglatadi.
- Agar `sendMessage` yoki `sendChatAction` xatolarini ko‘rsangiz, IPv6 marshrutlash va DNS’ni tekshiring.

Qo‘shimcha yordam: [Channel troubleshooting](/channels/troubleshooting).

Eslatmalar:

- Custom buyruqlar faqat **menyu elementlari**; OpenClaw ularni boshqa joyda qayta ishlamasangiz, bajarib bermaydi.
- 24. Ular yozib yuborilganda baribir ishlaydi (shunchaki `/commands` yoki menyuda ko‘rinmaydi). 25. Qurilma juftlash buyruqlari (`device-pair` plagini)
- Buyruq nomlari normallashtiriladi (oldidagi `/` olib tashlanadi, kichik harflarga o‘tkaziladi) va `a-z`, `0-9`, `_` ga mos bo‘lishi kerak (1–32 belgi).
- Custom buyruqlar **native buyruqlarni bekor qila olmaydi**. Ziddiyatlar e’tiborga olinmaydi va log qilinadi.
- Agar `commands.native` o‘chirilgan bo‘lsa, faqat custom buyruqlar ro‘yxatdan o‘tkaziladi (yoki bo‘lmasa tozalanadi).

### 26. Agar `device-pair` plagini o‘rnatilgan bo‘lsa, u yangi telefonni juftlash uchun Telegram-birinchi oqimini qo‘shadi:

27. `/pair` sozlash kodini yaratadi (oson nusxalash/joylash uchun alohida xabar sifatida yuboriladi).

1. 28. iOS ilovasida sozlash kodini joylashtirib ulaning.
2. 29. `/pair approve` eng so‘nggi kutilayotgan qurilma so‘rovini tasdiqlaydi.
3. 30. Batafsil ma’lumot: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

31) Video xabarlar (video va video eslatma o‘rtasidagi farq)

## Cheklovlar

- Chiquvchi matn `channels.telegram.textChunkLimit` ga bo‘laklanadi (sukut bo‘yicha 4000).
- Ixtiyoriy yangi qator bo‘yicha bo‘laklash: uzunlik bo‘yicha bo‘laklashdan oldin bo‘sh qatorlar (paragraf chegaralari) bo‘yicha ajratish uchun `channels.telegram.chunkMode="newline"` ni o‘rnating.
- Media yuklab olish/yuklash `channels.telegram.mediaMaxMb` bilan cheklanadi (sukut bo‘yicha 5).
- Telegram Bot API so‘rovlari `channels.telegram.timeoutSeconds` dan keyin timeout bo‘ladi (sukut bo‘yicha grammY orqali 500). 1. Uzoq osilib qolishlarning oldini olish uchun quyi qiymatni o‘rnating.
- Group history context uses `channels.telegram.historyLimit` (or `channels.telegram.accounts.*.historyLimit`), falling back to `messages.groupChat.historyLimit`. 3. O‘chirish uchun `0` ni o‘rnating (standart 50).
- 4. DM tarixini `channels.telegram.dmHistoryLimit` (foydalanuvchi almashuvlari) bilan cheklash mumkin. 5. Har bir foydalanuvchi uchun alohida sozlamalar: `channels.telegram.dms["<user_id>"].historyLimit`.

## Group activation modes

7. Standart holatda bot guruhlarda faqat eslatmalariga javob beradi (`@botname` yoki `agents.list[].groupChat.mentionPatterns` dagi naqshlar). 8. Bu xatti-harakatni o‘zgartirish uchun:

### 9. Konfiguratsiya orqali (tavsiya etiladi)

```json5
10. {
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // bu guruhda har doim javob beradi
      },
    },
  },
}
```

11. **Muhim:** `channels.telegram.groups` ni sozlash **allowlist** yaratadi — faqat ro‘yxatdagi guruhlar (yoki `"*"`) qabul qilinadi.
12. Forum mavzulari ota-guruh konfiguratsiyasini (allowFrom, requireMention, skills, prompts) meros qilib oladi, agar siz `channels.telegram.groups.<groupId>` ostida mavzu bo‘yicha alohida sozlamalar qo‘shmasangiz13. .topics.&lt;topicId&gt;14. \`.

15. Barcha guruhlarga doimo javob berishga ruxsat berish uchun:

```json5
16. {
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }, // barcha guruhlar, har doim javob beradi
      },
    },
  },
}
```

17. Barcha guruhlar uchun faqat eslatma bilan javob berishni saqlash (standart xatti-harakat):

```json5
18. {
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // yoki groups ni umuman ko‘rsatmaslik
      },
    },
  },
}
```

### 19. Buyruq orqali (sessiya darajasida)

20. Guruhda yuboring:

- 21. `/activation always` — barcha xabarlarga javob berish
- 22. `/activation mention` — eslatmani talab qilish (standart)

23. **Eslatma:** Buyruqlar faqat sessiya holatini yangilaydi. 24. Qayta ishga tushirishlar orasida doimiy xatti-harakat uchun konfiguratsiyadan foydalaning.

### 25. Guruh chat ID sini olish

26. Guruhdan istalgan xabarni Telegram’da `@userinfobot` yoki `@getidsbot` ga yo‘naltiring, chat ID ni ko‘rish uchun ( `-1001234567890` kabi manfiy raqam).

27. **Maslahat:** O‘zingizning foydalanuvchi ID’ingiz uchun botga DM yuboring — u sizning user ID’ingiz bilan javob beradi (juftlash xabari), yoki buyruqlar yoqilgach `/whoami` dan foydalaning.

28. **Maxfiylik eslatmasi:** `@userinfobot` — uchinchi tomon botidir. 29. Agar xohlasangiz, botni guruhga qo‘shing, xabar yuboring va `openclaw logs --follow` dan `chat.id` ni o‘qish uchun foydalaning yoki Bot API’dagi `getUpdates` dan foydalaning.

## 30. Konfiguratsiya yozuvlari

31. Standart holatda Telegram kanal hodisalari yoki `/config set|unset` tomonidan qo‘zg‘atilgan konfiguratsiya yangilanishlarini yozishga ruxsat etilgan.

32. Bu quyidagi hollarda sodir bo‘ladi:

- 33. Guruh superguruhga yangilanganda va Telegram `migrate_to_chat_id` ni yuborganda (chat ID o‘zgaradi). 34. OpenClaw `channels.telegram.groups` ni avtomatik ko‘chira oladi.
- 35. Telegram chatida `/config set` yoki `/config unset` ni ishga tushirsangiz ( `commands.config: true` talab etiladi).

36. O‘chirish uchun:

```json5
37. {
  channels: { telegram: { configWrites: false } },
}
```

## 38. Mavzular (forum superguruhlar)

39. Telegram forum mavzulari har bir xabar uchun `message_thread_id` ni o‘z ichiga oladi. 40. OpenClaw:

- 41. Har bir mavzu alohida bo‘lishi uchun Telegram guruh sessiya kalitiga `:topic:<threadId>` ni qo‘shadi.
- 42. Javoblar mavzuda qolishi uchun yozish indikatorlarini yuboradi va `message_thread_id` bilan javob beradi.
- 43. Umumiy mavzu (thread id `1`) maxsus: xabar yuborishda `message_thread_id` qo‘shilmaydi (Telegram rad etadi), ammo yozish indikatorlari baribir uni o‘z ichiga oladi.
- 44. Marshrutlash/shablonlash uchun shablon kontekstida `MessageThreadId` + `IsForum` ni ochib beradi.
- 45. Mavzu bo‘yicha maxsus konfiguratsiya `channels.telegram.groups.<chatId>` ostida mavjud46. .topics.&lt;threadId&gt;47. \` (skills, allowlists, auto-reply, system prompts, disable).
- 48. Mavzu konfiguratsiyalari guruh sozlamalarini (requireMention, allowlists, skills, prompts, enabled) meros qilib oladi, agar mavzu bo‘yicha alohida bekor qilinmasa.

49. Shaxsiy chatlar ayrim chekka holatlarda `message_thread_id` ni o‘z ichiga olishi mumkin. 50. OpenClaw DM sessiya kalitini o‘zgartirmaydi, ammo u mavjud bo‘lsa, javoblar/qoralama oqimi uchun thread id dan foydalanadi.

## Inline Buttons

Telegram supports inline keyboards with callback buttons.

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

For per-account configuration:

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

Scopes:

- `off` — inline buttons disabled
- `dm` — only DMs (group targets blocked)
- `group` — only groups (DM targets blocked)
- `all` — DMs + groups
- `allowlist` — DMs + groups, but only senders allowed by `allowFrom`/`groupAllowFrom` (same rules as control commands)

Default: `allowlist`.
Legacy: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

### Sending buttons

Use the message tool with the `buttons` parameter:

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

When a user clicks a button, the callback data is sent back to the agent as a message with the format:
`callback_data: value`

### Configuration options

Telegram capabilities can be configured at two levels (object form shown above; legacy string arrays still supported):

- `channels.telegram.capabilities`: Global default capability config applied to all Telegram accounts unless overridden.
- `channels.telegram.accounts.<account>.capabilities`: Per-account capabilities that override the global defaults for that specific account.

Use the global setting when all Telegram bots/accounts should behave the same. Use per-account configuration when different bots need different behaviors (for example, one account only handles DMs while another is allowed in groups).

## Access control (DMs + groups)

### DM access

- Default: `channels.telegram.dmPolicy = "pairing"`. Unknown senders receive a pairing code; messages are ignored until approved (codes expire after 1 hour).
- Approve via:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- Pairing is the default token exchange used for Telegram DMs. Details: [Pairing](/channels/pairing)
- `channels.telegram.allowFrom` accepts numeric user IDs (recommended) or `@username` entries. It is **not** the bot username; use the human sender’s ID. The wizard accepts `@username` and resolves it to the numeric ID when possible.

#### Finding your Telegram user ID

Safer (no third-party bot):

1. Start the gateway and DM your bot.
2. Run `openclaw logs --follow` and look for `from.id`.

Alternate (official Bot API):

1. DM your bot.
2. Fetch updates with your bot token and read `message.from.id`:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Third-party (less private):

- DM `@userinfobot` or `@getidsbot` and use the returned user id.

### Group access

Two independent controls:

**1. Which groups are allowed** (group allowlist via `channels.telegram.groups`):

- `groups` konfiguratsiyasi yo‘q = barcha guruhlarga ruxsat beriladi
- `groups` konfiguratsiyasi mavjud = faqat ro‘yxatdagi guruhlar yoki `"*"` ga ruxsat beriladi
- Misol: `"groups": { "-1001234567890": {}, "*": {} }` barcha guruhlarga ruxsat beradi

\*\*2. **Qaysi yuboruvchilarga ruxsat beriladi** (`channels.telegram.groupPolicy` orqali yuboruvchini filtrlash):

- `"open"` = ruxsat etilgan guruhlardagi barcha yuboruvchilar xabar yubora oladi
- `"allowlist"` = faqat `channels.telegram.groupAllowFrom` dagi yuboruvchilar xabar yubora oladi
- `"disabled"` = guruh xabarlari umuman qabul qilinmaydi
  Standart holat `groupPolicy: "allowlist"` (agar `groupAllowFrom` qo‘shmasangiz, bloklanadi).

Most users want: `groupPolicy: "allowlist"` + `groupAllowFrom` + specific groups listed in `channels.telegram.groups`

Muayyan guruhda **har qanday guruh a’zosi** gaplasha olishi uchun (boshqaruv buyruqlarini esa faqat vakolatli yuboruvchilarga cheklagan holda), guruh bo‘yicha override sozlang:

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

## Long-polling vs webhook

- Standart: long-polling (ommaviy URL talab qilinmaydi).
- Webhook mode: set `channels.telegram.webhookUrl` and `channels.telegram.webhookSecret` (optionally `channels.telegram.webhookPath`).
  - Mahalliy tinglovchi standart bo‘yicha `0.0.0.0:8787` ga ulanadi va `POST /telegram-webhook` ni xizmat qiladi.
  - Agar ommaviy URL’ingiz boshqacha bo‘lsa, reverse proxy’dan foydalaning va `channels.telegram.webhookUrl` ni ommaviy endpoint’ga yo‘naltiring.

## Javoblarni ipga bog‘lash (reply threading)

Telegram teglar orqali ixtiyoriy ipga bog‘langan javoblarni qo‘llab-quvvatlaydi:

- `[[reply_to_current]]` -- qo‘zg‘atuvchi xabarga javob berish.
- `[[reply_to:<id>]]` -- aniq xabar ID’siga javob berish.

`channels.telegram.replyToMode` orqali boshqariladi:

- `first` (standart), `all`, `off`.

## Audio xabarlar (ovozli vs fayl)

Telegram **ovozli eslatmalar**ni (dumaloq pufakcha) **audio fayllar**dan (metadata kartasi) ajratadi.
OpenClaw orqaga moslik uchun standart bo‘yicha audio fayllardan foydalanadi.

Agent javoblarida ovozli eslatma pufakchasini majburlash uchun, javobning istalgan joyiga ushbu tegni kiriting:

- `[[audio_as_voice]]` — audio fayl o‘rniga ovozli eslatma sifatida yuboradi.

Bu teg yetkazilgan matndan olib tashlanadi. Boshqa kanallar bu tegni e’tiborsiz qoldiradi.

Xabar yuborish vositalari uchun, ovozga mos audio `media` URL bilan `asVoice: true` ni o‘rnating
(`message` media mavjud bo‘lsa ixtiyoriy):

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## 32. Telegram **video eslatmalarni** (dumaloq pufak) va **video fayllarni** (to‘rtburchak) farqlaydi.

33. OpenClaw sukut bo‘yicha video fayllardan foydalanadi.
34. Xabar yuborish vositasi orqali jo‘natishda video `media` URL bilan `asVideoNote: true` ni o‘rnating:

35. {
    action: "send",
    channel: "telegram",
    to: "123456789",
    media: "https://example.com/video.mp4",
    asVideoNote: true,
    }

```json5
36. (Eslatma: Video eslatmalar sarlavhani qo‘llab-quvvatlamaydi.
```

37. Agar matnli xabar bersangiz, u alohida xabar sifatida yuboriladi.) 38. \`channels.telegram.groups.<id>

## Stikerlar

OpenClaw Telegram stikerlarini aqlli keshlash bilan qabul qilish va yuborishni qo‘llab-quvvatlaydi.

### Stikerlarni qabul qilish

Foydalanuvchi stiker yuborganda, OpenClaw uni stiker turiga qarab qayta ishlaydi:

- **Statik stikerlar (WEBP):** Yuklab olinadi va vision orqali qayta ishlanadi. Xabar mazmunida stiker `<media:sticker>` placeholder sifatida ko‘rinadi.
- **Animatsiyalangan stikerlar (TGS):** O‘tkazib yuboriladi (Lottie formati qayta ishlash uchun qo‘llab-quvvatlanmaydi).
- **Video stikerlar (WEBM):** O‘tkazib yuboriladi (video formati qayta ishlash uchun qo‘llab-quvvatlanmaydi).

Stikerlar qabul qilinganda mavjud bo‘lgan template context maydoni:

- `Sticker` — obyekt, quyidagilar bilan:
  - `emoji` — stiker bilan bog‘langan emoji
  - `setName` — stiker to‘plami nomi
  - `fileId` — Telegram fayl ID’si (xuddi shu stikerni qayta yuborish uchun)
  - `fileUniqueId` — kesh qidiruvi uchun barqaror ID
  - `cachedDescription` — mavjud bo‘lsa, keshlangan vision tavsifi

### Stikerlar keshi

Stikerlar tavsiflarni yaratish uchun AI’ning vision imkoniyatlari orqali qayta ishlanadi. Since the same stickers are often sent repeatedly, OpenClaw caches these descriptions to avoid redundant API calls.

**Qanday ishlaydi:**

1. 1. **Birinchi uchrashuv:** Stiker rasmi ko‘rish tahlili uchun AI’ga yuboriladi. 2. AI tavsif yaratadi (masalan, "Qiziqarli tarzda qo‘l silkitayotgan multfilm mushugi").
2. 3. **Keshda saqlash:** Tavsif stikerning fayl ID’si, emoji va to‘plam nomi bilan birga saqlanadi.
3. 4. **Keyingi uchrashuvlar:** Xuddi shu stiker yana ko‘rilganda, keshdagi tavsif to‘g‘ridan-to‘g‘ri ishlatiladi. 5. Rasm AI’ga yuborilmaydi.

6) **Kesh joylashuvi:** `~/.openclaw/telegram/sticker-cache.json`

7. **Kesh yozuvi formati:**

```json
8. {
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "Qiziqarli tarzda qo‘l silkitayotgan multfilm mushugi",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

9. **Afzalliklar:**

- Reduces API costs by avoiding repeated vision calls for the same sticker
- Keshlangan stikerlar uchun tezroq javob vaqtlari (ko‘rish qayta ishlashidagi kechikishlarsiz)
- Keshlangan tavsiflarga asoslangan stiker qidirish funksiyasini yoqadi

13. Kesh stikerlar qabul qilinganda avtomatik ravishda to‘ldiriladi. 14. Keshni qo‘lda boshqarish talab etilmaydi.

### 15. Stikerlarni yuborish

Agent `sticker` va `sticker-search` amallari orqali stikerlarni yuborishi va qidirishi mumkin. 17. Ular sukut bo‘yicha o‘chirilgan va konfiguratsiyada yoqilishi kerak:

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

19. **Stiker yuborish:**

```json5
20. {
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

21. Parametrlar:

- 22. `fileId` (majburiy) — stikerning Telegram fayl ID’si. 23. Buni stiker qabul qilinganda `Sticker.fileId` dan yoki `sticker-search` natijasidan olish mumkin.
- 24. `replyTo` (ixtiyoriy) — javob beriladigan xabar ID’si.
- 25. `threadId` (ixtiyoriy) — forum mavzulari uchun xabar oqimi ID’si.

26. **Stikerlarni qidirish:**

27. Agent keshdagi stikerlarni tavsif, emoji yoki to‘plam nomi bo‘yicha qidirishi mumkin:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

29. Keshdan mos keladigan stikerlarni qaytaradi:

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

31. Qidiruv tavsif matni, emoji belgilar va to‘plam nomlari bo‘ylab noaniq moslashtirishdan foydalanadi.

32. **Oqimlar bilan misol:**

```json5
33. {
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## 34. Oqim (qoralamalar)

35. Agent javobni yaratish jarayonida Telegram **qoralama pufakchalarini** oqimda ko‘rsatishi mumkin.
36. OpenClaw Bot API’ning `sendMessageDraft` (haqiqiy xabar emas) usulidan foydalanadi va so‘ng yakuniy javobni oddiy xabar sifatida yuboradi.

37. Talablar (Telegram Bot API 9.3+):

- 38. **Mavzular yoqilgan shaxsiy chatlar** (bot uchun forum mavzusi rejimi).
- 39. Kiruvchi xabarlar `message_thread_id` ni o‘z ichiga olishi kerak (shaxsiy mavzu oqimi).
- 40. Guruhlar/superguruhlar/kanallar uchun oqim e’tiborga olinmaydi.

41. Konfiguratsiya:

- 42. `channels.telegram.streamMode: "off" | "partial" | "block"` (sukut bo‘yicha: `partial`)
  - 43. `partial`: qoralama pufakchasini eng so‘nggi oqim matni bilan yangilaydi.
  - 44. `block`: qoralama pufakchasini kattaroq bloklarda (bo‘laklab) yangilaydi.
  - 45. `off`: qoralama oqimini o‘chiradi.
- 46. Ixtiyoriy (faqat `streamMode: "block"` uchun):
  - 47. \`channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference?
    48. }`49. sukut bo‘yicha:`minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (`channels.telegram.textChunkLimit\` ga moslab cheklanadi).
    - 50. Eslatma: qoralama oqimi **blok oqimi** (kanal xabarlari) dan alohida.

Note: draft streaming is separate from **block streaming** (channel messages).1) Blokli oqim (block streaming) sukut bo‘yicha o‘chiq va `channels.telegram.blockStreaming: true` ni talab qiladi,
   agar qoralama yangilanishlari o‘rniga Telegram xabarlarini erta olishni istasangiz.

2. Mulohaza oqimi (faqat Telegram):

- 3. `/reasoning stream` javob yaratilayotgan paytda mulohazani qoralama pufagiga uzatadi, so‘ngra mulohazasiz yakuniy javobni yuboradi.
- Agar `channels.telegram.streamMode` `off` bo‘lsa, mantiqiy oqim (reasoning stream) o‘chiriladi.
  5. Batafsil kontekst: [Streaming + chunking](/concepts/streaming).

## 6. Qayta urinish siyosati

Chiqayotgan Telegram API chaqiruvlari vaqtinchalik tarmoq/429 xatolarida eksponensial backoff va jitter bilan qayta urinadi. 8. `channels.telegram.retry` orqali sozlang. 9. [Retry policy](/concepts/retry) ga qarang.

## 10. Agent vositasi (xabarlar + reaksiyalar)

- 11. Vositasi: `telegram` — `sendMessage` amali (`to`, `content`, ixtiyoriy `mediaUrl`, `replyToMessageId`, `messageThreadId`).
- 12. Vositasi: `telegram` — `react` amali (`chatId`, `messageId`, `emoji`).
- 13. Vositasi: `telegram` — `deleteMessage` amali (`chatId`, `messageId`).
- 14. Reaksiyani olib tashlash semantikasi: [/tools/reactions](/tools/reactions) ga qarang.
- 15. Vosita cheklovlari: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (sukut bo‘yicha: yoqilgan) va `channels.telegram.actions.sticker` (sukut bo‘yicha: o‘chiq).

## 16. Reaksiya bildirishnomalari

17. **Reaksiyalar qanday ishlaydi:**
    Telegram reaksiyalari **alohida `message_reaction` hodisalari** sifatida keladi, xabar payloadlaridagi xususiyatlar sifatida emas. 18. Foydalanuvchi reaksiya qo‘shganda, OpenClaw:

1. 19. Telegram API’dan `message_reaction` yangilanishini qabul qiladi
2. Uni quyidagi formatdagi **tizim hodisasi**ga aylantiradi: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. 21. Tizim hodisasini oddiy xabarlar bilan **bir xil sessiya kaliti** yordamida navbatga qo‘shadi
4. 22. Shu suhbatda keyingi xabar kelganda, tizim hodisalari bo‘shatiladi va agent kontekstining boshiga qo‘shiladi

23) Agent reaksiyalarni suhbat tarixida **tizim bildirishnomalari** sifatida ko‘radi, xabar metama’lumotlari sifatida emas.

24. **Sozlama:**

- 25. `channels.telegram.reactionNotifications`: Qaysi reaksiyalar bildirishnoma berishini boshqaradi
  - 26. `"off"` — barcha reaksiyalarni e’tiborsiz qoldirish
  - 27. `"own"` — foydalanuvchilar bot xabarlariga reaksiya bildirganda xabardor qilish (best-effort; xotirada) (sukut bo‘yicha)
  - 28. `"all"` — barcha reaksiyalar uchun xabardor qilish

- 29. `channels.telegram.reactionLevel`: agentning reaksiya bildirish imkoniyatini boshqaradi
  - 30. `"off"` — agent xabarlarga reaksiya bildira olmaydi
  - 31. `"ack"` — bot tasdiqlovchi reaksiyalar yuboradi (qayta ishlash paytida 👀) (sukut bo‘yicha)
  - 32. `"minimal"` — agent kamdan-kam reaksiya bildirishi mumkin (qoida: 5–10 almashinuvga 1 ta)
  - 33. `"extensive"` — mos bo‘lganda agent erkinroq reaksiya bildirishi mumkin

34. **Forum guruhlari:** Forum guruhlaridagi reaksiyalar `message_thread_id` ni o‘z ichiga oladi va `agent:main:telegram:group:{chatId}:topic:{threadId}` kabi sessiya kalitlaridan foydalanadi. 35. Bu bir xil mavzudagi reaksiyalar va xabarlar birga qolishini ta’minlaydi.

36. **Namunaviy sozlama:**

```json5
37. {
  channels: {
    telegram: {
      reactionNotifications: "all", // Barcha reaksiyalarni ko‘rish
      reactionLevel: "minimal", // Agent kamdan-kam reaksiya bildirishi mumkin
    },
  },
}
```

38. **Talablar:**

- 39. Telegram botlari `allowed_updates` ichida `message_reaction` ni aniq so‘rashi kerak (OpenClaw tomonidan avtomatik sozlanadi)
- 40. Webhook rejimida reaksiyalar webhook `allowed_updates` ichiga kiritiladi
- 41. Polling rejimida reaksiyalar `getUpdates` `allowed_updates` ichiga kiritiladi

## 42. Yetkazib berish manzillari (CLI/cron)

- 43. Maqsad sifatida chat id (`123456789`) yoki foydalanuvchi nomidan (`@name`) foydalaning.
- 44. Misol: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

## 45. Nosozliklarni bartaraf etish

46. **Bot guruhda eslatilmagan xabarlarga javob bermaydi:**

- 47. Agar `channels.telegram.groups.*.requireMention=false` ni o‘rnatsangiz, Telegram Bot API’ning **maxfiylik rejimi** o‘chirilgan bo‘lishi kerak.
  - 48. BotFather: `/setprivacy` → **Disable** (so‘ng botni guruhdan olib tashlab, qayta qo‘shing)
- 49. `openclaw channels status` sozlama eslatilmagan guruh xabarlarini kutganda ogohlantirish ko‘rsatadi.
- 50. `openclaw channels status --probe` qo‘shimcha ravishda aniq raqamli guruh ID’lari uchun a’zolikni tekshirishi mumkin (u \`
- Tezkor sinov: `/activation always` (faqat sessiya uchun; doimiylik uchun konfiguratsiyadan foydalaning)

**Bot umuman guruh xabarlarini ko‘rmayapti:**

- Agar `channels.telegram.groups` o‘rnatilgan bo‘lsa, guruh ro‘yxatda bo‘lishi yoki `"*"` ishlatilishi kerak
- @BotFather → "Group Privacy" bo‘limida Maxfiylik sozlamalarini tekshiring, u **OFF** bo‘lishi kerak
- Bot haqiqatan ham a’zo ekanini tekshiring (o‘qish huquqisiz faqat admin bo‘lib qolmagan bo‘lsin)
- Gateway loglarini tekshiring: `openclaw logs --follow` ("skipping group message" ni qidiring)

**Bot eslatmalarga javob beradi, lekin `/activation always` ga emas:**

- `/activation` buyrug‘i sessiya holatini yangilaydi, lekin konfiguratsiyaga saqlanmaydi
- Doimiy xatti-harakat uchun guruhni `channels.telegram.groups` ga `requireMention: false` bilan qo‘shing

**`/status` kabi buyruqlar ishlamayapti:**

- Telegram foydalanuvchi ID’ingiz avtorizatsiyadan o‘tganini tekshiring (juftlash yoki `channels.telegram.allowFrom` orqali)
- `groupPolicy: "open"` bo‘lgan guruhlarda ham buyruqlar avtorizatsiyani talab qiladi

**Node 22+ da long-polling darhol to‘xtab qoladi (ko‘pincha proksi/custom fetch bilan):**

- Node 22+ `AbortSignal` instansiyalariga qat’iyroq; begona signallar `fetch` chaqiruvlarini darhol bekor qilishi mumkin.
- Abort signallarini normallashtiradigan OpenClaw build’iga yangilang yoki yangilay olmaguningizcha gateway’ni Node 20’da ishga tushiring.

**Bot ishga tushadi, keyin jimgina javob berishni to‘xtatadi (yoki `HttpError: Network request ... failed`):**

- Ba’zi xostlar `api.telegram.org` ni avval IPv6 ga yechadi. Agar serveringizda ishlaydigan IPv6 chiqishi bo‘lmasa, grammY faqat IPv6 so‘rovlarida tiqilib qolishi mumkin.
- IPv6 chiqishini yoqish **yoki** `api.telegram.org` uchun IPv4 yechimini majburlash orqali tuzating (masalan, IPv4 A yozuvidan foydalanib `/etc/hosts` yozuvini qo‘shish yoki OS DNS stekida IPv4 ni ustun qilish), so‘ng gateway’ni qayta ishga tushiring.
- Tezkor tekshiruv: `dig +short api.telegram.org A` va `dig +short api.telegram.org AAAA` — DNS nimani qaytarayotganini tasdiqlash uchun.

## Konfiguratsiya ma’lumotnomasi (Telegram)

To‘liq konfiguratsiya: [Configuration](/gateway/configuration)

Provayder opsiyalari:

- `channels.telegram.enabled`: kanal ishga tushishini yoqish/o‘chirish.
- `channels.telegram.botToken`: bot tokeni (BotFather).
- `channels.telegram.tokenFile`: tokenni fayl yo‘lidan o‘qish.
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (standart: pairing).
- `channels.telegram.allowFrom`: DM ruxsat ro‘yxati (id/foydalanuvchi nomlari). `open` uchun `"*"` talab qilinadi.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (standart: allowlist).
- `channels.telegram.groupAllowFrom`: guruh jo‘natuvchilari uchun ruxsat ro‘yxati (id/foydalanuvchi nomlari).
- `channels.telegram.groups`: har bir guruh uchun standartlar + ruxsat ro‘yxati (`"*"` global standartlar uchun).
  - 39. **Bir dona WhatsApp raqamida bir nechta OpenClaw instansiyasidan foydalanish mumkinmi?**  
        Ha, har bir jo‘natuvchini `bindings` orqali turli agentlarga yo‘naltirish orqali (peer `kind: "direct"`, jo‘natuvchi E.164 masalan `+15551234567`)..groupPolicy`: groupPolicy uchun guruh bo‘yicha override (`open | allowlist | disabled\`).
  - `channels.telegram.groups.<id>.requireMention`: eslatma talabini boshqarish uchun standart.
  - `channels.telegram.groups.<id>.skills`: skill filtri (qoldirilsa = barcha skilllar, bo‘sh = hech biri).
  - `channels.telegram.groups.<id>.allowFrom`: guruh bo‘yicha jo‘natuvchilar ruxsat ro‘yxatini override qilish.
  - `channels.telegram.groups.<id>.systemPrompt`: guruh uchun qo‘shimcha system prompt.
  - `channels.telegram.groups.<id>.enabled`: `false` bo‘lsa, guruhni o‘chirish.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: mavzu bo‘yicha override’lar (guruh bilan bir xil maydonlar).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: groupPolicy uchun mavzu bo‘yicha override (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: har bir mavzu uchun mention (eslatib o‘tish) talabini alohida sozlash.
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (standart: allowlist).
- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: har bir akkaunt uchun alohida sozlama.
- `channels.telegram.replyToMode`: `off | first | all` (default: `first`).
- `channels.telegram.textChunkLimit`: chiqish xabarlari uchun bo‘lak hajmi (belgilar soni).
- `channels.telegram.chunkMode`: `length` (standart) yoki `newline` — uzunlik bo‘yicha bo‘lishdan oldin bo‘sh qatorlar (abzats chegaralari) bo‘yicha ajratish uchun.
- `channels.telegram.linkPreview`: chiqish xabarlari uchun havola prevyularini yoqish/o‘chirish (standart: true).
- `channels.telegram.streamMode`: `off | partial | block` (draft streaming).
- `channels.telegram.mediaMaxMb`: kiruvchi/chiquvchi media uchun maksimal hajm (MB).
- `channels.telegram.retry`: chiqish Telegram API chaqiruvlari uchun qayta urinish siyosati (urinishlar soni, minDelayMs, maxDelayMs, jitter).
- `channels.telegram.network.autoSelectFamily`: Node `autoSelectFamily` sozlamasini bekor qilish (true=yoqish, false=o‘chirish). Node 22 da Happy Eyeballs timeout muammolarini oldini olish uchun standart holatda o‘chirilgan.
- `channels.telegram.proxy`: Bot API chaqiruvlari uchun proksi URL (SOCKS/HTTP).
- `channels.telegram.webhookUrl`: enable webhook mode (requires `channels.telegram.webhookSecret`).
- `channels.telegram.webhookSecret`: webhook secret (required when webhookUrl is set).
- `channels.telegram.webhookPath`: local webhook path (default `/telegram-webhook`).
- `channels.telegram.actions.reactions`: gate Telegram tool reactions.
- `channels.telegram.actions.sendMessage`: gate Telegram tool message sends.
- `channels.telegram.actions.deleteMessage`: gate Telegram tool message deletes.
- `channels.telegram.actions.sticker`: gate Telegram sticker actions — send and search (default: false).
- `channels.telegram.reactionNotifications`: `off | own | all` — control which reactions trigger system events (default: `own` when not set).
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — control agent's reaction capability (default: `minimal` when not set).

Related global options:

- `agents.list[].groupChat.mentionPatterns` (mention gating patterns).
- `messages.groupChat.mentionPatterns` (global fallback).
- `commands.native` (defaults to `"auto"` → on for Telegram/Discord, off for Slack), `commands.text`, `commands.useAccessGroups` (command behavior). Override with `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.

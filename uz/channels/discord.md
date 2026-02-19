---
summary: "Discord bot qo‘llab-quvvatlash holati, imkoniyatlari va sozlamalari"
read_when:
  - Working on Discord channel features
title: "Discord"
---

# Discord (Bot API)

Status: ready for DM and guild text channels via the official Discord bot gateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DM’lari standart bo‘yicha pairing rejimiga o‘rnatilgan.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native buyruq xatti-harakati va buyruqlar katalogi.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallararo diagnostika va tuzatish jarayoni.
  
</Card>
</CardGroup>

## Tezkor sozlash

<Steps>
  <Step title="Create a Discord bot and enable intents">
    Discord Developer Portal’da ilova yarating, bot qo‘shing, so‘ng quyidagilarni yoqing:


    ```
    - **Message Content Intent**
    - **Server Members Intent** (rol allowlistlari va rolga asoslangan yo‘naltirish uchun talab qilinadi; nomdan ID’ga allowlist moslashtirish uchun tavsiya etiladi)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Standart akkaunt uchun muhit (env) fallback:
    ```

```bash
DISCORD_BOT_TOKEN=...
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    Botni serveringizga xabar yuborish ruxsatlari bilan taklif qiling.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Juftlash (pairing) kodlari 1 soatdan keyin muddati tugaydi.
    ```

  
</Step>
</Steps>

<Note>
Tokenni aniqlash hisobga (account) bog‘liq tarzda amalga oshiriladi. Config ichidagi token qiymatlari env fallback’dan ustun turadi. `DISCORD_BOT_TOKEN` faqat standart (default) hisob uchun ishlatiladi.
</Note>

## Runtime modeli

- Gateway Discord ulanishiga egalik qiladi.
- Javoblarni marshrutlash deterministik: Discord’dan kirgan javoblar yana Discord’ga qaytadi.
- Standart holatda (`session.dmScope=main`), to‘g‘ridan-to‘g‘ri chatlar agentning asosiy sessiyasi (`agent:main:main`) bilan bo‘lishiladi.
- Guild kanallari alohida sessiya kalitlariga ega (`agent:<agentId>:discord:channel:<channelId>`).
- Group DM’lar standart holatda e’tiborsiz qoldiriladi (`channels.discord.dm.groupEnabled=false`).
- Native slash buyruqlari alohida buyruq sessiyalarida ishlaydi (`agent:<agentId>:discord:slash:<userId>`), shu bilan birga marshrutlangan suhbat sessiyasiga `CommandTargetSessionKey` uzatiladi.

## Kirishni boshqarish va marshrutlash

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` DM kirishini boshqaradi (legacy: `channels.discord.dm.policy`):

    ```
    - `pairing` (standart)
    - `allowlist`
    - `open` (`channels.discord.allowFrom` ichida `"*"` bo‘lishi kerak; legacy: `channels.discord.dm.allowFrom`)
    - `disabled`
    
    Agar DM policy `open` bo‘lmasa, noma’lum foydalanuvchilar bloklanadi (yoki `pairing` rejimida juftlash so‘raladi).
    
    Yetkazib berish uchun DM target formati:
    
    - `user:<id>`
    - `<@id>` mention
    
    Faqat raqamli ID’lar noaniq hisoblanadi va aniq user/channel target turi ko‘rsatilmasa rad etiladi.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Guild boshqaruvi `channels.discord.groupPolicy` orqali nazorat qilinadi:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Agar `channels.discord` mavjud bo‘lsa, xavfsiz standart sozlama `allowlist` hisoblanadi.
    
    `allowlist` xatti-harakati:
    
    - guild `channels.discord.guilds` ga mos kelishi kerak (`id` afzal, slug ham qabul qilinadi)
    - ixtiyoriy yuboruvchi allowlist’lari: `users` (ID yoki nomlar) va `roles` (faqat role ID’lari); agar ulardan biri sozlangan bo‘lsa, yuboruvchi `users` YOKI `roles` ga mos kelsa ruxsat beriladi
    - agar guild’da `channels` sozlangan bo‘lsa, ro‘yxatda bo‘lmagan kanallar rad etiladi
    - agar guild’da `channels` bloki bo‘lmasa, allowlist qilingan guild’dagi barcha kanallarga ruxsat beriladi
    
    Misol:
    ```

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

    ```
    Agar siz faqat `DISCORD_BOT_TOKEN` ni sozlab, `channels.discord` blokini yaratmasangiz, runtime fallback `groupPolicy="open"` bo‘ladi (loglarda ogohlantirish bilan).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild xabarlari standart holatda mention orqali cheklanadi.

    ```
    Mention aniqlash quyidagilarni o‘z ichiga oladi:
    
    - bot’ni to‘g‘ridan-to‘g‘ri mention qilish
    - sozlangan mention pattern’lar (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - qo‘llab-quvvatlanadigan holatlarda botga javob berish (implicit reply-to-bot)
    
    `requireMention` har bir guild/channel uchun alohida sozlanadi (`channels.discord.guilds...`).
    
    Group DM’lar:
    
    - standart: e’tiborsiz qoldiriladi (`dm.groupEnabled=false`)
    - ixtiyoriy allowlist `dm.groupChannels` orqali (kanal ID yoki slug’lar)
    ```

  
</Tab>
</Tabs>

### Rolga asoslangan agent marshrutlash

Discord guild a’zolarini role ID orqali turli agentlarga marshrutlash uchun `bindings[].match.roles` dan foydalaning. Rolga asoslangan binding’lar faqat role ID’larini qabul qiladi va peer yoki parent-peer binding’lardan keyin, ammo faqat guild binding’laridan oldin baholanadi. Agar binding boshqa match maydonlarini ham belgilasa (masalan `peer` + `guildId` + `roles`), barcha ko‘rsatilgan maydonlar mos kelishi kerak.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Developer Portal sozlamalari

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. Bot tokenini nusxalang
    ```

  
</Accordion>

  <Accordion title="Privileged intents">
    **Bot -> Privileged Gateway Intents** bo‘limida quyidagilarni yoqing:

    ```
    - Message Content Intent
    - Server Members Intent (tavsiya etiladi)
    
    Presence intent ixtiyoriy va faqat presence yangilanishlarini olishni istasangiz kerak bo‘ladi. Bot presence’ini (`setPresence`) sozlash uchun a’zolar presence yangilanishlarini yoqish talab etilmaydi.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    OAuth URL generator:

    ```
    - scopes: `bot`, `applications.commands`
    
    Odatdagi asosiy ruxsatlar:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (ixtiyoriy)
    
    Alohida zarurat bo‘lmasa `Administrator` dan foydalanmang.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Discord Developer Mode’ni yoqing, so‘ng quyidagilarni nusxalang:

    ```
    - server ID
    - channel ID
    - user ID
    
    OpenClaw konfiguratsiyasida ishonchli audit va tekshiruvlar uchun raqamli ID’lardan foydalanish tavsiya etiladi.
    ```

  
</Accordion>
</AccordionGroup>

## Mahalliy buyruqlar va buyruq autentifikatsiyasi

- `commands.native` sukut bo‘yicha `"auto"` va Discord uchun yoqilgan.
- Har bir kanal uchun bekor qilish: `channels.discord.commands.native`.
- `commands.native=false` avval ro‘yxatdan o‘tkazilgan Discord mahalliy buyruqlarini aniq o‘chiradi.
- Mahalliy buyruq autentifikatsiyasi odatiy xabarlarni qayta ishlash bilan bir xil Discord allowlist/policy qoidalaridan foydalanadi.
- Buyruqlar Discord UI’da ruxsatsiz foydalanuvchilarga ko‘rinishi mumkin; bajarish jarayoni baribir OpenClaw autentifikatsiyasini qo‘llaydi va "not authorized" qaytaradi.

Buyruqlar katalogi va xatti-harakatlari uchun [Slash commands](/tools/slash-commands) ga qarang.

## Funksiya tafsilotlari

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord agent chiqishida javob teglarini qo‘llab-quvvatlaydi:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    `channels.discord.replyToMode` orqali boshqariladi:
    
    - `off` (sukut bo‘yicha)
    - `first`
    - `all`
    
    Eslatma: `off` yashirin javob tarmog‘ini o‘chiradi. Aniq `[[reply_to_*]]` teglari baribir inobatga olinadi.
    
    Xabar ID’lari agentlar aniq xabarlarni nishonga olishi uchun kontekst/tarixda ko‘rsatiladi.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild tarix konteksti:

    ```
    - `channels.discord.historyLimit` sukut bo‘yicha `20`
    - zaxira: `messages.groupChat.historyLimit`
    - `0` o‘chiradi
    
    DM tarix boshqaruvi:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Thread xatti-harakati:
    
    - Discord thread’lari kanal sessiyalari sifatida yo‘naltiriladi
    - ota thread metama’lumotlari ota-sessiya bog‘lanishi uchun ishlatilishi mumkin
    - agar thread uchun alohida sozlama bo‘lmasa, thread konfiguratsiyasi ota kanal konfiguratsiyasini meros qiladi
    
    Kanal mavzulari **ishonchsiz** kontekst sifatida kiritiladi (system prompt sifatida emas).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">
    Har bir guild uchun reaksiya bildirishnoma rejimi:

    ```
    - `off`
    - `own` (sukut bo‘yicha)
    - `all`
    - `allowlist` (`guilds.<id>.users` dan foydalanadi)
    
    Reaksiya hodisalari system hodisalariga aylantiriladi va yo‘naltirilgan Discord sessiyasiga biriktiriladi.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` OpenClaw kiruvchi xabarni qayta ishlayotgan paytda tasdiqlovchi emoji yuboradi.

    ```
    Ustuvorlik tartibi:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - agent identifikatsiya emoji zaxirasi (`agents.list[].identity.emoji`, aks holda "👀")
    
    Eslatmalar:
    
    - Discord unicode emoji yoki maxsus emoji nomlarini qabul qiladi.
    - Kanal yoki akkaunt uchun reaksiyani o‘chirish uchun `""` dan foydalaning.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Kanal tomonidan boshlangan konfiguratsiya yozuvlari sukut bo‘yicha yoqilgan.

    ```
    Bu `/config set|unset` oqimlariga ta’sir qiladi (buyruq funksiyalari yoqilgan bo‘lsa).
    
    O‘chirish:
    ```

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Discord gateway WebSocket trafigini `channels.discord.proxy` orqali HTTP(S) proksi orqali yo‘naltiring.

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

    ```
    Har bir akkaunt uchun bekor qilish:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    Proksilangan xabarlarni tizim a’zosi identifikatsiyasiga moslashtirish uchun PluralKit resolution’ni yoqing:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // ixtiyoriy; yopiq tizimlar uchun kerak
      },
    },
  },
}
```

    ```
    Eslatmalar:
    
    - allowlist’larda `pk:<memberId>` dan foydalanish mumkin
    - a’zo ko‘rsatish nomlari name/slug bo‘yicha moslashtiriladi
    - qidiruvlar asl xabar ID’sidan foydalanadi va vaqt oynasi bilan cheklangan
    - agar qidiruv muvaffaqiyatsiz bo‘lsa, proksilangan xabarlar bot xabarlari sifatida ko‘riladi va `allowBots=true` bo‘lmasa, bekor qilinadi
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Presence yangilanishlari faqat status yoki activity maydoni o‘rnatilganda qo‘llaniladi.

    ```
    Faqat status misoli:
    ```

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    ```
    Activity misoli (custom status sukut bo‘yicha activity turi):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Streaming misoli:
    ```

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    ```
    Activity turi xaritasi:
    
    - 0: Playing
    - 1: Streaming (`activityUrl` talab qilinadi)
    - 2: Listening
    - 3: Watching
    - 4: Custom (activity matnini status holati sifatida ishlatadi; emoji ixtiyoriy)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord DM’larda tugma asosidagi exec tasdiqlarini qo‘llab-quvvatlaydi va ixtiyoriy ravishda tasdiq so‘rovlarini yuborilgan kanalga joylashi mumkin.

    ```
    Konfiguratsiya yo‘li:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, sukut bo‘yicha: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    `target` `channel` yoki `both` bo‘lsa, tasdiq so‘rovi kanalda ko‘rinadi. Faqat sozlangan tasdiqlovchilar tugmalardan foydalana oladi; boshqa foydalanuvchilar vaqtinchalik rad javobini oladi. Tasdiq so‘rovlari buyruq matnini o‘z ichiga oladi, shuning uchun kanalga yetkazishni faqat ishonchli kanallarda yoqing. Agar kanal ID’sini sessiya kalitidan aniqlab bo‘lmasa, OpenClaw DM orqali yuborishga qaytadi.
    
    Agar tasdiqlar noma’lum tasdiq ID’lari bilan muvaffaqiyatsiz tugasa, tasdiqlovchilar ro‘yxati va funksiya yoqilganini tekshiring.
    
    Tegishli hujjatlar: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Vositalar va amal cheklovlari

Discord xabar amallari xabar yuborish, kanal administratsiyasi, moderatsiya, presence va metadata amallarini o‘z ichiga oladi.

Asosiy misollar:

- messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reaksiyalar: `react`, `reactions`, `emojiList`
- moderatsiya: `timeout`, `kick`, `ban`
- holat: `setPresence`

Action gate’lar `channels.discord.actions.*` ostida joylashgan.

Standart gate xatti-harakati:

| Amallar guruhi                                                                                                                                                           | Standart                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | yoqilgan                               |
| 45. rollar                                                                                                                                        | 12. oʻchirilgan |
| 46. moderatsiya                                                                                                                                   | 15. oʻchirilgan |
| 17. presence                                                                                                                                      | 18. oʻchirilgan |

## Components v2 UI

OpenClaw exec tasdiqlari va kontekstlararo belgilar uchun Discord components v2’dan foydalanadi. Discord xabar amallari maxsus UI uchun `components`ni ham qabul qilishi mumkin (kengaytirilgan; Carbon component instansiyalarini talab qiladi), legacy `embeds` esa mavjud, biroq tavsiya etilmaydi.

- `channels.discord.ui.components.accentColor` Discord komponent konteynerlarida ishlatiladigan aksent rangni (hex) belgilaydi.
- Har bir akkaunt uchun `channels.discord.accounts.<id>.ui.components.accentColor` orqali sozlanadi.
- Agar components v2 mavjud bo‘lsa, `embeds` e’tiborga olinmaydi.

Misol:

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## Ovozli xabarlar

Discord ovozli xabarlari to‘lqin shakli (waveform) ko‘rinishini ko‘rsatadi va OGG/Opus audio hamda metama’lumotlarni talab qiladi. OpenClaw to‘lqin shaklini avtomatik yaratadi, biroq audio fayllarni tekshirish va konvertatsiya qilish uchun gateway xostida `ffmpeg` va `ffprobe` mavjud bo‘lishi kerak.

Talablar va cheklovlar:

- **Lokal fayl yo‘li**ni taqdim eting (URL’lar rad etiladi).
- Matn kontentini kiritmang (Discord bir xil payload’da matn + ovozli xabarga ruxsat bermaydi).
- Istalgan audio formati qabul qilinadi; kerak bo‘lsa OpenClaw OGG/Opus’ga konvertatsiya qiladi.

Misol:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Birinchi qadam: `openclaw doctor` va `openclaw channels status --probe` ni ishga tushiring (amaliy ogohlantirishlar + tezkor auditlar).

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - Message Content Intent’ni yoqing
    - foydalanuvchi/a’zo aniqlashga bog‘liq bo‘lsangiz, Server Members Intent’ni yoqing
    - intent’larni o‘zgartirgandan so‘ng gateway’ni qayta ishga tushiring
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - `groupPolicy`ni tekshiring
    - `channels.discord.guilds` ostidagi guild allowlist’ni tekshiring
    - agar guild `channels` xaritasi mavjud bo‘lsa, faqat ro‘yxatdagi kanallarga ruxsat beriladi
    - `requireMention` xatti-harakati va mention andozalarini tekshiring
    
    Foydali tekshiruvlar:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Keng tarqalgan sabablar:

    ```
    - mos guild/channel allowlist bo‘lmagan holda `groupPolicy="allowlist"`
    - `requireMention` noto‘g‘ri joyda sozlangan (`channels.discord.guilds` yoki kanal yozuvi ostida bo‘lishi kerak)
    - yuboruvchi guild/channel `users` allowlist’i tomonidan bloklangan
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` ruxsat tekshiruvlari faqat raqamli kanal ID’lari uchun ishlaydi.

    ```
    Agar slug kalitlaridan foydalansangiz, runtime moslash ishlashi mumkin, biroq probe ruxsatlarni to‘liq tekshira olmaydi.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - DM o‘chirilgan: `channels.discord.dm.enabled=false`
    - DM siyosati o‘chirilgan: `channels.discord.dmPolicy="disabled"` (legacy: `channels.discord.dm.policy`)
    - `pairing` rejimida juftlash tasdig‘i kutilmoqda
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    Standart holatda bot tomonidan yozilgan xabarlar e’tiborga olinmaydi.

    ```
    Agar `channels.discord.allowBots=true` ni yoqsangiz, sikl (loop) xatti-harakatining oldini olish uchun qat’iy mention va allowlist qoidalaridan foydalaning.
    ```

  
</Accordion>
</AccordionGroup>

## Konfiguratsiya bo‘yicha havola ko‘rsatkichlari

Asosiy ma’lumotnoma:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Muhim Discord maydonlari:

- ishga tushirish/autentifikatsiya: `enabled`, `token`, `accounts.*`, `allowBots`
- siyosat: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- buyruq: `commands.native`, `commands.useAccessGroups`, `configWrites`
- javob/tarix: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- yetkazish: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/qayta urinish: `mediaMaxMb`, `retry`
- harakatlar: `actions.*`
- holat: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- xususiyatlar: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Xavfsizlik va operatsiyalar

- Bot tokenlarini maxfiy ma’lumot sifatida saqlang (`DISCORD_BOT_TOKEN` nazorat qilinadigan muhitlarda tavsiya etiladi).
- Eng kam zarur Discord ruxsatlarini bering.
- Agar buyruq deploy/state eskirgan bo‘lsa, gateway’ni qayta ishga tushiring va `openclaw channels status --probe` bilan yana tekshiring.

## Bog‘liq

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- [Slash commands](/tools/slash-commands)


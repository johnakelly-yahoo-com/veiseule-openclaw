---
summary: "WhatsApp (web channel) integration: login, inbox, replies, media, and ops"
read_when:
  - Working on WhatsApp/web channel behavior or inbox routing
title: "WhatsApp"
---

# WhatsApp (web channel)

Holat: WhatsApp Web (Baileys) orqali production-ready. Gateway bog‘langan sessiya(lar)ga egalik qiladi.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Standart DM siyosati noma’lum yuboruvchilar uchun pairing hisoblanadi.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallararo diagnostika va tuzatish bo‘yicha yo‘riqnomalar.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Kanal konfiguratsiyasining to‘liq andozalari va misollari.
  
</Card>
</CardGroup>

## Tezkor sozlash

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Muayyan akkaunt uchun:
    ```

```bash
4. {
  channels: { whatsapp: { configWrites: false } },
}
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    ```
    21. {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Step>
</Steps>

<Note>
OpenClaw imkon qadar WhatsApp’ni alohida raqamda ishga tushirishni tavsiya qiladi. (Kanal metama’lumotlari va onboarding jarayoni shu sozlama uchun optimallashtirilgan, ammo shaxsiy raqam bilan sozlash ham qo‘llab-quvvatlanadi.)
</Note>

## Joylashtirish andozalari

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Bu eng toza operatsion rejim:


    ````
    - OpenClaw uchun alohida WhatsApp identifikatori
    - DM allowlistlari va marshrutlash chegaralari aniqroq
    - o‘zingiz bilan chatda chalkashlik ehtimoli pastroq
    
    Minimal siyosat namunasi:
    
    ```json5
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Personal-number fallback">
    Onboarding shaxsiy raqam rejimini qo‘llab-quvvatlaydi va self-chat uchun mos bazaviy sozlamani yozadi:


    ```
    - `dmPolicy: "allowlist"`
    - `allowFrom` ichiga shaxsiy raqamingiz kiritiladi
    - `selfChatMode: true`
    
    Ish jarayonida self-chat himoyasi bog‘langan self raqam va `allowFrom` asosida ishlaydi.
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    Xabar almashish platformasi kanali joriy OpenClaw kanal arxitekturasida WhatsApp Web (`Baileys`) asosida ishlaydi.


    ```
    O‘rnatilgan chat-kanal reyestrida alohida Twilio WhatsApp xabar almashish kanali mavjud emas.
    ```

  
</Accordion>
</AccordionGroup>

## Inbound flow (DM + group)

- WhatsApp events come from `messages.upsert` (Baileys).
- Inbox listeners are detached on shutdown to avoid accumulating event handlers in tests/restarts.
- Status/broadcast chats are ignored.
- Direct chats use E.164; groups use group JID.
- **DM policy**: `channels.whatsapp.dmPolicy` controls direct chat access (default: `pairing`).

## Personal-number mode (fallback)

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` to‘g‘ridan-to‘g‘ri chat kirishini boshqaradi:


    ```
    - `pairing` (standart)
    - `allowlist`
    - `open` (`allowFrom` ichida `"*"` bo‘lishi kerak)
    - `disabled`
    
    `allowFrom` E.164 formatidagi raqamlarni qabul qiladi (ichki tarzda normallashtiriladi).
    
    Ko‘p akkauntli override: `channels.whatsapp.accounts.<id>.dmPolicy` (va `allowFrom`) shu akkaunt uchun kanal darajasidagi standart sozlamalardan ustun turadi.
    
    Runtime xulq-atvor tafsilotlari:
    
    - pairinglar kanal allow-store’da saqlanadi va sozlangan `allowFrom` bilan birlashtiriladi
    - agar allowlist sozlanmagan bo‘lsa, bog‘langan self raqam standart tarzda ruxsat etiladi
    - chiquvchi `fromMe` DM’lar hech qachon avtomatik pairing qilinmaydi
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Guruhga kirish ikki qatlamdan iborat:


    ```
    1. **Guruh a’zoligi allowlisti** (`channels.whatsapp.groups`)
       - agar `groups` ko‘rsatilmagan bo‘lsa, barcha guruhlar mos hisoblanadi
       - agar `groups` mavjud bo‘lsa, u guruh allowlisti sifatida ishlaydi (`"*"` ruxsat etiladi)
    
    2. **Guruh yuboruvchi siyosati** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: yuboruvchi allowlisti chetlab o‘tiladi
       - `allowlist`: yuboruvchi `groupAllowFrom` (yoki `*`) bilan mos kelishi kerak
       - `disabled`: guruhdan keluvchi barcha kiruvchi xabarlarni bloklaydi
    
    Yuboruvchi allowlisti fallback:
    
    - agar `groupAllowFrom` o‘rnatilmagan bo‘lsa, mavjud bo‘lganda runtime `allowFrom` ga qaytadi
    
    Eslatma: agar umuman `channels.whatsapp` bloki mavjud bo‘lmasa, runtime’dagi guruh siyosati fallback amalda `open` bo‘ladi.
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Guruh javoblari standart bo‘yicha mention talab qiladi.


    ```
    Mention aniqlash quyidagilarni o‘z ichiga oladi:
    
    - bot identifikatorining aniq WhatsApp mentionlari
    - sozlangan mention regex andozalari (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - botga javob berishni implicit aniqlash (reply yuboruvchisi bot identifikatoriga mos keladi)
    
    Sessiya darajasidagi faollashtirish buyrug‘i:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` sessiya holatini yangilaydi (global konfiguratsiyani emas). Bu owner tomonidan cheklangan.
    ```

  
</Tab>
</Tabs>

## Shaxsiy raqam va self-chat xulq-atvori

Bog‘langan self raqam `allowFrom` ichida ham mavjud bo‘lsa, WhatsApp self-chat himoyasi faollashadi:

- self-chat xabarlari uchun o‘qilganlik kvitansiyalarini o‘tkazib yuborish
- aks holda o‘zingizni ping qilishi mumkin bo‘lgan mention-JID avtomatik trigger xulq-atvorini e’tiborsiz qoldirish
- agar `messages.responsePrefix` o‘rnatilmagan bo‘lsa, self-chat javoblari standart tarzda `[{identity.name}]` yoki `[openclaw]` bilan boshlanadi

## Xabarni normallashtirish va kontekst

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Kiruvchi WhatsApp xabarlari umumiy inbound envelope ichiga o‘raladi.


    ````
    Agar iqtibosli javob mavjud bo‘lsa, kontekst quyidagi shaklda qo‘shiladi:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Mavjud bo‘lganda javob metama’lumot maydonlari ham to‘ldiriladi (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    Faqat media tarkibli kiruvchi xabarlar quyidagi kabi placeholder’lar bilan normallashtiriladi:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Joylashuv va kontakt payload’lari marshrutlashdan oldin matnli kontekstga normallashtiriladi.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Guruhlar uchun qayta ishlanmagan xabarlar buferga saqlanishi va bot oxir-oqibat ishga tushganda kontekst sifatida qo‘shilishi mumkin.

    ```
    - standart limit: `50`
    - sozlama: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` o‘chiradi
    
    Qo‘shish (injection) markerlari:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    Qabul qilingan kiruvchi WhatsApp xabarlari uchun o‘qilganlik tasdiqlari (read receipts) sukut bo‘yicha yoqilgan.

    ````
    Global darajada o‘chirish:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    Har bir akkaunt uchun alohida sozlash:
    
    ```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```
    
    Self-chat burilishlarida (turns) global yoqilgan bo‘lsa ham read receipt yuborilmaydi.
    ````

  
</Accordion>
</AccordionGroup>

## Yetkazib berish, bo‘laklash (chunking) va media

<AccordionGroup>
  <Accordion title="Text chunking">
    - standart bo‘lak limiti: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - `newline` rejimi avval paragraf chegaralarini (bo‘sh qatorlar) afzal ko‘radi, so‘ng uzunlik bo‘yicha xavfsiz bo‘laklashga o‘tadi
  
</Accordion>

  <Accordion title="Outbound media behavior">
    - rasm, video, audio (PTT voice-note) va hujjat payload’larini qo‘llab-quvvatlaydi
    - `audio/ogg` voice-note mosligi uchun `audio/ogg; codecs=opus` ga qayta yoziladi
    - animatsion GIF ijrosi video yuborishda `gifPlayback: true` orqali qo‘llab-quvvatlanadi
    - bir nechta media bilan javob yuborilganda caption birinchi media elementiga qo‘llanadi
    - media manbasi HTTP(S), `file://` yoki lokal yo‘llar bo‘lishi mumkin
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - kiruvchi media saqlash limiti: `channels.whatsapp.mediaMaxMb` (standart `50`)
    - avtomatik javoblar uchun chiquvchi media limiti: `agents.defaults.mediaMaxMb` (standart `5MB`)
    - rasmlar limitlarga moslash uchun avtomatik optimallashtiriladi (o‘lcham/sifat moslashtirish)
    - media yuborishda xatolik bo‘lsa, javobni jim tashlab yuborish o‘rniga birinchi element fallback sifatida ogohlantiruvchi matn yuboriladi
  
</Accordion>
</AccordionGroup>

## Tasdiqlovchi reaksiyalar

WhatsApp kiruvchi xabar qabul qilinganda darhol ack reaksiyalarini `channels.whatsapp.ackReaction` orqali qo‘llab-quvvatlaydi.

```json5
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

Xulq-atvor bo‘yicha eslatmalar:

- kiruvchi xabar qabul qilingach darhol (javobdan oldin) yuboriladi
- xatoliklar log qilinadi, ammo odatiy javob yetkazilishini to‘xtatmaydi
- `mentions` guruh rejimi mention-trigger qilingan burilishlarda reaksiya bildiradi; guruh aktivatsiyasi `always` bo‘lsa, bu tekshiruvni chetlab o‘tadi
- WhatsApp `channels.whatsapp.ackReaction` dan foydalanadi (legacy `messages.ackReaction` bu yerda ishlatilmaydi)

## Multi-account va credentials

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - akkaunt id’lari `channels.whatsapp.accounts` dan olinadi
    - standart akkaunt tanlovi: agar mavjud bo‘lsa `default`, aks holda sozlangan akkaunt id’laridan birinchisi (saralangan)
    - akkaunt id’lari qidiruv uchun ichki tarzda normallashtiriladi
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - joriy auth yo‘li: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - zaxira fayl: `creds.json.bak`
    - `~/.openclaw/credentials/` ichidagi legacy standart auth hanuz tan olinadi/migratsiya qilinadi (default-account oqimlari uchun)
  
</Accordion>

  <Accordion title="Logout behavior">
    `openclaw channels logout --channel whatsapp [--account <id>]` ushbu akkaunt uchun WhatsApp auth holatini tozalaydi.

    ```
    Legacy auth kataloglarida `oauth.json` saqlab qolinadi, Baileys auth fayllari esa o‘chiriladi.
    ```

  
</Accordion>
</AccordionGroup>

## Vositalar, amallar va config yozuvlari

- Agent vositalari WhatsApp reaksiya amalini (`react`) qo‘llab-quvvatlaydi.
- `[[audio_as_voice]]` WhatsApp uchun e’tiborga olinmaydi (audio allaqachon ovozli xabar sifatida yuboriladi).
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Channel tomonidan boshlangan config yozuvlari sukut bo‘yicha yoqilgan (`channels.whatsapp.configWrites=false` orqali o‘chiriladi).

## Media limitlari + optimallashtirish

<AccordionGroup>
  <Accordion title="Not linked (QR required)">
    Alomat: channel holati bog‘lanmagan deb ko‘rsatilmoqda.

    ````
    Yechim:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Alomat: ulangan akkauntda takroriy uzilishlar yoki qayta ulanish urinishlari kuzatilmoqda.

    ````
    Yechim:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    Zarur bo‘lsa, `channels login` orqali qayta ulang.
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">
    Maqsadli akkaunt uchun faol gateway listener mavjud bo‘lmasa, chiquvchi yuborishlar darhol xato bilan to‘xtaydi.

    ```
    Gateway ishlayotganiga va akkaunt ulanganiga ishonch hosil qiling.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Quyidagi tartibda tekshiring:

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` allowlist yozuvlari
    - mention gating (`requireMention` + mention andozalari)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway ishga tushirish muhiti Node’dan foydalanishi kerak. Bun barqaror WhatsApp/Telegram gateway ishlashi uchun mos emas deb belgilangan.
  
</Accordion>
</AccordionGroup>

## Konfiguratsiya ma’lumotnomasi havolalari

**Bun runtime**

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

WhatsApp uchun muhim maydonlar:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>.enabled`, `accounts.<id>.authDir`, account-level overrides
- operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- session xatti-harakati: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## Bog‘liq

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)

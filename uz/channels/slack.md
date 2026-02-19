---
summary: "Socket yoki HTTP webhook rejimi uchun Slack sozlamalari"
read_when:
  - "Holat: Slack ilova integratsiyalari orqali DM + kanallar uchun production-ready."
title: "Slack"
---

# Slack

Standart rejim — Socket Mode; HTTP Events API rejimi ham qo‘llab-quvvatlanadi.
Slack DM’lar sukut bo‘yicha pairing rejimida.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Native buyruq xatti-harakati va buyruqlar katalogi.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Kanallararo diagnostika va tuzatish playbook’lari.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">Tezkor sozlash
</Card>
</CardGroup>

## ```
    Slack ilovasi sozlamalarida:
```

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
    Env fallback (faqat default account uchun):
```

        ```
        SLACK_APP_TOKEN=xapp-...
        SLACK_BOT_TOKEN=xoxb-...
        ```

```bash
  
</Step>

  <Step title="Ilova hodisalariga obuna bo‘lish">
    Quyidagilar uchun bot hodisalariga obuna bo‘ling:

    - `app_mention`
    - `message.channels`, `message.groups`, `message.im`, `message.mpim`
    - `reaction_added`, `reaction_removed`
    - `member_joined_channel`, `member_left_channel`
    - `channel_rename`
    - `pin_added`, `pin_removed`

    Shuningdek, DM’lar uchun App Home **Messages Tab** ni yoqing.
  
</Step>

  <Step title="Gateway’ni ishga tushirish">
```

      openclaw gateway

```bash
  
</Step>
</Steps>
```

        
</Step>
      
        <Step title="Multi-account HTTP uchun noyob webhook yo‘llaridan foydalaning">
          Har bir account uchun HTTP rejimi qo‘llab-quvvatlanadi.
      
          Ro‘yxatdan o‘tishlar to‘qnashmasligi uchun har bir account’ga alohida `webhookPath` bering.
        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

        
</Tab>

  
</Tab>
</Tabs>

## Socket Mode uchun `botToken` + `appToken` talab qilinadi.

- HTTP rejimi `botToken` + `signingSecret` talab qiladi.
- Konfiguratsiyadagi tokenlar env fallback’dan ustun turadi.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback faqat default account uchun amal qiladi.
- `userToken` (`xoxp-...`) faqat konfiguratsiyada beriladi (env fallback yo‘q) va sukut bo‘yicha faqat o‘qish rejimida ishlaydi (`userTokenReadOnly: true`).
- `userToken` (`xoxp-...`) is config-only (no env fallback) and defaults to read-only behavior (`userTokenReadOnly: true`).
- Ixtiyoriy: agar chiqish xabarlari faol agent identifikatsiyasidan (maxsus `username` va ikonka) foydalanishini istasangiz, `chat:write.customize` ni qo‘shing. `icon_emoji` `:emoji_name:` sintaksisidan foydalanadi.

<Tip>
Amallar/katalog o‘qishlar uchun, sozlangan bo‘lsa, foydalanuvchi tokeni afzal ko‘rilishi mumkin. Yozish amallari uchun bot tokeni ustun hisoblanadi; foydalanuvchi tokeni orqali yozish faqat `userTokenReadOnly: false` bo‘lganda va bot tokeni mavjud bo‘lmaganda ruxsat etiladi.
</Tip>

## Kirish nazorati va marshrutlash

<Tabs>
  <Tab title="DM policy">
    `channels.slack.dmPolicy` DM kirishini boshqaradi (eski: `channels.slack.dm.policy`):

    ```
    - `pairing` (standart)
    - `allowlist`
    - `open` (`channels.slack.allowFrom` ichida `"*"` bo‘lishi talab qilinadi; eski: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM flaglari:
    
    - `dm.enabled` (standart true)
    - `channels.slack.allowFrom` (afzal)
    - `dm.allowFrom` (eski)
    - `dm.groupEnabled` (guruh DMlar uchun standart false)
    - `dm.groupChannels` (ixtiyoriy MPIM allowlist)
    
    DMlarda pairing `openclaw pairing approve slack <code>` orqali amalga oshiriladi.
    ```

  
</Tab>

  <Tab title="Channel policy">
    `channels.slack.groupPolicy` kanal bilan ishlashni boshqaradi:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Kanal allowlisti `channels.slack.channels` ostida joylashadi.
    
    Runtime eslatma: agar `channels.slack` butunlay mavjud bo‘lmasa (faqat env sozlamasi) va `channels.defaults.groupPolicy` o‘rnatilmagan bo‘lsa, runtime `groupPolicy="open"` ga qaytadi va ogohlantirishni log qiladi.
    
    Nom/ID aniqlash:
    
    - kanal allowlist yozuvlari va DM allowlist yozuvlari token ruxsati imkon berganda ishga tushishda aniqlanadi
    - aniqlanmagan yozuvlar sozlangandek saqlanadi
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Kanal xabarlari standart holatda mention orqali cheklanadi.

    ```
    Mention manbalari:
    
    - aniq app mention (`<@botId>`)
    - mention regex andozalari (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - botga javob berilgan thread xatti-harakati
    
    Har bir kanal uchun boshqaruvlar (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (allowlist)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>
</Tabs>

## OpenClaw konfiguratsiyasi (minimal)

- Slack uchun native command auto-mode **o‘chiq** (`commands.native: "auto"` Slack native commandlarni yoqmaydi).
- Native Slack command handlerlarini `channels.slack.commands.native: true` (yoki global `commands.native: true`) bilan yoqing.
- Native commandlar yoqilganda, Slack’da mos slash commandlarni (`/<command>` nomlari) ro‘yxatdan o‘tkazing.
- Agar native commandlar yoqilmagan bo‘lsa, `channels.slack.slashCommand` orqali bitta sozlangan slash commandni ishga tushirishingiz mumkin.

Standart slash command sozlamalari:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Slash sessiyalari alohida kalitlardan foydalanadi:

- `agent:<agentId>:slack:slash:<userId>`

va baribir buyruq bajarilishini maqsadli suhbat sessiyasiga (`CommandTargetSessionKey`) yo‘naltiradi.

## Scope’lar (joriy va ixtiyoriy)

- DMlar `direct` sifatida; kanallar `channel`; MPIMlar `group` sifatida yo‘naltiriladi.
- Standart `session.dmScope=main` bilan Slack DMlar agentning asosiy sessiyasiga birlashtiriladi.
- Kanal sessiyalari: `agent:<agentId>:slack:channel:<channelId>`.
- Thread javoblari tegishli holatda thread sessiya suffikslarini (`:thread:<threadTs>`) yaratishi mumkin.
- `channels.slack.thread.historyScope` standarti `thread`; `thread.inheritParent` standarti `false`.
- `channels.slack.thread.initialHistoryLimit` yangi thread sessiyasi boshlanganda mavjud thread xabarlaridan nechta yuklanishini boshqaradi (standart `20`; o‘chirish uchun `0` qo‘ying).

Javob threading boshqaruvlari:

- `chat:write` (`chat.postMessage` orqali xabar yuborish/yangilash/o‘chirish)
  [https://docs.slack.dev/reference/methods/chat.postMessage](https://docs.slack.dev/reference/methods/chat.postMessage)
- `im:write` (`conversations.open` orqali foydalanuvchi DM’larini ochish)
  [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
- `channels:history`, `groups:history`, `im:history`, `mpim:history`
  [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)

Qo‘lda reply teglar qo‘llab-quvvatlanadi:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Eslatma: `replyToMode="off"` implicit reply threadingni o‘chiradi. Aniq `[[reply_to_*]]` teglari baribir inobatga olinadi.

## Not needed today (but likely future)

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack fayl biriktirmalari Slack’da joylashtirilgan shaxsiy URLlardan (token-autentifikatsiyalangan so‘rov oqimi) yuklab olinadi va yuklab olish muvaffaqiyatli bo‘lsa hamda hajm cheklovlari ruxsat etsa media saqlash joyiga yoziladi.

    ```
    Runtime kiruvchi hajm cheklovi standart `20MB`, agar `channels.slack.mediaMaxMb` bilan o‘zgartirilmagan bo‘lsa.
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - matn bo‘laklari `channels.slack.textChunkLimit` dan foydalanadi (standart 4000)
    - `channels.slack.chunkMode="newline"` paragraf-birinchi bo‘lishni yoqadi
    - fayl yuborishlar Slack upload APIlaridan foydalanadi va thread javoblarini (`thread_ts`) o‘z ichiga olishi mumkin
    - chiquvchi media cheklovi sozlangan bo‘lsa `channels.slack.mediaMaxMb` ga amal qiladi; aks holda kanal yuborishlari media pipeline’dan MIME turiga oid standartlardan foydalanadi
  
</Accordion>

  <Accordion title="Delivery targets">
    Afzal aniq maqsadlar:

    ```
    - DMlar uchun `user:<id>`
    - Kanallar uchun `channel:<id>`
    
    Slack DMlar foydalanuvchi manzillariga yuborilganda Slack conversation APIlari orqali ochiladi.
    ```

  
</Accordion>
</AccordionGroup>

## Limits

Slack harakatlari `channels.slack.actions.*` orqali boshqariladi.

Joriy Slack vositalarida mavjud harakat guruhlari:

| Guruh       | Standart |
| ----------- | -------- |
| xabarlar    | yoqilgan |
| reaksiyalar | yoqilgan |
| pinlar      | yoqilgan |
| memberInfo  | yoqilgan |
| emojiList   | yoqilgan |

## Hodisalar va operatsion xatti-harakatlar

- Xabarni tahrirlash/o‘chirish/tredda e’lon qilish tizim hodisalariga moslashtiriladi.
- Reaksiyani qo‘shish/olib tashlash hodisalari tizim hodisalariga moslashtiriladi.
- A’zo qo‘shilishi/chiqishi, kanal yaratilishi/qayta nomlanishi va pin qo‘shish/olib tashlash hodisalari tizim hodisalariga moslashtiriladi.
- `channel_id_changed` `configWrites` yoqilgan bo‘lsa, kanal konfiguratsiya kalitlarini migratsiya qilishi mumkin.
- Kanal mavzusi/maqsadi metama’lumotlari ishonchsiz kontekst sifatida ko‘riladi va marshrutlash kontekstiga kiritilishi mumkin.

## Per-chat-type threading

You can configure different threading behavior per chat type by setting `channels.slack.replyToModeByChatType`:

Yechish tartibi:

- `channels.slack.accounts.<accountId> .ackReaction`
- `channels.slack.ackReaction`
- `messages.ackReaction`
- agent identity emoji fallback (`agents.list[].identity.emoji`, aks holda "👀")

Eslatmalar:

- Slack shortcodelarni kutadi (masalan, `"eyes"`).
- Kanal yoki akkaunt uchun reaksiyani o‘chirish uchun `""` dan foydalaning.

## Manifest va scope tekshiruv ro‘yxati

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "OpenClaw uchun Slack konnektori"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "OpenClaw’ga xabar yuborish",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    Agar siz `channels.slack.userToken` ni sozlasangiz, odatiy o‘qish scopelari quyidagilar:

    ```
    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (agar Slack qidiruv o‘qishlariga tayanadigan bo‘lsangiz)
    ```

  
</Accordion>
</AccordionGroup>

## Nosozliklarni bartaraf etish

<AccordionGroup>
  <Accordion title="No replies in channels">
    Tartib bilan tekshiring:

    ```
    - `groupPolicy`
    - kanal allowlisti (`channels.slack.channels`)
    - `requireMention`
    - har bir kanal uchun `users` allowlisti
    
    Foydali buyruqlar:
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Tekshiring:

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (yoki eski `channels.slack.dm.policy`)
    - juftlash tasdiqlari / allowlist yozuvlari
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Slack ilova sozlamalarida bot + app tokenlari va Socket Mode yoqilganini tekshiring.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Tekshiring:

    ```
    - signing secret
    - webhook yo‘li
    - Slack Request URLlari (Events + Interactivity + Slash Commands)
    - har bir HTTP akkaunt uchun noyob `webhookPath`
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Quyidagilardan qaysi biri mo‘ljallanganini tekshiring:

    ```
    - Slack’da mos slash buyruqlar ro‘yxatdan o‘tkazilgan holda native buyruq rejimi (`channels.slack.commands.native: true`)
    - yoki yagona slash buyruq rejimi (`channels.slack.slashCommand.enabled: true`)
    
    Shuningdek, `commands.useAccessGroups` va kanal/foydalanuvchi allowlistlarini tekshiring.
    ```

  
</Accordion>
</AccordionGroup>

## Asbob harakatlari

Slack asbob harakatlari `channels.slack.actions.*` orqali cheklanishi mumkin:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  Muhim Slack maydonlari:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM kirish: `dm.enabled`, `dmPolicy`, `allowFrom` (eski: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - kanalga kirish: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - tarmoqlash/tarix: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - yetkazib berish: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - operatsiyalar/xususiyatlar: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Xavfsizlik bo‘yicha eslatmalar

- Yozishlar sukut bo‘yicha bot tokenidan foydalanadi, shuning uchun holatni o‘zgartiruvchi harakatlar ilovaning bot ruxsatlari va identiteti doirasida qoladi.
- [Kanal yo‘naltirish](/channels/channel-routing)
- Agar foydalanuvchi-token yozishlarini yoqsangiz, foydalanuvchi tokenida kutilgan yozish scope’lari (`chat:write`, `reactions:write`, `pins:write`,
  `files:write`) mavjudligiga ishonch hosil qiling, aks holda bu amallar bajarilmaydi.
- [Konfiguratsiya](/gateway/configuration)
- [Slash buyruqlar](/tools/slash-commands)


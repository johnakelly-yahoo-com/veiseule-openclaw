---
summary: "imsg orqali legacy iMessage qo‘llab-quvvatlovi (stdio orqali JSON-RPC). Yangi sozlamalar BlueBubbles dan foydalanishi kerak."
read_when:
  - iMessage qo‘llab-quvvatlovini sozlash
  - Debugging iMessage send/receive
title: "iMessage"
---

# iMessage (eski: imsg)

<Warning>
Yangi iMessage o‘rnatishlari uchun <a href="/channels/bluebubbles">BlueBubbles</a> dan foydalaning.

`imsg` integratsiyasi eskirgan va kelajakdagi relizda olib tashlanishi mumkin. 
</Warning>

Holat: legacy tashqi CLI integratsiyasi. Gateway `imsg rpc` ni ishga tushiradi va stdio orqali JSON-RPC bilan aloqa qiladi (alohida daemon/port yo‘q).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Yangi sozlamalar uchun afzal iMessage yo‘li.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage DMs sukut bo‘yicha pairing rejimida.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    iMessage maydonlari bo‘yicha to‘liq ma’lumotnoma.
  
</Card>
</CardGroup>

## Tezkor sozlash

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="OpenClaw ni sozlash">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Gateway’ni ishga tushirish">

```bash
openclaw gateway
```

      {
        channels: { imessage: { configWrites: false } },
      }

```bash
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

        ```
            Pairing so‘rovlari 1 soatdan keyin muddati tugaydi.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw faqat stdio bilan mos keladigan `cliPath` ni talab qiladi, shuning uchun `cliPath` ni masofaviy Mac’ga SSH orqali ulanib `imsg` ni ishga tushiradigan wrapper skriptga yo‘naltirishingiz mumkin.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Ilovalar (attachments) yoqilganida tavsiya etiladigan konfiguratsiya:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // used for SCP attachment fetches
      includeAttachments: true,
    },
  },
}
```

    ```
    Agar `remoteHost` o‘rnatilmagan bo‘lsa, OpenClaw SSH wrapper skriptini tahlil qilish orqali uni avtomatik aniqlashga harakat qiladi.
    ```

  
</Tab>
</Tabs>

## Talablar va ruxsatlar (macOS)

- Xabarlar `imsg` ishlayotgan Mac’da tizimga kirilgan bo‘lishi kerak.
- OpenClaw/`imsg` ishlayotgan jarayon konteksti uchun Full Disk Access talab qilinadi (Messages DB kirishi).
- Messages.app orqali xabar yuborish uchun Automation ruxsati talab qilinadi.

<Tip>
Ruxsatlar har bir jarayon konteksti bo‘yicha beriladi. Agar gateway headless (LaunchAgent/SSH) rejimida ishlasa, so‘rovlarni ishga tushirish uchun aynan shu kontekstda bir martalik interaktiv buyruqni bajaring:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Kirishni boshqarish va marshrutlash

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` to‘g‘ridan-to‘g‘ri xabarlarni boshqaradi:

    ```
    - `pairing` (standart)
    - `allowlist`
    - `open` (`allowFrom` ichida `"*"` bo‘lishi talab qilinadi)
    - `disabled`
    
    Allowlist maydoni: `channels.imessage.allowFrom`.
    
    Allowlist yozuvlari handle yoki chat maqsadlari (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`) bo‘lishi mumkin.
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` guruhlarni boshqarishni nazorat qiladi:

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DMlar to‘g‘ridan-to‘g‘ri marshrutlashdan foydalanadi; guruhlar esa guruh marshrutlashdan foydalanadi.
    - Standart `session.dmScope=main` bilan iMessage DMlar agentning asosiy sessiyasiga birlashtiriladi.
    - Guruh sessiyalari izolyatsiya qilingan (`agent:<agentId> :imessage:group:<chat_id>`).
    - Javoblar iMessage’ga boshlang‘ich kanal/maqsad metama’lumotlari orqali qayta marshrutlanadi.

    ```
    Guruhga o‘xshash mavzu xatti-harakati:
    
    Ba’zi ko‘p ishtirokchili iMessage mavzulari `is_group=false` bilan kelishi mumkin.
    Agar o‘sha `chat_id` `channels.imessage.groups` ostida aniq sozlangan bo‘lsa, OpenClaw uni guruh trafigi sifatida qabul qiladi (guruh nazorati + guruh sessiyasini izolyatsiya qilish).
    ```

  
</Tab>
</Tabs>

## Joylashtirish (deployment) usullari

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    Bot trafigi shaxsiy Messages profilingizdan ajratilgan bo‘lishi uchun alohida Apple ID va macOS foydalanuvchisidan foydalaning.

    ```
    {
      channels: {
        imessage: {
          cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
          remoteHost: "user@gateway-host", // for SCP file transfer
          includeAttachments: true,
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Odatdagi topologiya:

    ```
    - gateway Linux/VM’da ishlaydi
    - iMessage + `imsg` tailnet’ingizdagi Mac’da ishlaydi
    - `cliPath` wrapper `imsg`ni ishga tushirish uchun SSH’dan foydalanadi
    - `remoteHost` SCP orqali biriktirmalarni yuklab olishni yoqadi
    
    Misol:
    ```

```json5
3. {
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
12. Telegram orqali juftlash (iOS uchun tavsiya etiladi)
```

    ```
    SSH kalitlaridan foydalaning, shunda SSH ham, SCP ham interaktiv bo‘lmaydi.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    iMessage `channels.imessage.accounts` ostida har bir akkaunt uchun alohida konfiguratsiyani qo‘llab-quvvatlaydi.

    ```
    5. #!/usr/bin/env bash
    exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
    ```

  
</Accordion>
</AccordionGroup>

## Media, bo‘lib yuborish (chunking) va yetkazib berish maqsadlari

<AccordionGroup>
  <Accordion title="Attachments and media">
    - kiruvchi biriktirmalarni qabul qilish ixtiyoriy: `channels.imessage.includeAttachments`
    - `remoteHost` o‘rnatilganda masofaviy biriktirma yo‘llarini SCP orqali yuklab olish mumkin
    - chiquvchi media hajmi `channels.imessage.mediaMaxMb` bilan belgilanadi (standart 16 MB)
  
</Accordion>

  <Accordion title="Outbound chunking">
    - matn bo‘lagi limiti: `channels.imessage.textChunkLimit` (standart 4000)
    - bo‘lish rejimi: `channels.imessage.chunkMode`
      - `length` (standart)
      - `newline` (avval paragraf bo‘yicha bo‘lish)
  
</Accordion>

  <Accordion title="Addressing formats">
    Afzal ko‘riladigan aniq maqsadlar:

    ```
    - `chat_id:123` (barqaror marshrutlash uchun tavsiya etiladi)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle maqsadlari ham qo‘llab-quvvatlanadi:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Konfiguratsiya yozuvlari

iMessage sukut bo‘yicha kanal tomonidan boshlangan konfiguratsiya yozuvlariga ruxsat beradi ( `/config set|unset` uchun, agar `commands.config: true` bo‘lsa ).

O‘chirish:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Muammolarni bartaraf etish

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Binary va RPC qo‘llab-quvvatlanishini tekshiring:

```bash
36. {
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { requireMention: false },
      },
    },
  },
}
```

    ```
    Agar probe RPC qo‘llab-quvvatlanmaydi deb xabar bersa, `imsg`ni yangilang.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Tekshiring:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - juftlash tasdiqlari (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">    Tekshiring:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - `channels.imessage.groups` allowlist xatti-harakati
    - eslatma (mention) namunasi sozlamasi (`agents.list[].groupChat.mentionPatterns`)
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">    Tekshiring:

    ```
    - `channels.imessage.remoteHost`
    - gateway xostidan SSH/SCP kalit autentifikatsiyasi
    - Messages ishlayotgan Mac’dagi masofaviy yo‘lning o‘qish uchun ochiqligi
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">    Xuddi shu foydalanuvchi/sessiya kontekstida interaktiv GUI terminalda qayta ishga tushiring va so‘rovlarni tasdiqlang:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    OpenClaw/`imsg` ishlayotgan jarayon konteksti uchun Full Disk Access + Automation ruxsatlari berilganini tasdiqlang.
    ```

  
</Accordion>
</AccordionGroup>

## Konfiguratsiya bo‘yicha ma’lumotnomalar

- [Configuration reference - iMessage](/gateway/configuration-reference#imessage)
- [Gateway configuration](/gateway/configuration)
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)


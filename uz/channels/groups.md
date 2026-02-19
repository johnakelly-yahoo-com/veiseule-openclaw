---
summary: "Turli platformalarda (WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams) guruh chat xatti-harakati"
read_when:
  - Changing group chat behavior or mention gating
title: "Guruhlar"
---

# Guruhlar

OpenClaw guruh chatlarini barcha platformalarda bir xil tarzda boshqaradi: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams.

## Boshlang‘ich kirish (2 daqiqa)

OpenClaw “lives” on your own messaging accounts. There is no separate WhatsApp bot user.
If **you** are in a group, OpenClaw can see that group and respond there.

Standart xatti-harakat:

- Guruhlar cheklangan (`groupPolicy: "allowlist"`).
- Javoblar uchun siz mention gating’ni ataylab o‘chirib qo‘ymaguningizcha, tilga olish (mention) talab qilinadi.

Tarjimasi: allowlist ro‘yxatidagi jo‘natuvchilar OpenClaw’ni tilga olib uni ishga tushirishi mumkin.

> TL;DR
>
> - **DM access** is controlled by `*.allowFrom`.
> - **Group access** is controlled by `*.groupPolicy` + allowlists (`*.groups`, `*.groupAllowFrom`).
> - **Reply triggering** is controlled by mention gating (`requireMention`, `/activation`).

Quick flow (what happens to a group message):

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Group message flow](/images/groups-flow.svg)

If you want...

| Goal                                                                | What to set                                                                          |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Allow all groups but only reply on @mentions           | `groups: { "*": { requireMention: true } }`                                          |
| Disable all group replies                                           | `groupPolicy: "disabled"`                                                            |
| Only specific groups                                                | `groups: { "<group-id>": { ... 1. } }` (`"*"` kalitisiz)          |
| 2. Guruhlarda faqat siz ishga tushira olasiz | 3. `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

## 4. Sessiya kalitlari

- 5. Guruh sessiyalari `agent:<agentId>:<channel>:group:<id>` sessiya kalitlaridan foydalanadi (xonalar/kanallar `agent:<agentId>:<channel>:channel:<id>` dan foydalanadi).
- 6. Telegram forum mavzulari guruh identifikatoriga `:topic:<threadId>` qoʻshadi, shuning uchun har bir mavzu alohida sessiyaga ega boʻladi.
- 7. Toʻgʻridan-toʻgʻri chatlar asosiy sessiyadan foydalanadi (yoki sozlangan boʻlsa, joʻnatuvchi boʻyicha alohida).
- 8. Yurak urishlari (heartbeats) guruh sessiyalari uchun oʻtkazib yuboriladi.

## 9. Andoza: shaxsiy DMlar + ommaviy guruhlar (bitta agent)

10. Ha — agar “shaxsiy” trafikingiz **DMlar** va “ommaviy” trafikingiz **guruhlar** boʻlsa, bu juda yaxshi ishlaydi.

11. Sabab: bitta-agent rejimida DMlar odatda **asosiy** sessiya kalitiga (`agent:main:main`) tushadi, guruhlar esa har doim **asosiy boʻlmagan** sessiya kalitlaridan foydalanadi (`agent:main:<channel>:group:<id>`). 12. Agar `mode: "non-main"` bilan sandboxingni yoqsangiz, guruh sessiyalari Docker’da ishlaydi, asosiy DM sessiyangiz esa xostda qoladi.

13. Bu sizga bitta agent “miyasi”ni (umumiy ish maydoni + xotira), ammo ikki xil bajarilish holatini beradi:

- Misol (DMlar xostda, guruhlar sandboxlangan + faqat xabar almashish vositalari):
- 15. **Guruhlar**: sandbox + cheklangan vositalar (Docker)

> 16. Agar haqiqatan ham alohida ish maydonlari/personalar kerak boʻlsa (“shaxsiy” va “ommaviy” hech qachon aralashmasligi kerak), ikkinchi agent + bogʻlashlardan foydalaning. 17. Qarang: [Multi-Agent Routing](/concepts/multi-agent).

18. Misol (DMlar xostda, guruhlar sandboxlangan + faqat xabar almashish vositalari):

```json5
22. {
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "~/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

20. “Guruhlar faqat X papkani koʻra olsin” deganini “xostga umuman kirish yoʻq” oʻrniga xohlaysizmi? 21. `workspaceAccess: "none"` ni saqlang va faqat ruxsat etilgan yoʻllarni sandboxga ulang:

```json5
22. {
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "~/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

23. Bogʻliq:

- 24. Sozlama kalitlari va standartlari: [Gateway configuration](/gateway/configuration#agentsdefaultssandbox)
- 25. Vosita nima uchun bloklanganini sozlash: [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated)
- 26. Bind mount tafsilotlari: [Sandboxing](/gateway/sandboxing#custom-bind-mounts)

## 27. Koʻrsatish yorliqlari

- Har bir kanal boʻyicha guruh/xona xabarlari qanday qayta ishlanishini boshqaring:
- 29. `#room` xonalar/kanallar uchun band qilingan; guruh chatlari `g-<slug>` dan foydalanadi (kichik harflar, bo‘shliqlar -> `-`, `#@+._-` saqlanadi).

## 30. Guruh siyosati

31. Har bir kanal boʻyicha guruh/xona xabarlari qanday qayta ishlanishini boshqaring:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // numeric Telegram user id (wizard can resolve @username)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| 33. Siyosat       | 34. Xulq-atvor                                                                                                                       |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 35. `"open"`      | 36. Guruhlar allowlistlarni chetlab oʻtadi; eslatma-cheklash (mention-gating) baribir qoʻllanadi. |
| 37. `"disabled"`  | 38. Barcha guruh xabarlarini butunlay bloklash.                                                                      |
| 39. `"allowlist"` | 40. Faqat sozlangan allowlistga mos keladigan guruh/xonalarga ruxsat berish.                                         |

Tezkor mental model (guruh xabarlari uchun baholash tartibi):

- 42. `groupPolicy` eslatma-cheklashdan alohida (u @eslatmalarni talab qiladi).
- guruh allowlistlari (`*.groups`, `*.groupAllowFrom`, kanalga xos allowlist)
- 44. Discord: allowlist `channels.discord.guilds.<id>` dan foydalanadi45. `.channels`.
- 46. Slack: allowlist `channels.slack.channels` dan foydalanadi.
- 47. Matrix: allowlist `channels.matrix.groups` dan foydalanadi (xona IDlari, aliaslar yoki nomlar). 48. Joʻnatuvchilarni cheklash uchun `channels.matrix.groupAllowFrom` dan foydalaning; xonaga xos `users` allowlistlari ham qoʻllab-quvvatlanadi.
- 49. Guruh DMlari alohida boshqariladi (`channels.discord.dm.*`, `channels.slack.dm.*`).
- 50. Telegram allowlist foydalanuvchi IDlarini (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) yoki foydalanuvchi nomlarini (`"@alice"` yoki `"alice"`) moslashtirishi mumkin; prefikslar katta-kichik harfga sezgir emas.
- Default is `groupPolicy: "allowlist"`; if your group allowlist is empty, group messages are blocked.

Tezkor mental model (guruh xabarlari uchun baholash tartibi):

1. `groupPolicy` (open/disabled/allowlist)
2. guruh allowlistlari (`*.groups`, `*.groupAllowFrom`, kanalga xos allowlist)
3. mention gating (`requireMention`, `/activation`)

## Mention gating (standart)

Guruh xabarlari, agar guruh bo‘yicha alohida o‘zgartirilmagan bo‘lsa, mention talab qiladi. Standartlar har bir subsistema uchun `*.groups."*"` ostida joylashgan.

Bot xabariga javob berish (kanal javob metamaʼlumotlarini qo‘llab-quvvatlasa) implicit mention hisoblanadi. Bu Telegram, WhatsApp, Slack, Discord va Microsoft Teams uchun amal qiladi.

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

Eslatmalar:

- `mentionPatterns` katta-kichik harfga sezgir bo‘lmagan regexlardir.
- Aniq mentionlarni taqdim etadigan interfeyslar baribir o‘tkaziladi; patternlar zaxira sifatida ishlatiladi.
- Agent bo‘yicha override: `agents.list[].groupChat.mentionPatterns` (bir nechta agentlar bitta guruhni bo‘lishganda foydali).
- Mention gating is only enforced when mention detection is possible (native mentions or `mentionPatterns` are configured).
- Discord standartlari `channels.discord.guilds."*"` da joylashgan (guild/kanal bo‘yicha o‘zgartirilishi mumkin).
- Guruh tarix konteksti barcha kanallarda bir xil tarzda o‘raladi va **faqat pending** (mention gating sababli o‘tkazib yuborilgan xabarlar); global standart uchun `messages.groupChat.historyLimit` dan foydalaning va override uchun `channels.<channel>`.historyLimit`(yoki`channels.<channel>`.accounts.*.historyLimit`) dan foydalaning. O‘chirish uchun `0` ni o‘rnating.

## Guruh/kanal asbob cheklovlari (ixtiyoriy)

Qaror chiqarish tartibi (eng aniq ustun):

- guruh/kanal `toolsBySender` mos kelishi
- `toolsBySender`: guruh ichida jo‘natuvchi bo‘yicha override (kalitlar kanalga qarab jo‘natuvchi IDlari/foydalanuvchi nomlari/email/telefon raqamlari bo‘lishi mumkin). Wildcard sifatida `"*"` dan foydalaning.

Misol (Telegram):

1. guruh/kanal `toolsBySender` mos kelishi
2. guruh/kanal `tools`
3. standart (`"*"`) `toolsBySender` mos kelishi
4. standart (`"*"`) `tools`

Eslatmalar:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

Eslatmalar:

- Guruh/kanal asbob cheklovlari global/agent asbob siyosatiga qo‘shimcha ravishda qo‘llanadi (deny baribir ustun).
- Baʼzi kanallar xonalar/kanallar uchun boshqa ichki tuzilmani ishlatadi (masalan, Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

## Guruh allowlistlari

`channels.whatsapp.groups`, `channels.telegram.groups` yoki `channels.imessage.groups` sozlanganda, kalitlar guruh allowlisti sifatida ishlaydi. Barcha guruhlarga ruxsat berib, shu bilan birga standart mention xatti-harakatini sozlash uchun `"*"` dan foydalaning.

Keng tarqalgan intentlar (copy/paste):

1. Disable all group replies

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2. Barcha guruhlarga ruxsat berish, lekin mention talab qilish (aniq)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3. Guruhlarda faqat egasi trigger qila oladi (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4. Guruhlarda faqat egasi trigger qila oladi (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## Faollashtirish (faqat egasi)

Guruh egalari har bir guruh bo‘yicha faollashtirishni yoqib/o‘chirishi mumkin:

- `/activation mention`
- `/activation always`

Ega `channels.whatsapp.allowFrom` orqali aniqlanadi (yoki sozlanmagan bo‘lsa, botning o‘z E.164 raqami). Buyruqni alohida xabar sifatida yuboring. Hozirda boshqa interfeyslar `/activation` ni e’tiborsiz qoldiradi.

## Kontekst maydonlari

Guruhdan keluvchi payloadlar quyidagilarni o‘rnatadi:

- `ChatType=group`
- `GroupSubject` (agar ma’lum bo‘lsa)
- `GroupMembers` (agar ma’lum bo‘lsa)
- `WasMentioned` (mention gating natijasi)
- Telegram forum mavzulari, shuningdek, `MessageThreadId` va `IsForum` ni ham o‘z ichiga oladi.

Agent tizim prompti yangi guruh sessiyasining birinchi navbatida guruh tanishtiruvini o‘z ichiga oladi. U modelga inson kabi javob berishni, Markdown jadvallaridan qochishni va literal `\n` ketma-ketliklarini yozmaslikni eslatadi.

## WhatsApp ga xos jihatlar

- Yo‘naltirish yoki ruxsat ro‘yxatiga olishda `chat_id:<id>` ni afzal ko‘ring.
- Chatlarni ro‘yxatlash: `imsg chats --limit 20`.
- Guruh javoblari har doim bir xil `chat_id` ga qaytadi.

## WhatsApp ga xos jihatlar

WhatsApp-only xatti-harakatlar (tarixni kiritish, mentionlarni qayta ishlash tafsilotlari) uchun [Group messages](/channels/group-messages) ga qarang.


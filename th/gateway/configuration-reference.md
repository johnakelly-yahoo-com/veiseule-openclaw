---
title: "เอกสารอ้างอิงการตั้งค่า"
description: "เอกสารอ้างอิงแบบครบถ้วนทุกฟิลด์สำหรับ ~/.openclaw/openclaw.json"
---

# เอกสารอ้างอิงการตั้งค่า

ทุกฟิลด์ที่มีอยู่ใน `~/.openclaw/openclaw.json` สำหรับภาพรวมที่เน้นงานเป็นหลัก โปรดดู [Configuration](/gateway/configuration)

รูปแบบไฟล์ตั้งค่าเป็น **JSON5** (อนุญาตให้มีคอมเมนต์และ comma ต่อท้ายได้) ทุกฟิลด์เป็นตัวเลือก — OpenClaw จะใช้ค่าเริ่มต้นที่ปลอดภัยเมื่อไม่ได้ระบุ

---

## ช่องทาง (Channels)

แต่ละช่องจะเริ่มทำงานโดยอัตโนมัติเมื่อมีส่วนการตั้งค่าของช่องนั้นอยู่ (ยกเว้นตั้งค่า `enabled: false`).

### การเข้าถึงแบบ DM และกลุ่ม

ทุกช่องรองรับนโยบาย DM และนโยบายกลุ่ม:

| นโยบาย DM                                  | พฤติกรรม                                                                                   |
| ------------------------------------------ | ------------------------------------------------------------------------------------------ |
| `pairing` (ค่าเริ่มต้น) | ผู้ส่งที่ไม่รู้จักจะได้รับโค้ดจับคู่แบบครั้งเดียว; เจ้าของต้องอนุมัติ                      |
| `allowlist`                                | อนุญาตเฉพาะผู้ส่งที่อยู่ใน `allowFrom` (หรือในรายการอนุญาตที่จับคู่ไว้) |
| `open`                                     | อนุญาต DM ขาเข้าทั้งหมด (ต้องตั้งค่า `allowFrom: ["*"]`)                |
| `disabled`                                 | ไม่รับ DM ขาเข้าทั้งหมด                                                                    |

| นโยบายกลุ่ม                                  | พฤติกรรม                                                                         |
| -------------------------------------------- | -------------------------------------------------------------------------------- |
| `allowlist` (ค่าเริ่มต้น) | เฉพาะกลุ่มที่ตรงกับ allowlist ที่กำหนดค่าไว้                                     |
| `open`                                       | ข้ามการตรวจสอบ allowlist ของกลุ่ม (ยังคงใช้การบังคับกล่าวถึง) |
| `disabled`                                   | บล็อกข้อความจากกลุ่ม/ห้องทั้งหมด                                                 |

<Note>
`channels.defaults.groupPolicy` ใช้กำหนดค่าเริ่มต้นเมื่อ `groupPolicy` ของผู้ให้บริการไม่ได้ตั้งค่า
โค้ดจับคู่จะหมดอายุภายใน 1 ชั่วโมง คำขอจับคู่ DM ที่รอดำเนินการจำกัดไว้ที่ **3 ต่อช่อง**
Slack/Discord มี fallback พิเศษ: หากไม่มีส่วนการตั้งค่าของผู้ให้บริการเลย นโยบายกลุ่มขณะรันไทม์อาจถูกกำหนดเป็น `open` (พร้อมคำเตือนตอนเริ่มต้นระบบ)
</Note>

### WhatsApp

WhatsApp ทำงานผ่าน web channel ของ gateway (Baileys Web) จะเริ่มทำงานอัตโนมัติเมื่อมีเซสชันที่เชื่อมโยงอยู่

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blue ticks (false in self-chat mode)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- คำสั่งขาออกจะใช้งานบัญชี `default` เป็นค่าเริ่มต้นหากมี; มิฉะนั้นจะใช้รหัสบัญชีแรกที่กำหนดค่าไว้ (เรียงลำดับแล้ว)
- ไดเรกทอรี auth แบบบัญชีเดียวของ Baileys รุ่นเก่าจะถูกย้ายโดย `openclaw doctor` ไปยัง `whatsapp/default`
- การกำหนดค่าแทนที่รายบัญชี: `channels.whatsapp.accounts.<id>`.sendReadReceipts`, `channels.whatsapp.accounts.<id>`.dmPolicy`, `channels.whatsapp.accounts.<id>`.allowFrom\`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Bot token: `channels.telegram.botToken` หรือ `channels.telegram.tokenFile` โดยใช้ `TELEGRAM_BOT_TOKEN` เป็นตัวเลือกสำรองสำหรับบัญชีค่าเริ่มต้น
- `configWrites: false` จะบล็อกการเขียนค่าคอนฟิกที่เริ่มจาก Telegram (การย้าย ID ของ supergroup, `/config set|unset`).
- ตัวอย่างพรีวิวสตรีมของ Telegram ใช้ `sendMessage` + `editMessageText` (ใช้ได้ทั้งในแชทส่วนตัวและแชทกลุ่ม)
- นโยบายการลองใหม่: ดูที่ [Retry policy](/concepts/retry)

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- โทเค็น: `channels.discord.token` โดยใช้ `DISCORD_BOT_TOKEN` เป็นค่า fallback สำหรับบัญชีเริ่มต้น
- ใช้ `user:<id>` (DM) หรือ `channel:<id>` (ช่องของ guild) สำหรับเป้าหมายการส่ง; ไม่รองรับ ID ตัวเลขล้วน
- slug ของ Guild ใช้ตัวพิมพ์เล็กและแทนที่ช่องว่างด้วย `-`; คีย์ของช่องใช้ชื่อที่ทำเป็น slug แล้ว (ไม่มี `#`) ควรใช้ Guild ID เป็นหลัก
- ข้อความที่บอทเป็นผู้ส่งจะถูกละเว้นโดยค่าเริ่มต้น `allowBots: true` จะเปิดใช้งานข้อความเหล่านั้น (แต่ยังคงกรองข้อความของบอทเอง)
- `maxLinesPerMessage` (ค่าเริ่มต้น 17) จะแบ่งข้อความที่ยาวเป็นหลายบรรทัด แม้จะไม่เกิน 2000 ตัวอักษร
- `channels.discord.ui.components.accentColor` กำหนดสีเน้นสำหรับคอนเทนเนอร์ Discord components v2

**โหมดการแจ้งเตือนรีแอคชัน:** `off` (ไม่มี), `own` (เฉพาะข้อความของบอท, ค่าเริ่มต้น), `all` (ทุกข้อความ), `allowlist` (จาก `guilds.<id>` `.users` สำหรับทุกข้อความ)

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- Service account JSON: แบบ inline (`serviceAccount`) หรือแบบไฟล์ (`serviceAccountFile`)
- ค่า fallback ของตัวแปรแวดล้อม: `GOOGLE_CHAT_SERVICE_ACCOUNT` หรือ `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`
- ใช้ `spaces/<spaceId>` หรือ `users/<userId|email>` สำหรับเป้าหมายการส่ง

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **โหมด Socket** ต้องใช้ทั้ง `botToken` และ `appToken` (ใช้ `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` เป็นค่า fallback ของบัญชีเริ่มต้นจาก env)
- **โหมด HTTP** ต้องใช้ `botToken` ร่วมกับ `signingSecret` (ที่ระดับ root หรือรายบัญชี)
- `configWrites: false` จะบล็อกการเขียนคอนฟิกที่เริ่มต้นจาก Slack
- ใช้ `user:<id>` (DM) หรือ `channel:<id>` สำหรับเป้าหมายการส่ง

**โหมดการแจ้งเตือนรีแอคชัน:** `off`, `own` (ค่าเริ่มต้น), `all`, `allowlist` (จาก `reactionAllowlist`)

**การแยก session ของเธรด:** `thread.historyScope` เป็นแบบต่อเธรด (ค่าเริ่มต้น) หรือแชร์ร่วมกันทั้งช่อง `thread.inheritParent` คัดลอกทรานสคริปต์ของช่องหลักไปยังเธรดใหม่

| กลุ่มการทำงาน | ค่าเริ่มต้น | หมายเหตุ                      |
| ------------- | ----------- | ----------------------------- |
| reactions     | เปิดใช้งาน  | โต้ตอบ + แสดงรายการรีแอคชัน   |
| messages      | เปิดใช้งาน  | อ่าน/ส่ง/แก้ไข/ลบ             |
| pins          | เปิดใช้งาน  | ปักหมุด/ยกเลิกหมุด/แสดงรายการ |
| memberInfo    | enabled     | ข้อมูลสมาชิก                  |
| emojiList     | enabled     | รายการอีโมจิที่กำหนดเอง       |

### Mattermost

Mattermost มาในรูปแบบปลั๊กอิน: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

โหมดแชท: `oncall` (ตอบกลับเมื่อมีการ @-mention, ค่าเริ่มต้น), `onmessage` (ทุกข้อความ), `onchar` (ข้อความที่ขึ้นต้นด้วยคำนำหน้าที่กำหนด).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**โหมดการแจ้งเตือนรีแอคชัน:** `off`, `own` (ค่าเริ่มต้น), `all`, `allowlist` (จาก `reactionAllowlist`).

### iMessage

OpenClaw จะเรียกใช้ `imsg rpc` (JSON-RPC ผ่าน stdio). ไม่ต้องใช้ daemon หรือพอร์ต

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- ต้องให้สิทธิ์ Full Disk Access กับฐานข้อมูล Messages
- แนะนำให้ใช้เป้าหมายรูปแบบ `chat_id:<id>` ใช้ `imsg chats --limit 20` เพื่อแสดงรายการแชท
- `cliPath` สามารถชี้ไปยัง SSH wrapper ได้; ตั้งค่า `remoteHost` สำหรับดึงไฟล์แนบผ่าน SCP

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### หลายบัญชี (ทุกช่องทาง)

เรียกใช้หลายบัญชีต่อหนึ่งช่องทางได้ (แต่ละบัญชีมี `accountId` ของตนเอง):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- ระบบจะใช้ `default` เมื่อไม่ได้ระบุ `accountId` (CLI + routing)
- Env tokens จะมีผลกับบัญชี **default** เท่านั้น
- การตั้งค่าพื้นฐานของช่องทางจะมีผลกับทุกบัญชี เว้นแต่จะกำหนดทับในระดับบัญชี
- ใช้ `bindings[].match.accountId` เพื่อกำหนดเส้นทางแต่ละบัญชีไปยังเอเจนต์ที่แตกต่างกัน

### การบังคับให้กล่าวถึงในแชทกลุ่ม

ข้อความในกลุ่มจะตั้งค่าเริ่มต้นเป็น **ต้องมีการกล่าวถึง** (การกล่าวถึงใน metadata หรือรูปแบบ regex) ใช้กับแชทกลุ่มของ WhatsApp, Telegram, Discord, Google Chat และ iMessage

**ประเภทของการกล่าวถึง:**

- **การกล่าวถึงใน Metadata**: การ @-mention แบบเนทีฟของแพลตฟอร์ม จะถูกละเว้นในโหมดแชทกับตนเองของ WhatsApp
- **รูปแบบข้อความ**: รูปแบบ regex ใน `agents.list[].groupChat.mentionPatterns` จะถูกตรวจสอบเสมอ
- การบังคับให้กล่าวถึงจะถูกใช้เฉพาะเมื่อสามารถตรวจจับได้ (มีการกล่าวถึงแบบเนทีฟ หรือมีอย่างน้อยหนึ่งรูปแบบ)

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` กำหนดค่าเริ่มต้นแบบโกลบอล Channels สามารถกำหนดค่า override ด้วย `channels.<channel>` `.historyLimit` (หรือกำหนดต่อบัญชีได้) ตั้งค่าเป็น `0` เพื่อปิดการใช้งาน

#### ขีดจำกัดประวัติ DM

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

ลำดับการพิจารณา: override ต่อ DM → ค่าเริ่มต้นของผู้ให้บริการ → ไม่จำกัด (เก็บทั้งหมด)

รองรับ: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### โหมดแชทกับตัวเอง

เพิ่มหมายเลขของคุณใน `allowFrom` เพื่อเปิดใช้งานโหมดแชทกับตัวเอง (จะไม่สนใจ @-mentions แบบ native และตอบกลับเฉพาะข้อความที่ตรงกับรูปแบบที่กำหนด):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### คำสั่ง (การจัดการคำสั่งในแชท)

```json5
{
  commands: {
    native: "auto", // ลงทะเบียนคำสั่งแบบ native เมื่อรองรับ
    text: true, // แยกวิเคราะห์ /commands ในข้อความแชท
    bash: false, // อนุญาต ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // อนุญาต /config
    debug: false, // อนุญาต /debug
    restart: false, // อนุญาต /restart + เครื่องมือรีสตาร์ท gateway
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- คำสั่งแบบข้อความต้องเป็นข้อความแบบ **เดี่ยวเท่านั้น** และขึ้นต้นด้วย `/`
- `native: "auto"` จะเปิดใช้คำสั่งแบบ native สำหรับ Discord/Telegram และปิดไว้สำหรับ Slack
- กำหนดค่า override รายช่องทางได้ที่: `channels.discord.commands.native` (ค่าเป็น bool หรือ `"auto"`) `false` จะล้างคำสั่งที่เคยลงทะเบียนไว้ก่อนหน้า
- `channels.telegram.customCommands` ใช้เพิ่มรายการเมนูบอท Telegram เพิ่มเติม
- `bash: true` เปิดใช้งาน `! <cmd>` สำหรับเชลล์ของโฮสต์ ต้องตั้งค่า `tools.elevated.enabled` และผู้ส่งต้องอยู่ใน \`tools.elevated.allowFrom.<channel>\`\`.
- `config: true` เปิดใช้งาน `/config` (อ่าน/เขียน `openclaw.json`)
- `channels.<provider>`.configWrites\` ใช้ควบคุมการแก้ไข config แยกตามแต่ละช่องทาง (ค่าเริ่มต้น: true)
- `allowFrom` กำหนดแยกตามผู้ให้บริการ เมื่อกำหนดค่าแล้ว จะถือเป็นแหล่งการยืนยันสิทธิ์ **เพียงแหล่งเดียว** (จะไม่ใช้ allowlists/pairing ของช่องทาง และ `useAccessGroups`)
- `useAccessGroups: false` อนุญาตให้คำสั่งข้ามนโยบาย access-group ได้เมื่อไม่ได้ตั้งค่า `allowFrom`

</Accordion>

---

## ค่าเริ่มต้นของเอเจนต์

### `agents.defaults.workspace`

ค่าเริ่มต้น: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

รากของ repository แบบกำหนดเอง ซึ่งจะแสดงในบรรทัด Runtime ของ system prompt หากไม่ได้ตั้งค่า OpenClaw จะตรวจจับอัตโนมัติโดยไล่ค้นหาขึ้นไปจาก workspace

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

ปิดการสร้างไฟล์ bootstrap ของ workspace โดยอัตโนมัติ (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`)

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

จำนวนอักขระสูงสุดต่อไฟล์ bootstrap ของ workspace ก่อนถูกตัดทอน ค่าเริ่มต้น: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

จำนวนอักขระรวมสูงสุดที่แทรกจากไฟล์ bootstrap ทั้งหมดของ workspace ค่าเริ่มต้น: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

โซนเวลาสำหรับบริบทของ system prompt (ไม่ใช่เวลาของข้อความ) หากไม่กำหนด จะใช้โซนเวลาของโฮสต์แทน

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

รูปแบบเวลาที่ใช้ใน system prompt ค่าเริ่มต้น: `auto` (ตามค่าที่ระบบปฏิบัติการกำหนด)

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: รูปแบบ `provider/model` (เช่น `anthropic/claude-opus-4-6`). หากไม่ระบุ provider, OpenClaw จะถือว่าเป็น `anthropic` (เลิกใช้งานแล้ว)
- `models`: แคตตาล็อกโมเดลและ allowlist ที่กำหนดไว้สำหรับ `/model` แต่ละรายการสามารถมี `alias` (ชื่อย่อ) และ `params` (เฉพาะของ provider: `temperature`, `maxTokens`)
- `imageModel`: ใช้เฉพาะเมื่อโมเดลหลักไม่รองรับอินพุตรูปภาพ
- `maxConcurrent`: จำนวนสูงสุดของการรันเอเจนต์แบบขนานในทุกเซสชัน (แต่ละเซสชันยังคงทำงานแบบลำดับ) ค่าเริ่มต้น: 1.

**ชื่อย่อ alias ที่มีมาให้ในตัว** (ใช้ได้เฉพาะเมื่อโมเดลอยู่ใน `agents.defaults.models`):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

alias ที่คุณกำหนดเองจะมีลำดับความสำคัญเหนือค่าดีฟอลต์เสมอ

โมเดล Z.AI GLM-4.x จะเปิดโหมด thinking โดยอัตโนมัติ เว้นแต่คุณจะตั้งค่า `--thinking off` หรือกำหนด `agents.defaults.models["zai/<model>"].params.thinking` เอง

### `agents.defaults.cliBackends`

แบ็กเอนด์ CLI แบบทางเลือกสำหรับการรันสำรองเฉพาะข้อความ (ไม่มีการเรียกใช้เครื่องมือ) มีประโยชน์เป็นตัวสำรองเมื่อผู้ให้บริการ API ขัดข้อง

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- แบ็กเอนด์ CLI เน้นข้อความเป็นหลัก; เครื่องมือจะถูกปิดใช้งานเสมอ
- รองรับเซสชันเมื่อมีการตั้งค่า `sessionArg`
- รองรับการส่งต่อรูปภาพเมื่อ `imageArg` รับพาธไฟล์

### `agents.defaults.heartbeat`

การรัน heartbeat เป็นระยะ

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: สตริงระยะเวลา (ms/s/m/h) ค่าเริ่มต้น: `30m`
- ต่อเอเจนต์: ตั้งค่า `agents.list[].heartbeat` เมื่อมีเอเจนต์ใดกำหนด `heartbeat` จะมี **เฉพาะเอเจนต์เหล่านั้นเท่านั้น** ที่รัน heartbeat
- Heartbeat จะรันหนึ่งรอบการทำงานเต็มของเอเจนต์ — ช่วงเวลาที่สั้นลงจะใช้โทเค็นมากขึ้น

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`: `default` หรือ `safeguard` (การสรุปแบบแบ่งเป็นช่วงสำหรับประวัติที่ยาว) ดู [Compaction](/concepts/compaction)
- `memoryFlush`: รอบการทำงานแบบเอเจนต์ที่ทำงานเงียบก่อนการบีบอัดอัตโนมัติ เพื่อจัดเก็บความทรงจำถาวร จะถูกข้ามเมื่อเวิร์กสเปซเป็นแบบอ่านอย่างเดียว

### `agents.defaults.contextPruning`

ตัดแต่ง **ผลลัพธ์เครื่องมือเก่า** ออกจากบริบทในหน่วยความจำก่อนส่งไปยัง LLM **ไม่** แก้ไขประวัติเซสชันที่จัดเก็บบนดิสก์

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` เปิดใช้งานการตัดแต่ง
- `ttl` ควบคุมความถี่ที่สามารถรันการตัดแต่งอีกครั้งได้ (หลังจากการใช้งานแคชครั้งล่าสุด)
- การตัดแต่งจะทำ soft-trim กับผลลัพธ์เครื่องมือที่มีขนาดใหญ่เกินก่อน จากนั้นจึง hard-clear ผลลัพธ์เครื่องมือที่เก่ากว่าหากจำเป็น

**Soft-trim** จะคงส่วนต้น + ส่วนท้ายไว้ และแทรก `...` ไว้ตรงกลาง

**Hard-clear** จะแทนที่ผลลัพธ์เครื่องมือทั้งหมดด้วยข้อความ placeholder

หมายเหตุ:

- บล็อกรูปภาพจะไม่ถูกตัดแต่ง/ลบล้าง
- อัตราส่วนคำนวณจากจำนวนอักขระ (โดยประมาณ) ไม่ใช่จำนวนโทเค็นที่แน่นอน
- หากมีข้อความจากผู้ช่วยน้อยกว่า `keepLastAssistants` จะข้ามการตัดแต่ง

</Accordion>

ดู [Session Pruning](/concepts/session-pruning) สำหรับรายละเอียดพฤติกรรม

### บล็อกการสตรีม

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- ช่องทางที่ไม่ใช่ Telegram ต้องกำหนด `*.blockStreaming: true` อย่างชัดเจนเพื่อเปิดใช้งานการตอบกลับแบบบล็อก
- การกำหนดค่าสำหรับแต่ละช่องทาง: `channels.<channel>``.blockStreamingCoalesce` (รวมถึงตัวแปรต่อบัญชี) ค่าเริ่มต้น `minChars: 1500` สำหรับ Signal/Slack/Discord/Google Chat
- `humanDelay`: การหน่วงเวลาแบบสุ่มระหว่างการตอบกลับแต่ละบล็อก `natural` = 800–2500ms. การตั้งค่าแทนที่ต่อเอเจนต์: `agents.list[].humanDelay`.

ดู [Streaming](/concepts/streaming) สำหรับรายละเอียดพฤติกรรมและการแบ่งข้อความ (chunking)

### ตัวบ่งชี้การพิมพ์

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- ค่าเริ่มต้น: `instant` สำหรับแชทส่วนตัว/การ mention และ `message` สำหรับแชทกลุ่มที่ไม่มีการ mention
- การตั้งค่าแทนที่ต่อเซสชัน: `session.typingMode`, `session.typingIntervalSeconds`.

ดู [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

ตัวเลือก **Docker sandboxing** สำหรับเอเจนต์แบบฝังในระบบ ดู [Sandboxing](/gateway/sandboxing) สำหรับคู่มือฉบับเต็ม

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

<Accordion title="Sandbox details">

**การเข้าถึง Workspace:**

- `none`: sandbox workspace ตามขอบเขต (scope) ภายใต้ `~/.openclaw/sandboxes`
- `ro`: sandbox workspace ที่ `/workspace` และเมานต์ agent workspace แบบอ่านอย่างเดียวที่ `/agent`
- `rw`: เมานต์ agent workspace แบบอ่าน/เขียนที่ `/workspace`

**ขอบเขต (Scope):**

- `session`: คอนเทนเนอร์และ workspace แยกต่อเซสชัน
- `agent`: หนึ่งคอนเทนเนอร์และ workspace ต่อเอเจนต์ (ค่าเริ่มต้น)
- `shared`: ใช้คอนเทนเนอร์และ workspace ร่วมกัน (ไม่มีการแยกข้ามเซสชัน)

**`setupCommand`** จะทำงานหนึ่งครั้งหลังจากสร้างคอนเทนเนอร์ (ผ่าน `sh -lc`) ต้องการ network egress, root ที่เขียนได้, และผู้ใช้ root

**ค่าเริ่มต้นของคอนเทนเนอร์คือ `network: "none"`** — ตั้งค่าเป็น `"bridge"` หากเอเจนต์ต้องการการเข้าถึงภายนอก

**ไฟล์แนบขาเข้า (Inbound attachments)** จะถูกจัดเก็บไว้ที่ `media/inbound/*` ใน workspace ที่กำลังใช้งาน

**`docker.binds`** ใช้เมานต์ไดเรกทอรีจากโฮสต์เพิ่มเติม; การตั้งค่าระดับ global และต่อเอเจนต์จะถูกรวมกัน

**Sandboxed browser** (`sandbox.browser.enabled`): Chromium + CDP ภายในคอนเทนเนอร์ URL ของ noVNC จะถูกแทรกเข้าไปใน system prompt ไม่จำเป็นต้องเปิด `browser.enabled` ในการตั้งค่าหลัก

- `allowHostControl: false` (ค่าเริ่มต้น) จะบล็อกไม่ให้เซสชันใน sandbox ควบคุมเบราว์เซอร์ของโฮสต์
- `sandbox.browser.binds` เมานต์ไดเรกทอรีโฮสต์เพิ่มเติมเข้าเฉพาะคอนเทนเนอร์ sandbox browser เมื่อกำหนดค่า (รวมถึง `[]`) จะใช้แทนที่ `docker.binds` สำหรับคอนเทนเนอร์เบราว์เซอร์

</Accordion>

สร้างอิมเมจ:

```bash
scripts/sandbox-setup.sh           # main sandbox image
scripts/sandbox-browser-setup.sh   # optional browser image
```

### `agents.list` (การตั้งค่าแทนที่ต่อเอเจนต์)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // or { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: รหัสเอเจนต์แบบคงที่ (จำเป็น)
- `default`: หากตั้งค่าหลายรายการ จะใช้รายการแรก (มีการบันทึกคำเตือน) หากไม่ตั้งค่าใด ๆ จะใช้รายการแรกใน list เป็นค่าเริ่มต้น
- `model`: รูปแบบ string จะเขียนทับเฉพาะ `primary`; รูปแบบ object `{ primary, fallbacks }` จะเขียนทับทั้งสองค่า (`[]` จะปิดการใช้ global fallbacks)
- `identity.avatar`: พาธแบบอ้างอิงจาก workspace, URL แบบ `http(s)`, หรือ URI แบบ `data:`
- `identity` จะสร้างค่าเริ่มต้นอัตโนมัติ: `ackReaction` จาก `emoji`, `mentionPatterns` จาก `name`/`emoji`
- `subagents.allowAgents`: allowlist ของ agent id สำหรับ `sessions_spawn` (`["*"]` = ได้ทุกตัว; ค่าเริ่มต้น: เฉพาะ agent เดียวกัน)

---

## การกำหนดเส้นทางแบบหลายเอเจนต์

รันหลายเอเจนต์ที่แยกกันอย่างอิสระภายใน Gateway เดียว ดู [Multi-Agent](/concepts/multi-agent)

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

### ฟิลด์การจับคู่ของ Binding

- `match.channel` (จำเป็น)
- `match.accountId` (ไม่บังคับ; `*` = ทุกบัญชี; ไม่ระบุ = บัญชีค่าเริ่มต้น)
- `match.peer` (ไม่บังคับ; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (ไม่บังคับ; เฉพาะบาง channel)

**ลำดับการจับคู่แบบกำหนดแน่นอน (Deterministic):**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (ตรงกันแบบ exact, ไม่มี peer/guild/team)
5. `match.accountId: "*"` (ครอบคลุมทั้ง channel)
6. เอเจนต์เริ่มต้น

ภายในแต่ละระดับ รายการ `bindings` ที่ตรงเงื่อนไขรายการแรกจะถูกเลือก

### โปรไฟล์การเข้าถึงแยกตามเอเจนต์

<Accordion title="Full access (no sandbox)">

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

</Accordion>

ดู [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) สำหรับรายละเอียดลำดับความสำคัญ

---

## Session

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // legacy (runtime always uses "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: วิธีการจัดกลุ่ม DM
  - `main`: DM ทั้งหมดใช้ session หลักร่วมกัน
  - `per-peer`: แยกตาม sender id ข้ามทุก channel
  - `per-channel-peer`: แยกตาม channel + sender (แนะนำสำหรับ inbox ที่มีผู้ใช้หลายคน)
  - `per-account-channel-peer`: แยกตาม account + channel + sender (แนะนำสำหรับหลายบัญชี)
- **`identityLinks`**: แมป canonical id กับ peer ที่มี prefix ของผู้ให้บริการ เพื่อแชร์ session ข้าม channel
- **`reset`**: นโยบายรีเซ็ตหลัก `daily` จะรีเซ็ตตามเวลา `atHour` ท้องถิ่น; `idle` จะรีเซ็ตหลังจาก `idleMinutes` หากตั้งค่าทั้งสองแบบ ระบบจะยึดตามเงื่อนไขที่หมดอายุก่อน
- **`resetByType`**: การตั้งค่าแทนที่แยกตามประเภท (`direct`, `group`, `thread`) รองรับ `dm` แบบ legacy เป็นชื่อเรียกแทนของ `direct`
- **`mainKey`**: ฟิลด์แบบ legacy Runtime ตอนนี้จะใช้ "main" สำหรับบัคเก็ตแชตแบบ direct หลักเสมอ
- **`sendPolicy`**: จับคู่ตาม `channel`, `chatType` (`direct|group|channel` โดยมี alias แบบเดิมคือ `dm`), `keyPrefix` หรือ `rawKeyPrefix` กฎ deny ที่มาก่อนจะมีผลก่อน
- **`maintenance`**: `warn` จะแจ้งเตือนเซสชันที่กำลังใช้งานเมื่อถูกถอดออก; `enforce` จะทำการล้างข้อมูล (pruning) และหมุนเวียน (rotation)

</Accordion>

---

## ข้อความ

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### คำนำหน้าการตอบกลับ

การตั้งค่าแทนที่ราย channel/account: `channels.<channel>.responsePrefix`, `channels.<channel>.accounts.<id>.responsePrefix`.

ลำดับการพิจารณา (เจาะจงมากที่สุดมีผลก่อน): account → channel → global `""` จะปิดการทำงานและหยุดการไล่ลำดับ (cascade) `"auto"` จะสร้างจาก `[{identity.name}]`

**ตัวแปรเทมเพลต:**

| ตัวแปร            | คำอธิบาย                | ตัวอย่าง                                |
| ----------------- | ----------------------- | --------------------------------------- |
| `{model}`         | ชื่อโมเดลแบบสั้น        | `claude-opus-4-6`                       |
| `{modelFull}`     | ตัวระบุโมเดลแบบเต็ม     | `anthropic/claude-opus-4-6`             |
| `{provider}`      | ชื่อผู้ให้บริการ        | `anthropic`                             |
| `{thinkingLevel}` | ระดับการคิดปัจจุบัน     | `high`, `low`, `off`                    |
| `{identity.name}` | ชื่อเอกลักษณ์ของเอเจนต์ | (เหมือนกับ `"auto"`) |

ตัวแปรไม่สนใจตัวพิมพ์เล็ก/ใหญ่ `{think}` เป็นชื่อแทนของ `{thinkingLevel}`

### การตอบรับ (Ack reaction)

- ค่าเริ่มต้นคือ `identity.emoji` ของเอเจนต์ที่กำลังใช้งาน มิฉะนั้นใช้ "👀" ตั้งค่าเป็น `""` เพื่อปิดการใช้งาน
- การตั้งค่าแทนที่ราย channel: `channels.<channel>.ackReaction`, `channels.<channel>`.accounts.<id>.ackReaction\`.
- ลำดับการประมวลผล: account → channel → `messages.ackReaction` → ใช้ identity สำรอง
- ขอบเขต: `group-mentions` (ค่าเริ่มต้น), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: ลบ ack หลังจากตอบกลับ (เฉพาะ Slack/Discord/Telegram/Google Chat)

### การหน่วงเวลาข้อความขาเข้า

รวมข้อความที่เป็นข้อความล้วนและส่งมาอย่างรวดเร็วจากผู้ส่งคนเดียวกันให้เป็นการโต้ตอบของเอเจนต์ครั้งเดียว สื่อ/ไฟล์แนบจะถูกส่งทันที คำสั่งควบคุมจะไม่ถูกหน่วงเวลา

### TTS (ข้อความเป็นเสียงพูด)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

- `auto` ควบคุมการเปิดใช้ TTS อัตโนมัติ `/tts off|always|inbound|tagged` ใช้แทนค่าต่อเซสชัน
- `summaryModel` ใช้แทนค่า `agents.defaults.model.primary` สำหรับการสรุปอัตโนมัติ
- API key จะย้อนกลับไปใช้ `ELEVENLABS_API_KEY`/`XI_API_KEY` และ `OPENAI_API_KEY`

---

## Talk

ค่าเริ่มต้นสำหรับโหมด Talk (macOS/iOS/Android)

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Voice ID จะย้อนกลับไปใช้ `ELEVENLABS_VOICE_ID` หรือ `SAG_VOICE_ID`
- `apiKey` จะย้อนกลับไปใช้ `ELEVENLABS_API_KEY`
- `voiceAliases` ช่วยให้คำสั่ง Talk ใช้ชื่อที่จำง่ายได้

---

## เครื่องมือ

### โปรไฟล์เครื่องมือ

`tools.profile` กำหนด allowlist พื้นฐานก่อน `tools.allow`/`tools.deny`:

| โปรไฟล์     | ประกอบด้วย                                                                                |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | เฉพาะ `session_status`                                                                    |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | ไม่มีข้อจำกัด (เหมือนกับไม่ตั้งค่า)                                    |

### กลุ่มเครื่องมือ

| กลุ่ม              | เครื่องมือ                                                                               |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` ใช้เป็นชื่อแทนของ `exec` ได้)               |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | เครื่องมือในตัวทั้งหมด (ไม่รวม provider plugins)                      |

### `tools.allow` / `tools.deny`

นโยบาย allow/deny ของเครื่องมือแบบ global (deny มีผลเหนือกว่า) ไม่สนตัวพิมพ์เล็ก-ใหญ่ และรองรับ wildcard `*` มีผลแม้ปิดการใช้งาน Docker sandbox อยู่

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

จำกัดการใช้งานเครื่องมือเพิ่มเติมสำหรับ provider หรือโมเดลที่ระบุ ลำดับ: base profile → provider profile → allow/deny

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

ควบคุมสิทธิ์การรันคำสั่งแบบ elevated (บนโฮสต์):

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- การกำหนดค่าระดับเอเจนต์ (`agents.list[].tools.elevated`) สามารถจำกัดเพิ่มเติมได้เท่านั้น
- `/elevated on|off|ask|full` จะบันทึกสถานะแยกตามแต่ละเซสชัน; คำสั่งแบบ inline จะมีผลกับข้อความเดียวเท่านั้น
- `exec` แบบ elevated จะรันบนโฮสต์และข้ามการทำ sandboxing

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // หรือ BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

กำหนดค่าการทำความเข้าใจสื่อขาเข้า (image/audio/video):

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Media model entry fields">

**Provider entry** (`type: "provider"` หรือเว้นว่างไว้):

- `provider`: รหัสผู้ให้บริการ API (`openai`, `anthropic`, `google`/`gemini`, `groq` เป็นต้น)
- `model`: ระบุรหัสโมเดลที่ต้องการใช้แทนค่าเริ่มต้น
- `profile` / `preferredProfile`: เลือกโปรไฟล์สำหรับการยืนยันตัวตน

**CLI entry** (`type: "cli"`):

- `command`: ไฟล์ปฏิบัติการที่จะเรียกใช้งาน
- `args`: อาร์กิวเมนต์แบบเทมเพลต (รองรับ `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` เป็นต้น)

**Common fields:**

- `capabilities`: รายการความสามารถเพิ่มเติม (ไม่บังคับ) (`image`, `audio`, `video`) ค่าเริ่มต้น: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: กำหนดค่าแทนที่เฉพาะแต่ละรายการ
- หากเกิดความล้มเหลว ระบบจะสลับไปใช้รายการถัดไป

การยืนยันตัวตนของผู้ให้บริการเป็นไปตามลำดับมาตรฐาน: โปรไฟล์การยืนยันตัวตน → ตัวแปรสภาพแวดล้อม → `models.providers.*.apiKey`

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: โมเดลเริ่มต้นสำหรับ sub-agents ที่ถูกสร้างขึ้น หากไม่ระบุ sub-agents จะสืบทอดโมเดลจากผู้เรียกใช้งาน
- นโยบายเครื่องมือราย subagent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`

---

## ผู้ให้บริการแบบกำหนดเองและ base URL

OpenClaw ใช้แคตตาล็อกโมเดลของ pi-coding-agent เพิ่มผู้ให้บริการแบบกำหนดเองผ่าน `models.providers` ในไฟล์ config หรือ `~/.openclaw/agents/<agentId>/agent/models.json`

```json5
{
  models: {
    mode: "merge", // รวม (ค่าเริ่มต้น) | แทนที่
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- ใช้ `authHeader: true` ร่วมกับ `headers` สำหรับความต้องการยืนยันตัวตนแบบกำหนดเอง
- กำหนดค่า root ของ agent config ใหม่ด้วย `OPENCLAW_AGENT_DIR` (หรือ `PI_CODING_AGENT_DIR`)

### ตัวอย่างผู้ให้บริการ

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

ใช้ `cerebras/zai-glm-4.7` สำหรับ Cerebras; ใช้ `zai/glm-4.7` สำหรับ Z.AI โดยตรง

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

ตั้งค่า `OPENCODE_API_KEY` (หรือ `OPENCODE_ZEN_API_KEY`) ทางลัด: `openclaw onboard --auth-choice opencode-zen`

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

ตั้งค่า `ZAI_API_KEY` รองรับ alias `z.ai/*` และ `z-ai/*` ทางลัด: `openclaw onboard --auth-choice zai-api-key`

- Endpoint ทั่วไป: `https://api.z.ai/api/paas/v4`
- Coding endpoint (ค่าเริ่มต้น): `https://api.z.ai/api/coding/paas/v4`
- สำหรับ endpoint ทั่วไป ให้กำหนดผู้ให้บริการแบบกำหนดเองพร้อมแทนที่ base URL

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

สำหรับ endpoint ในประเทศจีน: `baseUrl: "https://api.moonshot.cn/v1"` หรือ `openclaw onboard --auth-choice moonshot-api-key-cn`

</Accordion>

<Accordion title="Kimi Coding">

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

รองรับ Anthropic และมี provider ในตัว ทางลัด: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Base URL ควรละเว้น `/v1` (ไคลเอนต์ของ Anthropic จะเติมให้เอง) ทางลัด: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

ตั้งค่า `MINIMAX_API_KEY`. ทางลัด: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

ดู [โมเดลภายในเครื่อง](/gateway/local-models). สรุปสั้น ๆ: รัน MiniMax M2.1 ผ่าน LM Studio Responses API บนฮาร์ดแวร์ประสิทธิภาพสูง และคงการรวมโมเดลแบบโฮสต์ไว้เพื่อใช้เป็นตัวสำรอง

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: allowlist แบบเลือกใช้สำหรับ skills ที่มากับระบบเท่านั้น (ไม่กระทบ managed/workspace skills).
- `entries.<skillKey>`.enabled: false\` จะปิดการใช้งาน skill แม้ว่าจะมากับระบบ/ติดตั้งไว้แล้ว
- `entries.<skillKey>`.apiKey\`: ตัวช่วยอำนวยความสะดวกสำหรับ skills ที่กำหนดตัวแปร env หลักไว้

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- โหลดจาก `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` และ `plugins.load.paths`.
- **การเปลี่ยนแปลงการตั้งค่าต้องรีสตาร์ท gateway**
- `allow`: allowlist แบบเลือกใช้ (จะโหลดเฉพาะ plugins ที่ระบุไว้เท่านั้น) `deny` มีผลเหนือกว่า

ดู [Plugins](/tools/plugin).

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` จะปิดการใช้งาน `act:evaluate` และ `wait --fn`.
- โปรไฟล์ระยะไกล (Remote) เป็นแบบ attach-only (ไม่รองรับ start/stop/reset)
- ลำดับการตรวจจับอัตโนมัติ: เบราว์เซอร์เริ่มต้นหากเป็น Chromium-based → Chrome → Brave → Edge → Chromium → Chrome Canary.
- บริการควบคุม: เฉพาะ loopback (พอร์ตอ้างอิงจาก `gateway.port`, ค่าเริ่มต้น `18791`)

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: สีเน้นสำหรับ UI ของแอปเนทีฟ (เช่น สีฟองใน Talk Mode เป็นต้น)
- `assistant`: กำหนดอัตลักษณ์ของ Control UI แทนค่าเริ่มต้น หากไม่กำหนด จะใช้อัตลักษณ์ของ agent ที่กำลังใช้งานอยู่

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (รัน gateway) หรือ `remote` (เชื่อมต่อกับ gateway ระยะไกล). Gateway จะไม่เริ่มทำงานหากไม่ได้ตั้งค่าเป็น `local`.
- `port`: พอร์ตแบบมัลติเพล็กซ์เดียวสำหรับ WS + HTTP ลำดับความสำคัญ: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`
- `bind`: `auto`, `loopback` (ค่าเริ่มต้น), `lan` (`0.0.0.0`), `tailnet` (เฉพาะ Tailscale IP) หรือ `custom`
- **Auth**: จำเป็นโดยค่าเริ่มต้น การ bind ที่ไม่ใช่ loopback ต้องใช้โทเค็น/รหัสผ่านที่แชร์ร่วมกัน ตัวช่วยตั้งค่าเริ่มต้น (Onboarding wizard) จะสร้างโทเค็นให้โดยอัตโนมัติ
- `auth.mode: "trusted-proxy"`: มอบหมายการยืนยันตัวตนให้ reverse proxy ที่รับรู้ตัวตน และเชื่อถือ identity headers จาก `gateway.trustedProxies` (ดู [Trusted Proxy Auth](/gateway/trusted-proxy-auth))
- `auth.allowTailscale`: เมื่อเป็น `true`, identity headers จาก Tailscale Serve จะผ่านการยืนยันตัวตน (ตรวจสอบผ่าน `tailscale whois`) ค่าเริ่มต้นเป็น `true` เมื่อ `tailscale.mode = "serve"`
- `auth.rateLimit`: ตัวจำกัดความพยายามยืนยันตัวตนที่ล้มเหลว (ไม่บังคับ) มีผลต่อแต่ละ client IP และแต่ละขอบเขตการยืนยันตัวตน (shared-secret และ device-token จะถูกติดตามแยกกัน) คำขอที่ถูกบล็อกจะส่งกลับ `429` + `Retry-After`
  - `auth.rateLimit.exemptLoopback` มีค่าเริ่มต้นเป็น `true`; ตั้งค่าเป็น `false` หากคุณต้องการให้ทราฟฟิกจาก localhost ถูกจำกัดอัตราด้วย (สำหรับสภาพแวดล้อมทดสอบหรือการติดตั้ง proxy แบบเข้มงวด)
- `tailscale.mode`: `serve` (เฉพาะ tailnet, bind กับ loopback) หรือ `funnel` (สาธารณะ ต้องมีการยืนยันตัวตน)
- `remote.transport`: `ssh` (ค่าเริ่มต้น) หรือ `direct` (ws/wss) สำหรับ `direct`, `remote.url` ต้องเป็น `ws://` หรือ `wss://`
- `gateway.remote.token` ใช้สำหรับการเรียก CLI ระยะไกลเท่านั้น; ไม่ได้เปิดใช้งานการยืนยันตัวตนของ local gateway
- `trustedProxies`: IP ของ reverse proxy ที่ทำหน้าที่ยุติ TLS ระบุเฉพาะ proxy ที่คุณควบคุมเท่านั้น
- `gateway.tools.deny`: ชื่อเครื่องมือเพิ่มเติมที่ถูกบล็อกสำหรับ HTTP `POST /tools/invoke` (ขยายจาก deny list เริ่มต้น)
- `gateway.tools.allow`: ลบชื่อเครื่องมือออกจาก HTTP deny list เริ่มต้น

</Accordion>

### เอ็นด์พอยต์ที่รองรับ OpenAI

- Chat Completions: ปิดใช้งานโดยค่าเริ่มต้น เปิดใช้งานด้วย `gateway.http.endpoints.chatCompletions.enabled: true`
- Responses API: `gateway.http.endpoints.responses.enabled`
- การเสริมความปลอดภัย URL-input สำหรับ Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### การแยกการทำงานหลายอินสแตนซ์

รัน gateway หลายตัวบนโฮสต์เดียวโดยใช้พอร์ตและ state dir ที่ไม่ซ้ำกัน:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

แฟล็กเพื่อความสะดวก: `--dev` (ใช้ `~/.openclaw-dev` + พอร์ต `19001`), `--profile <name>` (ใช้ `~/.openclaw-<name>`)

ดู [Multiple Gateways](/gateway/multiple-gateways)

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

Auth: `Authorization: Bearer <token>` หรือ `x-openclaw-token: <token>`

**Endpoints:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }` }\`
  - `sessionKey` จาก payload ของคำขอจะถูกรับได้เฉพาะเมื่อกำหนด `hooks.allowRequestSessionKey=true` (ค่าเริ่มต้น: `false`)
- `POST /hooks/<name>` → ถูก resolve ผ่าน `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` จับคู่ sub-path หลัง `/hooks` (เช่น `/hooks/gmail` → `gmail`)
- `match.source` จับคู่ฟิลด์ใน payload สำหรับเส้นทางแบบทั่วไป
- เทมเพลตอย่าง `{{messages[0].subject}}` จะอ่านค่าจาก payload
- `transform` สามารถชี้ไปยังโมดูล JS/TS ที่คืนค่า hook action ได้
  - `transform.module` ต้องเป็น relative path และต้องอยู่ภายใน `hooks.transformsDir` (จะปฏิเสธ absolute path และการไต่ระดับพาธ)
- `agentId` ใช้กำหนดเส้นทางไปยัง agent เฉพาะ; หากไม่รู้จัก ID จะย้อนกลับไปใช้ค่าเริ่มต้น
- `allowedAgentIds`: จำกัดการกำหนดเส้นทางแบบระบุชัด (`*` หรือไม่ระบุ = อนุญาตทั้งหมด, `[]` = ปฏิเสธทั้งหมด)
- `defaultSessionKey`: session key แบบคงที่ (ไม่บังคับ) สำหรับการรัน hook agent ที่ไม่มี `sessionKey` ระบุมาโดยชัดเจน
- `allowRequestSessionKey`: อนุญาตให้ผู้เรียก `/hooks/agent` ตั้งค่า `sessionKey` ได้ (ค่าเริ่มต้น: `false`)
- `allowedSessionKeyPrefixes`: allowlist ของคำนำหน้า (prefix) สำหรับค่า `sessionKey` ที่ระบุชัด (ทั้งจาก request + mapping) เช่น `["hook:"]`
- `deliver: true` จะส่งคำตอบสุดท้ายไปยัง channel; ค่า `channel` เริ่มต้นคือ `last`
- `model` ใช้ override LLM สำหรับการรัน hook นี้ (ต้องได้รับอนุญาตหากมีการตั้งค่า model catalog)

</Accordion>

### การผสานรวม Gmail

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- Gateway จะเริ่ม `gog gmail watch serve` โดยอัตโนมัติเมื่อบูต หากมีการตั้งค่าไว้ ตั้งค่า `OPENCLAW_SKIP_GMAIL_WATCHER=1` เพื่อปิดใช้งาน
- อย่ารัน `gog gmail watch serve` แยกต่างหากพร้อมกับ Gateway

---

## โฮสต์ Canvas

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- ให้บริการ HTML/CSS/JS ที่ agent แก้ไขได้ และ A2UI ผ่าน HTTP ภายใต้พอร์ตของ Gateway:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- เฉพาะในเครื่อง: คงค่า `gateway.bind: "loopback"` (ค่าเริ่มต้น)
- หาก bind แบบ non-loopback: เส้นทาง canvas จะต้องใช้การยืนยันตัวตนของ Gateway (token/password/trusted-proxy) เช่นเดียวกับพื้นผิว HTTP อื่น ๆ ของ Gateway
- โดยทั่วไป Node WebViews จะไม่ส่ง auth headers; หลังจาก node ถูกจับคู่และเชื่อมต่อแล้ว Gateway จะอนุญาตการ fallback ผ่าน private-IP เพื่อให้ node โหลด canvas/A2UI ได้โดยไม่ต้องเผยความลับใน URL
- แทรกไคลเอนต์ live-reload ลงใน HTML ที่ให้บริการ
- สร้าง `index.html` เริ่มต้นให้อัตโนมัติเมื่อไดเรกทอรีว่าง
- ให้บริการ A2UI ที่ `/__openclaw__/a2ui/` ด้วย
- การเปลี่ยนแปลงต้องรีสตาร์ท gateway
- ปิดใช้งาน live reload สำหรับไดเรกทอรีขนาดใหญ่หรือเมื่อเกิดข้อผิดพลาด `EMFILE`

---

## การค้นหา (Discovery)

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (ค่าเริ่มต้น): ไม่รวม `cliPath` + `sshPort` ใน TXT records
- `full`: รวม `cliPath` + `sshPort`
- Hostname เริ่มต้นคือ `openclaw` สามารถ override ได้ด้วย `OPENCLAW_MDNS_HOSTNAME`

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

เขียนโซน unicast DNS-SD ภายใต้ `~/.openclaw/dns/` สำหรับการค้นหาข้ามเครือข่าย ให้ใช้งานร่วมกับเซิร์ฟเวอร์ DNS (แนะนำ CoreDNS) + Tailscale split DNS

ตั้งค่า: `openclaw dns setup --apply`

---

## สภาพแวดล้อม

### `env` (ตัวแปรสภาพแวดล้อมแบบ inline)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- ตัวแปรสภาพแวดล้อมแบบ inline จะถูกใช้งานก็ต่อเมื่อ process env ไม่มีคีย์นั้นอยู่
- ไฟล์ `.env`: `.env` ใน CWD + `~/.openclaw/.env` (ทั้งสองจะไม่เขียนทับตัวแปรที่มีอยู่แล้ว)
- `shellEnv`: นำเข้าคีย์ที่คาดว่าจะมีแต่ยังขาดอยู่จากโปรไฟล์ login shell ของคุณ
- ดู [Environment](/help/environment) สำหรับลำดับความสำคัญทั้งหมด

### การแทนที่ค่าด้วยตัวแปรสภาพแวดล้อม

อ้างอิงตัวแปรสภาพแวดล้อมในสตริงคอนฟิกใด ๆ ด้วย `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- จับคู่เฉพาะชื่อที่เป็นตัวพิมพ์ใหญ่เท่านั้น: `[A-Z_][A-Z0-9_]*`
- ตัวแปรที่ขาดหาย/ว่างเปล่าจะทำให้เกิดข้อผิดพลาดตอนโหลดคอนฟิก
- ใช้ `$${VAR}` เพื่อ escape ให้เป็น `${VAR}` แบบตัวอักษรล้วน
- ทำงานร่วมกับ `$include` ได้

---

## การจัดเก็บข้อมูลยืนยันตัวตน

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- โปรไฟล์การยืนยันตัวตนต่อเอเจนต์จะถูกเก็บไว้ที่ `<agentDir>/auth-profiles.json`
- นำเข้า OAuth แบบเดิมจาก `~/.openclaw/credentials/oauth.json`
- ดู [OAuth](/concepts/oauth)

---

## การบันทึกล็อก

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- ไฟล์ล็อกเริ่มต้น: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- ตั้งค่า `logging.file` เพื่อใช้พาธแบบคงที่
- `consoleLevel` จะเพิ่มเป็น `debug` เมื่อใช้ `--verbose`

---

## Wizard

เมทาดาทาที่เขียนโดย CLI wizard (`onboard`, `configure`, `doctor`):

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## ข้อมูลประจำตัว

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

เขียนโดยผู้ช่วย onboarding บน macOS กำหนดค่าเริ่มต้นจาก:

- `messages.ackReaction` จาก `identity.emoji` (หากไม่มีจะใช้ 👀)
- `mentionPatterns` จาก `identity.name`/`identity.emoji`
- `avatar` รองรับ: พาธแบบอ้างอิง workspace, URL `http(s)`, หรือ `data:` URI

---

## Bridge (legacy, ถูกนำออกแล้ว)

เวอร์ชันปัจจุบันไม่รวม TCP bridge อีกต่อไป โหนดเชื่อมต่อผ่าน Gateway WebSocket คีย์ `bridge.*` ไม่ได้เป็นส่วนหนึ่งของโครงสร้างคอนฟิกอีกต่อไป (การตรวจสอบจะล้มเหลวจนกว่าจะลบออก; `openclaw doctor --fix` สามารถลบคีย์ที่ไม่รู้จักได้)

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`: ระยะเวลาที่เก็บเซสชัน cron ที่เสร็จสิ้นแล้วก่อนจะลบออก ค่าเริ่มต้น: `24h`.

ดู [Cron Jobs](/automation/cron-jobs).

---

## ตัวแปรเทมเพลตของโมเดลสื่อ

ตัวแทนค่าในเทมเพลตที่ถูกแทนค่าใน `tools.media.*.models[].args`:

| ตัวแปร             | คำอธิบาย                                                                    |
| ------------------ | --------------------------------------------------------------------------- |
| `{{Body}}`         | เนื้อหาข้อความขาเข้าทั้งหมด                                                 |
| `{{RawBody}}`      | เนื้อหาดิบ (ไม่มี history/sender wrappers)               |
| `{{BodyStripped}}` | เนื้อหาที่ลบการ mention กลุ่มออกแล้ว                                        |
| `{{From}}`         | ตัวระบุผู้ส่ง                                                               |
| `{{To}}`           | ตัวระบุปลายทาง                                                              |
| `{{MessageSid}}`   | รหัสข้อความของช่องทาง                                                       |
| `{{SessionId}}`    | UUID ของเซสชันปัจจุบัน                                                      |
| `{{IsNewSession}}` | `"true"` เมื่อมีการสร้างเซสชันใหม่                                          |
| `{{MediaUrl}}`     | pseudo-URL ของสื่อขาเข้า                                                    |
| `{{MediaPath}}`    | พาธไฟล์สื่อภายในเครื่อง                                                     |
| `{{MediaType}}`    | ประเภทสื่อ (image/audio/document/…)                      |
| `{{Transcript}}`   | บทถอดเสียงของไฟล์เสียง                                                      |
| `{{Prompt}}`       | พรอมป์ต์สื่อที่ประมวลผลแล้วสำหรับรายการ CLI                                 |
| `{{MaxChars}}`     | แก้ไขปัญหาจำนวนอักขระเอาต์พุตสูงสุดสำหรับรายการ CLI                         |
| `{{ChatType}}`     | `"direct"` หรือ `"group"`                                                   |
| `{{GroupSubject}}` | ชื่อกลุ่ม (เท่าที่สามารถดึงข้อมูลได้)                    |
| `{{GroupMembers}}` | ตัวอย่างสมาชิกในกลุ่ม (เท่าที่สามารถดึงข้อมูลได้)        |
| `{{SenderName}}`   | ชื่อที่แสดงของผู้ส่ง (เท่าที่สามารถดึงข้อมูลได้)         |
| `{{SenderE164}}`   | หมายเลขโทรศัพท์ของผู้ส่ง (เท่าที่สามารถดึงข้อมูลได้)     |
| `{{Provider}}`     | ข้อมูลผู้ให้บริการ (whatsapp, telegram, discord เป็นต้น) |

---

## การรวมไฟล์คอนฟิก (`$include`)

แยกคอนฟิกออกเป็นหลายไฟล์:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**พฤติกรรมการรวม:**

- ไฟล์เดี่ยว: แทนที่อ็อบเจ็กต์ที่ครอบอยู่ทั้งหมด
- อาร์เรย์ของไฟล์: รวมแบบ deep-merge ตามลำดับ (ไฟล์หลังจะเขียนทับไฟล์ก่อนหน้า)
- คีย์ระดับเดียวกัน: รวมหลังจาก include (จะเขียนทับค่าที่ถูกรวมมา)
- Nested includes: ได้ลึกสูงสุด 10 ระดับ
- พาธ: แบบสัมพัทธ์ (อ้างอิงจากไฟล์ที่ include), แบบสัมบูรณ์ หรืออ้างอิงโฟลเดอร์แม่ด้วย `../`
- ข้อผิดพลาด: แสดงข้อความชัดเจนเมื่อไฟล์หายไป, พาร์สผิดพลาด หรือมีการ include วนซ้ำ

---

_ที่เกี่ยวข้อง: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_


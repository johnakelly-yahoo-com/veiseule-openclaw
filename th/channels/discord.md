---
summary: "สถานะการรองรับ ความสามารถ และการกำหนดค่าของบอตDiscord"
read_when:
  - กำลังพัฒนาฟีเจอร์ของช่องทางDiscord
title: "Discord"
---

# Discord (Bot API)

สถานะ: พร้อมใช้งานสำหรับDMและช่องข้อความของกิลด์ผ่านเกตเวย์บอตDiscordอย่างเป็นทางการ

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DMs จะใช้โหมดการจับคู่ (pairing mode) เป็นค่าเริ่มต้น
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    พฤติกรรมคำสั่งแบบเนทีฟและรายการคำสั่ง
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    การวินิจฉัยและขั้นตอนการซ่อมแซมข้ามช่องทาง
  
</Card>
</CardGroup>

## Quick setup (beginner)

<Steps>
  <Step title="Create a Discord bot and enable intents">สร้างแอปพลิเคชันใน Discord Developer Portal เพิ่มบอท จากนั้นเปิดใช้งาน:

    ```
    - **Message Content Intent**
    - **Server Members Intent** (จำเป็นสำหรับ role allowlists และการกำหนดเส้นทางตาม role; แนะนำสำหรับการจับคู่ allowlist แบบชื่อเป็น ID)
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
    Env fallback สำหรับบัญชีเริ่มต้น:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">เชิญบอตเข้ามาในเซิร์ฟเวอร์ของคุณพร้อมสิทธิ์การส่งข้อความ (สร้างเซิร์ฟเวอร์ส่วนตัวหากต้องการใช้เฉพาะDM)

```bash
เริ่มต้น Gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    รหัสการจับคู่จะหมดอายุภายใน 1 ชั่วโมง
    ```

  
</Step>
</Steps>

<Note>
การ resolve โทเค็นจะอิงตามบัญชี ค่าโทเค็นในคอนฟิกจะมีลำดับความสำคัญเหนือ env fallback `DISCORD_BOT_TOKEN` จะถูกใช้เฉพาะกับบัญชีเริ่มต้นเท่านั้น
</Note>

## โมเดลรันไทม์

- Gateway เป็นผู้ดูแลการเชื่อมต่อกับ Discord
- คงการกำหนดเส้นทางให้เป็นแบบกำหนดแน่นอน: การตอบกลับจะส่งกลับไปยังช่องทางที่รับข้อความมาเสมอ
- แชตแบบตรงจะถูกรวมเข้าเป็นเซสชันหลักของเอเจนต์ (ค่าเริ่มต้น `agent:main:main`); ช่องกิลด์จะถูกแยกเป็น `agent:<agentId>:discord:channel:<channelId>` (ชื่อที่แสดงใช้ `discord:<guildSlug>#<channelSlug>`)
- กฎกิลด์เสริม: ตั้งค่า `channels.discord.guilds` โดยคีย์เป็นguild id (แนะนำ) หรือslug พร้อมกฎรายช่อง
- Group DMจะถูกละเว้นโดยค่าเริ่มต้น; เปิดใช้งานด้วย `channels.discord.dm.groupEnabled` และอาจจำกัดด้วย `channels.discord.dm.groupChannels`.
- คำสั่งเนทีฟใช้คีย์เซสชันแบบแยก (`agent:<agentId>:discord:slash:<userId>`) แทนเซสชันที่ใช้ร่วมกัน `main`

## การควบคุมการเข้าถึงและการกำหนดเส้นทาง

<Tabs>
  <Tab title="DM policy">หากต้องการละเว้นDMทั้งหมด: ตั้งค่า `channels.discord.dm.enabled=false` หรือ `channels.discord.dm.policy="disabled"`.

    ```
    หากต้องการคงพฤติกรรมแบบ “เปิดให้ใครก็ได้” เดิม: ตั้งค่า `channels.discord.dm.policy="open"` และ `channels.discord.dm.allowFrom=["*"]`.
    ```

  
</Tab>

  <Tab title="Guild policy">พฤติกรรมถูกควบคุมโดย `channels.discord.replyToMode`:

    ```
    หากต้องการรายการอนุญาตแบบเข้มงวด: ตั้งค่า `channels.discord.dm.policy="allowlist"` และระบุผู้ส่งใน `channels.discord.dm.allowFrom`.
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    หากคุณตั้งค่าเฉพาะ `DISCORD_BOT_TOKEN` และไม่เคยสร้างส่วน `channels.discord` ระบบรันไทม์จะตั้งค่าเริ่มต้น `groupPolicy` เป็น `open`.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    ข้อความใน Guild จะถูกกำหนดให้ต้องมีการ mention เป็นค่าเริ่มต้น

    ```
    การตรวจจับการ mention ครอบคลุม:
    
    - การ mention บอทโดยตรง
    - รูปแบบการ mention ที่กำหนดไว้ (`agents.list[].groupChat.mentionPatterns`, ใช้ `messages.groupChat.mentionPatterns` เป็นค่า fallback)
    - พฤติกรรมตอบกลับบอทโดยอัตโนมัติในกรณีที่รองรับ
    
    `requireMention` ถูกกำหนดแยกตาม guild/channel (`channels.discord.guilds...`).
    
    Group DMs:
    
    - ค่าเริ่มต้น: ถูกละเว้น (`dm.groupEnabled=false`)
    - สามารถกำหนด allowlist เพิ่มเติมผ่าน `dm.groupChannels` (channel IDs หรือ slugs)
    ```

  
</Tab>
</Tabs>

### การกำหนดเส้นทางเอเจนต์ตามบทบาท (Role-based)

ใช้ `bindings[].match.roles` เพื่อกำหนดเส้นทางสมาชิก Discord guild ไปยังเอเจนต์ที่แตกต่างกันตาม role ID Role-based bindings รองรับเฉพาะ role ID และจะถูกประเมินหลังจาก peer หรือ parent-peer bindings และก่อน guild-only bindings หาก binding มีการตั้งค่า match ฟิลด์อื่นร่วมด้วย (เช่น `peer` + `guildId` + `roles`) ทุกฟิลด์ที่กำหนดจะต้องตรงกันทั้งหมด

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

## การตั้งค่า Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">ใน **Bot** → **Privileged Gateway Intents** ให้เปิดใช้งาน:

    ```
    - Message Content Intent
    - Server Members Intent (แนะนำ)
    
    Presence intent เป็นตัวเลือก และจำเป็นเฉพาะเมื่อคุณต้องการรับการอัปเดตสถานะ presence เท่านั้น การตั้งค่า presence ของบอท (`setPresence`) ไม่จำเป็นต้องเปิดใช้งานการอัปเดต presence ของสมาชิก
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">ในแอปของคุณ: **OAuth2** → **URL Generator**

    ```
    - scopes: `bot`, `applications.commands`
    
    สิทธิ์พื้นฐานที่มักใช้:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (ไม่บังคับ)
    
    หลีกเลี่ยงการใช้ `Administrator` เว้นแต่จำเป็นอย่างชัดเจน
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    เปิดใช้งาน Discord Developer Mode จากนั้นคัดลอก:

    ```
    - server ID
    - channel ID
    - user ID
    
    แนะนำให้ใช้ numeric ID ในการตั้งค่า OpenClaw เพื่อความน่าเชื่อถือในการตรวจสอบและทดสอบ
    ```

  
</Accordion>
</AccordionGroup>

## คำสั่ง native และการยืนยันสิทธิ์คำสั่ง

- คำสั่งเนทีฟเสริม: `commands.native` ค่าเริ่มต้นเป็น `"auto"` (เปิดสำหรับDiscord/Telegram, ปิดสำหรับSlack).
- หรือคอนฟิก: `channels.discord.token: "..."`.
- `commands.native=false` จะล้างคำสั่ง Discord native ที่เคยลงทะเบียนไว้ก่อนหน้าอย่างชัดเจน
- คำสั่งเนทีฟเคารพรายการอนุญาตเดียวกับDM/ข้อความกิลด์ (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, กฎรายช่อง)
- Slash commands อาจยังมองเห็นได้ในUIของDiscordสำหรับผู้ใช้ที่ไม่ได้อยู่ในรายการอนุญาต; OpenClawจะบังคับใช้รายการอนุญาตเมื่อรันและตอบว่า “not authorized”

ดู [Exec approvals](/tools/exec-approvals) และ [Slash commands](/tools/slash-commands) สำหรับโฟลว์การอนุมัติและคำสั่งโดยรวม

## รายละเอียดฟีเจอร์

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord รองรับ reply tags ในผลลัพธ์ของเอเจนต์:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    ควบคุมด้วย `channels.discord.replyToMode`:
    
    - `off` (ค่าเริ่มต้น)
    - `first`
    - `all`
    
    หมายเหตุ: `off` จะปิดการทำ reply threading แบบอัตโนมัติ แต่ยังคงรองรับแท็ก `[[reply_to_*]]` ที่ระบุอย่างชัดเจน
    
    Message IDs จะถูกส่งผ่านใน context/history เพื่อให้เอเจนต์สามารถอ้างอิงข้อความเฉพาะได้
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    บริบทประวัติใน Guild:

    ```
    - `channels.discord.historyLimit` ค่าเริ่มต้น `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` คือปิดใช้งาน
    
    การควบคุมประวัติ DM:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    พฤติกรรมของ Thread:
    
    - Discord threads จะถูกกำหนดเส้นทางเป็น channel sessions
    - สามารถใช้ metadata ของ parent thread เพื่อเชื่อมโยงกับ parent-session
    - การตั้งค่า thread จะสืบทอดจากการตั้งค่า parent channel เว้นแต่จะมีการกำหนดเฉพาะสำหรับ thread นั้น
    
    หัวข้อ (topic) ของช่องจะถูกเพิ่มเข้าไปเป็นบริบทแบบ **untrusted** (ไม่ใช่ system prompt)
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    `guilds.<id> .reactionNotifications`: โหมดอีเวนต์ของระบบรีแอคชัน (`off`, `own`, `all`, `allowlist`)
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` จะส่งอีโมจิเพื่อยืนยันขณะ OpenClaw กำลังประมวลผลข้อความขาเข้า

    ```
    ลำดับการกำหนดค่า:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - fallback เป็นอีโมจิจาก identity ของเอเจนต์ (`agents.list[].identity.emoji`, หากไม่มีใช้ "👀")
    
    หมายเหตุ:
    
    - Discord รองรับทั้ง unicode emoji หรือชื่อ custom emoji
    - ใช้ `""` เพื่อปิดใช้งาน reaction สำหรับช่องหรือบัญชี
    ```

  
</Accordion>

  <Accordion title="Config writes">
    การเขียนค่าคอนฟิกที่เริ่มจากช่อง (Channel-initiated) เปิดใช้งานเป็นค่าเริ่มต้น

    ```
    มีผลกับกระบวนการ `/config set|unset` (เมื่อเปิดใช้งานฟีเจอร์คำสั่ง)
    
    ปิดใช้งาน:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    กำหนดเส้นทางทราฟฟิก WebSocket ของ Discord gateway ผ่าน HTTP(S) proxy ด้วย `channels.discord.proxy`

```json5
ช่อง (เช่น `#help`) → **Copy Channel ID**
```

    ```
    การกำหนดค่าแยกตามบัญชี:
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
    เปิดใช้งาน PluralKit resolution เพื่อแมปข้อความที่ถูก proxy ไปยังตัวตนสมาชิกของระบบ:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    หมายเหตุ:
    
    - allowlists สามารถใช้ `pk:<memberId>`
    - ชื่อแสดงผลของสมาชิกจะถูกจับคู่ตามชื่อ/slug
    - การค้นหาใช้ message ID เดิมและถูกจำกัดภายในช่วงเวลา
    - หากค้นหาไม่สำเร็จ ข้อความที่ถูก proxy จะถูกมองเป็นข้อความบอทและถูกละเว้น เว้นแต่ตั้งค่า `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    การอัปเดต Presence จะถูกใช้เฉพาะเมื่อคุณตั้งค่าฟิลด์ status หรือ activity

    ```
    ตัวอย่างเฉพาะการตั้งค่า Status:
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    ตัวอย่าง Activity (ค่าเริ่มต้นของประเภท activity คือ custom status):
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
    ตัวอย่างการสตรีม:
    ```

```json5
กำหนดค่าOpenClawด้วย `channels.discord.token` (หรือ `DISCORD_BOT_TOKEN` เป็นตัวสำรอง)
```

    ```
    แผนผังประเภท Activity:
    
    - 0: Playing
    - 1: Streaming (ต้องระบุ `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (ใช้ข้อความ activity เป็นสถานะ; อีโมจิเป็นตัวเลือก)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord รองรับการอนุมัติ exec แบบใช้ปุ่มใน DM และสามารถเลือกให้โพสต์ข้อความขออนุมัติในช่องต้นทางได้

    ```
    idผู้ใช้Discordของคุณอยู่ใน `channels.discord.execApprovals.approvers` (UIจะส่งให้ผู้อนุมัติเท่านั้น)
    ```

  
</Accordion>
</AccordionGroup>

## Tool actions

การดำเนินการกับข้อความใน Discord รวมถึงการส่งข้อความ การจัดการช่อง การดูแลควบคุม การตั้งค่าสถานะ (presence) และการจัดการเมตาดาต้า

ตัวอย่างหลัก:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- React + list reactions + emojiList
- `timeout`, `kick`, `ban`
- presence: `setPresence`

เกตของการดำเนินการอยู่ภายใต้ `channels.discord.actions.*`.

พฤติกรรมเกตเริ่มต้น:

| Action group                                                                                                  | Default   |
| ------------------------------------------------------------------------------------------------------------- | --------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | enabled   |
| บทบาท                                                                                                         | ปิดใช้งาน |
| moderation                                                                                                    | ปิดใช้งาน |
| presence                                                                                                      | ปิดใช้งาน |

## UI ของ Components v2

OpenClaw ใช้ Discord components v2 สำหรับการอนุมัติการรันคำสั่ง (exec approvals) และตัวบ่งชี้ข้ามบริบท การดำเนินการกับข้อความใน Discord ยังสามารถรับ `components` สำหรับ UI แบบกำหนดเองได้ (ขั้นสูง; ต้องใช้ Carbon component instances) ขณะที่ `embeds` แบบเดิมยังคงใช้ได้ แต่ไม่แนะนำ

- `channels.discord.ui.components.accentColor` กำหนดสีเน้นที่ใช้โดยคอนเทนเนอร์ของ Discord components (hex)
- ตั้งค่าแยกตามบัญชีด้วย `channels.discord.accounts.<id> .ui.components.accentColor`.
- ระบบจะไม่สนใจ `embeds` เมื่อมี components v2 อยู่

ตัวอย่าง:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
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

## messages

ข้อความเสียงใน Discord จะแสดงตัวอย่างคลื่นเสียงและต้องใช้ไฟล์เสียง OGG/Opus พร้อมเมตาดาต้า OpenClaw สร้างคลื่นเสียงให้อัตโนมัติ แต่จำเป็นต้องมี `ffmpeg` และ `ffprobe` พร้อมใช้งานบนโฮสต์ gateway เพื่อตรวจสอบและแปลงไฟล์เสียง

Capabilities & limits

- ระบุ **พาธไฟล์ในเครื่อง (local file path)** (ระบบจะปฏิเสธ URL)
- ละเว้นเนื้อหาข้อความ (Discord ไม่อนุญาตให้มีข้อความ + ข้อความเสียงใน payload เดียวกัน)
- รองรับไฟล์เสียงทุกฟอร์แมต; OpenClaw จะแปลงเป็น OGG/Opus เมื่อจำเป็น

ตัวอย่าง:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - เปิดใช้งาน Message Content Intent
    - เปิดใช้งาน Server Members Intent เมื่อคุณต้องพึ่งพาการแปลงผู้ใช้/สมาชิก
    - รีสตาร์ต gateway หลังจากเปลี่ยน intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - ตรวจสอบ `groupPolicy`
    - ตรวจสอบ guild allowlist ภายใต้ `channels.discord.guilds`
    - หากมีแผนที่ `channels` ของ guild จะอนุญาตเฉพาะช่องที่ระบุไว้เท่านั้น
    - ตรวจสอบพฤติกรรม `requireMention` และรูปแบบการ mention
    
    การตรวจสอบที่มีประโยชน์:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    สาเหตุที่พบบ่อย:

    ```
    หากต้องการอนุญาต **ไม่มีช่องใดเลย** ให้ตั้งค่า `channels.discord.groupPolicy: "disabled"` (หรือคงรายการอนุญาตว่าง)
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**การตรวจสอบสิทธิ์** (`channels status --probe`) จะตรวจสอบเฉพาะ channel ID แบบตัวเลขเท่านั้น **การตรวจสอบสิทธิ์** (`channels status --probe`) ตรวจสอบเฉพาะidช่องตัวเลข หากคุณใช้slug/ชื่อเป็นคีย์ `channels.discord.guilds.*.channels` การตรวจสอบจะยืนยันสิทธิ์ไม่ได้

    ```
    หากคุณใช้คีย์แบบ slug การจับคู่ตอนรันไทม์ยังคงทำงานได้ แต่ probe จะไม่สามารถตรวจสอบสิทธิ์ได้อย่างสมบูรณ์
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DMไม่ทำงาน**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, หรือคุณยังไม่ได้รับการอนุมัติ (`channels.discord.dm.policy="pairing"`)
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    โดยค่าเริ่มต้น ข้อความที่สร้างโดยบอทจะถูกเพิกเฉย

    ```
    หากคุณตั้งค่า `channels.discord.allowBots=true` ให้ใช้กฎการ mention และ allowlist ที่เข้มงวดเพื่อหลีกเลี่ยงพฤติกรรมลูป
    ```

  
</Accordion>
</AccordionGroup>

## จุดอ้างอิงการตั้งค่า

เอกสารอ้างอิงหลัก:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

ฟิลด์ Discord ที่สำคัญ:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- `guilds.<id> .channels.<channel> .allow`: อนุญาต/ปฏิเสธช่องเมื่อ `groupPolicy="allowlist"`
- ใช้ `commands.useAccessGroups: false` เพื่อข้ามการตรวจสอบ access-group สำหรับคำสั่ง
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Safety & ops

- ปฏิบัติต่อโทเค็นบอทเสมือนเป็นความลับ (แนะนำให้ใช้ `DISCORD_BOT_TOKEN` ในสภาพแวดล้อมที่มีการควบคุมดูแล)
- กำหนดสิทธิ์ Discord ตามหลักสิทธิ์น้อยที่สุด (least-privilege)
- หากสถานะการ deploy/state ของคำสั่งล้าสมัย ให้รีสตาร์ท gateway และตรวจสอบอีกครั้งด้วย `openclaw channels status --probe`

## ที่เกี่ยวข้อง

- [การจับคู่](/channels/pairing)
- รับids (guild/user/channel)
- [การแก้ไขปัญหา](/channels/troubleshooting)
- รายการคำสั่งทั้งหมด + คอนฟิก: [Slash commands](/tools/slash-commands)


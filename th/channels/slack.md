---
summary: "การตั้งค่า Slack สำหรับโหมด Socket หรือ HTTP webhook"
read_when:
  - การตั้งค่า Slack หรือการดีบักโหมด Socket/HTTP ของ Slack
title: "Slack"
---

# Slack

สถานะ: พร้อมใช้งานในระดับ production สำหรับ DM และ channels ผ่านการผสานรวม Slack app โหมด HTTP (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">    Slack DMs จะใช้โหมดการจับคู่ (pairing mode) เป็นค่าเริ่มต้น
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">    พฤติกรรมคำสั่งแบบเนทีฟและแคตตาล็อกคำสั่ง
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    คู่มือการวินิจฉัยและซ่อมแซมข้ามช่องทาง
  
</Card>
</CardGroup>

## ตั้งค่าอย่างรวดเร็ว(ผู้เริ่มต้น)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        ในการตั้งค่า Slack app:

        ```
        **Socket Mode** → เปิดใช้งาน **Socket Mode** → เปิดสวิตช์ จากนั้นไปที่ **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** พร้อมสโคป `connections:write` คัดลอก **App Token** (`xapp-...`) คัดลอก **App Token** (`xapp-...`)
        ```

```json5
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

        ```
            Env fallback (เฉพาะบัญชีเริ่มต้นเท่านั้น):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Subscribe app events">
          สมัครรับ bot events สำหรับ:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          และเปิดใช้งาน App Home **Messages Tab** สำหรับ DM
        
</Step>
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        ใช้โหมด HTTP webhook เมื่อ Gateway ของคุณเข้าถึงได้โดย Slack ผ่าน HTTPS (มักใช้กับการติดตั้งบนเซิร์ฟเวอร์)
        โหมด HTTP ใช้ Events API + Interactivity + Slash Commands โดยใช้ URL คำขอร่วมกัน
        10. โหมด HTTP ใช้ Events API + Interactivity + Slash Commands โดยใช้ URL คำขอร่วมกัน.
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

      โหมด HTTP หลายบัญชี: ตั้งค่า `channels.slack.accounts.<id> .mode = "http"` และกำหนด
      `webhookPath` ที่ไม่ซ้ำกันต่อบัญชี เพื่อให้แต่ละแอป Slack ชี้ไปยัง URL ของตนเอง

  
</Tab>
</Tabs>

## โมเดลโทเค็น

- `botToken` + `appToken` จำเป็นสำหรับ Socket Mode
- โหมด HTTP ต้องใช้ `botToken` + `signingSecret`
- โทเค็นในคอนฟิกจะมีลำดับความสำคัญเหนือ env fallback
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback ใช้ได้เฉพาะกับบัญชีเริ่มต้นเท่านั้น
- ตัวอย่างที่ตั้งค่า userTokenReadOnly อย่างชัดเจน(อนุญาตให้ user token เขียน):
- ตัวเลือกเพิ่มเติม: เพิ่ม `chat:write.customize` หากต้องการให้ข้อความขาออกใช้ตัวตนของเอเจนต์ที่กำลังใช้งาน (กำหนด `username` และไอคอนเอง) `icon_emoji` ใช้ไวยากรณ์ `:emoji_name:`

<Tip>
สำหรับการดำเนินการ/การอ่านไดเรกทอรี สามารถเลือกใช้ user token เมื่อมีการกำหนดค่า แม้ตั้งค่า `userTokenReadOnly: false` แล้ว bot token ก็ยังคง
ถูกเลือกใช้เป็นหลักสำหรับการเขียนเมื่อมีให้ใช้งาน.
</Tip>

## การควบคุมการเข้าถึงและการกำหนดเส้นทาง

<Tabs>
  <Tab title="DM policy">DM ถูกละเลย: ผู้ส่งยังไม่ได้รับอนุมัติเมื่อ `channels.slack.dm.policy="pairing"`

    ```
    หากคุณตั้งค่าเพียง `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` และไม่เคยสร้างส่วน `channels.slack`,
      ค่าเริ่มต้นขณะรันจะตั้ง `groupPolicy` เป็น `open` เพิ่ม `channels.slack.groupPolicy`,
    `channels.defaults.groupPolicy` หรือรายการอนุญาตช่องเพื่อจำกัดให้แน่นขึ้น 24. เพิ่ม `channels.slack.groupPolicy`,
    `channels.defaults.groupPolicy` หรือ allowlist ของช่องเพื่อจำกัดการใช้งาน.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` ควบคุมการจัดการช่อง (`open|disabled|allowlist`)

    ```
    เพื่ออนุญาตทุกคน: ตั้งค่า `channels.slack.dm.policy="open"` และ `channels.slack.dm.allowFrom=["*"]`
    ```

  
</Tab>

  <Tab title="Mentions and channel users">    โดยค่าเริ่มต้น ข้อความใน channel จะต้องมีการ mention ก่อนจึงจะทำงาน


    ```
    การควบคุมด้วยการกล่าวถึงตั้งค่าผ่าน `channels.slack.channels` (ตั้งค่า `requireMention` เป็น `true`); `agents.list[].groupChat.mentionPatterns` (หรือ `messages.groupChat.mentionPatterns`) นับเป็นการกล่าวถึงเช่นกัน
    ```

  
</Tab>
</Tabs>

## คำสั่งและพฤติกรรมของ slash

- คำสั่งแบบข้อความต้องเป็นข้อความ `/...` แบบเดี่ยว และสามารถปิดได้ด้วย `commands.text: false`. Slack slash commands are managed in the Slack app and are not removed automatically.
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- เมื่อเปิดใช้งานคำสั่งแบบเนทีฟ ให้ลงทะเบียน slash commands ที่ตรงกันใน Slack (ชื่อ `/<command>`)
- หากเปิดใช้คำสั่งเนทีฟ ให้เพิ่มรายการ `slash_commands` หนึ่งรายการต่อคำสั่งที่ต้องการเผยแพร่(ต้องตรงกับรายการ `/help`) สามารถเขียนทับด้วย `channels.slack.commands.native` 13. เขียนทับด้วย `channels.slack.commands.native`.

การตั้งค่าเริ่มต้นของ slash command:

- `enabled`: ตั้งค่า `false` เพื่อปิดใช้งานช่อง
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

เซสชันของ slash จะใช้คีย์ที่แยกกัน:

- Slash commands ใช้เซสชัน `agent:<agentId>:slack:slash:<userId>` (ตั้งค่าพรีฟิกซ์ได้ผ่าน `channels.slack.slashCommand.sessionPrefix`)

และยังคงกำหนดเส้นทางการเรียกใช้คำสั่งไปยังเซสชันการสนทนาเป้าหมาย (`CommandTargetSessionKey`)

## เธรด, เซสชัน และแท็กการตอบกลับ

- DM จะถูกกำหนดเส้นทางเป็น `direct`; channel เป็น `channel`; MPIM เป็น `group`
- ด้วยค่าเริ่มต้น `session.dmScope=main`, Slack DM จะถูกรวมเข้าเป็นเซสชันหลักของเอเจนต์
- ช่องแมปเป็นเซสชัน `agent:<agentId>:slack:channel:<channelId>`
- การตอบกลับในเธรดสามารถสร้าง suffix ของเซสชันเธรด (`:thread:<threadTs>`) ได้เมื่อเหมาะสม
- `channels.slack.thread.historyScope` ค่าเริ่มต้นคือ `thread`; `thread.inheritParent` ค่าเริ่มต้นคือ `false`
- `channels.slack.thread.initialHistoryLimit` กำหนดจำนวนข้อความในเธรดเดิมที่จะดึงมาเมื่อเริ่มเซสชันเธรดใหม่ (ค่าเริ่มต้น `20`; ตั้งค่าเป็น `0` เพื่อปิดการทำงาน)

การตอบกลับแบบเธรด

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- fallback แบบ legacy สำหรับแชทโดยตรง: `channels.slack.dm.replyToMode`

รองรับแท็กการตอบกลับแบบกำหนดเอง:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — ตอบกลับไปยังข้อความที่มี message id ระบุ

หมายเหตุ: การตั้งค่า `replyToMode="off"` จะปิดการทำงานของการตอบกลับแบบเธรดโดยอัตโนมัติ แท็ก `[[reply_to_*]]` แบบระบุชัดเจนยังคงได้รับการรองรับ

## สื่อ, การแบ่งข้อความ (chunking) และการจัดส่ง

<AccordionGroup>
  <Accordion title="Inbound attachments">ไฟล์แนบจะถูกดาวน์โหลดไปยังคลังสื่อเมื่อได้รับอนุญาตและมีขนาดไม่เกินขีดจำกัด

    ```
    การอัปโหลดสื่อจำกัดโดย `channels.slack.mediaMaxMb` (ค่าเริ่มต้น 20)
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - การแบ่งข้อความ (text chunks) ใช้ `channels.slack.textChunkLimit` (ค่าเริ่มต้น 4000)
    - `channels.slack.chunkMode="newline"` เปิดใช้งานการแยกตามย่อหน้าก่อน
    - การส่งไฟล์ใช้ Slack upload APIs และสามารถรวมการตอบกลับในเธรด (`thread_ts`) ได้
    - ขีดจำกัดสื่อขาออกเป็นไปตาม `channels.slack.mediaMaxMb` เมื่อมีการกำหนดค่า; มิฉะนั้น การส่งของช่องจะใช้ค่าเริ่มต้นตามชนิด MIME จาก media pipeline
  
</Accordion>

  <Accordion title="Delivery targets">
    เป้าหมายที่ระบุชัดเจนที่แนะนำ:

    ```
    `im:write` (เปิด DM ผ่าน `conversations.open` สำหรับ DM ผู้ใช้)
    [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
    ```

  
</Accordion>
</AccordionGroup>

## การดำเนินการและเกต

การกระทำของเครื่องมือ Slack สามารถจำกัดได้ด้วย `channels.slack.actions.*`:

กลุ่มการดำเนินการที่มีในเครื่องมือ Slack ปัจจุบัน:

| Action group | Default |
| ------------ | ------- |
| messages     | enabled |
| reactions    | enabled |
| pins         | enabled |
| memberInfo   | enabled |
| emojiList    | enabled |

## Behavior

- การแก้ไข/ลบข้อความ และการกระจายข้อความในเธรด จะถูกแมปเป็น system events
- เหตุการณ์เพิ่ม/ลบรีแอคชัน จะถูกแมปเป็น system events
- เหตุการณ์สมาชิกเข้า/ออก, การสร้าง/เปลี่ยนชื่อช่อง, และการปักหมุด/ยกเลิกปักหมุด จะถูกแมปเป็น system events
- `channel_id_changed` สามารถย้ายคีย์การตั้งค่าช่องได้เมื่อเปิดใช้งาน `configWrites`
- ข้อมูลเมทาดาทาหัวข้อ/วัตถุประสงค์ของช่องจะถูกมองว่าเป็นบริบทที่ไม่น่าเชื่อถือ และสามารถถูกฉีดเข้าไปในบริบทการกำหนดเส้นทางได้

## `reactions:read`

`ackReaction` จะส่งอีโมจิยืนยันในขณะที่ OpenClaw กำลังประมวลผลข้อความขาเข้า

ลำดับการพิจารณา:

- `หรือ`channels.slack.channels.<name>`.ackReaction`
- `channels.slack.ackReaction`
- `replyToMode`
- อีโมจิสำรองจากตัวตนเอเจนต์ (`agents.list[].identity.emoji`, มิฉะนั้นใช้ "👀")

Notes

- Slack คาดหวัง shortcodes (เช่น `"eyes"`)
- ใช้ `""` เพื่อปิดใช้งานรีแอคชันสำหรับช่องหรือบัญชี

## รายการตรวจสอบ Manifest และขอบเขตสิทธิ์ (scope)

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
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
        "description": "Send a message to OpenClaw",
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
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
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
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
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
    หากคุณกำหนดค่า `channels.slack.userToken` ขอบเขตการอ่าน (read scopes) ทั่วไปคือ:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## การแก้ไขปัญหา

<AccordionGroup>
  <Accordion title="No replies in channels">
    ตรวจสอบตามลำดับ:

    ```
    `users`: รายการอนุญาตผู้ใช้รายช่อง(ไม่บังคับ)
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    ตรวจสอบ:

    ```
    `allowlist` กำหนดให้ช่องต้องอยู่ในรายการ `channels.slack.channels`
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">สร้างแอป Slack และเปิดใช้งาน **Socket Mode**
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    ตรวจสอบความถูกต้อง:

    ```
    **Event Subscriptions** → เปิดใช้งานอีเวนต์และตั้งค่า **Request URL** เป็นพาธ webhook ของ Gateway (ค่าเริ่มต้น `/slack/events`)
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    ยืนยันว่าคุณตั้งใจ:

    ```
    การลงทะเบียนคำสั่งเนทีฟใช้ `commands.native` (ค่าเริ่มต้นระดับ global คือ `"auto"` → ปิด Slack) และสามารถเขียนทับต่อเวิร์กสเปซด้วย `channels.slack.commands.native` คำสั่งแบบข้อความต้องเป็นข้อความ `/...` แบบสแตนด์อโลน และสามารถปิดได้ด้วย `commands.text: false` Slack slash commands จัดการในแอป Slack และจะไม่ถูกลบอัตโนมัติ ใช้ `commands.useAccessGroups: false` เพื่อข้ามการตรวจสอบกลุ่มการเข้าถึงสำหรับคำสั่ง 20.
    ```

  
</Accordion>
</AccordionGroup>

## จุดอ้างอิงการตั้งค่า

ลำดับความสำคัญ:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  ฟิลด์ Slack ที่สำคัญ:

  - โหมด/การยืนยันตัวตน: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - การเข้าถึง DM: `dm.enabled`, `dmPolicy`, `allowFrom` (เดิม: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - เพื่อ **ไม่อนุญาตช่องใดเลย** ให้ตั้งค่า `channels.slack.groupPolicy: "disabled"` (หรือคงรายการอนุญาตว่างไว้)
  - เธรด/ประวัติ: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - การจัดส่ง: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - การปฏิบัติการ/ฟีเจอร์: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## ที่เกี่ยวข้อง

- [Pairing](/channels/pairing)
- `channel`: ช่องมาตรฐาน(สาธารณะ/ส่วนตัว)
- โฟลว์สำหรับการไตรอาจ: [/channels/troubleshooting](/channels/troubleshooting)
- [Configuration](/gateway/configuration)
- รายการคำสั่งทั้งหมด + คอนฟิก: [Slash commands](/tools/slash-commands)


---
summary: "สถานะการรองรับบอต Telegram ความสามารถ และการกำหนดค่า"
read_when:
  - ทำงานเกี่ยวกับฟีเจอร์ Telegram หรือ webhook
title: "Telegram"
---

# Telegram (Bot API)

สถานะ: พร้อมใช้งานระดับโปรดักชันสำหรับบอต DMs + กลุ่มผ่าน grammY ค่าเริ่มต้นเป็น long-polling; webhook เป็นตัวเลือก 27. ค่าเริ่มต้นเป็น long-polling; webhook เป็นตัวเลือก.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    นโยบาย DM เริ่มต้นสำหรับ Telegram คือการจับคู่ (pairing)
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    การวินิจฉัยข้ามช่องทางและคู่มือการแก้ไขปัญหา
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">    รูปแบบการตั้งค่า channel แบบครบถ้วนและตัวอย่าง
  
</Card>
</CardGroup>

## การตั้งค่า(ทางลัด)

<Steps>
  <Step title="Create the bot token in BotFather">เปิด Telegram และแชตกับ **@BotFather** ([ลิงก์ตรง](https://t.me/BotFather)) ยืนยันว่า handle เป็น `@BotFather` ตรงตัว 29.

    ```
    รัน `/newbot` แล้วทำตามคำแนะนำ (ชื่อ + ชื่อผู้ใช้ลงท้ายด้วย `bot`)
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Env fallback: `TELEGRAM_BOT_TOKEN=...` (เฉพาะบัญชีค่าเริ่มต้นเท่านั้น)
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    รหัสการจับคู่จะหมดอายุภายใน 1 ชั่วโมง
    ```

  
</Step>

  <Step title="Add the bot to a group">    เพิ่มบอทเข้าไปในกลุ่มของคุณ จากนั้นตั้งค่า `channels.telegram.groups` และ `groupPolicy` ให้สอดคล้องกับโมเดลการเข้าถึงของคุณ
  
</Step>
</Steps>

<Note>
ลำดับการเลือกใช้โทเค็นจะแยกตามบัญชี ในทางปฏิบัติ ค่าจาก config จะมีลำดับความสำคัญเหนือ env fallback และ `TELEGRAM_BOT_TOKEN` จะมีผลกับบัญชีค่าเริ่มต้นเท่านั้น
</Note>

## การตั้งค่าฝั่ง Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">เพิ่มบอตเป็น **แอดมิน** ของกลุ่ม (บอตแอดมินจะได้รับข้อความทั้งหมด)

    ```
    **หมายเหตุ:** เมื่อสลับ privacy mode Telegram ต้องการให้ลบแล้วเพิ่มบอตใหม่
    ในแต่ละกลุ่มเพื่อให้การเปลี่ยนแปลงมีผล
    ```

  
</Accordion>

  <Accordion title="Group permissions">สถานะผู้ดูแลตั้งค่าภายในกลุ่ม (ผ่าน UI ของ Telegram).

    ```
    บอทที่เป็นผู้ดูแลจะได้รับข้อความกลุ่มทั้งหมดเสมอ
        ดังนั้นให้ใช้สถานะผู้ดูแลหากต้องการการมองเห็นแบบเต็ม.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — อนุญาต/ปฏิเสธการเพิ่มบอตเข้ากลุ่ม
    ```

  
</Accordion>
</AccordionGroup>

## การควบคุมการเข้าถึงและการเปิดใช้งาน

<Tabs>
  <Tab title="DM policy">ควบคุมด้วย `channels.telegram.replyToMode`:

    ```
    - `pairing` (ค่าเริ่มต้น)
    - `allowlist`
    - `open` (ต้องตั้งค่า `allowFrom` ให้มี `"*"` รวมอยู่ด้วย)
    - `disabled`
    
    `channels.telegram.allowFrom` รองรับ Telegram user ID แบบตัวเลข สามารถใช้คำนำหน้า `telegram:` / `tg:` ได้ และระบบจะปรับให้อยู่ในรูปแบบมาตรฐาน
    ตัวช่วยตั้งค่าเริ่มต้น (onboarding wizard) รองรับการกรอก `@username` และจะแปลงเป็น ID ตัวเลขให้
    หากคุณอัปเกรดแล้วและใน config ยังมีรายการ allowlist แบบ `@username` ให้รัน `openclaw doctor --fix` เพื่อแปลงเป็น ID ตัวเลข (ทำแบบ best-effort; ต้องมี Telegram bot token)
    
    ### การค้นหา Telegram user ID ของคุณ
    
    วิธีที่ปลอดภัยกว่า (ไม่ใช้บอทบุคคลที่สาม):
    
    1. ส่งข้อความส่วนตัว (DM) ไปยังบอทของคุณ
    2. รัน `openclaw logs --follow`
    3. ดูค่า `from.id`
    
    วิธีผ่าน Official Bot API:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    DM `@userinfobot` หรือ `@getidsbot` และใช้ user id ที่ได้
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">มีการควบคุมอิสระสองส่วน:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
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

  <Tab title="Mention behavior">การตอบในกลุ่มต้องมีการกล่าวถึงเป็นค่าเริ่มต้น (native @mention หรือ `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`)

    ```
    การกล่าวถึง (mention) อาจมาจาก:
    
    - การ mention แบบ native `@botusername` หรือ
    - รูปแบบการ mention ที่กำหนดใน:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    คำสั่งสลับโหมดระดับเซสชัน:
    
    - `/activation always`
    - `/activation mention`
    
    คำสั่งเหล่านี้จะอัปเดตเฉพาะสถานะของเซสชัน ใช้ config หากต้องการให้มีผลถาวร
    
    ตัวอย่าง config แบบถาวร:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    ส่งต่อข้อความใดๆ จากกลุ่มไปยัง `@userinfobot` หรือ `@getidsbot` บน Telegram เพื่อดู chat ID (ตัวเลขติดลบเช่น `-1001234567890`)
    ```

  
</Tab>
</Tabs>

## พฤติกรรมขณะรันไทม์

- ช่องทาง Telegram Bot API ที่ Gateway เป็นเจ้าของ
- การกำหนดเส้นทางแบบกำหนดแน่นอน: การตอบกลับจะกลับไปที่ Telegram เสมอ โมเดลจะไม่เลือกช่องทางเอง
- ข้อความขาเข้าได้รับการปรับให้อยู่ในซองช่องทางร่วม พร้อมบริบทการตอบกลับและตัวแทนสื่อ
- การดึง group chat ID .topics.<threadId>
- ส่งตัวบ่งชี้การพิมพ์และการตอบกลับด้วย `message_thread_id` เพื่อให้การตอบอยู่ในหัวข้อ
- Long polling ใช้ grammY runner พร้อมการจัดลำดับต่อเนื่องแยกตามแชท/เธรด long-polling ใช้ grammY runner พร้อมการจัดลำดับต่อแชต; คอนเคอร์เรนซีรวมถูกจำกัดโดย `agents.defaults.maxConcurrent`
- Telegram Bot API ไม่รองรับ read receipts; ไม่มีตัวเลือก `sendReadReceipts`

## เอกสารอ้างอิงฟีเจอร์

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw สามารถสตรีมการตอบกลับบางส่วนใน Telegram DMs โดยใช้ `sendMessageDraft`

    ```
    ข้อกำหนด:
    
    - `channels.telegram.streamMode` ต้องไม่เป็น `"off"` (ค่าเริ่มต้น: `"partial"`)
    
    โหมด:
    
    - `off`: ไม่มีการแสดงตัวอย่างแบบเรียลไทม์
    - `partial`: อัปเดตตัวอย่างบ่อยครั้งจากข้อความที่ยังไม่สมบูรณ์
    - `block`: อัปเดตตัวอย่างแบบเป็นก้อนโดยใช้ `channels.telegram.draftChunk`
    
    ค่าเริ่มต้นของ `draftChunk` สำหรับ `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` จะถูกจำกัดตามค่า `channels.telegram.textChunkLimit`
    
    ทำงานได้ทั้งในแชทส่วนตัวและกลุ่ม/หัวข้อ
    
    สำหรับข้อความล้วน OpenClaw จะใช้ข้อความตัวอย่างเดิมและแก้ไขครั้งสุดท้ายในข้อความเดิม (ไม่มีการส่งข้อความที่สอง)
    
    สำหรับข้อความที่ซับซ้อน (เช่น มี media payload) OpenClaw จะกลับไปใช้การส่งผลลัพธ์แบบปกติ แล้วจึงลบข้อความตัวอย่าง
    
    `streamMode` แยกจาก block streaming หากเปิด block streaming สำหรับ Telegram อย่างชัดเจน OpenClaw จะข้ามการแสดงตัวอย่างเพื่อหลีกเลี่ยงการสตรีมซ้ำซ้อน
    
    การสตรีม reasoning เฉพาะ Telegram:
    
    - `/reasoning stream` จะส่ง reasoning ไปยังตัวอย่างแบบเรียลไทม์ระหว่างการสร้างคำตอบ
    - คำตอบสุดท้ายจะถูกส่งโดยไม่มีข้อความ reasoning รวมอยู่ด้วย
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">ข้อความขาออกของ Telegram ใช้ `parse_mode: "HTML"` (ชุดแท็กที่ Telegram รองรับ)

    ```
    - ข้อความสไตล์ Markdown จะถูกแปลงเป็น HTML ที่ปลอดภัยสำหรับ Telegram
    - HTML ดิบจากโมเดลจะถูก escape เพื่อลดความล้มเหลวในการ parse ของ Telegram
    - หาก Telegram ปฏิเสธ HTML ที่ parse แล้ว OpenClaw จะลองส่งใหม่เป็นข้อความธรรมดา
    
    การแสดงตัวอย่างลิงก์เปิดใช้งานตามค่าเริ่มต้น และสามารถปิดได้ด้วย `channels.telegram.linkPreview: false`
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">    การลงทะเบียนเมนูคำสั่ง Telegram จะดำเนินการตอนเริ่มต้นระบบด้วย `setMyCommands`

    ```
    OpenClaw ลงทะเบียนคำสั่งเนทีฟ (เช่น `/status`, `/reset`, `/model`) กับเมนูบอตของ Telegram เมื่อเริ่มต้น
    คุณสามารถเพิ่มคำสั่งกำหนดเองลงเมนูผ่านคอนฟิก:
    34. คุณสามารถเพิ่มคำสั่งกำหนดเองลงในเมนูผ่านคอนฟิก:
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
    กฎ:
    
    - ชื่อจะถูกปรับรูปแบบ (ตัด `/` ด้านหน้าออก และแปลงเป็นตัวพิมพ์เล็ก)
    - รูปแบบที่ถูกต้อง: `a-z`, `0-9`, `_`, ความยาว `1..32`
    - คำสั่งแบบกำหนดเองไม่สามารถแทนที่คำสั่ง native ได้
    - รายการที่ซ้ำหรือขัดแย้งจะถูกข้ามและบันทึก log
    
    หมายเหตุ:
    
    - คำสั่งแบบกำหนดเองเป็นเพียงรายการในเมนูเท่านั้น ไม่ได้มีการติดตั้งพฤติกรรมให้อัตโนมัติ
    - คำสั่งจาก plugin/skill ยังสามารถใช้งานได้เมื่อพิมพ์ แม้จะไม่แสดงในเมนู Telegram
    
    หากปิดคำสั่ง native คำสั่ง built-in จะถูกลบออก คำสั่งแบบกำหนดเอง/จาก plugin ยังสามารถลงทะเบียนได้หากมีการตั้งค่า
    
    ปัญหาที่พบบ่อยในการตั้งค่า:
    
    - `setMyCommands failed` มักหมายถึง DNS/HTTPS ขาออกไปยัง `api.telegram.org` ถูกบล็อก
    
    ### คำสั่งจับคู่อุปกรณ์ (`device-pair` plugin)
    
    เมื่อมีการติดตั้ง `device-pair` plugin:
    
    1. `/pair` สร้างรหัสตั้งค่า
    2. วางรหัสในแอป iOS
    3. `/pair approve` อนุมัติคำขอที่รอดำเนินการล่าสุด
    
    รายละเอียดเพิ่มเติม: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">    ตั้งค่าขอบเขตของ inline keyboard:

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
    สำหรับการกำหนดค่าต่อบัญชี:
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
    `"disabled"` = ไม่รับข้อความกลุ่มเลย
      ค่าเริ่มต้นคือ `groupPolicy: "allowlist"` (บล็อกจนกว่าจะเพิ่ม `groupAllowFrom`)
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
    เมื่อผู้ใช้คลิกปุ่ม ข้อมูล callback จะถูกส่งกลับไปยังเอเจนต์เป็นข้อความรูปแบบ:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">    การทำงานของเครื่องมือ Telegram ประกอบด้วย:

    ```
    เครื่องมือ: `telegram` พร้อมแอ็กชัน `react` (`chatId`, `messageId`, `emoji`)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram รองรับการตอบแบบเธรดผ่านแท็ก:

    ```
    - `[[reply_to_current]]` ตอบกลับข้อความที่เป็นตัวกระตุ้น
    - `[[reply_to:<id>]]` ตอบกลับ Telegram message ID ที่ระบุ
    
    `channels.telegram.replyToMode` ควบคุมพฤติกรรม:
    
    - `off` (ค่าเริ่มต้น)
    - `first`
    - `all`
    
    หมายเหตุ: `off` จะปิดการผูกเธรดตอบกลับโดยอัตโนมัติ แต่แท็ก `[[reply_to_*]]` แบบระบุชัดยังคงทำงาน
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">หัวข้อ (forum supergroups)

    ```
    - คีย์เซสชันของหัวข้อจะต่อท้ายด้วย `:topic:<threadId>`
    - การตอบกลับและการแสดงสถานะกำลังพิมพ์จะกำหนดเป้าหมายไปยังเธรดของหัวข้อ
    - path config ของหัวข้อ:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    กรณีพิเศษของหัวข้อทั่วไป (`threadId=1`):
    
    - การส่งข้อความจะไม่ใส่ `message_thread_id` (Telegram จะปฏิเสธ `sendMessage(...thread_id=1)`)
    - แต่การแสดงสถานะกำลังพิมพ์ยังคงใส่ `message_thread_id`
    
    การสืบทอดค่าของหัวข้อ: รายการหัวข้อจะสืบทอดการตั้งค่าจากกลุ่ม เว้นแต่จะมีการ override (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`)
    
    Template context ประกอบด้วย:
    
    - `MessageThreadId`
    - `IsForum`
    
    พฤติกรรมเธรดใน DM:
    
    - แชทส่วนตัวที่มี `message_thread_id` จะยังคง route แบบ DM แต่ใช้คีย์เซสชัน/เป้าหมายการตอบกลับที่รองรับเธรด
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">    ### ข้อความเสียง

    ```
    `[[audio_as_voice]]` — ส่งเสียงเป็น voice note แทนไฟล์
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
    ### ข้อความวิดีโอ
    
    Telegram แยกความแตกต่างระหว่างไฟล์วิดีโอและวิดีโอโน้ต
    
    ตัวอย่าง message action:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    วิดีโอโน้ตไม่รองรับคำบรรยาย (caption); ข้อความที่ให้มาจะถูกส่งแยกต่างหาก
    
    ### สติกเกอร์
    
    การจัดการสติกเกอร์ขาเข้า:
    
    - WEBP แบบ static: ดาวน์โหลดและประมวลผล (placeholder `<media:sticker>`)
    - TGS แบบเคลื่อนไหว: ข้าม
    - WEBM แบบวิดีโอ: ข้าม
    
    ฟิลด์บริบทของสติกเกอร์:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    ไฟล์แคชสติกเกอร์:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    สติกเกอร์จะถูกอธิบายหนึ่งครั้ง (เมื่อเป็นไปได้) และแคชไว้เพื่อลดการเรียก vision ซ้ำ
    
    เปิดใช้งานการทำงานกับสติกเกอร์:
    ```

```json5
{
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
    การส่งสติกเกอร์
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    แคชสติกเกอร์
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

  <Accordion title="Reaction notifications">รับอัปเดต `message_reaction` จาก Telegram API

    ```
    เมื่อเปิดใช้งาน OpenClaw จะเพิ่ม system events ลงคิว เช่น:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    การตั้งค่า:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (ค่าเริ่มต้น: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (ค่าเริ่มต้น: `minimal`)
    
    หมายเหตุ:
    
    - `own` หมายถึงปฏิกิริยาของผู้ใช้ต่อข้อความที่บอทส่งเท่านั้น (best-effort ผ่านแคชข้อความที่ส่งแล้ว)
    - Telegram ไม่ได้ให้ thread ID ในอัปเดต reaction
      - กลุ่มที่ไม่ใช่ฟอรัมจะ route ไปยังเซสชันแชทกลุ่ม
      - กลุ่มฟอรัมจะ route ไปยังเซสชันหัวข้อทั่วไปของกลุ่ม (`:topic:1`) ไม่ใช่หัวข้อที่มาที่แท้จริง
    
    `allowed_updates` สำหรับ polling/webhook จะรวม `message_reaction` โดยอัตโนมัติ
    ```

  
</Accordion>

  <Accordion title="Ack reactions">    `ackReaction` จะส่งอีโมจิยืนยันระหว่างที่ OpenClaw กำลังประมวลผลข้อความขาเข้า

    ```
    ลำดับการเลือกใช้ค่า:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - fallback เป็นอีโมจิจาก agent identity (`agents.list[].identity.emoji`, หากไม่มีใช้ "👀")
    
    หมายเหตุ:
    
    - Telegram ต้องการอีโมจิแบบ unicode (เช่น "👀")
    - ใช้ `""` เพื่อปิด reaction สำหรับ channel หรือบัญชี
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">    การเขียนค่า config ของ channel เปิดใช้งานตามค่าเริ่มต้น (`configWrites !== false`)

    ```
    คุณรัน `/config set` หรือ `/config unset` ในแชต Telegram (ต้องมี `commands.config: true`)
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">ค่าเริ่มต้น: long-polling (ไม่ต้องมี URL สาธารณะ)

    ```
    หาก URL สาธารณะต่างออกไป ให้ใช้ reverse proxy และชี้ `channels.telegram.webhookUrl` ไปยังปลายทางสาธารณะ
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    ข้อความขาออกถูกแบ่งเป็นชิ้นขนาด `channels.telegram.textChunkLimit` (ค่าเริ่มต้น 4000)
    การแบ่งตามบรรทัดใหม่ (ไม่บังคับ): ตั้งค่า `channels.telegram.chunkMode="newline"` เพื่อแบ่งตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนแบ่งตามความยาว
    การดาวน์โหลด/อัปโหลดสื่อจำกัดที่ `channels.telegram.mediaMaxMb` (ค่าเริ่มต้น 5)
    - `channels.telegram.timeoutSeconds` ใช้ override ค่า timeout ของ Telegram API client (หากไม่ตั้งค่า จะใช้ค่าเริ่มต้นของ grammY)
    บริบทประวัติกลุ่มใช้ `channels.telegram.historyLimit` (หรือ `channels.telegram.accounts.*.historyLimit`) โดย fallback เป็น `messages.groupChat.historyLimit` ตั้งค่า `0` เพื่อปิด (ค่าเริ่มต้น 50) ตั้งค่า `0` เพื่อปิด (ค่าเริ่มต้น 50)
    ประวัติ DM สามารถจำกัดด้วย `channels.telegram.dmHistoryLimit` (จำนวนรอบผู้ใช้) การ override ต่อผู้ใช้: `channels.telegram.dms["<user_id>"].historyLimit` `channels.signal.dmHistoryLimit`: ขีดจำกัดประวัติ DM ในรอบผู้ใช้ การเขียนทับต่อผู้ใช้: `channels.telegram.dms["<user_id>"].historyLimit`<user_id>"].historyLimit`
    - การ retry สำหรับ Telegram API ขาออกสามารถตั้งค่าได้ผ่าน `channels.telegram.retry`

    ```
    CLI send target สามารถเป็น chat ID แบบตัวเลขหรือ username:
    ```

```bash
ตัวอย่าง: `openclaw message send --channel telegram --target 123456789 --message "hi"`
```

  
</Accordion>
</AccordionGroup>

## การแก้ไขปัญหา

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    หากตั้งค่า `channels.telegram.groups.*.requireMention=false` ต้องปิด **privacy mode** ของ Bot API ของ Telegram
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - เมื่อมี `channels.telegram.groups` อยู่ ต้องระบุกลุ่มนั้น (หรือใส่ `"*"`)
    - ตรวจสอบว่าบอทเป็นสมาชิกในกลุ่ม
    - ตรวจสอบ log: `openclaw logs --follow` เพื่อดูสาเหตุที่ถูกข้าม
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - อนุญาตตัวตนผู้ส่งของคุณ (pairing และ/หรือ `allowFrom` แบบตัวเลข)
    - การอนุญาตคำสั่งยังคงมีผลแม้ `groupPolicy` จะเป็น `open`
    - `setMyCommands failed` มักบ่งชี้ปัญหาการเข้าถึง DNS/HTTPS ไปยัง `api.telegram.org`
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ พร้อม custom fetch/proxy อาจทำให้เกิดการยกเลิกทันที หากชนิดของ AbortSignal ไม่ตรงกัน
    - บางโฮสต์ resolve `api.telegram.org` ไปยัง IPv6 ก่อน; หาก IPv6 ขาออกมีปัญหา อาจทำให้ Telegram API ล้มเหลวเป็นบางครั้ง
    - ตรวจสอบคำตอบ DNS:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

ความช่วยเหลือเพิ่มเติม: [การแก้ไขปัญหาช่องทาง](/channels/troubleshooting)

## อ้างอิงการกำหนดค่า (Telegram)

Primary reference:

- `channels.telegram.enabled`: เปิด/ปิดการเริ่มต้นช่องทาง

- `channels.telegram.botToken`: โทเคนบอต (BotFather)

- `channels.telegram.tokenFile`: อ่านโทเคนจากพาธไฟล์

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (ค่าเริ่มต้น: การจับคู่)

- `channels.telegram.allowFrom`: allowlist DM (ids/ชื่อผู้ใช้) `open` ต้องใช้ `"*"` `open` ต้องใช้ `"*"`. `open` ต้องใช้ `"*"`. `openclaw doctor --fix` สามารถแก้ไขรายการ `@username` แบบเดิมให้เป็น IDs ได้

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (ค่าเริ่มต้น: allowlist)

- `channels.telegram.groupAllowFrom`: allowlist ผู้ส่งในกลุ่ม (ids/ชื่อผู้ใช้) `openclaw doctor --fix` สามารถแก้ไขรายการ `@username` แบบเดิมให้เป็น IDs ได้

- `channels.telegram.groups`: ค่าเริ่มต้นต่อกลุ่ม + allowlist (ใช้ `"*"` สำหรับค่าเริ่มต้นโกลบอล)
  - `channels.telegram.groups.<id>.groupPolicy`: override ต่อกลุ่มสำหรับ groupPolicy (`open | allowlist | disabled`)
  - `channels.telegram.groups.<id>.requireMention`: ค่าเริ่มต้น mention gating
  - `channels.telegram.groups.<id>.skills`: ตัวกรอง skill (ไม่ระบุ = ทุก Skills, ว่าง = ไม่มี)
  - `channels.telegram.groups.<id>.allowFrom`: override allowlist ผู้ส่งต่อกลุ่ม
  - `channels.telegram.groups.<id>.systemPrompt`: system prompt เพิ่มเติมสำหรับกลุ่ม
  - `channels.telegram.groups.<id>.enabled`: ปิดการใช้งานกลุ่มเมื่อ `false`
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: override ต่อหัวข้อ (ฟิลด์เดียวกับกลุ่ม)
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: override ต่อหัวข้อสำหรับ groupPolicy (`open | allowlist | disabled`)
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: override mention gating ต่อหัวข้อ

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (ค่าเริ่มต้น: allowlist)

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: override ต่อบัญชี

- `channels.telegram.replyToMode`: `off | first | all` (ค่าเริ่มต้น: `first`)

- `channels.telegram.textChunkLimit`: ขนาดชิ้นขาออก (ตัวอักษร)

- `channels.telegram.chunkMode`: `length` (ค่าเริ่มต้น) หรือ `newline` เพื่อแบ่งตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนแบ่งตามความยาว

- `channels.telegram.linkPreview`: สลับการแสดงตัวอย่างลิงก์สำหรับข้อความขาออก (ค่าเริ่มต้น: true)

- `channels.telegram.streamMode`: `off | partial | block` (draft streaming)

- `channels.telegram.mediaMaxMb`: ขีดจำกัดสื่อขาเข้า/ขาออก (MB)

- `channels.telegram.retry`: นโยบายลองใหม่สำหรับการเรียก Telegram API ขาออก (จำนวนครั้ง, minDelayMs, maxDelayMs, jitter)

- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=เปิด, false=ปิด) ค่าเริ่มต้นปิดบน Node 22 เพื่อหลีกเลี่ยงเวลาเกิน Happy Eyeballs `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=เปิด, false=ปิด) ค่าเริ่มต้นปิดบน Node 22 เพื่อหลีกเลี่ยงเวลาเกิน Happy Eyeballs Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`: URL พร็อกซีสำหรับการเรียก Bot API (SOCKS/HTTP)

- `channels.telegram.webhookUrl`: เปิดโหมด webhook (ต้องใช้ `channels.telegram.webhookSecret`)

- `channels.telegram.webhookSecret`: webhook secret (จำเป็นเมื่อกำหนด webhookUrl)

- `channels.telegram.webhookPath`: พาธ webhook ภายในเครื่อง (ค่าเริ่มต้น `/telegram-webhook`)

- ตัวรับฟังในเครื่อง bind ที่ `0.0.0.0:8787` และให้บริการ `POST /telegram-webhook` เป็นค่าเริ่มต้น

- `channels.telegram.actions.reactions`: ควบคุมปฏิกิริยาเครื่องมือ Telegram

- `channels.telegram.actions.sendMessage`: ควบคุมการส่งข้อความเครื่องมือ Telegram

- `channels.telegram.actions.deleteMessage`: ควบคุมการลบข้อความเครื่องมือ Telegram

- `channels.telegram.actions.sticker`: ควบคุมแอ็กชันสติกเกอร์ Telegram — ส่งและค้นหา (ค่าเริ่มต้น: false)

- `channels.telegram.reactionNotifications`: `off | own | all` — ควบคุมว่าปฏิกิริยาใดกระตุ้น system events (ค่าเริ่มต้น: `own` เมื่อไม่ตั้งค่า)

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — ควบคุมความสามารถปฏิกิริยาของเอเจนต์ (ค่าเริ่มต้น: `minimal` เมื่อไม่ตั้งค่า)

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

ฟิลด์สำคัญเฉพาะของ Telegram:

- startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- การควบคุมการเข้าถึง(DMs+กลุ่ม)
- command/menu: `commands.native`, `customCommands`
- threading/replies: `replyToMode`
- ตัวเลือก (เฉพาะ `streamMode: "block"`):
- formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- โหมด webhook: ตั้งค่า `channels.telegram.webhookUrl` และ `channels.telegram.webhookSecret` (เลือกตั้ง `channels.telegram.webhookPath`)
- actions/capabilities: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- การแจ้งเตือนปฏิกิริยา
- writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## ที่เกี่ยวข้อง

- [Pairing](/channels/pairing)
- การสตรีม (ดราฟต์)
- การแก้ไขปัญหาการตั้งค่า (คำสั่ง)

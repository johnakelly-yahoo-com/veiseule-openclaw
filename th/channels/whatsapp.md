---
summary: "การรองรับช่องทาง WhatsApp, การควบคุมการเข้าถึง, พฤติกรรมการส่งข้อความ และการดำเนินงาน"
read_when:
  - ทำงานเกี่ยวกับพฤติกรรมของช่องทางWhatsApp/เว็บหรือการกำหนดเส้นทางกล่องข้อความ
title: "WhatsApp"
---

# WhatsApp (ช่องทางเว็บ)

สถานะ: พร้อมใช้งานระดับ production ผ่าน WhatsApp Web (Baileys). Gateway เป็นผู้ดูแลเซสชันที่เชื่อมโยงไว้

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    นโยบาย DM เริ่มต้นคือการจับคู่ (pairing) สำหรับผู้ส่งที่ไม่รู้จัก
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    การวินิจฉัยข้ามช่องทางและคู่มือการแก้ไขปัญหา
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    รูปแบบและตัวอย่างการตั้งค่าช่องทางแบบครบถ้วน
  
</Card>
</CardGroup>

## ตั้งค่าอย่างรวดเร็ว(ผู้เริ่มต้น)

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
    สำหรับบัญชีเฉพาะ:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
เริ่มต้น Gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
อนุมัติด้วย: `openclaw pairing approve whatsapp <code>`(ดูรายการด้วย `openclaw pairing list whatsapp`)
```

    ```
    โค้ดหมดอายุใน1ชั่วโมง; คำขอที่รออยู่จำกัดที่3ต่อช่องทาง
    ```

  
</Step>
</Steps>

<Note>
OpenClaw แนะนำให้ใช้งาน WhatsApp ด้วยหมายเลขแยกต่างหากเมื่อเป็นไปได้ (ข้อมูลเมตาของช่องทางและขั้นตอน onboarding ถูกปรับให้เหมาะกับการตั้งค่านี้ แต่ก็รองรับการใช้หมายเลขส่วนตัวเช่นกัน)
</Note>

## รูปแบบการปรับใช้

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    นี่คือโหมดการทำงานที่สะอาดและเรียบร้อยที่สุด:


    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    การทำ Onboarding รองรับโหมดหมายเลขส่วนตัวและจะเขียนค่าเริ่มต้นที่เหมาะกับการแชทกับตัวเอง:


    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">ช่องทางแพลตฟอร์มส่งข้อความคือ WhatsApp Web (`Baileys`) ในสถาปัตยกรรมช่องทางของ OpenClaw ปัจจุบัน

    ```
    ไม่มีช่องทางส่งข้อความ WhatsApp ผ่าน Twilio แยกต่างหากใน registry ช่องทางแชทที่มีมาให้ในตัว
    ```

  
</Accordion>
</AccordionGroup>

## โมเดลการทำงานขณะรัน

- Gateway เป็นผู้ดูแล WhatsApp socket และลูปการเชื่อมต่อใหม่
- การส่งข้อความขาออกต้องมี WhatsApp listener ที่ใช้งานอยู่สำหรับบัญชีเป้าหมาย
- แชตสถานะ/บรอดแคสต์ถูกละเว้น
- แชทแบบตรงใช้กฎเซสชัน DM (`session.dmScope`; ค่าเริ่มต้น `main` จะรวม DM เข้ากับเซสชันหลักของเอเจนต์)
- กลุ่มแมปไปยังเซสชัน `agent:<agentId>:whatsapp:group:<jid>`

## การควบคุมการเข้าถึงและการเปิดใช้งาน

<Tabs>
  <Tab title="DM policy">**นโยบายDM**: `channels.whatsapp.dmPolicy` ควบคุมการเข้าถึงแชตตรง(ค่าเริ่มต้น: `pairing`)

    ```
    นโยบายกลุ่ม: `channels.whatsapp.groupPolicy = open|disabled|allowlist`(ค่าเริ่มต้น `allowlist`)
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    การเข้าถึงในกลุ่มมีสองชั้น:

    ```
    `channels.whatsapp.groups`(allowlistกลุ่ม+ค่าเริ่มต้นการกั้นด้วยmention; ใช้ `"*"` เพื่ออนุญาตทั้งหมด)
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    โดยค่าเริ่มต้น การตอบกลับในกลุ่มต้องมีการ mention

    ```
    การตรวจจับการ mention รวมถึง:
    
    - การ mention ตัวตนของบอทบน WhatsApp โดยตรง
    - รูปแบบ regex สำหรับการ mention ที่กำหนดไว้ (`agents.list[].groupChat.mentionPatterns`, ค่า fallback คือ `messages.groupChat.mentionPatterns`)
    - การตรวจจับการตอบกลับถึงบอทโดยปริยาย (ผู้ส่งการตอบกลับตรงกับตัวตนของบอท)
    
    คำสั่งเปิดใช้งานระดับเซสชัน:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` จะอัปเดตสถานะของเซสชัน (ไม่ใช่การตั้งค่าระดับ global) และจำกัดให้เฉพาะ owner เท่านั้น
    ```

  
</Tab>
</Tabs>

## พฤติกรรมของหมายเลขส่วนตัวและการแชทกับตัวเอง

เมื่อหมายเลขที่ลิงก์ไว้ของตนเองอยู่ใน `allowFrom` ด้วย ระบบป้องกันการแชทกับตัวเองของ WhatsApp จะทำงาน:

- ข้ามการส่ง read receipts สำหรับรอบการแชทกับตัวเอง
- ไม่ใช้พฤติกรรม auto-trigger จาก mention-JID ที่อาจทำให้คุณแจ้งเตือนตัวเอง
- การตอบกลับแชตกับตัวเองจะใช้ค่าเริ่มต้นเป็น `[{identity.name}]` เมื่อมีการตั้งค่า(มิฉะนั้นเป็น `[openclaw]`)
  หาก `messages.responsePrefix` ไม่ได้ตั้งค่า ให้ตั้งค่าอย่างชัดเจนเพื่อปรับแต่งหรือปิด
  คำนำหน้า(ใช้ `""` เพื่อลบออก) Set it explicitly to customize or disable
  the prefix (use `""` to remove it).

## การปรับรูปแบบข้อความและบริบท

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    ข้อความ WhatsApp ขาเข้าจะถูกห่อด้วย inbound envelope แบบรวมศูนย์

    ````
    หากมีการตอบกลับแบบอ้างอิง (quoted reply) ระบบจะเพิ่มบริบทในรูปแบบดังนี้:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    ฟิลด์เมทาดาทาการตอบกลับจะถูกเติมเมื่อมีข้อมูล (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">ข้อความขาเข้าที่มีเฉพาะสื่อใช้เพลซโฮลเดอร์:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    เพย์โหลดตำแหน่งที่ตั้งและรายชื่อผู้ติดต่อจะถูกแปลงเป็นบริบทข้อความก่อนทำการส่งต่อ (routing).
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    สำหรับกลุ่ม ข้อความที่ยังไม่ถูกประมวลผลสามารถถูกบัฟเฟอร์และแทรกเป็นบริบทเมื่อบอทถูกเรียกใช้งานในภายหลัง

    ```
    ข้อความล่าสุดที่ _ยังไม่ประมวลผล_(ค่าเริ่มต้น50)จะถูกแทรกภายใต้:
    `[Chat messages since your last reply - for context]`(ข้อความที่อยู่ในเซสชันแล้วจะไม่ถูกฉีดซ้ำ)
    ```

  
</Accordion>

  <Accordion title="Read receipts">โดยค่าเริ่มต้น Gatewayจะทำเครื่องหมายข้อความWhatsAppขาเข้าเป็นอ่านแล้ว(ติ๊กสีน้ำเงิน)เมื่อรับเข้า

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## การจัดส่ง การแบ่งข้อความ และสื่อ

<AccordionGroup>
  <Accordion title="Text chunking">การแบ่งตามบรรทัดใหม่แบบไม่บังคับ: ตั้ง `channels.whatsapp.chunkMode="newline"` เพื่อแยกตามบรรทัดว่าง(ขอบเขตย่อหน้า)ก่อนแบ่งตามความยาว
</Accordion>

  <Accordion title="Outbound media behavior">
    - รองรับเพย์โหลด image, video, audio (PTT voice-note) และ document
    - `audio/ogg` จะถูกเขียนใหม่เป็น `audio/ogg; codecs=opus` เพื่อให้เข้ากันได้กับ voice-note
    - รองรับการเล่น animated GIF ผ่าน `gifPlayback: true` เมื่อส่งวิดีโอ
    - คำบรรยาย (captions) จะถูกใส่กับสื่อรายการแรกเมื่อส่งเพย์โหลดตอบกลับแบบหลายสื่อ
    - แหล่งที่มาของสื่อสามารถเป็น HTTP(S), `file://` หรือพาธภายในเครื่อง
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - ขีดจำกัดการบันทึกสื่อขาเข้า: `channels.whatsapp.mediaMaxMb` (ค่าเริ่มต้น `50`)
    - ขีดจำกัดสื่อขาออกสำหรับการตอบกลับอัตโนมัติ: `agents.defaults.mediaMaxMb` (ค่าเริ่มต้น `5MB`)
    - รูปภาพจะถูกปรับให้เหมาะสมอัตโนมัติ (ปรับขนาด/คุณภาพ) เพื่อให้อยู่ในขีดจำกัด
    - หากการส่งสื่อล้มเหลว ระบบจะใช้ fallback โดยส่งข้อความเตือนแทนรายการแรก แทนที่จะปล่อยให้การตอบกลับหายไปโดยไม่มีการแจ้งเตือน
  
</Accordion>
</AccordionGroup>

## การตอบสนองแบบ Acknowledgment

`channels.whatsapp.ackReaction`(รีแอ็กชันอัตโนมัติเมื่อรับข้อความ: `{emoji, direct, group}`)

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

พฤติกรรม:

- ส่งทันทีหลังจากรับข้อความขาเข้า (ก่อนตอบกลับ)
- ความล้มเหลวจะถูกบันทึกใน log แต่จะไม่ขัดขวางการส่งการตอบกลับตามปกติ
- โหมดกลุ่ม `mentions` จะตอบสนองเฉพาะรอบที่ถูกกระตุ้นด้วยการ mention; การเปิดใช้งานกลุ่มแบบ `always` จะทำหน้าที่ข้ามการตรวจสอบนี้
- WhatsAppละเว้น `messages.ackReaction`; ให้ใช้ `channels.whatsapp.ackReaction` แทน

## หลายบัญชีและข้อมูลรับรอง

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*`(การตั้งค่าต่อบัญชี+ไม่บังคับ `authDir`)
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">กำหนดค่า WhatsApp ใน `~/.openclaw/openclaw.json`.<accountId>เก็บข้อมูลรับรองใน `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
</Accordion>

  <Accordion title="Logout behavior">ล็อกอินหลายบัญชี: `openclaw channels login --account <id>`(`<id>` = `accountId`)<id>`]` จะล้างสถานะการยืนยันตัวตน WhatsApp สำหรับบัญชีนั้น

    ```
    ในไดเรกทอรี auth แบบ legacy ไฟล์ `oauth.json` จะถูกเก็บไว้ ขณะที่ไฟล์ auth ของ Baileys จะถูกลบ
    ```

  
</Accordion>
</AccordionGroup>

## เครื่องมือ แอ็กชัน และการเขียนค่าคอนฟิก

- การรองรับเครื่องมือของเอเจนต์รวมถึงแอ็กชัน reaction ของ WhatsApp (`react`)
- เกตของแอ็กชัน:
  - `channels.whatsapp.actions.reactions`(กั้นเครื่องมือรีแอ็กชันWhatsApp)
  - `channels.whatsapp.groupPolicy`(นโยบายกลุ่ม)
- {
  channels: { whatsapp: { configWrites: false } },
  }

## การแก้ไขปัญหา(ย่อ)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">อาการ: `channels status` แสดง `linked: false` หรือเตือน “Not linked”

    ````
    การแก้ไข:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    อาการ: บัญชีที่ลิงก์มีการตัดการเชื่อมต่อหรือพยายามเชื่อมต่อใหม่ซ้ำๆ

    ```
    วิธีแก้ไข: `openclaw doctor` (หรือรีสตาร์ตเกตเวย์) วิธีแก้ไข: `openclaw doctor`(หรือรีสตาร์ตGateway) หากยังเป็นอยู่ ให้เชื่อมโยงใหม่ผ่าน `channels login` และตรวจสอบ `openclaw logs --follow`
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">
    การส่งข้อความขาออกจะล้มเหลวทันทีหากไม่มี gateway listener ที่ใช้งานอยู่สำหรับบัญชีเป้าหมาย

    ```
    ตรวจสอบให้แน่ใจว่า gateway กำลังทำงานและบัญชีถูกลิงก์แล้ว
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    ตรวจสอบตามลำดับนี้:

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - รายการ allowlist ใน `groups`
    - การบังคับ mention (`requireMention` + รูปแบบ mention)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    รันไทม์ของ WhatsApp gateway ควรใช้ Node Bun ถูกระบุว่าไม่เข้ากันสำหรับการใช้งาน WhatsApp/Telegram gateway อย่างเสถียร
  
</Accordion>
</AccordionGroup>

## ตัวชี้อ้างอิงการตั้งค่า (Configuration reference pointers)

ข้อมูลอ้างอิงหลัก:

- [ข้อมูลอ้างอิงการตั้งค่า - WhatsApp](/gateway/configuration-reference#whatsapp)

ฟิลด์ WhatsApp ที่มีความสำคัญสูง:

- การเข้าถึง: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- การส่งข้อความ: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- หลายบัญชี: `accounts.<id>`.enabled`, `accounts.<id>`.authDir`, การกำหนดค่าแทนที่ระดับบัญชี
- การดำเนินงาน: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- พฤติกรรมเซสชัน: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## ที่เกี่ยวข้อง

- [การจับคู่](/channels/pairing)
- [การกำหนดเส้นทางช่องทาง](/channels/channel-routing)
- คู่มือแก้ไขปัญหา: [Gateway troubleshooting](/gateway/troubleshooting)

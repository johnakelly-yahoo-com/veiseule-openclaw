---
title: "WhatsApp"
---

# WhatsApp (ช่องทางเว็บ)

สถานะ: ใช้ WhatsApp Web ผ่าน Baileys เท่านั้น Gateway เป็นผู้ถือครองเซสชัน Gateway owns the session(s).

## ตั้งค่าอย่างรวดเร็ว(ผู้เริ่มต้น)

1. ใช้ **หมายเลขโทรศัพท์แยกต่างหาก** หากเป็นไปได้(แนะนำ)
2. กำหนดค่า WhatsApp ใน `~/.openclaw/openclaw.json`.
3. รัน `openclaw channels login` เพื่อสแกนคิวอาร์โค้ด(อุปกรณ์ที่เชื่อมโยง)
4. เริ่มต้น Gateway

คอนฟิกขั้นต่ำ:

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

## เป้าหมาย

- รองรับหลายบัญชีWhatsApp(multi-account)ในหนึ่งกระบวนการGateway
- การกำหนดเส้นทางที่กำหนดแน่นอน: การตอบกลับกลับไปที่WhatsApp ไม่มีการกำหนดเส้นทางโมเดล
- โมเดลเห็นบริบทเพียงพอเพื่อเข้าใจการตอบกลับแบบอ้างอิง

## การเขียนคอนฟิก

โดยค่าเริ่มต้น WhatsApp ได้รับอนุญาตให้เขียนอัปเดตคอนฟิกที่ถูกทริกเกอร์โดย `/config set|unset`(ต้องใช้ `commands.config: true`)

ปิดการใช้งานด้วย:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## สถาปัตยกรรม(ใครเป็นเจ้าของอะไร)

- **Gateway** เป็นเจ้าของซ็อกเก็ตBaileysและลูปกล่องข้อความ
- **CLI / แอปmacOS** ติดต่อกับGateway; ไม่มีการใช้Baileysโดยตรง
- **ตัวรับฟังที่ทำงานอยู่** จำเป็นสำหรับการส่งออก มิฉะนั้นการส่งจะล้มเหลวทันที

## การขอหมายเลขโทรศัพท์(สองโหมด)

WhatsApp requires a real mobile number for verification. VoIP and virtual numbers are usually blocked. WhatsApp ต้องการหมายเลขมือถือจริงเพื่อยืนยันตัวตน หมายเลขVoIPและหมายเลขเสมือนมักถูกบล็อก มีสองวิธีที่รองรับในการรันOpenClawบนWhatsApp:

### หมายเลขเฉพาะ(แนะนำ)

Use a **separate phone number** for OpenClaw. Best UX, clean routing, no self-chat quirks. Ideal setup: **spare/old Android phone + eSIM**. Leave it on Wi‑Fi and power, and link it via QR.

**WhatsApp Business:** คุณสามารถใช้WhatsApp Businessบนอุปกรณ์เดียวกันด้วยหมายเลขอื่น เหมาะสำหรับแยกWhatsAppส่วนตัว—ติดตั้งWhatsApp BusinessและลงทะเบียนหมายเลขOpenClawที่นั่น Great for keeping your personal WhatsApp separate — install WhatsApp Business and register the OpenClaw number there.

**ตัวอย่างคอนฟิก(หมายเลขเฉพาะ, allowlistผู้ใช้เดียว):**

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

**โหมดการจับคู่(ไม่บังคับ):**
หากต้องการใช้การจับคู่แทนallowlist ให้ตั้งค่า `channels.whatsapp.dmPolicy` เป็น `pairing`. ผู้ส่งที่ไม่รู้จักจะได้รับโค้ดจับคู่ อนุมัติด้วย:
`openclaw pairing approve whatsapp <code>`

### หมายเลขส่วนตัว(ทางเลือกสำรอง)

Quick fallback: run OpenClaw on **your own number**. Message yourself (WhatsApp “Message yourself”) for testing so you don’t spam contacts. Expect to read verification codes on your main phone during setup and experiments. ทางลัด: รันOpenClawบน **หมายเลขของคุณเอง** ส่งข้อความถึงตัวเอง(WhatsApp “Message yourself”)เพื่อทดสอบเพื่อไม่สแปมรายชื่อ คาดว่าจะต้องอ่านรหัสยืนยันบนโทรศัพท์หลักระหว่างการตั้งค่าและทดลอง **ต้องเปิดโหมดแชตกับตัวเอง**
เมื่อวิซาร์ดถามหมายเลขWhatsAppส่วนตัวของคุณ ให้กรอกโทรศัพท์ที่คุณจะส่งข้อความจาก(เจ้าของ/ผู้ส่ง) ไม่ใช่หมายเลขผู้ช่วย

**ตัวอย่างคอนฟิก(หมายเลขส่วนตัว, แชตกับตัวเอง):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

การตอบกลับแชตกับตัวเองจะใช้ค่าเริ่มต้นเป็น `[{identity.name}]` เมื่อมีการตั้งค่า(มิฉะนั้นเป็น `[openclaw]`)
หาก `messages.responsePrefix` ไม่ได้ตั้งค่า ให้ตั้งค่าอย่างชัดเจนเพื่อปรับแต่งหรือปิด
คำนำหน้า(ใช้ `""` เพื่อลบออก) Set it explicitly to customize or disable
the prefix (use `""` to remove it).

### เคล็ดลับการหาเบอร์

- **eSIMท้องถิ่น** จากผู้ให้บริการมือถือในประเทศของคุณ(เชื่อถือได้ที่สุด)
  - ออสเตรีย: [hot.at](https://www.hot.at)
  - สหราชอาณาจักร: [giffgaff](https://www.giffgaff.com) — ซิมฟรี ไม่มีสัญญา
- **ซิมเติมเงิน** — ราคาถูก แค่ต้องรับSMSยืนยันหนึ่งครั้ง

**หลีกเลี่ยง:** TextNow, Google Voice และบริการ “SMSฟรี” ส่วนใหญ่—WhatsAppบล็อกอย่างเข้มงวด

**เคล็ดลับ:** หมายเลขต้องรับSMSยืนยันเพียงครั้งเดียว หลังจากนั้นเซสชันWhatsApp Webจะคงอยู่ผ่าน `creds.json`. After that, WhatsApp Web sessions persist via `creds.json`.

## ทำไมไม่ใช้Twilio?

- รุ่นเริ่มต้นของOpenClawรองรับการผสานรวมWhatsApp BusinessของTwilio
- หมายเลขWhatsApp Businessไม่เหมาะกับผู้ช่วยส่วนตัว
- Metaบังคับหน้าต่างการตอบกลับ24ชั่วโมง; หากคุณไม่ตอบใน24ชั่วโมงที่ผ่านมา หมายเลขธุรกิจไม่สามารถเริ่มส่งข้อความใหม่ได้
- การใช้งานปริมาณสูงหรือคุยถี่กระตุ้นการบล็อกอย่างรุนแรง เพราะบัญชีธุรกิจไม่ได้ออกแบบมาให้ส่งข้อความผู้ช่วยส่วนตัวจำนวนมาก
- ผลลัพธ์: การส่งไม่น่าเชื่อถือและถูกบล็อกบ่อย จึงยกเลิกการรองรับ

## การล็อกอิน+ข้อมูลรับรอง

- คำสั่งล็อกอิน: `openclaw channels login`(คิวอาร์ผ่านอุปกรณ์ที่เชื่อมโยง)
- ล็อกอินหลายบัญชี: `openclaw channels login --account <id>`(`<id>` = `accountId`)
- บัญชีเริ่มต้น(เมื่อไม่ระบุ `--account`): `default` หากมี มิฉะนั้นเป็นไอดีบัญชีที่ตั้งค่าไว้ลำดับแรก(เรียง)
- เก็บข้อมูลรับรองใน `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
- สำเนาสำรองที่ `creds.json.bak`(กู้คืนเมื่อข้อมูลเสียหาย)
- ความเข้ากันได้ย้อนหลัง: การติดตั้งเก่าเก็บไฟล์Baileysโดยตรงใน `~/.openclaw/credentials/`.
- ออกจากระบบ: `openclaw channels logout`(หรือ `--account <id>`) จะลบสถานะยืนยันตัวตนWhatsApp(แต่เก็บ `oauth.json` ที่ใช้ร่วมกัน)
- ซ็อกเก็ตที่ออกจากระบบแล้ว => แสดงข้อผิดพลาดให้เชื่อมโยงใหม่

## โฟลว์ขาเข้า(DM+กลุ่ม)

- อีเวนต์WhatsAppมาจาก `messages.upsert`(Baileys)
- ตัวรับฟังกล่องข้อความถูกถอดออกเมื่อปิดระบบเพื่อหลีกเลี่ยงการสะสมตัวจัดการอีเวนต์ในการทดสอบ/รีสตาร์ต
- แชตสถานะ/บรอดแคสต์ถูกละเว้น
- แชตตรงใช้E.164; กลุ่มใช้JIDกลุ่ม
- **นโยบายDM**: `channels.whatsapp.dmPolicy` ควบคุมการเข้าถึงแชตตรง(ค่าเริ่มต้น: `pairing`)
  - การจับคู่: ผู้ส่งที่ไม่รู้จักจะได้โค้ดจับคู่(อนุมัติผ่าน `openclaw pairing approve whatsapp <code>`; โค้ดหมดอายุใน1ชั่วโมง)
  - เปิด: ต้องให้ `channels.whatsapp.allowFrom` รวม `"*"`
  - หมายเลขWhatsAppที่เชื่อมโยงของคุณเชื่อถือโดยนัย ดังนั้นข้อความจากตัวเองจะข้ามการตรวจ `channels.whatsapp.dmPolicy` และ `channels.whatsapp.allowFrom`

### โหมดหมายเลขส่วนตัว(ทางเลือกสำรอง)

หากคุณรันOpenClawบน **หมายเลขWhatsAppส่วนตัว** ให้เปิด `channels.whatsapp.selfChatMode`(ดูตัวอย่างด้านบน)

พฤติกรรม:

- DMขาออกจะไม่ทริกเกอร์การตอบกลับจับคู่(ป้องกันการสแปมรายชื่อ)
- ผู้ส่งที่ไม่รู้จักขาเข้ายังคงทำตาม `channels.whatsapp.dmPolicy`
- โหมดแชตกับตัวเอง(allowFromรวมหมายเลขของคุณ)หลีกเลี่ยงใบตอบรับการอ่านอัตโนมัติและละเว้นmention JID
- ส่งใบตอบรับการอ่านสำหรับDMที่ไม่ใช่แชตกับตัวเอง

## ใบตอบรับการอ่าน

โดยค่าเริ่มต้น Gatewayจะทำเครื่องหมายข้อความWhatsAppขาเข้าเป็นอ่านแล้ว(ติ๊กสีน้ำเงิน)เมื่อรับเข้า

ปิดใช้งานทั่วทั้งระบบ:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

ปิดใช้งานต่อบัญชี:

```json5
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

หมายเหตุ:

- โหมดแชตกับตัวเองจะข้ามใบตอบรับการอ่านเสมอ

## WhatsApp FAQ: การส่งข้อความ+การจับคู่

**OpenClawจะส่งข้อความถึงรายชื่อสุ่มเมื่อฉันเชื่อมโยงWhatsAppหรือไม่?**  
ไม่ ค่าเริ่มต้นของนโยบายDMคือ **การจับคู่** ดังนั้นผู้ส่งที่ไม่รู้จักจะได้เพียงโค้ดจับคู่และข้อความของพวกเขา **จะไม่ถูกประมวลผล** OpenClawจะตอบเฉพาะแชตที่ได้รับ หรือการส่งที่คุณทริกเกอร์เอง(เอเจนต์/CLI) Default DM policy is **pairing**, so unknown senders only get a pairing code and their message is **not processed**. OpenClaw only replies to chats it receives, or to sends you explicitly trigger (agent/CLI).

**การจับคู่ทำงานอย่างไรบนWhatsApp?**  
การจับคู่คือประตูDMสำหรับผู้ส่งที่ไม่รู้จัก:

- DMแรกจากผู้ส่งใหม่จะส่งโค้ดสั้นๆกลับไป(ข้อความไม่ถูกประมวลผล)
- อนุมัติด้วย: `openclaw pairing approve whatsapp <code>`(ดูรายการด้วย `openclaw pairing list whatsapp`)
- โค้ดหมดอายุใน1ชั่วโมง; คำขอที่รออยู่จำกัดที่3ต่อช่องทาง

**หลายคนสามารถใช้ OpenClaw คนละอินสแตนซ์บนหมายเลข WhatsApp เดียวกันได้หรือไม่?**  
ได้ โดยกำหนดเส้นทางผู้ส่งแต่ละคนไปยังเอเจนต์ที่ต่างกันผ่าน `bindings` (peer `kind: "direct"`, ผู้ส่งแบบ E.164 เช่น `+15551234567`). การตอบกลับยังคงมาจาก **บัญชี WhatsApp เดียวกัน** และแชตแบบตรงจะถูกรวมเข้ากับเซสชันหลักของแต่ละเอเจนต์ ดังนั้นควรใช้ **หนึ่งเอเจนต์ต่อหนึ่งคน** การควบคุมการเข้าถึง DM (`dmPolicy`/`allowFrom`) เป็นแบบโกลบอลต่อบัญชี WhatsApp ดู [Multi-Agent Routing](/concepts/multi-agent)

**ทำไมวิซาร์ดถึงขอหมายเลขโทรศัพท์ของฉัน?**  
วิซาร์ดใช้หมายเลขนี้เพื่อตั้งค่า **allowlist/owner** เพื่ออนุญาต DM ของคุณเอง It’s not used for auto-sending. **ทำไมวิซาร์ดจึงขอหมายเลขโทรศัพท์ของฉัน?**  
วิซาร์ดใช้เพื่อตั้งค่า **allowlist/owner** เพื่ออนุญาตDMของคุณเอง ไม่ได้ใช้เพื่อส่งอัตโนมัติ หากคุณรันบนหมายเลขWhatsAppส่วนตัว ให้ใช้หมายเลขเดียวกันนั้นและเปิด `channels.whatsapp.selfChatMode`.

## การทำให้ข้อความเป็นมาตรฐาน(สิ่งที่โมเดลเห็น)

- `Body` คือเนื้อหาข้อความปัจจุบันพร้อมซองข้อมูล

- บริบทการตอบกลับแบบอ้างอิงจะถูก **ต่อท้ายเสมอ**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- ตั้งค่าเมทาดาทาการตอบกลับด้วย:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = เนื้อหาที่อ้างอิงหรือเพลซโฮลเดอร์สื่อ
  - `ReplyToSender` = E.164 เมื่อทราบ

- ข้อความขาเข้าที่มีเฉพาะสื่อใช้เพลซโฮลเดอร์:
  - `<media:image|video|audio|document|sticker>`

## กลุ่ม

- กลุ่มแมปไปยังเซสชัน `agent:<agentId>:whatsapp:group:<jid>`
- นโยบายกลุ่ม: `channels.whatsapp.groupPolicy = open|disabled|allowlist`(ค่าเริ่มต้น `allowlist`)
- โหมดการเปิดใช้งาน:
  - `mention`(ค่าเริ่มต้น): ต้อง @mention หรือจับคู่regex
  - `always`: ทริกเกอร์เสมอ
- `/activation mention|always` เป็นของเจ้าของเท่านั้นและต้องส่งเป็นข้อความเดี่ยว
- เจ้าของ = `channels.whatsapp.allowFrom`(หรือE.164ของตัวเองหากไม่ตั้งค่า)
- **การฉีดประวัติ**(เฉพาะที่ค้างอยู่):
  - ข้อความล่าสุดที่ _ยังไม่ประมวลผล_(ค่าเริ่มต้น50)จะถูกแทรกภายใต้:
    `[Chat messages since your last reply - for context]`(ข้อความที่อยู่ในเซสชันแล้วจะไม่ถูกฉีดซ้ำ)
  - ข้อความปัจจุบันภายใต้:
    `[Current message - respond to this]`
  - ต่อท้ายด้วยซัฟฟิกซ์ผู้ส่ง: `[from: Name (+E164)]`
- เมทาดาทากลุ่มแคช5นาที(หัวข้อ+ผู้เข้าร่วม)

## การส่งคำตอบ(การจัดเธรด)

- WhatsApp Webส่งข้อความมาตรฐาน(ไม่มีเธรดการตอบกลับแบบอ้างอิงในGatewayปัจจุบัน)
- แท็กการตอบกลับถูกละเว้นในช่องทางนี้

## การตอบรับด้วยรีแอ็กชัน(ตอบอัตโนมัติเมื่อรับ)

WhatsAppสามารถส่งรีแอ็กชันอีโมจิไปยังข้อความขาเข้าทันทีเมื่อรับ ก่อนที่บอตจะสร้างคำตอบ เพื่อให้ผู้ใช้ทราบทันทีว่าข้อความถูกรับแล้ว สิ่งนี้ให้การตอบกลับทันทีแก่ผู้ใช้ว่าระบบได้รับข้อความแล้ว

**การกำหนดค่า:**

```json
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

**ตัวเลือก:**

- `emoji`(สตริง): อีโมจิสำหรับการตอบรับ(เช่น "👀","✅","📨") เว้นว่างหรือไม่ระบุ=ปิดฟีเจอร์ ว่างหรือไม่ระบุ = ปิดใช้งานฟีเจอร์
- `direct`(บูลีน ค่าเริ่มต้น: `true`): ส่งรีแอ็กชันในแชตตรง/DM
- `group`(สตริง ค่าเริ่มต้น: `"mentions"`): พฤติกรรมในแชตกลุ่ม:
  - `"always"`: ตอบรีแอ็กชันทุกข้อความในกลุ่ม(แม้ไม่มี @mention)
  - `"mentions"`: ตอบเฉพาะเมื่อบอตถูก @mention
  - `"never"`: ไม่ตอบรีแอ็กชันในกลุ่ม

**การแทนที่ต่อบัญชี:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**หมายเหตุพฤติกรรม:**

- รีแอ็กชันจะถูกส่ง **ทันที** เมื่อรับข้อความ ก่อนตัวบ่งชี้การพิมพ์หรือคำตอบของบอต
- ในกลุ่มที่ตั้ง `requireMention: false`(การเปิดใช้งาน: เสมอ) `group: "mentions"` จะตอบรีแอ็กชันทุกข้อความ(ไม่ใช่แค่ @mention)
- ส่งแบบไฟร์แอนด์ฟอร์เก็ต: ความล้มเหลวของรีแอ็กชันจะถูกบันทึกแต่ไม่ขัดขวางการตอบของบอต
- JIDของผู้เข้าร่วมจะถูกรวมให้อัตโนมัติสำหรับรีแอ็กชันในกลุ่ม
- WhatsAppละเว้น `messages.ackReaction`; ให้ใช้ `channels.whatsapp.ackReaction` แทน

## เครื่องมือเอเจนต์(รีแอ็กชัน)

- เครื่องมือ: `whatsapp` พร้อมแอ็กชัน `react`(`chatJid`, `messageId`, `emoji`, ไม่บังคับ `remove`)
- ไม่บังคับ: `participant`(ผู้ส่งในกลุ่ม), `fromMe`(รีแอ็กชันต่อข้อความของคุณเอง), `accountId`(หลายบัญชี)
- ความหมายการลบรีแอ็กชัน: ดู [/tools/reactions](/tools/reactions)
- การกั้นเครื่องมือ: `channels.whatsapp.actions.reactions`(ค่าเริ่มต้น: เปิดใช้งาน)

## ข้อจำกัด

- ข้อความขาออกถูกแบ่งเป็นชิ้นที่ `channels.whatsapp.textChunkLimit`(ค่าเริ่มต้น4000)
- การแบ่งตามบรรทัดใหม่แบบไม่บังคับ: ตั้ง `channels.whatsapp.chunkMode="newline"` เพื่อแยกตามบรรทัดว่าง(ขอบเขตย่อหน้า)ก่อนแบ่งตามความยาว
- การบันทึกสื่อขาเข้าถูกจำกัดโดย `channels.whatsapp.mediaMaxMb`(ค่าเริ่มต้น50MB)
- รายการสื่อขาออกถูกจำกัดโดย `agents.defaults.mediaMaxMb`(ค่าเริ่มต้น5MB)

## การส่งขาออก(ข้อความ+สื่อ)

- ใช้ตัวรับฟังเว็บที่ทำงานอยู่; หากGatewayไม่ทำงานจะเกิดข้อผิดพลาด
- การแบ่งข้อความ: สูงสุด4kต่อข้อความ(กำหนดค่าได้ผ่าน `channels.whatsapp.textChunkLimit`, ไม่บังคับ `channels.whatsapp.chunkMode`)
- สื่อ:
  - รองรับภาพ/วิดีโอ/เสียง/เอกสาร
  - เสียงถูกส่งเป็นPTT; `audio/ogg` => `audio/ogg; codecs=opus`
  - คำบรรยายมีเฉพาะรายการสื่อแรก
  - การดึงสื่อรองรับHTTP(S)และพาธภายในเครื่อง
  - GIFเคลื่อนไหว: WhatsAppต้องการMP4พร้อม `gifPlayback: true` สำหรับการวนเล่นในบรรทัด
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: พารามิเตอร์ `send` รวม `gifPlayback: true`

## โน้ตเสียง(เสียงPTT)

WhatsAppส่งเสียงเป็น **โน้ตเสียง**(ฟองPTT)

- ผลลัพธ์ที่ดีที่สุด: OGG/Opus ผลลัพธ์ดีที่สุด: OGG/Opus OpenClawจะเขียน `audio/ogg` ใหม่เป็น `audio/ogg; codecs=opus`
- `[[audio_as_voice]]` ถูกละเว้นสำหรับWhatsApp(เสียงถูกส่งเป็นโน้ตเสียงอยู่แล้ว)

## ข้อจำกัดสื่อ+การปรับให้เหมาะสม

- ขีดจำกัดขาออกเริ่มต้น:5MB(ต่อรายการสื่อ)
- การแทนที่: `agents.defaults.mediaMaxMb`
- รูปภาพถูกปรับให้เหมาะสมอัตโนมัติเป็นJPEGต่ำกว่าขีดจำกัด(ปรับขนาด+กวาดคุณภาพ)
- สื่อเกินขนาด => ข้อผิดพลาด; การตอบด้วยสื่อจะถอยกลับเป็นคำเตือนแบบข้อความ

## Heartbeats

- **ฮาร์ตบีตGateway** บันทึกสุขภาพการเชื่อมต่อ(`web.heartbeatSeconds`, ค่าเริ่มต้น60วินาที)
- **ฮาร์ตบีตเอเจนต์** กำหนดค่าได้ต่อเอเจนต์(`agents.list[].heartbeat`)หรือแบบส่วนกลาง
  ผ่าน `agents.defaults.heartbeat`(ใช้เมื่อไม่มีรายการต่อเอเจนต์)
  - ใช้พรอมป์ต์ฮาร์ตบีตที่ตั้งค่าไว้(ค่าเริ่มต้น: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + พฤติกรรมข้าม `HEARTBEAT_OK`
  - การส่งค่าเริ่มต้นไปยังช่องทางที่ใช้ล่าสุด(หรือเป้าหมายที่กำหนด)

## พฤติกรรมการเชื่อมต่อใหม่

- นโยบายแบ็กออฟ: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`
- หากถึงmaxAttempts การเฝ้าดูเว็บจะหยุด(เสื่อมสภาพ)
- ออกจากระบบแล้ว => หยุดและต้องเชื่อมโยงใหม่

## แผนที่คอนฟิกแบบย่อ

- `channels.whatsapp.dmPolicy`(นโยบายDM: pairing/allowlist/open/disabled)
- `channels.whatsapp.selfChatMode`(การตั้งค่าโทรศัพท์เดียวกัน; บอตใช้หมายเลขWhatsAppส่วนตัวของคุณ)
- `channels.whatsapp.allowFrom` (DM allowlist) `channels.whatsapp.allowFrom`(allowlistDM) WhatsAppใช้หมายเลขโทรศัพท์E.164(ไม่มีชื่อผู้ใช้)
- `channels.whatsapp.mediaMaxMb`(ขีดจำกัดการบันทึกสื่อขาเข้า)
- `channels.whatsapp.ackReaction`(รีแอ็กชันอัตโนมัติเมื่อรับข้อความ: `{emoji, direct, group}`)
- `channels.whatsapp.accounts.<accountId>.*`(การตั้งค่าต่อบัญชี+ไม่บังคับ `authDir`)
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb`(ขีดจำกัดสื่อขาเข้าต่อบัญชี)
- `channels.whatsapp.accounts.<accountId>.ackReaction`(การแทนที่รีแอ็กชันตอบรับต่อบัญชี)
- `channels.whatsapp.groupAllowFrom`(allowlistผู้ส่งในกลุ่ม)
- `channels.whatsapp.groupPolicy`(นโยบายกลุ่ม)
- `channels.whatsapp.historyLimit`/`channels.whatsapp.accounts.<accountId>.historyLimit`(บริบทประวัติกลุ่ม; `0` ปิดการใช้งาน)
- `channels.whatsapp.dmHistoryLimit`(ขีดจำกัดประวัติDMในจำนวนเทิร์นผู้ใช้) การแทนที่ต่อผู้ใช้: `channels.whatsapp.dms["<phone>"].historyLimit` `channels.signal.dmHistoryLimit`: ขีดจำกัดประวัติ DM ในรอบผู้ใช้ การเขียนทับต่อผู้ใช้: `channels.whatsapp.dms["<phone>"].historyLimit`
- `channels.whatsapp.groups`(allowlistกลุ่ม+ค่าเริ่มต้นการกั้นด้วยmention; ใช้ `"*"` เพื่ออนุญาตทั้งหมด)
- `channels.whatsapp.actions.reactions`(กั้นเครื่องมือรีแอ็กชันWhatsApp)
- `agents.list[].groupChat.mentionPatterns`(หรือ `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix`(คำนำหน้าขาเข้า; ต่อบัญชี: `channels.whatsapp.accounts.<accountId>.messagePrefix`; เลิกใช้แล้ว: `messages.messagePrefix`)
- `messages.responsePrefix`(คำนำหน้าขาออก)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model`(การแทนที่ไม่บังคับ)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*`(การแทนที่ต่อเอเจนต์)
- `session.*`(scope, idle, store, mainKey)
- `web.enabled`(ปิดการเริ่มต้นช่องทางเมื่อเป็นfalse)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## บันทึก+การแก้ไขปัญหา

- ระบบย่อย: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`
- ไฟล์บันทึก: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`(กำหนดค่าได้)
- คู่มือแก้ไขปัญหา: [Gateway troubleshooting](/gateway/troubleshooting)

## การแก้ไขปัญหา(ย่อ)

**ยังไม่เชื่อมโยง/ต้องล็อกอินด้วยคิวอาร์**

- อาการ: `channels status` แสดง `linked: false` หรือเตือน “Not linked”
- วิธีแก้ไข: รัน `openclaw channels login` บนโฮสต์Gatewayและสแกนคิวอาร์(WhatsApp→Settings→Linked Devices)

**เชื่อมโยงแล้วแต่หลุด/วนเชื่อมต่อใหม่**

- อาการ: `channels status` แสดง `running, disconnected` หรือเตือน “Linked but disconnected”
- วิธีแก้ไข: `openclaw doctor` (หรือรีสตาร์ตเกตเวย์) วิธีแก้ไข: `openclaw doctor`(หรือรีสตาร์ตGateway) หากยังเป็นอยู่ ให้เชื่อมโยงใหม่ผ่าน `channels login` และตรวจสอบ `openclaw logs --follow`

**รันไทม์Bun**

- ไม่แนะนำให้ใช้ **Bun** WhatsApp (Baileys) และ Telegram ทำงานไม่เสถียรบน Bun
  รันเกตเวย์ด้วย **Node** (ดูหมายเหตุ runtime ใน Getting Started)

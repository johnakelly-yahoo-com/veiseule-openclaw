---
title: "Signal"
---

# Signal (signal-cli)

สถานะ: การผสานรวม CLI ภายนอก สถานะ: การผสานรวม CLI ภายนอก Gateway สื่อสารกับ `signal-cli` ผ่าน HTTP JSON-RPC + SSE

## Quick setup (beginner)

1. ใช้ **หมายเลข Signal แยกต่างหาก** สำหรับบอต (แนะนำ)
2. ติดตั้ง `signal-cli` (ต้องใช้ Java)
3. เชื่อมโยงอุปกรณ์บอตและเริ่มเดมอน:
   - `signal-cli link -n "OpenClaw"`
4. กำหนดค่า OpenClaw และเริ่ม Gateway

คอนฟิกขั้นต่ำ:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

## What it is

- ช่องทาง Signal ผ่าน `signal-cli` (ไม่ใช่ libsignal แบบฝัง)
- การกำหนดเส้นทางแบบกำหนดแน่นอน: การตอบกลับจะย้อนกลับไปที่ Signal เสมอ
- DMs ใช้เซสชันหลักของเอเจนต์ร่วมกัน; กลุ่มถูกแยก (`agent:<agentId>:signal:group:<groupId>`)

## Config writes

ตามค่าเริ่มต้น Signal อนุญาตให้เขียนการอัปเดตคอนฟิกที่ถูกกระตุ้นโดย `/config set|unset` (ต้องใช้ `commands.config: true`)

ปิดการทำงานด้วย:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## The number model (important)

- Gateway เชื่อมต่อกับ **อุปกรณ์ Signal** (บัญชี `signal-cli`)
- หากรันบอตบน **บัญชี Signal ส่วนตัวของคุณ** ระบบจะเพิกเฉยต่อข้อความของคุณเอง (การป้องกันลูป)
- สำหรับรูปแบบ “ฉันส่งข้อความหาบอตแล้วบอตตอบกลับ” ให้ใช้ **หมายเลขบอตแยกต่างหาก**

## Setup (fast path)

1. ติดตั้ง `signal-cli` (ต้องใช้ Java)
2. เชื่อมโยงบัญชีบอต:
   - `signal-cli link -n "OpenClaw"` แล้วสแกน QR ใน Signal
3. กำหนดค่า Signal และเริ่ม Gateway

ตัวอย่าง:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

การรองรับหลายบัญชี: ใช้ `channels.signal.accounts` พร้อมคอนฟิกต่อบัญชีและ `name` แบบไม่บังคับ ดู [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) สำหรับรูปแบบร่วมกัน อย่าคอมมิต `~/.openclaw/openclaw.json` (มักมีโทเคน) รองรับหลายบัญชี: ใช้ `channels.signal.accounts` พร้อมคอนฟิกต่อบัญชีและ `name` แบบไม่บังคับ ดู [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) สำหรับรูปแบบที่ใช้ร่วมกัน

## External daemon mode (httpUrl)

หากต้องการจัดการ `signal-cli` ด้วยตนเอง (การเริ่ม JVM ช้า, การเริ่มคอนเทนเนอร์, หรือ CPU ร่วม) ให้รันเดมอนแยกต่างหากและชี้ OpenClaw ไปยังมัน:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

การตั้งค่านี้จะข้ามการสร้างอัตโนมัติและการรอเริ่มต้นภายใน OpenClaw วิธีนี้จะข้ามการสปอว์นอัตโนมัติและการรอเริ่มต้นภายใน OpenClaw สำหรับการเริ่มช้าเมื่อสปอว์นอัตโนมัติ ให้ตั้งค่า `channels.signal.startupTimeoutMs`

## Access control (DMs + groups)

DMs:

- ค่าเริ่มต้น: `channels.signal.dmPolicy = "pairing"`
- ผู้ส่งที่ไม่รู้จักจะได้รับรหัสจับคู่; ข้อความจะถูกเพิกเฉยจนกว่าจะอนุมัติ (รหัสหมดอายุหลัง 1 ชั่วโมง)
- อนุมัติผ่าน:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- การจับคู่เป็นการแลกเปลี่ยนโทเคนเริ่มต้นสำหรับ Signal DMs รายละเอียด: [Pairing](/channels/pairing) การจับคู่เป็นการแลกเปลี่ยนโทเคนเริ่มต้น รายละเอียด: [Pairing](/channels/pairing)
- ผู้ส่งแบบ UUID เท่านั้น (จาก `sourceUuid`) จะถูกจัดเก็บเป็น `uuid:<id>` ใน `channels.signal.allowFrom`

Groups:

- `channels.signal.groupPolicy = open | allowlist | disabled`
- `channels.signal.groupAllowFrom` ควบคุมว่าใครสามารถกระตุ้นในกลุ่มเมื่อมีการตั้งค่า `allowlist`

## How it works (behavior)

- `signal-cli` ทำงานเป็นเดมอน; Gateway อ่านอีเวนต์ผ่าน SSE
- ข้อความขาเข้าถูกทำให้เป็นมาตรฐานเป็นซองช่องทางที่ใช้ร่วมกัน
- การตอบกลับจะถูกส่งกลับไปยังหมายเลขหรือกลุ่มเดิมเสมอ

## Media + limits

- ข้อความขาออกถูกแบ่งเป็นชิ้นตาม `channels.signal.textChunkLimit` (ค่าเริ่มต้น 4000)
- การแบ่งตามบรรทัดใหม่แบบไม่บังคับ: ตั้งค่า `channels.signal.chunkMode="newline"` เพื่อแบ่งตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนการแบ่งตามความยาว
- รองรับไฟล์แนบ (base64 ดึงจาก `signal-cli`)
- ขีดจำกัดสื่อเริ่มต้น: `channels.signal.mediaMaxMb` (ค่าเริ่มต้น 8)
- ใช้ `channels.signal.ignoreAttachments` เพื่อข้ามการดาวน์โหลดสื่อ
- บริบทประวัติกลุ่มใช้ `channels.signal.historyLimit` (หรือ `channels.signal.accounts.*.historyLimit`) และจะถอยกลับไปใช้ `messages.groupChat.historyLimit` ตั้งค่า `0` เพื่อปิด (ค่าเริ่มต้น 50) ตั้งค่า `0` เพื่อปิด (ค่าเริ่มต้น 50)

## Typing + read receipts

- **ตัวบ่งชี้การพิมพ์**: OpenClaw ส่งสัญญาณการพิมพ์ผ่าน `signal-cli sendTyping` และรีเฟรชระหว่างที่กำลังสร้างคำตอบ
- **ใบรับการอ่าน**: เมื่อ `channels.signal.sendReadReceipts` เป็น true OpenClaw จะส่งต่อใบรับการอ่านสำหรับ DMs ที่ได้รับอนุญาต
- signal-cli ไม่เปิดเผยใบรับการอ่านสำหรับกลุ่ม

## Reactions (message tool)

- ใช้ `message action=react` กับ `channel=signal`
- เป้าหมาย: ผู้ส่งแบบ E.164 หรือ UUID (ใช้ `uuid:<id>` จากผลลัพธ์การจับคู่; ใช้ UUID เปล่าได้เช่นกัน)
- `messageId` คือไทม์สแตมป์ Signal ของข้อความที่คุณกำลังโต้ตอบ
- ปฏิกิริยาในกลุ่มต้องใช้ `targetAuthor` หรือ `targetAuthorUuid`

ตัวอย่าง:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

คอนฟิก:

- `channels.signal.actions.reactions`: เปิด/ปิดการกระทำปฏิกิริยา (ค่าเริ่มต้น true)
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`
  - `off`/`ack` ปิดปฏิกิริยาของเอเจนต์ (เครื่องมือข้อความ `react` จะเกิดข้อผิดพลาด)
  - `minimal`/`extensive` เปิดปฏิกิริยาของเอเจนต์และตั้งค่าระดับคำแนะนำ
- การเขียนทับต่อบัญชี: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`

## Delivery targets (CLI/cron)

- DMs: `signal:+15551234567` (หรือ E.164 แบบปกติ)
- UUID DMs: `uuid:<id>` (หรือ UUID เปล่า)
- กลุ่ม: `signal:group:<groupId>`
- ชื่อผู้ใช้: `username:<name>` (หากบัญชี Signal ของคุณรองรับ)

## Troubleshooting

รันลำดับขั้นนี้ก่อน:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

จากนั้นยืนยันสถานะการจับคู่ DM หากจำเป็น:

```bash
openclaw pairing list signal
```

ความล้มเหลวที่พบบ่อย:

- เดมอนเข้าถึงได้แต่ไม่มีการตอบกลับ: ตรวจสอบการตั้งค่าบัญชี/เดมอน (`httpUrl`, `account`) และโหมดรับ
- DMs ถูกเพิกเฉย: ผู้ส่งกำลังรอการอนุมัติการจับคู่
- ข้อความกลุ่มถูกเพิกเฉย: การควบคุมผู้ส่ง/การกล่าวถึงของกลุ่มบล็อกการส่งมอบ

สำหรับโฟลว์การคัดแยก: [/channels/troubleshooting](/channels/troubleshooting)

## Configuration reference (Signal)

คอนฟิกเต็มรูปแบบ: [Configuration](/gateway/configuration)

ตัวเลือกผู้ให้บริการ:

- `channels.signal.enabled`: เปิด/ปิดการเริ่มต้นช่องทาง
- `channels.signal.account`: E.164 สำหรับบัญชีบอต
- `channels.signal.cliPath`: พาธไปยัง `signal-cli`
- `channels.signal.httpUrl`: URL เดมอนแบบเต็ม (แทนที่โฮสต์/พอร์ต)
- `channels.signal.httpHost`, `channels.signal.httpPort`: การผูกเดมอน (ค่าเริ่มต้น 127.0.0.1:8080)
- `channels.signal.autoStart`: สปอว์นเดมอนอัตโนมัติ (ค่าเริ่มต้น true หากไม่ได้ตั้งค่า `httpUrl`)
- `channels.signal.startupTimeoutMs`: เวลารอเริ่มต้นเป็นมิลลิวินาที (จำกัด 120000)
- `channels.signal.receiveMode`: `on-start | manual`
- `channels.signal.ignoreAttachments`: ข้ามการดาวน์โหลดไฟล์แนบ
- `channels.signal.ignoreStories`: เพิกเฉยสตอรี่จากเดมอน
- `channels.signal.sendReadReceipts`: ส่งต่อใบรับการอ่าน
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (ค่าเริ่มต้น: pairing)
- `channels.signal.allowFrom`: DM allowlist (E.164 หรือ `uuid:<id>`) `open` ต้องใช้ `"*"` Signal ไม่มีชื่อผู้ใช้; ใช้รหัสโทรศัพท์/UUID `open` ต้องใช้ `"*"`. Signal ไม่มีชื่อผู้ใช้; ใช้รหัสโทรศัพท์/UUID
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (ค่าเริ่มต้น: allowlist)
- `channels.signal.groupAllowFrom`: allowlist ผู้ส่งในกลุ่ม
- `channels.signal.historyLimit`: จำนวนข้อความกลุ่มสูงสุดที่จะรวมเป็นบริบท (ตั้ง 0 เพื่อปิด)
- `channels.signal.dmHistoryLimit`: ขีดจำกัดประวัติ DM เป็นจำนวนเทิร์นของผู้ใช้ การแทนที่ต่อผู้ใช้: `channels.imessage.dms["<handle>"].historyLimit` `channels.signal.dmHistoryLimit`: ขีดจำกัดประวัติ DM ในรอบผู้ใช้ การเขียนทับต่อผู้ใช้: `channels.signal.dms["<phone_or_uuid>"].historyLimit`
- `channels.signal.textChunkLimit`: ขนาดการแบ่งชิ้นขาออก (อักขระ)
- `channels.signal.chunkMode`: `length` (ค่าเริ่มต้น) หรือ `newline` เพื่อแบ่งตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนการแบ่งตามความยาว
- `channels.signal.mediaMaxMb`: ขีดจำกัดสื่อขาเข้า/ขาออก (MB)

ตัวเลือกส่วนกลางที่เกี่ยวข้อง:

- `agents.list[].groupChat.mentionPatterns` (Signal ไม่รองรับการกล่าวถึงแบบเนทีฟ)
- `messages.groupChat.mentionPatterns` (ตัวสำรองส่วนกลาง)
- `messages.responsePrefix`

---
title: "Telegram"
---

# Telegram (Bot API)

สถานะ: พร้อมใช้งานระดับโปรดักชันสำหรับบอต DMs + กลุ่มผ่าน grammY ค่าเริ่มต้นเป็น long-polling; webhook เป็นตัวเลือก 27. ค่าเริ่มต้นเป็น long-polling; webhook เป็นตัวเลือก.

## เริ่มต้นอย่างรวดเร็ว(ผู้เริ่มต้น)

1. 28. สร้างบอทด้วย **@BotFather** ([ลิงก์โดยตรง](https://t.me/BotFather)). สร้างบอตด้วย **@BotFather** ([ลิงก์ตรง](https://t.me/BotFather)) ยืนยันว่า handle เป็น `@BotFather` ตรงตัว จากนั้นคัดลอกโทเคน
2. ตั้งค่าโทเคน:
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - หรือคอนฟิก: `channels.telegram.botToken: "..."`
   - หากตั้งค่าทั้งสองอย่าง คอนฟิกจะมีลำดับความสำคัญสูงกว่า (env ใช้เป็นค่า fallback สำหรับบัญชีเริ่มต้นเท่านั้น)
3. เริ่มต้น Gateway
4. การเข้าถึง DM ใช้การจับคู่เป็นค่าเริ่มต้น อนุมัติรหัสจับคู่เมื่อมีการติดต่อครั้งแรก

คอนฟิกขั้นต่ำ:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
    },
  },
}
```

## คืออะไร

- ช่องทาง Telegram Bot API ที่ Gateway เป็นเจ้าของ
- การกำหนดเส้นทางแบบกำหนดแน่นอน: การตอบกลับจะกลับไปที่ Telegram เสมอ โมเดลจะไม่เลือกช่องทางเอง
- DM ใช้เซสชันหลักของเอเจนต์ร่วมกัน; กลุ่มจะแยกจากกัน (`agent:<agentId>:telegram:group:<chatId>`)

## การตั้งค่า(ทางลัด)

### 1. สร้างโทเคนบอต (BotFather)

1. เปิด Telegram และแชตกับ **@BotFather** ([ลิงก์ตรง](https://t.me/BotFather)) ยืนยันว่า handle เป็น `@BotFather` ตรงตัว 29. ยืนยันว่าแฮนเดิลเป็น `@BotFather` ตรงตัว.
2. รัน `/newbot` แล้วทำตามคำแนะนำ (ชื่อ + ชื่อผู้ใช้ลงท้ายด้วย `bot`)
3. คัดลอกโทเคนและเก็บรักษาอย่างปลอดภัย

การตั้งค่า BotFather (ไม่บังคับ):

- `/setjoingroups` — อนุญาต/ปฏิเสธการเพิ่มบอตเข้ากลุ่ม
- `/setprivacy` — ควบคุมว่าบอตจะเห็นข้อความทั้งหมดในกลุ่มหรือไม่

### 2. กำหนดค่าโทเคน (env หรือคอนฟิก)

ตัวอย่าง:

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

ตัวเลือก Env: `TELEGRAM_BOT_TOKEN=...` (ใช้ได้กับบัญชีเริ่มต้น)
หากตั้งค่าทั้ง env และคอนฟิก คอนฟิกจะมีลำดับความสำคัญสูงกว่า
หากตั้งค่าทั้ง env และคอนฟิก คอนฟิกจะมีลำดับความสำคัญสูงกว่า

รองรับหลายบัญชี: ใช้ `channels.telegram.accounts` พร้อมโทเคนต่อบัญชีและ `name` (ไม่บังคับ) รองรับหลายบัญชี: ใช้ `channels.telegram.accounts` พร้อมโทเคนต่อบัญชี และ `name` (ไม่บังคับ) ดู [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) สำหรับรูปแบบที่ใช้ร่วมกัน

3. เริ่มต้น Gateway เริ่มต้น Gateway Telegram จะเริ่มทำงานเมื่อมีการแก้ไขโทเคน (คอนฟิกก่อน แล้วจึง env fallback)
4. 30. การเข้าถึง DM ค่าเริ่มต้นเป็นการจับคู่. การเข้าถึง DM ค่าเริ่มต้นเป็นการจับคู่ อนุมัติรหัสเมื่อบอตถูกติดต่อครั้งแรก
5. สำหรับกลุ่ม: เพิ่มบอต ตัดสินใจพฤติกรรมความเป็นส่วนตัว/แอดมิน (ด้านล่าง) จากนั้นตั้งค่า `channels.telegram.groups` เพื่อควบคุม mention gating + allowlists

## โทเคน + ความเป็นส่วนตัว + สิทธิ์ (ฝั่ง Telegram)

### การสร้างโทเคน (BotFather)

- `/newbot` สร้างบอตและส่งคืนโทเคน (เก็บเป็นความลับ)
- หากโทเคนรั่ว ให้เพิกถอน/สร้างใหม่ผ่าน @BotFather และอัปเดตคอนฟิก

### การมองเห็นข้อความกลุ่ม (Privacy Mode)

บอต Telegram ค่าเริ่มต้นอยู่ใน **Privacy Mode** ซึ่งจำกัดข้อความกลุ่มที่บอตจะได้รับ
หากบอตต้องเห็นข้อความกลุ่ม _ทั้งหมด_ มีสองทางเลือก:
31. หากบอทของคุณต้องเห็นข้อความกลุ่ม _ทั้งหมด_ คุณมีสองทางเลือก:

- ปิด privacy mode ด้วย `/setprivacy` **หรือ**
- เพิ่มบอตเป็น **แอดมิน** ของกลุ่ม (บอตแอดมินจะได้รับข้อความทั้งหมด)

**หมายเหตุ:** เมื่อสลับ privacy mode Telegram ต้องการให้ลบแล้วเพิ่มบอตใหม่
ในแต่ละกลุ่มเพื่อให้การเปลี่ยนแปลงมีผล

### สิทธิ์กลุ่ม (สิทธิ์แอดมิน)

32. สถานะผู้ดูแลตั้งค่าภายในกลุ่ม (ผ่าน UI ของ Telegram). 33. บอทที่เป็นผู้ดูแลจะได้รับข้อความกลุ่มทั้งหมดเสมอ
    ดังนั้นให้ใช้สถานะผู้ดูแลหากต้องการการมองเห็นแบบเต็ม.

## ทำงานอย่างไร(พฤติกรรม)

- ข้อความขาเข้าได้รับการปรับให้อยู่ในซองช่องทางร่วม พร้อมบริบทการตอบกลับและตัวแทนสื่อ
- การตอบในกลุ่มต้องมีการกล่าวถึงเป็นค่าเริ่มต้น (native @mention หรือ `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`)
- การ override หลายเอเจนต์: ตั้งค่าแพตเทิร์นต่อเอเจนต์ที่ `agents.list[].groupChat.mentionPatterns`
- การตอบกลับจะถูกส่งกลับไปยังแชต Telegram เดิมเสมอ
- long-polling ใช้ grammY runner พร้อมการจัดลำดับต่อแชต; คอนเคอร์เรนซีรวมถูกจำกัดโดย `agents.defaults.maxConcurrent`
- Telegram Bot API ไม่รองรับ read receipts; ไม่มีตัวเลือก `sendReadReceipts`

## Draft streaming

OpenClaw สามารถสตรีมการตอบกลับบางส่วนใน Telegram DMs โดยใช้ `sendMessageDraft`

ข้อกำหนด:

- เปิด Threaded Mode สำหรับบอตใน @BotFather (forum topic mode)
- เฉพาะเธรดแชตส่วนตัว (Telegram จะใส่ `message_thread_id` ในข้อความขาเข้า)
- `channels.telegram.streamMode` ไม่ตั้งค่าเป็น `"off"` (ค่าเริ่มต้น: `"partial"`, `"block"` เปิดการอัปเดตแบบบล็อก)

Draft streaming ใช้ได้เฉพาะ DM; Telegram ไม่รองรับในกลุ่มหรือแชนเนล

## การจัดรูปแบบ (Telegram HTML)

- ข้อความขาออกของ Telegram ใช้ `parse_mode: "HTML"` (ชุดแท็กที่ Telegram รองรับ)
- อินพุตสไตล์ Markdown ถูกเรนเดอร์เป็น **HTML ที่ปลอดภัยสำหรับ Telegram** (ตัวหนา/เอียง/ขีดฆ่า/โค้ด/ลิงก์); องค์ประกอบบล็อกจะถูกรวมเป็นข้อความพร้อมบรรทัดใหม่/สัญลักษณ์หัวข้อ
- HTML ดิบจากโมเดลจะถูก escape เพื่อหลีกเลี่ยงข้อผิดพลาดการแยกวิเคราะห์ของ Telegram
- หาก Telegram ปฏิเสธ payload HTML OpenClaw จะลองส่งข้อความเดิมเป็น plain text

## คำสั่ง (เนทีฟ + กำหนดเอง)

OpenClaw ลงทะเบียนคำสั่งเนทีฟ (เช่น `/status`, `/reset`, `/model`) กับเมนูบอตของ Telegram เมื่อเริ่มต้น
คุณสามารถเพิ่มคำสั่งกำหนดเองลงเมนูผ่านคอนฟิก:
34. คุณสามารถเพิ่มคำสั่งกำหนดเองลงในเมนูผ่านคอนฟิก:

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

## การแก้ไขปัญหาการตั้งค่า (คำสั่ง)

- `setMyCommands failed` ในล็อกมักหมายถึง HTTPS/DNS ขาออกไปยัง `api.telegram.org` ถูกบล็อก
- หากเห็นความล้มเหลว `sendMessage` หรือ `sendChatAction` ให้ตรวจสอบเส้นทาง IPv6 และ DNS

ความช่วยเหลือเพิ่มเติม: [การแก้ไขปัญหาช่องทาง](/channels/troubleshooting)

หมายเหตุ:

- คำสั่งกำหนดเองเป็น **รายการเมนูเท่านั้น**; OpenClaw จะไม่ทำงานให้ เว้นแต่คุณจัดการเองที่อื่น
- คำสั่งบางอย่างสามารถถูกจัดการโดยปลั๊กอิน/สกิลได้โดยไม่ต้องลงทะเบียนในเมนูคำสั่งของ Telegram คำสั่งเหล่านี้ยังคงใช้งานได้เมื่อพิมพ์ (เพียงแต่จะไม่แสดงใน `/commands` / เมนู)
- ชื่อคำสั่งจะถูกทำให้เป็นมาตรฐาน (ตัด `/` นำหน้า แปลงเป็นตัวพิมพ์เล็ก) และต้องตรงตาม `a-z`, `0-9`, `_` (1–32 ตัวอักษร)
- คำสั่งกำหนดเอง **ไม่สามารถแทนที่คำสั่งเนทีฟได้** ความขัดแย้งจะถูกละเว้นและบันทึก 35. ความขัดแย้งจะถูกละเว้นและบันทึกไว้.
- หาก `commands.native` ถูกปิด จะลงทะเบียนเฉพาะคำสั่งกำหนดเอง (หรือถูกล้างหากไม่มี)

### คำสั่งการจับคู่อุปกรณ์ (ปลั๊กอิน `device-pair`)

หากติดตั้งปลั๊กอิน `device-pair` แล้ว จะเพิ่มโฟลว์แบบ Telegram-first สำหรับการจับคู่โทรศัพท์เครื่องใหม่:

1. `/pair` จะสร้าง setup code (ส่งเป็นข้อความแยกต่างหากเพื่อให้ง่ายต่อการคัดลอก/วาง)
2. วาง setup code ในแอป iOS เพื่อเชื่อมต่อ
3. `/pair approve` ใช้อนุมัติคำขออุปกรณ์ที่รอดำเนินการล่าสุด

รายละเอียดเพิ่มเติม: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## ข้อจำกัด

- ข้อความขาออกถูกแบ่งเป็นชิ้นขนาด `channels.telegram.textChunkLimit` (ค่าเริ่มต้น 4000)
- การแบ่งตามบรรทัดใหม่ (ไม่บังคับ): ตั้งค่า `channels.telegram.chunkMode="newline"` เพื่อแบ่งตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนแบ่งตามความยาว
- การดาวน์โหลด/อัปโหลดสื่อจำกัดที่ `channels.telegram.mediaMaxMb` (ค่าเริ่มต้น 5)
- คำขอ Telegram Bot API หมดเวลาหลัง `channels.telegram.timeoutSeconds` (ค่าเริ่มต้น 500 ผ่าน grammY) ตั้งค่าต่ำลงเพื่อหลีกเลี่ยงการค้างยาว 36. ตั้งค่าให้ต่ำลงเพื่อหลีกเลี่ยงการค้างนาน.
- บริบทประวัติกลุ่มใช้ `channels.telegram.historyLimit` (หรือ `channels.telegram.accounts.*.historyLimit`) โดย fallback เป็น `messages.groupChat.historyLimit` ตั้งค่า `0` เพื่อปิด (ค่าเริ่มต้น 50) ตั้งค่า `0` เพื่อปิด (ค่าเริ่มต้น 50)
- ประวัติ DM สามารถจำกัดด้วย `channels.telegram.dmHistoryLimit` (จำนวนรอบผู้ใช้) การ override ต่อผู้ใช้: `channels.telegram.dms["<user_id>"].historyLimit` `channels.signal.dmHistoryLimit`: ขีดจำกัดประวัติ DM ในรอบผู้ใช้ การเขียนทับต่อผู้ใช้: `channels.telegram.dms["<user_id>"].historyLimit`

## โหมดการเปิดใช้งานกลุ่ม

ค่าเริ่มต้น บอตจะตอบเฉพาะเมื่อมีการกล่าวถึงในกลุ่ม (`@botname` หรือแพตเทิร์นใน `agents.list[].groupChat.mentionPatterns`) หากต้องการเปลี่ยนพฤติกรรม: 37. เพื่อเปลี่ยนพฤติกรรมนี้:

### ผ่านคอนฟิก (แนะนำ)

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // always respond in this group
      },
    },
  },
}
```

**สำคัญ:** การตั้งค่า `channels.telegram.groups` จะสร้าง **allowlist** — จะยอมรับเฉพาะกลุ่มที่ระบุ (หรือ `"*"`)
Forum topics จะสืบทอดคอนฟิกของกลุ่มแม่ (allowFrom, requireMention, skills, prompts) เว้นแต่คุณเพิ่มการ override ต่อหัวข้อภายใต้ `channels.telegram.groups.<groupId> 38. หัวข้อฟอรัมจะสืบทอดคอนฟิกของกลุ่มแม่ (allowFrom, requireMention, skills, prompts) เว้นแต่คุณจะเพิ่มการเขียนทับรายหัวข้อภายใต้ `channels.telegram.groups.<groupId>`.topics.<topicId>`

เพื่ออนุญาตทุกกลุ่มและตอบเสมอ:

```json5
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

เพื่อคงโหมดกล่าวถึงเท่านั้นสำหรับทุกกลุ่ม (ค่าเริ่มต้น):

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

### ผ่านคำสั่ง (ระดับเซสชัน)

ส่งในกลุ่ม:

- `/activation always` - ตอบทุกข้อความ
- `/activation mention` - ต้องมีการกล่าวถึง (ค่าเริ่มต้น)

**หมายเหตุ:** คำสั่งจะอัปเดตเฉพาะสถานะเซสชัน สำหรับพฤติกรรมถาวรข้ามการรีสตาร์ต ให้ใช้คอนฟิก 39. สำหรับพฤติกรรมถาวรข้ามการรีสตาร์ต ให้ใช้คอนฟิก.

### การดึง group chat ID

ส่งต่อข้อความใดๆ จากกลุ่มไปยัง `@userinfobot` หรือ `@getidsbot` บน Telegram เพื่อดู chat ID (ตัวเลขติดลบเช่น `-1001234567890`)

**เคล็ดลับ:** สำหรับ user ID ของคุณเอง DM บอต แล้วบอตจะตอบด้วย user ID (ข้อความจับคู่) หรือใช้ `/whoami` เมื่อเปิดใช้คำสั่งแล้ว

40. **หมายเหตุด้านความเป็นส่วนตัว:** `@userinfobot` เป็นบอทของบุคคลที่สาม. **หมายเหตุความเป็นส่วนตัว:** `@userinfobot` เป็นบอตของบุคคลที่สาม หากต้องการหลีกเลี่ยง ให้เพิ่มบอตของคุณเข้ากลุ่ม ส่งข้อความ แล้วใช้ `openclaw logs --follow` เพื่ออ่าน `chat.id` หรือใช้ Bot API `getUpdates`

## การเขียนคอนฟิก

ค่าเริ่มต้น Telegram อนุญาตให้เขียนอัปเดตคอนฟิกที่เกิดจากอีเวนต์ของช่องทางหรือ `/config set|unset`

สิ่งนี้จะเกิดขึ้นเมื่อ:

- กลุ่มถูกอัปเกรดเป็นซูเปอร์กรุ๊ปและ Telegram ส่ง `migrate_to_chat_id` (chat ID เปลี่ยน) OpenClaw สามารถย้าย `channels.telegram.groups` อัตโนมัติ 41. OpenClaw สามารถย้าย `channels.telegram.groups` ให้อัตโนมัติ.
- คุณรัน `/config set` หรือ `/config unset` ในแชต Telegram (ต้องมี `commands.config: true`)

ปิดใช้งานด้วย:

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## หัวข้อ (forum supergroups)

หัวข้อฟอรัมของ Telegram จะมี `message_thread_id` ต่อข้อความ OpenClaw จะ: OpenClaw:

- ต่อท้าย `:topic:<threadId>` กับคีย์เซสชันกลุ่ม Telegram เพื่อให้แต่ละหัวข้อแยกกัน
- ส่งตัวบ่งชี้การพิมพ์และการตอบกลับด้วย `message_thread_id` เพื่อให้การตอบอยู่ในหัวข้อ
- หัวข้อทั่วไป (thread id `1`) เป็นกรณีพิเศษ: การส่งข้อความจะไม่ใส่ `message_thread_id` (Telegram ปฏิเสธ) แต่ตัวบ่งชี้การพิมพ์ยังคงใส่
- เปิดเผย `MessageThreadId` + `IsForum` ในบริบทเทมเพลตสำหรับการกำหนดเส้นทาง/เทมเพลต
- มีการกำหนดค่าเฉพาะหัวข้อภายใต้ `channels.telegram.groups.<chatId>.topics.<threadId>` (skills, allowlists, auto-reply, system prompts, disable)
- คอนฟิกหัวข้อจะสืบทอดการตั้งค่ากลุ่ม (requireMention, allowlists, skills, prompts, enabled) เว้นแต่ถูก override ต่อหัวข้อ

42. แชตส่วนตัวอาจมี `message_thread_id` ในบางกรณีขอบ. แชตส่วนตัวอาจมี `message_thread_id` ในบางกรณีขอบ OpenClaw จะคงคีย์เซสชัน DM เดิม แต่ยังใช้ thread id สำหรับการตอบกลับ/การสตรีมดราฟต์เมื่อมี

## ปุ่มอินไลน์

Telegram รองรับคีย์บอร์ดอินไลน์พร้อมปุ่ม callback

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

สำหรับการกำหนดค่าต่อบัญชี:

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

ขอบเขต:

- `off` — ปิดปุ่มอินไลน์
- `dm` — เฉพาะ DM (บล็อกเป้าหมายกลุ่ม)
- `group` — เฉพาะกลุ่ม (บล็อกเป้าหมาย DM)
- `all` — DM + กลุ่ม
- `allowlist` — DM + กลุ่ม แต่เฉพาะผู้ส่งที่อนุญาตโดย `allowFrom`/`groupAllowFrom` (กติกาเดียวกับคำสั่งควบคุม)

43. ค่าเริ่มต้น: `allowlist`.
    ค่าเริ่มต้น: `allowlist`
    Legacy: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`

### การส่งปุ่ม

ใช้ message tool พร้อมพารามิเตอร์ `buttons`:

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

เมื่อผู้ใช้คลิกปุ่ม ข้อมูล callback จะถูกส่งกลับไปยังเอเจนต์เป็นข้อความรูปแบบ:
`callback_data: value`

### ตัวเลือกการกำหนดค่า

ความสามารถของ Telegram สามารถกำหนดค่าได้สองระดับ (แสดงรูปแบบอ็อบเจ็กต์ด้านบน; ยังรองรับอาร์เรย์สตริงแบบ legacy):

- `channels.telegram.capabilities`: ค่าดีฟอลต์ระดับโกลบอล ใช้กับบัญชี Telegram ทั้งหมด เว้นแต่ถูก override
- `channels.telegram.accounts.<account>.capabilities`: ความสามารถต่อบัญชีที่ override ค่าดีฟอลต์โกลบอลสำหรับบัญชีนั้น

44. ใช้การตั้งค่าระดับโกลบอลเมื่อบอท/บัญชี Telegram ทั้งหมดควรมีพฤติกรรมเหมือนกัน. ใช้ค่าระดับโกลบอลเมื่อบอต/บัญชี Telegram ทั้งหมดควรมีพฤติกรรมเหมือนกัน ใช้การกำหนดค่าต่อบัญชีเมื่อบอตต่างกันต้องการพฤติกรรมต่างกัน (เช่น บัญชีหนึ่งรับเฉพาะ DM อีกบัญชีอนุญาตในกลุ่ม)

## การควบคุมการเข้าถึง(DMs+กลุ่ม)

### การเข้าถึง DM

- ค่าเริ่มต้น: `channels.telegram.dmPolicy = "pairing"`. ค่าเริ่มต้น: `channels.telegram.dmPolicy = "pairing"` ผู้ส่งที่ไม่รู้จักจะได้รับรหัสจับคู่ ข้อความจะถูกละเว้นจนกว่าจะอนุมัติ (รหัสหมดอายุภายใน 1 ชั่วโมง)
- อนุมัติผ่าน:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- การจับคู่เป็นการแลกเปลี่ยนโทเคนค่าเริ่มต้นที่ใช้สำหรับ Telegram DMs รายละเอียด: [การจับคู่](/channels/pairing) การจับคู่เป็นการแลกเปลี่ยนโทเคนเริ่มต้น รายละเอียด: [Pairing](/channels/pairing)
- `channels.telegram.allowFrom` รับ user ID แบบตัวเลข (แนะนำ) หรือรายการ `@username` **ไม่ใช่** ชื่อผู้ใช้ของบอต ใช้ ID ของผู้ส่งที่เป็นมนุษย์ ตัวช่วยตั้งค่ารับ `@username` และจะแก้เป็น ID ตัวเลขเมื่อทำได้ 45. นี่ **ไม่ใช่** ชื่อผู้ใช้ของบอท; ให้ใช้ ID ของผู้ส่งที่เป็นมนุษย์. 46. ตัวช่วยตั้งค่ารองรับ `@username` และจะแปลงเป็น ID ตัวเลขเมื่อเป็นไปได้.

#### การค้นหา Telegram user ID ของคุณ

ปลอดภัยกว่า (ไม่มีบอตบุคคลที่สาม):

1. เริ่มต้น Gateway และ DM บอตของคุณ
2. รัน `openclaw logs --follow` และมองหา `from.id`

ทางเลือก (Bot API ทางการ):

1. 47. ส่ง DM ไปยังบอทของคุณ.
2. ดึงอัปเดตด้วยโทเคนบอตของคุณและอ่าน `message.from.id`:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

บุคคลที่สาม (ความเป็นส่วนตัวน้อยกว่า):

- DM `@userinfobot` หรือ `@getidsbot` และใช้ user id ที่ได้

### การเข้าถึงกลุ่ม

มีการควบคุมอิสระสองส่วน:

**1. กลุ่มใดบ้างที่อนุญาต** (allowlist กลุ่มผ่าน `channels.telegram.groups`):

- ไม่มีคอนฟิก `groups` = อนุญาตทุกกลุ่ม
- มีคอนฟิก `groups` = อนุญาตเฉพาะกลุ่มที่ระบุหรือ `"*"`
- ตัวอย่าง: `"groups": { "-1001234567890": {}, "*": {} }` อนุญาตทุกกลุ่ม

**2. ผู้ส่งใดบ้างที่อนุญาต** (การกรองผู้ส่งผ่าน `channels.telegram.groupPolicy`):

- `"open"` = ผู้ส่งทุกคนในกลุ่มที่อนุญาตสามารถส่งข้อความได้
- `"allowlist"` = เฉพาะผู้ส่งใน `channels.telegram.groupAllowFrom` เท่านั้น
- `"disabled"` = ไม่รับข้อความกลุ่มเลย
  ค่าเริ่มต้นคือ `groupPolicy: "allowlist"` (บล็อกจนกว่าจะเพิ่ม `groupAllowFrom`)

ผู้ใช้ส่วนใหญ่ต้องการ: `groupPolicy: "allowlist"` + `groupAllowFrom` + ระบุกลุ่มเฉพาะใน `channels.telegram.groups`

เพื่ออนุญาตให้ **สมาชิกกลุ่มใดๆ** พูดคุยในกลุ่มเฉพาะ (ขณะยังคงจำกัดคำสั่งควบคุมให้ผู้ส่งที่ได้รับอนุญาต) ให้ตั้งค่า override ต่อกลุ่ม:

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

## Long-polling เทียบกับ webhook

- ค่าเริ่มต้น: long-polling (ไม่ต้องมี URL สาธารณะ)
- โหมด webhook: ตั้งค่า `channels.telegram.webhookUrl` และ `channels.telegram.webhookSecret` (เลือกตั้ง `channels.telegram.webhookPath`)
  - ตัวรับฟังในเครื่อง bind ที่ `0.0.0.0:8787` และให้บริการ `POST /telegram-webhook` เป็นค่าเริ่มต้น
  - หาก URL สาธารณะต่างออกไป ให้ใช้ reverse proxy และชี้ `channels.telegram.webhookUrl` ไปยังปลายทางสาธารณะ

## เธรดการตอบกลับ

Telegram รองรับการตอบแบบเธรดผ่านแท็ก:

- `[[reply_to_current]]` -- ตอบกลับข้อความที่กระตุ้น
- `[[reply_to:<id>]]` -- ตอบกลับ message id ที่ระบุ

ควบคุมด้วย `channels.telegram.replyToMode`:

- `first` (ค่าเริ่มต้น), `all`, `off`

## ข้อความเสียง (voice vs file)

Telegram แยก **voice notes** (ฟองกลม) ออกจาก **ไฟล์เสียง** (การ์ดเมทาดาทา)
OpenClaw ค่าเริ่มต้นใช้ไฟล์เสียงเพื่อความเข้ากันได้ย้อนหลัง
48. OpenClaw ใช้ไฟล์เสียงเป็นค่าเริ่มต้นเพื่อความเข้ากันได้ย้อนหลัง.

เพื่อบังคับส่งเป็นฟอง voice note ในการตอบของเอเจนต์ ให้ใส่แท็กนี้ที่ใดก็ได้ในคำตอบ:

- `[[audio_as_voice]]` — ส่งเสียงเป็น voice note แทนไฟล์

49. แท็กจะถูกตัดออกจากข้อความที่ส่งมอบ. แท็กจะถูกลบออกจากข้อความที่ส่งจริง ช่องทางอื่นจะเพิกเฉยแท็กนี้

สำหรับการส่งด้วย message tool ให้ตั้งค่า `asVoice: true` พร้อมเสียงที่รองรับ voice ที่ URL `media`
(`message` เป็นตัวเลือกเมื่อมีสื่อ):

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## ข้อความวิดีโอ (วิดีโอ vs วิดีโอโน้ต)

Telegram แยกแยะ **วิดีโอโน้ต** (ฟองกลม) ออกจาก **ไฟล์วิดีโอ** (สี่เหลี่ยมผืนผ้า)
OpenClaw ตั้งค่าเริ่มต้นเป็นไฟล์วิดีโอ

สำหรับการส่งด้วย message tool ให้ตั้งค่า `asVideoNote: true` พร้อม URL วิดีโอใน `media`:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(หมายเหตุ: วิดีโอโน้ตไม่รองรับคำบรรยาย) หากคุณใส่ข้อความ ข้อความนั้นจะถูกส่งเป็นอีกข้อความหนึ่งแยกต่างหาก)

## สติกเกอร์

OpenClaw รองรับการรับและส่งสติกเกอร์ Telegram พร้อมแคชอัจฉริยะ

### การรับสติกเกอร์

เมื่อผู้ใช้ส่งสติกเกอร์ OpenClaw จะจัดการตามประเภท:

- **สติกเกอร์แบบคงที่ (WEBP):** ดาวน์โหลดและประมวลผลผ่าน vision สติกเกอร์จะแสดงเป็นตัวแทน `<media:sticker>` ในเนื้อหาข้อความ 50. สติกเกอร์จะแสดงเป็นตัวแทน `<media:sticker>` ในเนื้อหาข้อความ.
- **สติกเกอร์แอนิเมชัน (TGS):** ข้าม (รูปแบบ Lottie ไม่รองรับการประมวลผล)
- **สติกเกอร์วิดีโอ (WEBM):** ข้าม (รูปแบบวิดีโอไม่รองรับการประมวลผล)

ฟิลด์บริบทเทมเพลตที่ใช้ได้เมื่อรับสติกเกอร์:

- `Sticker` — อ็อบเจ็กต์ที่มี:
  - `emoji` — อีโมจิที่เกี่ยวข้องกับสติกเกอร์
  - `setName` — ชื่อชุดสติกเกอร์
  - `fileId` — Telegram file ID (ใช้ส่งสติกเกอร์เดิมกลับ)
  - `fileUniqueId` — ID คงที่สำหรับค้นหาในแคช
  - `cachedDescription` — คำอธิบายจาก vision ที่แคชไว้เมื่อมี

### แคชสติกเกอร์

Stickers are processed through the AI's vision capabilities to generate descriptions. สติกเกอร์ถูกประมวลผลผ่านความสามารถ vision ของ AI เพื่อสร้างคำอธิบาย เนื่องจากสติกเกอร์เดียวกันมักถูกส่งซ้ำ OpenClaw จึงแคชคำอธิบายเพื่อหลีกเลี่ยงการเรียก API ซ้ำ

**ทำงานอย่างไร:**

1. **พบครั้งแรก:** ส่งภาพสติกเกอร์ไปวิเคราะห์ vision AI สร้างคำอธิบาย (เช่น "แมวการ์ตูนโบกมืออย่างกระตือรือร้น") The AI generates a description (e.g., "A cartoon cat waving enthusiastically").
2. **บันทึกแคช:** บันทึกคำอธิบายพร้อม file ID อีโมจิ และชื่อชุด
3. **Subsequent encounters:** When the same sticker is seen again, the cached description is used directly. The image is not sent to the AI.

**ตำแหน่งแคช:** `~/.openclaw/telegram/sticker-cache.json`

**รูปแบบรายการแคช:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**ประโยชน์:**

- ลดค่าใช้จ่าย API โดยหลีกเลี่ยงการเรียก vision ซ้ำสำหรับสติกเกอร์เดิม
- ตอบสนองเร็วขึ้นสำหรับสติกเกอร์ที่แคชแล้ว (ไม่มีดีเลย์จากการประมวลผล vision)
- เปิดใช้ฟังก์ชันค้นหาสติกเกอร์ตามคำอธิบายที่แคช

แคชจะถูกเติมอัตโนมัติเมื่อรับสติกเกอร์ ไม่ต้องจัดการแคชด้วยตนเอง There is no manual cache management required.

### การส่งสติกเกอร์

เอเจนต์สามารถส่งและค้นหาสติกเกอร์ด้วยแอ็กชัน `sticker` และ `sticker-search` ซึ่งถูกปิดเป็นค่าเริ่มต้นและต้องเปิดในคอนฟิก: These are disabled by default and must be enabled in config:

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

**ส่งสติกเกอร์:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

พารามิเตอร์:

- `fileId` (required) — the Telegram file ID of the sticker. `fileId` (จำเป็น) — Telegram file ID ของสติกเกอร์ รับจาก `Sticker.fileId` เมื่อรับสติกเกอร์ หรือจากผลลัพธ์ `sticker-search`
- `replyTo` (ไม่บังคับ) — message ID ที่จะตอบกลับ
- `threadId` (ไม่บังคับ) — message thread ID สำหรับหัวข้อฟอรัม

**ค้นหาสติกเกอร์:**

เอเจนต์สามารถค้นหาสติกเกอร์ที่แคชไว้ตามคำอธิบาย อีโมจิ หรือชื่อชุด:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

ส่งคืนสติกเกอร์ที่ตรงจากแคช:

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

การค้นหาใช้การจับคู่แบบฟัซซีข้ามข้อความคำอธิบาย อักขระอีโมจิ และชื่อชุด

**ตัวอย่างพร้อมเธรด:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## การสตรีม (ดราฟต์)

Telegram can stream **draft bubbles** while the agent is generating a response.
Telegram สามารถสตรีม **ฟองดราฟต์** ระหว่างที่เอเจนต์กำลังสร้างคำตอบ
OpenClaw ใช้ Bot API `sendMessageDraft` (ไม่ใช่ข้อความจริง) จากนั้นส่งคำตอบสุดท้ายเป็นข้อความปกติ

ข้อกำหนด (Telegram Bot API 9.3+):

- **แชตส่วนตัวที่เปิดใช้หัวข้อ** (forum topic mode สำหรับบอต)
- ข้อความขาเข้าต้องมี `message_thread_id` (เธรดหัวข้อส่วนตัว)
- การสตรีมจะถูกละเว้นสำหรับกลุ่ม/ซูเปอร์กรุ๊ป/แชนเนล

คอนฟิก:

- `channels.telegram.streamMode: "off" | "partial" | "block"` (ค่าเริ่มต้น: `partial`)
  - `partial`: อัปเดตฟองดราฟต์ด้วยข้อความสตรีมล่าสุด
  - `block`: อัปเดตฟองดราฟต์เป็นบล็อกใหญ่ (แบ่งชิ้น)
  - `off`: ปิดการสตรีมดราฟต์
- ตัวเลือก (เฉพาะ `streamMode: "block"`):
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    - ค่าเริ่มต้น: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (จำกัดที่ `channels.telegram.textChunkLimit`)

Note: draft streaming is separate from **block streaming** (channel messages).
หมายเหตุ: การสตรีมดราฟต์แยกจาก **block streaming** (ข้อความช่องทาง)
block streaming ปิดเป็นค่าเริ่มต้นและต้องใช้ `channels.telegram.blockStreaming: true`
หากต้องการข้อความ Telegram ก่อนกำหนดแทนการอัปเดตดราฟต์

สตรีมเหตุผล (เฉพาะ Telegram):

- `/reasoning stream` จะสตรีมเหตุผลลงในฟองดราฟต์ขณะสร้างคำตอบ
  จากนั้นส่งคำตอบสุดท้ายโดยไม่มีเหตุผล
- หาก `channels.telegram.streamMode` เป็น `off` จะปิดสตรีมเหตุผล
  บริบทเพิ่มเติม: [การสตรีม + การแบ่งชิ้น](/concepts/streaming)
  More context: [Streaming + chunking](/concepts/streaming).

## นโยบายการลองใหม่

การเรียก Telegram API ขาออกจะลองใหม่เมื่อเกิดข้อผิดพลาดเครือข่ายชั่วคราว/429 ด้วย exponential backoff และ jitter กำหนดค่าผ่าน `channels.telegram.retry` ดู [นโยบายการลองใหม่](/concepts/retry) Configure via `channels.telegram.retry`. See [Retry policy](/concepts/retry).

## เครื่องมือเอเจนต์ (ข้อความ + ปฏิกิริยา)

- เครื่องมือ: `telegram` พร้อมแอ็กชัน `sendMessage` (`to`, `content`, ตัวเลือก `mediaUrl`, `replyToMessageId`, `messageThreadId`)
- เครื่องมือ: `telegram` พร้อมแอ็กชัน `react` (`chatId`, `messageId`, `emoji`)
- เครื่องมือ: `telegram` พร้อมแอ็กชัน `deleteMessage` (`chatId`, `messageId`)
- ความหมายการลบปฏิกิริยา: ดู [/tools/reactions](/tools/reactions)
- การจำกัดเครื่องมือ: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (ค่าเริ่มต้น: เปิด), และ `channels.telegram.actions.sticker` (ค่าเริ่มต้น: ปิด)

## การแจ้งเตือนปฏิกิริยา

**การทำงานของปฏิกิริยา:**
ปฏิกิริยาของ Telegram เข้ามาเป็น **อีเวนต์ `message_reaction` แยกต่างหาก** ไม่ได้เป็นพร็อพเพอร์ตีใน payload ข้อความ เมื่อผู้ใช้เพิ่มปฏิกิริยา OpenClaw จะ: When a user adds a reaction, OpenClaw:

1. รับอัปเดต `message_reaction` จาก Telegram API
2. แปลงเป็น **system event** รูปแบบ: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. ใส่คิว system event โดยใช้ **คีย์เซสชันเดียวกัน** กับข้อความปกติ
4. เมื่อมีข้อความถัดไปในบทสนทนา system events จะถูกระบายและนำหน้าบริบทของเอเจนต์

เอเจนต์จะเห็นปฏิกิริยาเป็น **การแจ้งเตือนระบบ** ในประวัติการสนทนา ไม่ใช่เมทาดาทาของข้อความ

**การกำหนดค่า:**

- `channels.telegram.reactionNotifications`: ควบคุมว่าปฏิกิริยาใดกระตุ้นการแจ้งเตือน
  - `"off"` — เพิกเฉยทุกปฏิกิริยา
  - `"own"` — แจ้งเตือนเมื่อผู้ใช้ตอบสนองต่อข้อความบอต (best-effort; ในหน่วยความจำ) (ค่าเริ่มต้น)
  - `"all"` — แจ้งเตือนทุกปฏิกิริยา

- `channels.telegram.reactionLevel`: ควบคุมความสามารถปฏิกิริยาของเอเจนต์
  - `"off"` — เอเจนต์ไม่สามารถตอบสนอง
  - `"ack"` — บอตส่งปฏิกิริยารับทราบ (👀 ระหว่างประมวลผล) (ค่าเริ่มต้น)
  - `"minimal"` — เอเจนต์ตอบสนองอย่างประหยัด (แนวทาง: 1 ครั้งต่อ 5–10 รอบ)
  - `"extensive"` — เอเจนต์ตอบสนองอย่างเสรีเมื่อเหมาะสม

**กลุ่มฟอรัม:** ปฏิกิริยาในกลุ่มฟอรัมจะมี `message_thread_id` และใช้คีย์เซสชันเช่น `agent:main:telegram:group:{chatId}:topic:{threadId}` เพื่อให้ปฏิกิริยาและข้อความในหัวข้อเดียวกันอยู่ด้วยกัน This ensures reactions and messages in the same topic stay together.

**ตัวอย่างคอนฟิก:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**ข้อกำหนด:**

- บอต Telegram ต้องร้องขอ `message_reaction` อย่างชัดเจนใน `allowed_updates` (OpenClaw ตั้งค่าให้อัตโนมัติ)
- สำหรับโหมด webhook ปฏิกิริยาจะรวมอยู่ใน webhook `allowed_updates`
- สำหรับโหมด polling ปฏิกิริยาจะรวมอยู่ใน `getUpdates` `allowed_updates`

## เป้าหมายการส่งมอบ(CLI/cron)

- ใช้ chat id (`123456789`) หรือชื่อผู้ใช้ (`@name`) เป็นเป้าหมาย
- ตัวอย่าง: `openclaw message send --channel telegram --target 123456789 --message "hi"`

## การแก้ไขปัญหา

**บอตไม่ตอบข้อความที่ไม่ถูกกล่าวถึงในกลุ่ม:**

- หากตั้งค่า `channels.telegram.groups.*.requireMention=false` ต้องปิด **privacy mode** ของ Bot API ของ Telegram
  - BotFather: `/setprivacy` → **Disable** (จากนั้นลบแล้วเพิ่มบอตใหม่เข้ากลุ่ม)
- `openclaw channels status` จะแสดงคำเตือนเมื่อคอนฟิกคาดหวังข้อความกลุ่มที่ไม่ถูกกล่าวถึง
- `openclaw channels status --probe` สามารถตรวจสอบสมาชิกเพิ่มเติมสำหรับ group ID ตัวเลขที่ระบุ (ไม่สามารถตรวจสอบกฎ wildcard `"*"`)
- ทดสอบด่วน: `/activation always` (เฉพาะเซสชัน; ใช้คอนฟิกเพื่อความถาวร)

**บอตไม่เห็นข้อความกลุ่มเลย:**

- หากตั้งค่า `channels.telegram.groups` กลุ่มต้องถูกระบุหรือใช้ `"*"`
- ตรวจสอบ Privacy Settings ใน @BotFather → "Group Privacy" ควรเป็น **OFF**
- ยืนยันว่าบอตเป็นสมาชิกจริง (ไม่ใช่แค่แอดมินที่ไม่มีสิทธิ์อ่าน)
- ตรวจสอบล็อก Gateway: `openclaw logs --follow` (มองหา "skipping group message")

**บอตตอบเมื่อถูกกล่าวถึง แต่ไม่ตอบ `/activation always`:**

- คำสั่ง `/activation` อัปเดตสถานะเซสชันแต่ไม่บันทึกถาวรในคอนฟิก
- เพื่อความถาวร เพิ่มกลุ่มไปที่ `channels.telegram.groups` พร้อม `requireMention: false`

**คำสั่งอย่าง `/status` ใช้ไม่ได้:**

- ตรวจสอบว่า Telegram user ID ของคุณได้รับอนุญาตแล้ว (ผ่านการจับคู่หรือ `channels.telegram.allowFrom`)
- คำสั่งต้องการการอนุญาต แม้อยู่ในกลุ่มที่ตั้งค่า `groupPolicy: "open"`

**long-polling ยุติทันทีบน Node 22+ (มักเกิดกับ proxy/fetch แบบกำหนดเอง):**

- Node 22+ เข้มงวดกับอินสแตนซ์ `AbortSignal` มากขึ้น สัญญาณจากภายนอกอาจยุติการเรียก `fetch` ทันที
- อัปเกรดเป็นบิลด์ OpenClaw ที่ทำ normalization สัญญาณ abort หรือรัน Gateway บน Node 20 จนกว่าจะอัปเกรดได้

**บอตเริ่มทำงานแล้วหยุดตอบเงียบๆ (หรือบันทึก `HttpError: Network request ... failed`):**

- Some hosts resolve `api.telegram.org` to IPv6 first. บางโฮสต์แก้ชื่อ `api.telegram.org` เป็น IPv6 ก่อน หากเซิร์ฟเวอร์ของคุณไม่มี IPv6 egress ที่ใช้งานได้ grammY อาจค้างกับคำขอ IPv6 เท่านั้น
- แก้ไขโดยเปิด IPv6 egress **หรือ** บังคับให้แก้ชื่อเป็น IPv4 สำหรับ `api.telegram.org` (เช่น เพิ่มรายการ `/etc/hosts` ด้วย A record ของ IPv4 หรือกำหนดให้ระบบปฏิบัติการเลือก IPv4) แล้วรีสตาร์ต Gateway
- ตรวจสอบด่วน: `dig +short api.telegram.org A` และ `dig +short api.telegram.org AAAA` เพื่อยืนยันผล DNS

## อ้างอิงการกำหนดค่า (Telegram)

การกำหนดค่าแบบเต็ม: [การกำหนดค่า](/gateway/configuration)

ตัวเลือกผู้ให้บริการ:

- `channels.telegram.enabled`: เปิด/ปิดการเริ่มต้นช่องทาง
- `channels.telegram.botToken`: โทเคนบอต (BotFather)
- `channels.telegram.tokenFile`: อ่านโทเคนจากพาธไฟล์
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (ค่าเริ่มต้น: การจับคู่)
- `channels.telegram.allowFrom`: allowlist DM (ids/ชื่อผู้ใช้) `open` ต้องใช้ `"*"` `open` ต้องใช้ `"*"`.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (ค่าเริ่มต้น: allowlist)
- `channels.telegram.groupAllowFrom`: allowlist ผู้ส่งในกลุ่ม (ids/ชื่อผู้ใช้)
- `channels.telegram.groups`: ค่าเริ่มต้นต่อกลุ่ม + allowlist (ใช้ `"*"` สำหรับค่าเริ่มต้นโกลบอล)
  - `channels.telegram.groups.<id>.groupPolicy`: override ต่อกลุ่มสำหรับ groupPolicy (`open | allowlist | disabled`)
  - `channels.telegram.groups.<id>.requireMention`: ค่าเริ่มต้น mention gating
  - `channels.telegram.groups.<id>.skills`: ตัวกรอง skill (ไม่ระบุ = ทุก Skills, ว่าง = ไม่มี)
  - `channels.telegram.groups.<id>.allowFrom`: override allowlist ผู้ส่งต่อกลุ่ม
  - `channels.telegram.groups.<id>.systemPrompt`: system prompt เพิ่มเติมสำหรับกลุ่ม
  - `channels.telegram.groups.<id>.enabled`: ปิดการใช้งานกลุ่มเมื่อ `false`
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: override ต่อหัวข้อ (ฟิลด์เดียวกับกลุ่ม)
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: override ต่อหัวข้อสำหรับ groupPolicy (`open | allowlist | disabled`)
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: override mention gating ต่อหัวข้อ
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (ค่าเริ่มต้น: allowlist)
- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: override ต่อบัญชี
- `channels.telegram.replyToMode`: `off | first | all` (ค่าเริ่มต้น: `first`)
- `channels.telegram.textChunkLimit`: ขนาดชิ้นขาออก (ตัวอักษร)
- `channels.telegram.chunkMode`: `length` (ค่าเริ่มต้น) หรือ `newline` เพื่อแบ่งตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนแบ่งตามความยาว
- `channels.telegram.linkPreview`: สลับการแสดงตัวอย่างลิงก์สำหรับข้อความขาออก (ค่าเริ่มต้น: true)
- `channels.telegram.streamMode`: `off | partial | block` (draft streaming)
- `channels.telegram.mediaMaxMb`: ขีดจำกัดสื่อขาเข้า/ขาออก (MB)
- `channels.telegram.retry`: นโยบายลองใหม่สำหรับการเรียก Telegram API ขาออก (จำนวนครั้ง, minDelayMs, maxDelayMs, jitter)
- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=เปิด, false=ปิด) ค่าเริ่มต้นปิดบน Node 22 เพื่อหลีกเลี่ยงเวลาเกิน Happy Eyeballs Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.
- `channels.telegram.proxy`: URL พร็อกซีสำหรับการเรียก Bot API (SOCKS/HTTP)
- `channels.telegram.webhookUrl`: เปิดโหมด webhook (ต้องใช้ `channels.telegram.webhookSecret`)
- `channels.telegram.webhookSecret`: webhook secret (จำเป็นเมื่อกำหนด webhookUrl)
- `channels.telegram.webhookPath`: พาธ webhook ภายในเครื่อง (ค่าเริ่มต้น `/telegram-webhook`)
- `channels.telegram.actions.reactions`: ควบคุมปฏิกิริยาเครื่องมือ Telegram
- `channels.telegram.actions.sendMessage`: ควบคุมการส่งข้อความเครื่องมือ Telegram
- `channels.telegram.actions.deleteMessage`: ควบคุมการลบข้อความเครื่องมือ Telegram
- `channels.telegram.actions.sticker`: ควบคุมแอ็กชันสติกเกอร์ Telegram — ส่งและค้นหา (ค่าเริ่มต้น: false)
- `channels.telegram.reactionNotifications`: `off | own | all` — ควบคุมว่าปฏิกิริยาใดกระตุ้น system events (ค่าเริ่มต้น: `own` เมื่อไม่ตั้งค่า)
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — ควบคุมความสามารถปฏิกิริยาของเอเจนต์ (ค่าเริ่มต้น: `minimal` เมื่อไม่ตั้งค่า)

ตัวเลือกโกลบอลที่เกี่ยวข้อง:

- `agents.list[].groupChat.mentionPatterns` (แพตเทิร์น mention gating)
- `messages.groupChat.mentionPatterns` (fallback โกลบอล)
- `commands.native` (ค่าเริ่มต้นเป็น `"auto"` → เปิดสำหรับ Telegram/Discord ปิดสำหรับ Slack), `commands.text`, `commands.useAccessGroups` (พฤติกรรมคำสั่ง) override ด้วย `channels.telegram.commands.native` Override with `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`

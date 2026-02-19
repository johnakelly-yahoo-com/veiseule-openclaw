---
title: IRC
description: เชื่อมต่อ OpenClaw เข้ากับช่อง IRC และข้อความส่วนตัว
---

ใช้ IRC เมื่อคุณต้องการให้ OpenClaw อยู่ในช่องแบบคลาสสิก (`#room`) และข้อความส่วนตัว
IRC มาพร้อมเป็นปลั๊กอินส่วนขยาย แต่จะตั้งค่าในไฟล์คอนฟิกหลักภายใต้ `channels.irc`

## เริ่มต้นอย่างรวดเร็ว

1. เปิดใช้งานการตั้งค่า IRC ใน `~/.openclaw/openclaw.json`
2. ตั้งค่าอย่างน้อยดังนี้:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. เริ่ม/รีสตาร์ท gateway:

```bash
openclaw gateway run
```

## ค่าเริ่มต้นด้านความปลอดภัย

- `channels.irc.dmPolicy` มีค่าเริ่มต้นเป็น `"pairing"`
- `channels.irc.groupPolicy` มีค่าเริ่มต้นเป็น `"allowlist"`
- เมื่อใช้ `groupPolicy="allowlist"` ให้ตั้งค่า `channels.irc.groups` เพื่อกำหนดช่องที่อนุญาต
- ใช้ TLS (`channels.irc.tls=true`) เว้นแต่คุณตั้งใจยอมรับการส่งข้อมูลแบบไม่เข้ารหัส

## การควบคุมการเข้าถึง

มี “ด่าน” แยกกันสองส่วนสำหรับช่อง IRC:

1. **การเข้าถึงช่อง** (`groupPolicy` + `groups`): บอทยอมรับข้อความจากช่องนั้นหรือไม่
2. **การเข้าถึงของผู้ส่ง** (`groupAllowFrom` / ต่อช่อง `groups["#channel"].allowFrom`): ใครบ้างที่สามารถเรียกใช้บอทภายในช่องนั้นได้

คีย์การตั้งค่า:

- DM allowlist (สิทธิ์ผู้ส่ง DM): `channels.irc.allowFrom`
- Group sender allowlist (สิทธิ์ผู้ส่งในช่อง): `channels.irc.groupAllowFrom`
- การควบคุมต่อช่อง (ช่อง + ผู้ส่ง + กฎการ mention): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` อนุญาตช่องที่ไม่ได้ตั้งค่าไว้ (**ยังคงต้อง mention ตามค่าเริ่มต้น**)

รายการใน allowlist สามารถใช้รูปแบบ nick หรือ `nick!user@host` ได้

### ข้อผิดพลาดที่พบบ่อย: `allowFrom` ใช้สำหรับ DM ไม่ใช่ช่อง

หากคุณเห็นบันทึก (logs) เช่น:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…หมายความว่าผู้ส่งไม่ได้รับอนุญาตสำหรับข้อความใน **กลุ่ม/ช่อง** แก้ไขได้โดย:

- ตั้งค่า `channels.irc.groupAllowFrom` (มีผลกับทุกช่อง), หรือ
- ตั้งค่า allowlist ผู้ส่งต่อช่อง: `channels.irc.groups["#channel"].allowFrom`

ตัวอย่าง (อนุญาตให้ทุกคนใน `#tuirc-dev` คุยกับบอทได้):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## การทริกเกอร์การตอบกลับ (mentions)

แม้ว่าช่องจะได้รับอนุญาตแล้ว (ผ่าน `groupPolicy` + `groups`) และผู้ส่งได้รับอนุญาต OpenClaw จะตั้งค่าเริ่มต้นให้ **ต้องมีการ mention** ในบริบทของกลุ่ม

ซึ่งหมายความว่าคุณอาจเห็นบันทึกเช่น `drop channel … (missing-mention)` เว้นแต่ข้อความจะมีรูปแบบ mention ที่ตรงกับบอท

หากต้องการให้บอทตอบกลับในช่อง IRC **โดยไม่ต้องมีการ mention** ให้ปิดการบังคับ mention สำหรับช่องนั้น:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

หรือเพื่ออนุญาต **ทั้งหมด** ของช่อง IRC (ไม่ใช้ allowlist แยกต่อช่อง) และยังคงตอบกลับโดยไม่ต้องมีการ mention:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## หมายเหตุด้านความปลอดภัย (แนะนำสำหรับช่องสาธารณะ)

หากคุณตั้งค่า `allowFrom: ["*"]` ในช่องสาธารณะ ใครก็ตามสามารถส่งพรอมป์ให้บอทได้
เพื่อลดความเสี่ยง ควรจำกัดเครื่องมือสำหรับช่องนั้น

### เครื่องมือเดียวกันสำหรับทุกคนในช่อง

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### เครื่องมือแตกต่างกันตามผู้ส่ง (owner ได้สิทธิ์มากกว่า)

ใช้ `toolsBySender` เพื่อกำหนดนโยบายที่เข้มงวดกว่าสำหรับ `"*"` และผ่อนปรนมากขึ้นสำหรับ nick ของคุณ:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

หมายเหตุ:

- คีย์ของ `toolsBySender` สามารถเป็น nick (เช่น `"eigen"`) หรือ hostmask แบบเต็ม (`"eigen!~eigen@174.127.248.171"`) เพื่อการยืนยันตัวตนที่รัดกุมยิ่งขึ้น
- นโยบายของผู้ส่งที่ตรงเงื่อนไขรายการแรกจะถูกใช้; `"*"` คือค่าทั่วไป (wildcard) สำรอง

ดูข้อมูลเพิ่มเติมเกี่ยวกับการเข้าถึงแบบกลุ่มเทียบกับ mention-gating (และการทำงานร่วมกันของทั้งสอง) ได้ที่: [/channels/groups](/channels/groups).

## NickServ

ระบุตัวตนกับ NickServ หลังจากเชื่อมต่อ:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

การลงทะเบียนแบบครั้งเดียวเมื่อเชื่อมต่อ (ไม่บังคับ):

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

ปิดการใช้งาน `register` หลังจากลงทะเบียน nick แล้ว เพื่อหลีกเลี่ยงความพยายาม REGISTER ซ้ำ ๆ

## ตัวแปรสภาพแวดล้อม

บัญชีค่าเริ่มต้นรองรับ:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (คั่นด้วยเครื่องหมายจุลภาค)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## การแก้ไขปัญหา

- หากบอทเชื่อมต่อได้แต่ไม่เคยตอบกลับในช่อง ให้ตรวจสอบ `channels.irc.groups` **และ** ตรวจสอบว่า mention-gating กำลังกรองข้อความทิ้งหรือไม่ (`missing-mention`) หากต้องการให้ตอบกลับโดยไม่ต้อง ping ให้ตั้งค่า `requireMention:false` สำหรับช่องนั้น
- หากเข้าสู่ระบบไม่สำเร็จ ให้ตรวจสอบว่า nick ยังว่างและรหัสผ่านเซิร์ฟเวอร์ถูกต้อง
- หาก TLS ล้มเหลวบนเครือข่ายแบบกำหนดเอง ให้ตรวจสอบ host/port และการตั้งค่าใบรับรอง

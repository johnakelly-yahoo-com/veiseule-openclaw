---
title: "hooks"
---

# `openclaw hooks`

จัดการ agent hooks (อัตโนมัติแบบขับเคลื่อนด้วยอีเวนต์สำหรับคำสั่งอย่าง `/new`, `/reset` และการเริ่มต้นGateway)

เกี่ยวข้อง:

- Hooks: [Hooks](/automation/hooks)
- Plugin hooks: [Plugins](/tools/plugin#plugin-hooks)

## แสดงรายการHooksทั้งหมด

```bash
openclaw hooks list
```

แสดงรายการ hooks ทั้งหมดที่ค้นพบจากไดเรกทอรี workspace, managed และ bundled

**ตัวเลือก:**

- `--eligible`: แสดงเฉพาะ hooks ที่มีสิทธิ์ใช้งานได้ (ผ่านข้อกำหนดแล้ว)
- `--json`: เอาต์พุตเป็น JSON
- `-v, --verbose`: แสดงข้อมูลรายละเอียดรวมถึงข้อกำหนดที่ขาดหายไป

**ตัวอย่างเอาต์พุต:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**ตัวอย่าง (verbose):**

```bash
openclaw hooks list --verbose
```

แสดงข้อกำหนดที่ขาดหายไปสำหรับ hooks ที่ไม่มีสิทธิ์ใช้งาน

**ตัวอย่าง (JSON):**

```bash
openclaw hooks list --json
```

ส่งคืน JSON แบบมีโครงสร้างสำหรับการใช้งานเชิงโปรแกรม

## ดูข้อมูลHook

```bash
openclaw hooks info <name>
```

แสดงข้อมูลรายละเอียดของ hook เฉพาะรายการ

**อาร์กิวเมนต์:**

- `<name>`: ชื่อ hook (เช่น `session-memory`)

**ตัวเลือก:**

- `--json`: เอาต์พุตเป็น JSON

**ตัวอย่าง:**

```bash
openclaw hooks info session-memory
```

**เอาต์พุต:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## ตรวจสอบคุณสมบัติการใช้งาน Hooks

```bash
openclaw hooks check
```

แสดงสรุปสถานะความพร้อมของ hooks (จำนวนที่พร้อมใช้งานเทียบกับยังไม่พร้อม)

**ตัวเลือก:**

- `--json`: เอาต์พุตเป็น JSON

**ตัวอย่างเอาต์พุต:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## เปิดใช้งานHook

```bash
openclaw hooks enable <name>
```

เปิดใช้งาน hook เฉพาะโดยเพิ่มลงในคอนฟิกของคุณ (`~/.openclaw/config.json`)

**หมายเหตุ:** Hooks ที่ถูกจัดการโดยปลั๊กอินจะแสดง `plugin:<id>` ใน `openclaw hooks list` และ
ไม่สามารถเปิด/ปิดได้ที่นี่ ให้เปิด/ปิดปลั๊กอินแทน Enable/disable the plugin instead.

**อาร์กิวเมนต์:**

- `<name>`: ชื่อ hook (เช่น `session-memory`)

**ตัวอย่าง:**

```bash
openclaw hooks enable session-memory
```

**เอาต์พุต:**

```
✓ Enabled hook: 💾 session-memory
```

**สิ่งที่ทำ:**

- ตรวจสอบว่า hook มีอยู่และมีสิทธิ์ใช้งานได้
- อัปเดต `hooks.internal.entries.<name>.enabled = true` ในคอนฟิกของคุณ
- บันทึกคอนฟิกลงดิสก์

**หลังจากเปิดใช้งาน:**

- รีสตาร์ทGatewayเพื่อให้โหลด hooks ใหม่ (รีสตาร์ทแอปแถบเมนูบน macOS หรือรีสตาร์ทโปรเซสGatewayของคุณในโหมดพัฒนา)

## ปิดใช้งานHook

```bash
openclaw hooks disable <name>
```

ปิดใช้งาน hook เฉพาะโดยอัปเดตคอนฟิกของคุณ

**อาร์กิวเมนต์:**

- `<name>`: ชื่อ hook (เช่น `command-logger`)

**ตัวอย่าง:**

```bash
openclaw hooks disable command-logger
```

**เอาต์พุต:**

```
⏸ Disabled hook: 📝 command-logger
```

**หลังจากปิดใช้งาน:**

- รีสตาร์ทGatewayเพื่อให้โหลด hooks ใหม่

## ติดตั้งHooks

```bash
openclaw hooks install <path-or-spec>
```

ติดตั้งแพ็ก hook จากโฟลเดอร์/อาร์ไคฟ์ในเครื่องหรือจาก npm

**สิ่งที่ทำ:**

- คัดลอกแพ็ก hook ไปยัง `~/.openclaw/hooks/<id>`
- เปิดใช้งาน hooks ที่ติดตั้งใน `hooks.internal.entries.*`
- บันทึกการติดตั้งไว้ภายใต้ `hooks.internal.installs`

**ตัวเลือก:**

- `-l, --link`: ลิงก์ไดเรกทอรีในเครื่องแทนการคัดลอก (เพิ่มไปยัง `hooks.internal.load.extraDirs`)

**อาร์ไคฟ์ที่รองรับ:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**ตัวอย่าง:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Link a local directory without copying
openclaw hooks install -l ./my-hook-pack
```

## อัปเดตHooks

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

อัปเดตแพ็ก hook ที่ติดตั้งแล้ว (เฉพาะการติดตั้งจาก npm)

**ตัวเลือก:**

- `--all`: อัปเดตแพ็ก hook ที่ติดตามทั้งหมด
- `--dry-run`: แสดงสิ่งที่จะเปลี่ยนแปลงโดยไม่เขียนลงดิสก์

## Hooks ที่มาพร้อมชุด

### session-memory

บันทึกบริบทของเซสชันไว้ในหน่วยความจำเมื่อคุณสั่ง `/new`

**เปิดใช้งาน:**

```bash
openclaw hooks enable session-memory
```

**เอาต์พุต:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**ดู:** [เอกสารsession-memory](/automation/hooks#session-memory)

### command-logger

บันทึกอีเวนต์คำสั่งทั้งหมดไปยังไฟล์ตรวจสอบส่วนกลาง

**เปิดใช้งาน:**

```bash
openclaw hooks enable command-logger
```

**เอาต์พุต:** `~/.openclaw/logs/commands.log`

**ดูบันทึก:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**ดู:** [เอกสารcommand-logger](/automation/hooks#command-logger)

### soul-evil

สลับเนื้อหา `SOUL.md` ที่ถูกแทรกด้วย `SOUL_EVIL.md` ระหว่างช่วง purge หรือแบบสุ่ม

**เปิดใช้งาน:**

```bash
openclaw hooks enable soul-evil
```

**ดู:** [SOUL Evil Hook](/hooks/soul-evil)

### boot-md

รัน `BOOT.md` เมื่อGatewayเริ่มต้น (หลังจากช่องทางเริ่มทำงาน)

**อีเวนต์**: `gateway:startup`

**เปิดใช้งาน**:

```bash
openclaw hooks enable boot-md
```

**ดู:** [เอกสารboot-md](/automation/hooks#boot-md)


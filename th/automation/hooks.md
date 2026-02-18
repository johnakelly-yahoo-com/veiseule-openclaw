---
title: "Hooks"
---

# Hooks

Hooks provide an extensible event-driven system for automating actions in response to agent commands and events. Hooks มอบระบบแบบขยายได้ที่ขับเคลื่อนด้วยอีเวนต์สำหรับทำงานอัตโนมัติเพื่อตอบสนองต่อคำสั่งและอีเวนต์ของเอเจนต์ Hooks จะถูกค้นพบโดยอัตโนมัติจากไดเรกทอรี และสามารถจัดการได้ผ่านคำสั่ง CLI คล้ายกับวิธีที่ Skills ทำงานใน OpenClaw

## เริ่มต้นทำความรู้จัก

Hooks คือสคริปต์ขนาดเล็กที่ทำงานเมื่อมีบางสิ่งเกิดขึ้น โดยมีอยู่สองประเภท: There are two kinds:

- **Hooks** (หน้านี้): ทำงานภายใน Gateway เมื่อเกิดอีเวนต์ของเอเจนต์ เช่น `/new`, `/reset`, `/stop` หรืออีเวนต์วงจรชีวิต
- **Webhooks**: external HTTP webhooks that let other systems trigger work in OpenClaw. **Webhooks**: HTTP webhook ภายนอกที่เปิดให้ระบบอื่นเรียกใช้งานใน OpenClaw ดู [Webhook Hooks](/automation/webhook) หรือใช้ `openclaw webhooks` สำหรับคำสั่งผู้ช่วย Gmail

Hooks ยังสามารถบรรจุรวมอยู่ในปลั๊กอินได้ ดู [Plugins](/tools/plugin#plugin-hooks)

กรณีการใช้งานทั่วไป:

- บันทึกสแนปช็อตหน่วยความจำเมื่อคุณรีเซ็ตเซสชัน
- เก็บบันทึกการตรวจสอบคำสั่งสำหรับการแก้ไขปัญหาหรือการปฏิบัติตามข้อกำหนด
- กระตุ้นระบบอัตโนมัติต่อเนื่องเมื่อเซสชันเริ่มหรือสิ้นสุด
- เขียนไฟล์ลงในเวิร์กสเปซของเอเจนต์หรือเรียกใช้ API ภายนอกเมื่อเกิดอีเวนต์

หากคุณเขียนฟังก์ชัน TypeScript เล็กๆได้ คุณก็สามารถเขียน hook ได้ Hooks จะถูกค้นพบโดยอัตโนมัติ และคุณสามารถเปิดหรือปิดการใช้งานผ่าน CLI Hooks are discovered automatically, and you enable or disable them via the CLI.

## ภาพรวม

ระบบ hooks ช่วยให้คุณสามารถ:

- บันทึกบริบทของเซสชันลงในหน่วยความจำเมื่อมีการออกคำสั่ง `/new`
- บันทึกคำสั่งทั้งหมดเพื่อการตรวจสอบ
- เรียกใช้ระบบอัตโนมัติที่กำหนดเองเมื่อเกิดอีเวนต์วงจรชีวิตของเอเจนต์
- ขยายพฤติกรรมของ OpenClaw โดยไม่ต้องแก้ไขโค้ดแกนหลัก

## เริ่มต้นใช้งาน

### Hooks ที่มาพร้อมกับระบบ

OpenClaw มาพร้อม hook ที่บันเดิลไว้ 4 ตัวซึ่งจะถูกค้นพบโดยอัตโนมัติ:

- **💾 session-memory**: บันทึกบริบทของเซสชันลงในเวิร์กสเปซของเอเจนต์ (ค่าเริ่มต้น `~/.openclaw/workspace/memory/`) เมื่อคุณออกคำสั่ง `/new`
- **📝 command-logger**: บันทึกอีเวนต์คำสั่งทั้งหมดไปยัง `~/.openclaw/logs/commands.log`
- **🚀 boot-md**: รัน `BOOT.md` เมื่อ Gateway เริ่มทำงาน (ต้องเปิดใช้งาน internal hooks)
- **😈 soul-evil**: สลับเนื้อหา `SOUL.md` ที่ถูก inject กับ `SOUL_EVIL.md` ระหว่างช่วง purge หรือแบบสุ่ม

แสดงรายการ hook ที่มีอยู่:

```bash
openclaw hooks list
```

เปิดใช้งาน hook:

```bash
openclaw hooks enable session-memory
```

ตรวจสอบสถานะ hook:

```bash
openclaw hooks check
```

ดูข้อมูลโดยละเอียด:

```bash
openclaw hooks info session-memory
```

### การเริ่มต้นใช้งาน

ในระหว่างการเริ่มต้นใช้งาน (`openclaw onboard`) ระบบจะให้คุณเลือกเปิดใช้งาน hooks ที่แนะนำ วิซาร์ดจะค้นหา hooks ที่เข้าเกณฑ์โดยอัตโนมัติและแสดงรายการให้คุณเลือก

## การค้นพบ Hooks

Hooks จะถูกค้นพบโดยอัตโนมัติจากสามไดเรกทอรี (ตามลำดับความสำคัญ):

1. **Workspace hooks**: `<workspace>/hooks/` (ต่อเอเจนต์ มีความสำคัญสูงสุด)
2. **Managed hooks**: `~/.openclaw/hooks/` (ติดตั้งโดยผู้ใช้ ใช้ร่วมกันข้ามเวิร์กสเปซ)
3. **Bundled hooks**: `<openclaw>/dist/hooks/bundled/` (มากับ OpenClaw)

ไดเรกทอรี managed hook อาจเป็น **hook เดี่ยว** หรือ **hook pack** (ไดเรกทอรีแพ็กเกจ)

แต่ละ hook เป็นไดเรกทอรีที่ประกอบด้วย:

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## แพ็กเกจ Hook (npm/ไฟล์เก็บถาวร)

Hook pack คือแพ็กเกจ npm มาตรฐานที่ส่งออก hook หนึ่งตัวหรือมากกว่าผ่าน `openclaw.hooks` ใน
`package.json` ติดตั้งด้วยคำสั่ง: Install them with:

```bash
openclaw hooks install <path-or-spec>
```

ตัวอย่าง `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

แต่ละรายการชี้ไปยังไดเรกทอรี hook ที่มี `HOOK.md` และ `handler.ts` (หรือ `index.ts`)
Hook pack สามารถมาพร้อม dependency ได้ โดยจะถูกติดตั้งภายใต้ `~/.openclaw/hooks/<id>`
Hook packs can ship dependencies; they will be installed under `~/.openclaw/hooks/<id>`.

## โครงสร้าง Hook

### รูปแบบ HOOK.md

ไฟล์ `HOOK.md` ประกอบด้วยเมตาดาตาในรูปแบบ YAML frontmatter พร้อมเอกสาร Markdown:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### ฟิลด์เมตาดาตา

อ็อบเจ็กต์ `metadata.openclaw` รองรับ:

- **`emoji`**: อีโมจิที่แสดงใน CLI (เช่น `"💾"`)
- **`events`**: อาร์เรย์ของอีเวนต์ที่ต้องการรับฟัง (เช่น `["command:new", "command:reset"]`)
- **`export`**: named export ที่จะใช้ (ค่าเริ่มต้นคือ `"default"`)
- **`homepage`**: URL เอกสารประกอบ
- **`requires`**: ข้อกำหนดเพิ่มเติม (ไม่บังคับ)
  - **`bins`**: ไบนารีที่ต้องมีบน PATH (เช่น `["git", "node"]`)
  - **`anyBins`**: ต้องมีอย่างน้อยหนึ่งไบนารีจากรายการนี้
  - **`env`**: ตัวแปรสภาพแวดล้อมที่จำเป็น
  - **`config`**: พาธคอนฟิกที่จำเป็น (เช่น `["workspace.dir"]`)
  - **`os`**: แพลตฟอร์มที่รองรับ (เช่น `["darwin", "linux"]`)
- **`always`**: ข้ามการตรวจสอบคุณสมบัติ (บูลีน)
- **`install`**: วิธีการติดตั้ง (สำหรับ hook ที่บันเดิล: `[{"id":"bundled","kind":"bundled"}]`)

### Handler Implementation

ไฟล์ `handler.ts` จะ export ฟังก์ชัน `HookHandler`:

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const myHandler: HookHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### บริบทของอีเวนต์

แต่ละอีเวนต์ประกอบด้วย:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

## ประเภทอีเวนต์

### อีเวนต์คำสั่ง

ถูกกระตุ้นเมื่อมีการออกคำสั่งของเอเจนต์:

- **`command`**: อีเวนต์คำสั่งทั้งหมด (ตัวรับฟังทั่วไป)
- **`command:new`**: เมื่อมีการออกคำสั่ง `/new`
- **`command:reset`**: เมื่อมีการออกคำสั่ง `/reset`
- **`command:stop`**: เมื่อมีการออกคำสั่ง `/stop`

### อีเวนต์เอเจนต์

- **`agent:bootstrap`**: ก่อนที่จะ inject ไฟล์ bootstrap ของเวิร์กสเปซ (hook อาจแก้ไข `context.bootstrapFiles`)

### อีเวนต์ Gateway

ถูกกระตุ้นเมื่อ Gateway เริ่มทำงาน:

- **`gateway:startup`**: หลังจากช่องทางเริ่มทำงานและโหลด hook แล้ว

### Tool Result Hooks (Plugin API)

hook เหล่านี้ไม่ใช่ตัวรับฟังสตรีมอีเวนต์ แต่เปิดให้ปลั๊กอินปรับผลลัพธ์ของเครื่องมือแบบ synchronous ก่อนที่ OpenClaw จะบันทึก

- **`tool_result_persist`**: แปลงผลลัพธ์ของเครื่องมือก่อนเขียนลง transcript ของเซสชัน ต้องเป็น synchronous; ส่งคืน payload ผลลัพธ์ที่อัปเดตแล้ว หรือ `undefined` เพื่อคงค่าเดิม ดู [Agent Loop](/concepts/agent-loop) Must be synchronous; return the updated tool result payload or `undefined` to keep it as-is. See [Agent Loop](/concepts/agent-loop).

### อีเวนต์ในอนาคต

ประเภทอีเวนต์ที่วางแผนไว้:

- **`session:start`**: เมื่อเริ่มเซสชันใหม่
- **`session:end`**: เมื่อเซสชันสิ้นสุด
- **`agent:error`**: เมื่อเอเจนต์พบข้อผิดพลาด
- **`message:sent`**: เมื่อมีการส่งข้อความ
- **`message:received`**: เมื่อมีการรับข้อความ

## การสร้าง Hook แบบกำหนดเอง

### 1. เลือกตำแหน่ง

- **Workspace hooks** (`<workspace>/hooks/`): ต่อเอเจนต์ ความสำคัญสูงสุด
- **Managed hooks** (`~/.openclaw/hooks/`): ใช้ร่วมกันข้ามเวิร์กสเปซ

### 2. สร้างโครงสร้างไดเรกทอรี

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. สร้าง HOOK.md

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. สร้าง handler.ts

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const handler: HookHandler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // Your logic here
};

export default handler;
```

### 5. เปิดใช้งานและทดสอบ

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```

## การกำหนดค่า

### รูปแบบคอนฟิกใหม่ (แนะนำ)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### การกำหนดค่าราย Hook

Hooks สามารถมีการกำหนดค่าเฉพาะได้:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### ไดเรกทอรีเพิ่มเติม

โหลด hook จากไดเรกทอรีเพิ่มเติม:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### รูปแบบคอนฟิกเดิม (ยังรองรับ)

รูปแบบคอนฟิกแบบเดิมยังคงใช้งานได้เพื่อความเข้ากันได้ย้อนหลัง:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**การย้ายระบบ**: สำหรับ hook ใหม่ แนะนำให้ใช้ระบบค้นพบแบบใหม่ Legacy handler จะถูกโหลดหลัง hook แบบไดเรกทอรี Legacy handlers are loaded after directory-based hooks.

## คำสั่ง CLI

### แสดงรายการ Hooks

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Verbose output (show missing requirements)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

### ข้อมูล Hook

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### ตรวจสอบคุณสมบัติ

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### เปิด/ปิดการใช้งาน

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```

## เอกสารอ้างอิง Hook ที่บันเดิลมา

### session-memory

บันทึกบริบทของเซสชันลงในหน่วยความจำเมื่อคุณออกคำสั่ง `/new`.

**Events**: `command:new`

**Requirements**: ต้องตั้งค่า `workspace.dir`

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (ค่าเริ่มต้น `~/.openclaw/workspace`)

**ทำอะไรบ้าง**:

1. ใช้รายการเซสชันก่อนรีเซ็ตเพื่อระบุ transcript ที่ถูกต้อง
2. ดึง 15 บรรทัดล่าสุดของการสนทนา
3. ใช้ LLM เพื่อสร้าง slug ชื่อไฟล์แบบอธิบายได้
4. บันทึกเมตาดาตาของเซสชันลงในไฟล์หน่วยความจำตามวันที่

**ตัวอย่างเอาต์พุต**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**ตัวอย่างชื่อไฟล์**:

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (ใช้ timestamp สำรองหากการสร้าง slug ล้มเหลว)

**เปิดใช้งาน**:

```bash
openclaw hooks enable session-memory
```

### command-logger

บันทึกอีเวนต์คำสั่งทั้งหมดไปยังไฟล์ตรวจสอบส่วนกลาง

**Events**: `command`

**Requirements**: ไม่มี

**Output**: `~/.openclaw/logs/commands.log`

**ทำอะไรบ้าง**:

1. จับรายละเอียดอีเวนต์ (การกระทำของคำสั่ง, เวลา, คีย์เซสชัน, ID ผู้ส่ง, แหล่งที่มา)
2. ต่อท้ายไฟล์บันทึกในรูปแบบ JSONL
3. ทำงานแบบเงียบในเบื้องหลัง

**ตัวอย่างรายการบันทึก**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**ดูบันทึก**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**เปิดใช้งาน**:

```bash
openclaw hooks enable command-logger
```

### soul-evil

สลับเนื้อหา `SOUL.md` ที่ถูก inject กับ `SOUL_EVIL.md` ระหว่างช่วง purge หรือแบบสุ่ม

**Events**: `agent:bootstrap`

**Docs**: [SOUL Evil Hook](/hooks/soul-evil)

**Output**: ไม่มีการเขียนไฟล์ การสลับเกิดขึ้นในหน่วยความจำเท่านั้น

**เปิดใช้งาน**:

```bash
openclaw hooks enable soul-evil
```

**Config**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

### boot-md

รัน `BOOT.md` เมื่อ Gateway เริ่มทำงาน (หลังจากช่องทางเริ่มแล้ว)
ต้องเปิดใช้งาน internal hooks เพื่อให้ทำงานได้
Internal hooks must be enabled for this to run.

**Events**: `gateway:startup`

**Requirements**: ต้องตั้งค่า `workspace.dir`

**ทำอะไรบ้าง**:

1. อ่าน `BOOT.md` จากเวิร์กสเปซของคุณ
2. รันคำสั่งผ่าน agent runner
3. ส่งข้อความขาออกตามที่ร้องขอผ่าน message tool

**เปิดใช้งาน**:

```bash
openclaw hooks enable boot-md
```

## แนวปฏิบัติที่ดีที่สุด

### ทำให้ Handler เร็ว

Hooks run during command processing. Keep them lightweight:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### จัดการข้อผิดพลาดอย่างเหมาะสม

ห่อการทำงานที่มีความเสี่ยงเสมอ:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### กรองอีเวนต์ตั้งแต่เนิ่นๆ

คืนค่าทันทีหากอีเวนต์ไม่เกี่ยวข้อง:

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### ใช้คีย์อีเวนต์ที่เฉพาะเจาะจง

ระบุอีเวนต์ที่ชัดเจนในเมตาดาตาเมื่อเป็นไปได้:

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

แทนที่จะเป็น:

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## การดีบัก

### เปิดการบันทึก Hook

Gateway จะบันทึกการโหลด hook เมื่อเริ่มทำงาน:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### ตรวจสอบการค้นพบ

แสดง hook ที่ค้นพบทั้งหมด:

```bash
openclaw hooks list --verbose
```

### ตรวจสอบการลงทะเบียน

ใน handler ของคุณ ให้บันทึกเมื่อถูกเรียกใช้:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Verify Eligibility

ตรวจสอบสาเหตุที่ hook ไม่เข้าเกณฑ์:

```bash
openclaw hooks info my-hook
```

มองหาข้อกำหนดที่ขาดหายในเอาต์พุต

## การทดสอบ

### บันทึก Gateway

ติดตามบันทึกของ Gateway เพื่อดูการทำงานของ hook:

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### ทดสอบ Hook โดยตรง

ทดสอบ handler ของคุณแบบแยกส่วน:

```typescript
import { test } from "vitest";
import { createHookEvent } from "./src/hooks/hooks.js";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = createHookEvent("command", "new", "test-session", {
    foo: "bar",
  });

  await myHandler(event);

  // Assert side effects
});
```

## สถาปัตยกรรม

### องค์ประกอบหลัก

- **`src/hooks/types.ts`**: นิยามชนิดข้อมูล
- **`src/hooks/workspace.ts`**: การสแกนและโหลดไดเรกทอรี
- **`src/hooks/frontmatter.ts`**: การพาร์สเมตาดาตา HOOK.md
- **`src/hooks/config.ts`**: การตรวจสอบคุณสมบัติ
- **`src/hooks/hooks-status.ts`**: การรายงานสถานะ
- **`src/hooks/loader.ts`**: ตัวโหลดโมดูลแบบไดนามิก
- **`src/cli/hooks-cli.ts`**: คำสั่ง CLI
- **`src/gateway/server-startup.ts`**: โหลด hook เมื่อ Gateway เริ่มทำงาน
- **`src/auto-reply/reply/commands-core.ts`**: กระตุ้นอีเวนต์คำสั่ง

### โฟลว์การค้นพบ

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

### โฟลว์อีเวนต์

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

## การแก้ไขปัญหา

### Hook ไม่ถูกค้นพบ

1. ตรวจสอบโครงสร้างไดเรกทอรี:

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. ตรวจสอบรูปแบบ HOOK.md:

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. แสดง hook ที่ค้นพบทั้งหมด:

   ```bash
   openclaw hooks list
   ```

### Hook ไม่เข้าเกณฑ์

ตรวจสอบข้อกำหนด:

```bash
openclaw hooks info my-hook
```

มองหาสิ่งที่ขาดหายไป:

- ไบนารี (ตรวจสอบ PATH)
- ตัวแปรสภาพแวดล้อม
- ค่าคอนฟิก
- ความเข้ากันได้ของระบบปฏิบัติการ

### Hook ไม่ทำงาน

1. ตรวจสอบว่า hook เปิดใช้งานแล้ว:

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. รีสตาร์ตโปรเซส Gateway เพื่อให้โหลด hook ใหม่

3. ตรวจสอบบันทึก Gateway สำหรับข้อผิดพลาด:

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### ข้อผิดพลาดของ Handler

ตรวจสอบข้อผิดพลาด TypeScript/import:

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## คู่มือการย้ายระบบ

### 1. จาก Legacy Config สู่ Discovery

**ก่อน**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**หลัง**:

1. สร้างไดเรกทอรี hook:

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. สร้าง HOOK.md:

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. อัปเดตคอนฟิก:

   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. ตรวจสอบและรีสตาร์ตโปรเซส Gateway:

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**ประโยชน์ของการย้ายระบบ**:

- การค้นพบอัตโนมัติ
- การจัดการผ่าน CLI
- 2. การตรวจสอบคุณสมบัติ
- เอกสารที่ดีขึ้น
- โครงสร้างที่สอดคล้องกัน

## ดูเพิ่มเติม

- [CLI Reference: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Configuration](/gateway/configuration#hooks)

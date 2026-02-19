---
summary: "กระจายข้อความ WhatsApp ไปยังเอเจนต์หลายตัว"
read_when:
  - การกำหนดค่ากลุ่มบรอดแคสต์
  - การดีบักการตอบกลับจากหลายเอเจนต์ใน WhatsApp
status: experimental
title: "กลุ่มบรอดแคสต์"
---

# กลุ่มบรอดแคสต์

**สถานะ:** ทดลองใช้  
**เวอร์ชัน:** เพิ่มใน 2026.1.9

## ภาพรวม

Broadcast Groups ช่วยให้เอเจนต์หลายตัวสามารถประมวลผลและตอบกลับข้อความเดียวกันพร้อมกันได้ กลุ่มบรอดแคสต์ช่วยให้เอเจนต์หลายตัวประมวลผลและตอบกลับข้อความเดียวกันได้พร้อมกัน ซึ่งช่วยให้คุณสร้างทีมเอเจนต์เฉพาะทางที่ทำงานร่วมกันในกลุ่ม WhatsApp หรือ DM เดียว โดยใช้หมายเลขโทรศัพท์เพียงหมายเลขเดียว กลุ่มบรอดแคสต์ช่วยให้เอเจนต์หลายตัวประมวลผลและตอบกลับข้อความเดียวกันได้พร้อมกัน ซึ่งช่วยให้คุณสร้างทีมเอเจนต์เฉพาะทางที่ทำงานร่วมกันในกลุ่ม WhatsApp หรือ DM เดียว โดยใช้หมายเลขโทรศัพท์เพียงหมายเลขเดียว

ขอบเขตปัจจุบัน: **เฉพาะ WhatsApp** (ช่องทางเว็บ)

32. Broadcast groups จะถูกประเมินหลังจาก channel allowlists และกฎการเปิดใช้งานกลุ่ม Broadcast groups จะถูกประเมินหลังจาก channel allowlists และกฎการเปิดใช้งานกลุ่ม กลุ่มบรอดแคสต์จะถูกประเมินหลังจากรายการอนุญาตของช่องทางและกฎการเปิดใช้งานกลุ่ม ในกลุ่ม WhatsApp หมายความว่าการบรอดแคสต์จะเกิดขึ้นเมื่อ OpenClaw ควรตอบกลับตามปกติ (เช่น เมื่อมีการกล่าวถึง ทั้งนี้ขึ้นอยู่กับการตั้งค่ากลุ่มของคุณ)

## กรณีใช้งาน

### 1. ทีมเอเจนต์เฉพาะทาง

ปรับใช้เอเจนต์หลายตัวที่มีความรับผิดชอบแบบอะตอมมิกและโฟกัสชัดเจน:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

เอเจนต์แต่ละตัวจะประมวลผลข้อความเดียวกันและให้มุมมองเฉพาะทางของตน

### 2. การรองรับหลายภาษา

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. เวิร์กโฟลว์การประกันคุณภาพ

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. การทำงานอัตโนมัติของงาน

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## การกำหนดค่า

### การตั้งค่าพื้นฐาน

เพิ่มส่วน `broadcast` ระดับบนสุด (ถัดจาก `bindings`) คีย์คือ WhatsApp peer id: 33. คีย์คือ WhatsApp peer ids:

- แชทกลุ่ม: group JID (เช่น `120363403215116621@g.us`)
- DMs: หมายเลขโทรศัพท์รูปแบบ E.164 (เช่น `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**ผลลัพธ์:** เมื่อ OpenClaw ควรตอบกลับในแชทนี้ ระบบจะรันเอเจนต์ทั้งสามตัว

### กลยุทธ์การประมวลผล

ควบคุมวิธีที่เอเจนต์ประมวลผลข้อความ:

#### แบบขนาน (ค่าเริ่มต้น)

เอเจนต์ทั้งหมดประมวลผลพร้อมกัน:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### แบบลำดับ

เอเจนต์ประมวลผลตามลำดับ (ตัวถัดไปรอให้ตัวก่อนหน้าจบ):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### ตัวอย่างครบถ้วน

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## ทำงานอย่างไร

### โฟลว์ของข้อความ

1. **ข้อความขาเข้า** มาถึงในกลุ่ม WhatsApp
2. **ตรวจสอบบรอดแคสต์**: ระบบตรวจสอบว่า peer ID อยู่ใน `broadcast` หรือไม่
3. **หากอยู่ในรายการบรอดแคสต์**:
   - เอเจนต์ที่ระบุทั้งหมดประมวลผลข้อความ
   - เอเจนต์แต่ละตัวมีคีย์เซสชันของตนเองและบริบทที่แยกจากกัน
   - เอเจนต์ประมวลผลแบบขนาน (ค่าเริ่มต้น) หรือแบบลำดับ
4. **หากไม่อยู่ในรายการบรอดแคสต์**:
   - ใช้การกำหนดเส้นทางตามปกติ (binding ที่ตรงกันตัวแรก)

หมายเหตุ: กลุ่มบรอดแคสต์ไม่ข้ามรายการอนุญาตของช่องทางหรือกฎการเปิดใช้งานกลุ่ม (การกล่าวถึง/คำสั่ง ฯลฯ) พวกมันเพียงเปลี่ยนว่า _เอเจนต์ใดจะรัน_ เมื่อข้อความมีสิทธิ์ถูกประมวลผล 34. พวกมันจะเปลี่ยนแปลงเพียง _ว่าเอเจนต์ใดทำงาน_ เมื่อข้อความมีสิทธิ์ถูกประมวลผล

### การแยกเซสชัน

เอเจนต์แต่ละตัวในกลุ่มบรอดแคสต์จะคงไว้ซึ่งการแยกอย่างสมบูรณ์ดังต่อไปนี้:

- **คีย์เซสชัน** (`agent:alfred:whatsapp:group:120363...` เทียบกับ `agent:baerbel:whatsapp:group:120363...`)
- **ประวัติการสนทนา** (เอเจนต์ไม่เห็นข้อความของเอเจนต์อื่น)
- **เวิร์กสเปซ** (sandbox แยกกันหากกำหนดค่า)
- **การเข้าถึงเครื่องมือ** (รายการอนุญาต/ปฏิเสธที่ต่างกัน)
- **หน่วยความจำ/บริบท** (IDENTITY.md, SOUL.md ฯลฯ แยกกัน)
- **บัฟเฟอร์บริบทของกลุ่ม** (ข้อความกลุ่มล่าสุดที่ใช้เป็นบริบท) ถูกแชร์ต่อ peer ดังนั้นเอเจนต์บรอดแคสต์ทั้งหมดจะเห็นบริบทเดียวกันเมื่อถูกทริกเกอร์

สิ่งนี้ทำให้เอเจนต์แต่ละตัวสามารถมี:

- บุคลิกที่แตกต่างกัน
- การเข้าถึงเครื่องมือที่แตกต่างกัน (เช่น อ่านอย่างเดียว vs. อ่าน-เขียน)
- โมเดลที่แตกต่างกัน (เช่น opus vs. sonnet)
- Skills ที่ติดตั้งแตกต่างกัน

### ตัวอย่าง: เซสชันที่แยกจากกัน

ในกลุ่ม `120363403215116621@g.us` ที่มีเอเจนต์ `["alfred", "baerbel"]`:

**บริบทของ Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**บริบทของ Bärbel:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## แนวปฏิบัติที่ดีที่สุด

### 1. โฟกัสของเอเจนต์

ออกแบบเอเจนต์แต่ละตัวให้มีความรับผิดชอบเดียวที่ชัดเจน:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **ดี:** เอเจนต์แต่ละตัวมีงานเดียว  
❌ **ไม่ดี:** เอเจนต์ทั่วไปแบบ “dev-helper” ตัวเดียว

### 2. ใช้ชื่อที่สื่อความหมาย

ทำให้ชัดเจนว่าเอเจนต์แต่ละตัวทำอะไร:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. กำหนดการเข้าถึงเครื่องมือให้แตกต่าง

ให้เอเจนต์มีเฉพาะเครื่องมือที่จำเป็น:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. เฝ้าระวังประสิทธิภาพ

เมื่อมีเอเจนต์จำนวนมาก ควรพิจารณา:

- ใช้ `"strategy": "parallel"` (ค่าเริ่มต้น) เพื่อความเร็ว
- จำกัดกลุ่มบรอดแคสต์ไว้ที่ 5-10 เอเจนต์
- ใช้โมเดลที่เร็วกว่าสำหรับเอเจนต์ที่ง่ายกว่า

### 5. จัดการความล้มเหลวอย่างเหมาะสม

35. เอเจนต์ล้มเหลวแยกจากกัน เอเจนต์ล้มเหลวแยกจากกัน เอเจนต์ล้มเหลวแยกจากกัน ความผิดพลาดของเอเจนต์หนึ่งจะไม่บล็อกผู้อื่น:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## ความเข้ากันได้

### ผู้ให้บริการ

กลุ่มบรอดแคสต์ปัจจุบันทำงานร่วมกับ:

- ✅ WhatsApp (ใช้งานแล้ว)
- 🚧 Telegram (วางแผน)
- 🚧 Discord (วางแผน)
- 🚧 Slack (วางแผน)

### การกำหนดเส้นทาง

กลุ่มบรอดแคสต์ทำงานควบคู่กับการกำหนดเส้นทางเดิม:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: มีเพียง alfred ที่ตอบกลับ (การกำหนดเส้นทางปกติ)
- `GROUP_B`: agent1 และ agent2 ตอบกลับ (บรอดแคสต์)

**ลำดับความสำคัญ:** `broadcast` มีลำดับสูงกว่า `bindings`.

## การแก้ไขปัญหา

### เอเจนต์ไม่ตอบกลับ

**ตรวจสอบ:**

1. มี Agent ID อยู่ใน `agents.list`
2. รูปแบบ Peer ID ถูกต้อง (เช่น `120363403215116621@g.us`)
3. เอเจนต์ไม่อยู่ในรายการปฏิเสธ

**ดีบัก:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### มีเพียงเอเจนต์เดียวตอบกลับ

**สาเหตุ:** Peer ID อาจอยู่ใน `bindings` แต่ไม่อยู่ใน `broadcast`.

**วิธีแก้ไข:** เพิ่มเข้าไปในการกำหนดค่าบรอดแคสต์หรือลบออกจาก bindings

### ปัญหาด้านประสิทธิภาพ

**หากช้ากับเอเจนต์จำนวนมาก:**

- ลดจำนวนเอเจนต์ต่อกลุ่ม
- ใช้โมเดลที่เบากว่า (sonnet แทน opus)
- ตรวจสอบเวลาเริ่มต้นของ sandbox

## ตัวอย่าง

### ตัวอย่างที่ 1: ทีมตรวจทานโค้ด

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**ผู้ใช้ส่ง:** โค้ดสแนปเป็ต  
**การตอบกลับ:**

- code-formatter: "แก้ไขการย่อหน้าและเพิ่ม type hints แล้ว"
- security-scanner: "⚠️ พบช่องโหว่ SQL injection ที่บรรทัด 12"
- test-coverage: "Coverage คือ 45% ขาดการทดสอบกรณีข้อผิดพลาด"
- docs-checker: "ขาด docstring สำหรับฟังก์ชัน `process_data`"

### ตัวอย่างที่ 2: การรองรับหลายภาษา

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## เอกสารอ้างอิง API

### 36. สคีมาคอนฟิก

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### ฟิลด์

- `strategy` (ไม่บังคับ): วิธีการประมวลผลเอเจนต์
  - `"parallel"` (ค่าเริ่มต้น): เอเจนต์ทั้งหมดประมวลผลพร้อมกัน
  - `"sequential"`: เอเจนต์ประมวลผลตามลำดับของอาร์เรย์
- `[peerId]`: WhatsApp group JID, หมายเลข E.164 หรือ peer ID อื่น
  - ค่า: อาร์เรย์ของ Agent ID ที่ควรประมวลผลข้อความ

## ข้อจำกัด

1. **จำนวนเอเจนต์สูงสุด:** ไม่มีขีดจำกัดตายตัว แต่ 10+ เอเจนต์อาจช้า
2. **บริบทที่แชร์:** เอเจนต์ไม่เห็นการตอบกลับของกันและกัน (ออกแบบมาเช่นนี้)
3. **ลำดับข้อความ:** การตอบกลับแบบขนานอาจมาถึงไม่เรียงลำดับ
4. **อัตราการใช้งาน:** เอเจนต์ทั้งหมดนับรวมกับข้อจำกัดอัตราของ WhatsApp

## การปรับปรุงในอนาคต

ฟีเจอร์ที่วางแผนไว้:

- [ ] โหมดบริบทที่แชร์ (เอเจนต์เห็นการตอบกลับของกันและกัน)
- [ ] การประสานงานเอเจนต์ (เอเจนต์สามารถส่งสัญญาณถึงกัน)
- [ ] การเลือกเอเจนต์แบบไดนามิก (เลือกตามเนื้อหาข้อความ)
- [ ] ลำดับความสำคัญของเอเจนต์ (บางเอเจนต์ตอบก่อน)

## ดูเพิ่มเติม

- [การกำหนดค่าแบบหลายเอเจนต์](/tools/multi-agent-sandbox-tools)
- [การกำหนดค่าเส้นทาง](/channels/channel-routing)
- [การจัดการเซสชัน](/concepts/sessions)

---
title: "บริบท"
---

# บริบท

“Context” คือ **ทุกอย่างที่ OpenClaw ส่งให้โมเดลสำหรับการรันหนึ่งครั้ง** โดยถูกจำกัดด้วย **context window** ของโมเดล (ขีดจำกัดโทเคน) It is bounded by the model’s **context window** (token limit).

แบบจำลองทางความคิดสำหรับผู้เริ่มต้น:

- **System prompt** (สร้างโดย OpenClaw): กฎ เครื่องมือ รายการSkills เวลา/รันไทม์ และไฟล์เวิร์กสเปซที่ถูกฉีดเข้าไป
- **Conversation history**: ข้อความของคุณ + ข้อความของผู้ช่วยสำหรับเซสชันนี้
- **Tool calls/results + attachments**: เอาต์พุตคำสั่ง การอ่านไฟล์ รูปภาพ/เสียง ฯลฯ

Context _ไม่ใช่สิ่งเดียวกัน_ กับ “memory”: memory สามารถบันทึกลงดิสก์และโหลดกลับมาได้ภายหลัง ส่วน context คือสิ่งที่อยู่ภายในหน้าต่างปัจจุบันของโมเดล

## เริ่มต้นอย่างรวดเร็ว (ตรวจสอบ context)

- `/status` → มุมมองด่วนว่า “หน้าต่างของฉันเต็มแค่ไหน?” + การตั้งค่าเซสชัน
- `/context list` → สิ่งที่ถูกฉีดเข้าไป + ขนาดโดยประมาณ (ต่อไฟล์ + รวมทั้งหมด)
- `/context detail` → การแจกแจงเชิงลึก: ต่อไฟล์ ต่อขนาดสคีมาของเครื่องมือ ต่อขนาดเอนทรีของแต่ละSkill และขนาดของ system prompt
- `/usage tokens` → เพิ่มฟุตเตอร์การใช้งานต่อคำตอบในคำตอบปกติ
- `/compact` → สรุปประวัติที่เก่ากว่าให้เป็นเอนทรีแบบกะทัดรัดเพื่อคืนพื้นที่หน้าต่าง

ดูเพิ่มเติม: [Slash commands](/tools/slash-commands), [Token use & costs](/reference/token-use), [Compaction](/concepts/compaction)

## ตัวอย่างเอาต์พุต

ค่าจะแตกต่างกันตามโมเดล ผู้ให้บริการ นโยบายเครื่องมือ และสิ่งที่อยู่ในเวิร์กสเปซของคุณ

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## อะไรบ้างที่นับรวมใน context window

ทุกอย่างที่โมเดลได้รับจะถูกนับรวม ได้แก่:

- System prompt (ทุกส่วน)
- Conversation history
- Tool calls + tool results
- Attachments/transcripts (รูปภาพ/เสียง/ไฟล์)
- สรุปจากการ compaction และอาร์ติแฟกต์จากการตัดทอน
- “wrapper” หรือเฮดเดอร์ที่ซ่อนอยู่ของผู้ให้บริการ (มองไม่เห็นแต่ยังนับรวม)

## OpenClaw สร้าง system prompt อย่างไร

System prompt เป็น **ของ OpenClaw** และถูกสร้างใหม่ทุกครั้งที่รัน โดยประกอบด้วย: It includes:

- รายการเครื่องมือ + คำอธิบายสั้นๆ
- รายการSkills (เฉพาะเมทาดาทา; ดูด้านล่าง)
- ตำแหน่งเวิร์กสเปซ
- เวลา (UTC + เวลาผู้ใช้ที่แปลงแล้วหากตั้งค่าไว้)
- เมทาดาทารันไทม์ (โฮสต์/OS/โมเดล/การคิด)
- ไฟล์บูตสแตรปจากเวิร์กสเปซที่ถูกฉีดเข้าไปภายใต้ **Project Context**

แจกแจงทั้งหมด: [System Prompt](/concepts/system-prompt)

## ไฟล์เวิร์กสเปซที่ถูกฉีดเข้าไป (Project Context)

โดยค่าเริ่มต้น OpenClaw จะฉีดไฟล์เวิร์กสเปซชุดคงที่ (ถ้ามี):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (เฉพาะครั้งแรกที่รัน)

ไฟล์ขนาดใหญ่จะถูกตัดทอนต่อไฟล์ด้วย `agents.defaults.bootstrapMaxChars` (ค่าเริ่มต้น `20000` ตัวอักษร) โดย `/context` จะแสดงขนาด **ดิบเทียบกับที่ถูกฉีด** และบอกว่ามีการตัดทอนหรือไม่ `/context` shows **raw vs injected** sizes and whether truncation happened.

## Skills: อะไรถูกฉีดเข้าไป vs โหลดตามต้องการ

System prompt จะมี **skills list** แบบกะทัดรัด (ชื่อ + คำอธิบาย + ตำแหน่ง) ซึ่งมีโอเวอร์เฮดจริง This list has real overhead.

Skill instructions are _not_ included by default. คำสั่งของSkill _จะไม่_ ถูกรวมมาโดยค่าเริ่มต้น โมเดลคาดว่าจะ `read` `SKILL.md` ของSkill **เฉพาะเมื่อจำเป็น**

## Tools: มีต้นทุนสองส่วน

เครื่องมือมีผลต่อ context สองทาง:

1. **ข้อความรายการเครื่องมือ** ใน system prompt (สิ่งที่คุณเห็นเป็น “Tooling”)
2. **โครงร่างของเครื่องมือ** (JSON) ข้อมูลเหล่านี้จะถูกส่งไปยังโมเดลเพื่อให้สามารถเรียกใช้เครื่องมือได้ และจะถูกนับรวมเป็นส่วนหนึ่งของบริบท แม้ว่าคุณจะไม่เห็นเป็นข้อความปกติก็ตาม

`/context detail` จะแจกแจงสคีมาเครื่องมือที่ใหญ่ที่สุดเพื่อให้คุณเห็นว่าอะไรเป็นตัวหลักที่กินพื้นที่

## Commands, directives และ “inline shortcuts”

Slash commands ถูกจัดการโดย Gateway มีพฤติกรรมที่แตกต่างกันดังนี้: There are a few different behaviors:

- **Standalone commands**: ข้อความที่เป็นเพียง `/...` จะรันเป็นคำสั่ง
- **Directives**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` จะถูกตัดออกก่อนที่โมเดลจะเห็นข้อความ
  - ข้อความที่มีเฉพาะ directive จะคงการตั้งค่าเซสชันไว้
  - directive แบบ inline ในข้อความปกติจะทำหน้าที่เป็นคำใบ้ต่อข้อความนั้น
- **Inline shortcuts** (เฉพาะผู้ส่งที่อยู่ในรายการอนุญาต): โทเคน `/...` บางตัวภายในข้อความปกติสามารถรันทันทีได้ (เช่น “hey /status”) และจะถูกตัดออกก่อนที่โมเดลจะเห็นข้อความที่เหลือ

รายละเอียด: [Slash commands](/tools/slash-commands)

## Sessions, compaction และ pruning (อะไรที่คงอยู่)

สิ่งที่คงอยู่ข้ามข้อความขึ้นอยู่กับกลไก:

- **Normal history** คงอยู่ในทรานสคริปต์ของเซสชันจนกว่าจะถูก compacted/pruned ตามนโยบาย
- **Compaction** จะคงสรุปไว้ในทรานสคริปต์และเก็บข้อความล่าสุดไว้ครบถ้วน
- **Pruning** จะลบผลลัพธ์เครื่องมือเก่าออกจากพรอมต์ _ในหน่วยความจำ_ สำหรับการรันหนึ่งครั้ง แต่จะไม่เขียนทรานสคริปต์ใหม่

เอกสาร: [Session](/concepts/session), [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning)

## สิ่งที่ `/context` รายงานจริงๆ

`/context` จะเลือกใช้รายงาน system prompt ที่ **สร้างจากการรันล่าสุด** เมื่อมีให้ใช้:

- `System prompt (run)` = จับมาจากการรันแบบฝัง (รองรับเครื่องมือ) ครั้งล่าสุด และบันทึกไว้ในสโตร์ของเซสชัน
- `System prompt (estimate)` = คำนวณแบบทันทีเมื่อไม่มีรายงานจากการรัน (หรือเมื่อรันผ่านแบ็กเอนด์ CLI ที่ไม่สร้างรายงาน)

ไม่ว่าจะกรณีใด จะรายงานขนาดและผู้มีส่วนร่วมหลักๆ โดย **ไม่** ดัมพ์ system prompt หรือสคีมาเครื่องมือทั้งหมด



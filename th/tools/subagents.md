---
title: "ซับเอเจนต์"
---

# ซับเอเจนต์

ซับเอเจนต์คือการรันเอเจนต์เบื้องหลังที่ถูกสร้างจากการรันเอเจนต์ที่มีอยู่แล้ว โดยจะทำงานในเซสชันของตัวเอง (`agent:<agentId>:subagent:<uuid>`) และเมื่อเสร็จสิ้นจะ **ประกาศ (announce)** ผลลัพธ์กลับไปยังช่องแชตของผู้ร้องขอ

## คำสั่ง Slash

ใช้ `/subagents` เพื่อตรวจสอบหรือควบคุมการรันซับเอเจนต์สำหรับ **เซสชันปัจจุบัน**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` จะแสดงเมทาดาทาของการรัน (สถานะ เวลา session id พาธทรานสคริปต์ การล้างค่า)

เป้าหมายหลัก:

- ทำงานประเภท “ค้นคว้า / งานยาว / เครื่องมือที่ช้า” แบบขนาน โดยไม่บล็อกการรันหลัก
- แยกซับเอเจนต์ออกจากกันโดยค่าเริ่มต้น (แยกเซสชัน + sandbox เสริมได้)
- ทำให้พื้นผิวเครื่องมือใช้งานผิดพลาดได้ยาก: ซับเอเจนต์จะ **ไม่ได้รับ** session tools โดยค่าเริ่มต้น
- รองรับความลึกของการ spawn ที่กำหนดค่าได้ สำหรับรูปแบบ orchestrator

หมายเหตุเรื่องค่าใช้จ่าย: ซับเอเจนต์แต่ละตัวมีบริบทและการใช้โทเค็นเป็น **ของตัวเอง** สำหรับงานหนักหรือทำซ้ำบ่อย ควรตั้งค่าโมเดลที่ถูกกว่าสำหรับซับเอเจนต์ และคงโมเดลคุณภาพสูงไว้ให้เอเจนต์หลัก คุณสามารถตั้งค่าได้ผ่าน `agents.defaults.subagents.model` หรือ override รายเอเจนต์

## เครื่องมือ

ใช้ `sessions_spawn`:

- เริ่มการรันซับเอเจนต์ (`deliver: false`, global lane: `subagent`)
- จากนั้นรันขั้นตอน announce และโพสต์คำตอบ announce ไปยังช่องแชตของผู้ร้องขอ
- โมเดลเริ่มต้น: สืบทอดจากผู้เรียก เว้นแต่คุณจะตั้งค่า `agents.defaults.subagents.model` (หรือ `agents.list[].subagents.model` รายเอเจนต์); หากระบุ `sessions_spawn.model` อย่างชัดเจนจะมีผลเหนือกว่า
- การคิด (thinking) เริ่มต้น: สืบทอดจากผู้เรียก เว้นแต่คุณจะตั้งค่า `agents.defaults.subagents.thinking` (หรือ `agents.list[].subagents.thinking` รายเอเจนต์); หากระบุ `sessions_spawn.thinking` อย่างชัดเจนจะมีผลเหนือกว่า

พารามิเตอร์ของเครื่องมือ:

- `task` (จำเป็น)
- `label?` (ไม่บังคับ)
- `agentId?` (ไม่บังคับ; สร้างภายใต้ agent id อื่นหากได้รับอนุญาต)
- `model?` (ไม่บังคับ; override โมเดลของซับเอเจนต์; ค่าที่ไม่ถูกต้องจะถูกข้าม และซับเอเจนต์จะรันด้วยโมเดลเริ่มต้นพร้อมคำเตือนในผลลัพธ์ของเครื่องมือ)
- `thinking?` (ไม่บังคับ; override ระดับการคิดสำหรับการรันซับเอเจนต์)
- `runTimeoutSeconds?` (ค่าเริ่มต้น `0`; หากตั้งค่าไว้ ซับเอเจนต์จะถูกยกเลิกหลัง N วินาที)
- `cleanup?` (`delete|keep`, ค่าเริ่มต้น `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: รายการ agent id ที่สามารถระบุผ่าน `agentId` ได้ (`["*"]` เพื่ออนุญาตทั้งหมด) ค่าเริ่มต้น: เฉพาะเอเจนต์ผู้ร้องขอ

Discovery:

- ใช้ `agents_list` เพื่อดูว่า agent id ใดบ้างที่อนุญาตสำหรับ `sessions_spawn` ในขณะนี้

Auto-archive:

- เซสชันซับเอเจนต์จะถูกเก็บถาวรโดยอัตโนมัติหลัง `agents.defaults.subagents.archiveAfterMinutes` (ค่าเริ่มต้น: 60)
- การ archive ใช้ `sessions.delete` และเปลี่ยนชื่อทรานสคริปต์เป็น `*.deleted.<timestamp>` (โฟลเดอร์เดียวกัน)
- `cleanup: "delete"` จะ archive ทันทีหลัง announce (ยังคงเก็บทรานสคริปต์ผ่านการเปลี่ยนชื่อ)
- Auto-archive เป็นแบบ best-effort; timer ที่รอดำเนินการจะหายไปหาก gateway รีสตาร์ต
- `runTimeoutSeconds` จะ **ไม่** auto-archive; จะเพียงหยุดการรัน เซสชันจะคงอยู่จนกว่า auto-archive ทำงาน
- Auto-archive ใช้กับทั้งเซสชัน depth-1 และ depth-2

## Nested Sub-Agents

โดยค่าเริ่มต้น ซับเอเจนต์ไม่สามารถสร้างซับเอเจนต์ของตนเองได้ (`maxSpawnDepth: 1`) คุณสามารถเปิดใช้งานการซ้อนหนึ่งระดับโดยตั้งค่า `maxSpawnDepth: 2` ซึ่งจะรองรับรูปแบบ **orchestrator**: main → orchestrator sub-agent → worker sub-sub-agents

### วิธีเปิดใช้งาน

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // allow sub-agents to spawn children (default: 1)
        maxChildrenPerAgent: 5, // max active children per agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
      },
    },
  },
}
```

### ระดับความลึก

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | เอเจนต์หลัก                                   | เสมอ                        |
| 1     | `agent:<id>:subagent:<uuid>`                 | ซับเอเจนต์ (orchestrator เมื่ออนุญาต depth 2) | เฉพาะเมื่อ `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | ซับซับเอเจนต์ (worker ปลายทาง)                | ไม่ได้                       |

### Announce chain

ผลลัพธ์จะไหลย้อนกลับตามลำดับ:

1. worker ระดับ depth-2 เสร็จสิ้น → announce ไปยัง parent (orchestrator ระดับ depth-1)
2. orchestrator ระดับ depth-1 รับ announce สังเคราะห์ผลลัพธ์ แล้วเสร็จสิ้น → announce ไปยัง main
3. เอเจนต์หลักรับ announce และส่งต่อให้ผู้ใช้

แต่ละระดับจะเห็น announce เฉพาะจากลูกโดยตรงของตนเท่านั้น

### นโยบายเครื่องมือตามระดับความลึก

- **Depth 1 (orchestrator เมื่อ `maxSpawnDepth >= 2`)**: ได้รับ `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` เพื่อจัดการลูกของตน เครื่องมือ session/system อื่นยังถูกปฏิเสธ
- **Depth 1 (leaf เมื่อ `maxSpawnDepth == 1`)**: ไม่มี session tools (พฤติกรรมค่าเริ่มต้นปัจจุบัน)
- **Depth 2 (leaf worker)**: ไม่มี session tools — `sessions_spawn` ถูกปฏิเสธเสมอที่ depth 2 ไม่สามารถสร้างลูกต่อได้

### ขีดจำกัดการ spawn ต่อเอเจนต์

แต่ละเซสชันเอเจนต์ (ทุกระดับความลึก) สามารถมีลูกที่ active ได้สูงสุด `maxChildrenPerAgent` (ค่าเริ่มต้น: 5) ในเวลาเดียวกัน ป้องกันการแตกแขนงเกินควบคุมจาก orchestrator เดียว

### Cascade stop

การหยุด orchestrator ระดับ depth-1 จะหยุดลูกระดับ depth-2 ทั้งหมดโดยอัตโนมัติ:

- `/stop` ในแชตหลักจะหยุดเอเจนต์ depth-1 ทั้งหมด และ cascade ไปยังลูก depth-2
- `/subagents kill <id>` จะหยุดซับเอเจนต์เฉพาะ และ cascade ไปยังลูกของมัน
- `/subagents kill all` จะหยุดซับเอเจนต์ทั้งหมดของผู้ร้องขอ และ cascade

## การยืนยันตัวตน

การยืนยันตัวตนของซับเอเจนต์ถูกกำหนดโดย **agent id** ไม่ใช่ประเภทของเซสชัน:

- คีย์เซสชันของซับเอเจนต์คือ `agent:<agentId>:subagent:<uuid>`
- โหลด auth store จาก `agentDir` ของเอเจนต์นั้น
- โปรไฟล์ auth ของเอเจนต์หลักจะถูกรวมเข้ามาเป็น **fallback**; โปรไฟล์ของเอเจนต์จะมีผลเหนือกว่าเมื่อมีความขัดแย้ง

หมายเหตุ: การ merge เป็นแบบ additive ดังนั้นโปรไฟล์ของเอเจนต์หลักจะพร้อมใช้งานเสมอในฐานะ fallback ยังไม่รองรับการแยก auth แบบสมบูรณ์ต่อเอเจนต์

## Announce

ซับเอเจนต์รายงานผลผ่านขั้นตอน announce:

- ขั้นตอน announce จะรันภายในเซสชันของซับเอเจนต์ (ไม่ใช่เซสชันของผู้ร้องขอ)
- หากซับเอเจนต์ตอบกลับว่า `ANNOUNCE_SKIP` พอดี จะไม่มีการโพสต์ใด ๆ
- มิฉะนั้น คำตอบ announce จะถูกโพสต์ไปยังช่องแชตของผู้ร้องขอผ่านการเรียก `agent` แบบ follow-up (`deliver=true`)
- คำตอบ announce จะคงการกำหนดเส้นทางเธรด/หัวข้อเมื่อมีให้ (Slack threads, Telegram topics, Matrix threads)
- ข้อความ announce ถูกทำให้เป็นรูปแบบมาตรฐาน:
  - `Status:` มาจากผลลัพธ์ของการรัน (`success`, `error`, `timeout`, หรือ `unknown`)
  - `Result:` เนื้อหาสรุปจากขั้นตอน announce (หรือ `(not available)` หากไม่มี)
  - `Notes:` รายละเอียดข้อผิดพลาดและบริบทที่เป็นประโยชน์อื่น ๆ
- ค่า `Status` ไม่ได้อนุมานจากเอาต์พุตของโมเดล แต่มาจากสัญญาณผลลัพธ์ของ runtime

payload ของ announce จะมีบรรทัดสถิติท้ายข้อความ (แม้เมื่อถูกครอบไว้):

- Runtime (เช่น `runtime 5m12s`)
- การใช้โทเค็น (input/output/total)
- ค่าใช้จ่ายโดยประมาณเมื่อมีการตั้งค่าราคาของโมเดล (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` และพาธทรานสคริปต์ (เพื่อให้เอเจนต์หลักเรียก `sessions_history` หรือเปิดไฟล์บนดิสก์ได้)

## Tool Policy (sub-agent tools)

โดยค่าเริ่มต้น ซับเอเจนต์จะได้รับ **เครื่องมือทั้งหมด ยกเว้น session tools** และ system tools:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

เมื่อ `maxSpawnDepth >= 2`, ซับเอเจนต์ระดับ depth-1 แบบ orchestrator จะได้รับ `sessions_spawn`, `subagents`, `sessions_list`, และ `sessions_history` เพิ่มเติม เพื่อจัดการลูกของตน

Override ผ่านการตั้งค่า:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny มีผลเหนือเสมอ
        deny: ["gateway", "cron"],
        // หากตั้ง allow จะกลายเป็น allow-only (deny ยังมีผลเหนือ)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

ซับเอเจนต์ใช้คิว lane ภายในโปรเซสโดยเฉพาะ:

- ชื่อ lane: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (ค่าเริ่มต้น `8`)

## Stopping

- การส่ง `/stop` ในแชตของผู้ร้องขอจะยกเลิกเซสชันผู้ร้องขอ และหยุดการรันซับเอเจนต์ที่ active ทั้งหมดที่ถูกสร้างจากมัน พร้อม cascade ไปยังลูกที่ซ้อนกัน
- `/subagents kill <id>` จะหยุดซับเอเจนต์เฉพาะ และ cascade ไปยังลูกของมัน

## Limitations

- การ announce ของซับเอเจนต์เป็นแบบ **best-effort** หาก gateway รีสตาร์ต งาน “announce back” ที่รอดำเนินการจะสูญหาย
- ซับเอเจนต์ยังคงใช้ทรัพยากรของ gateway โปรเซสเดียวกัน; ใช้ `maxConcurrent` เป็นวาล์วความปลอดภัย
- `sessions_spawn` เป็นแบบ non-blocking เสมอ: จะคืนค่า `{ status: "accepted", runId, childSessionKey }` ทันที
- บริบทของซับเอเจนต์จะ inject เฉพาะ `AGENTS.md` + `TOOLS.md` (ไม่มี `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, หรือ `BOOTSTRAP.md`)
- ความลึกการซ้อนสูงสุดคือ 5 (`maxSpawnDepth` ช่วง: 1–5) แนะนำ depth 2 สำหรับกรณีใช้งานส่วนใหญ่
- `maxChildrenPerAgent` จำกัดจำนวนลูกที่ active ต่อเซสชัน (ค่าเริ่มต้น: 5, ช่วง: 1–20)
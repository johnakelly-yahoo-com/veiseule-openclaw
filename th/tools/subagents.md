---
summary: "ซับเอเจนต์: การสปอว์นการรันเอเจนต์ที่แยกจากกันและประกาศผลกลับไปยังแชทของผู้ร้องขอ"
read_when:
  - คุณต้องการงานเบื้องหลัง/ขนานผ่านเอเจนต์
  - คุณกำลังเปลี่ยน sessions_spawn หรือนโยบายเครื่องมือของซับเอเจนต์
title: "ซับเอเจนต์"
---

# ซับเอเจนต์

Sub-agents คือการรันเอเจนต์แบบเบื้องหลังที่ถูกสร้างจากการรันเอเจนต์ที่มีอยู่ พวกมันทำงานในเซสชันของตนเอง (`agent:<agentId>:subagent:<uuid>`) และเมื่อเสร็จสิ้น จะ **ประกาศ** ผลลัพธ์กลับไปยังช่องแชตของผู้ร้องขอ

## คำสั่ง

ใช้คำสั่งสแลช `/subagents` เพื่อตรวจสอบและควบคุมการทำงานของซับเอเจนต์สำหรับเซสชันปัจจุบัน:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` แสดงเมทาดาทาของการรัน (สถานะ, เวลา, session id, พาธ transcript, การล้างข้อมูล)

เป้าหมายหลัก:

- ทำงานประเภท "research / งานยาว / เครื่องมือช้า" แบบขนานโดยไม่บล็อกรันหลัก
- แยก sub-agents ออกจากกันเป็นค่าเริ่มต้น (แยกเซสชัน + sandbox แบบกำหนดได้)
- ทำให้พื้นผิวของเครื่องมือใช้งานผิดวิธีได้ยาก: sub-agents จะ **ไม่ได้** รับ session tools โดยค่าเริ่มต้น
- รองรับความลึกของการซ้อนแบบกำหนดค่าได้สำหรับรูปแบบ orchestrator

หมายเหตุด้านต้นทุน: แต่ละ sub-agent มี context และการใช้โทเค็นเป็นของตนเอง สำหรับงานหนักหรือทำซ้ำ
ให้ตั้งค่าโมเดลที่ประหยัดกว่าสำหรับ sub-agents และคงเอเจนต์หลักของคุณไว้บนโมเดลคุณภาพสูงกว่า
In a multi-agent setup, you can set sub-agent defaults per agent:

## เครื่องมือ

Explicit `model` parameter in the `sessions_spawn` call

- Global default: `agents.defaults.subagents.thinking`
- จากนั้นจะรันขั้นตอน announce และโพสต์ข้อความตอบกลับ announce ไปยังแชนแนลแชทของผู้ร้องขอ
- โมเดลเริ่มต้น: สืบทอดจากตัวเรียกใช้งาน เว้นแต่คุณจะตั้งค่า `agents.defaults.subagents.model` (หรือกำหนดรายเอเจนต์ที่ `agents.list[].subagents.model`); หากระบุ `sessions_spawn.model` อย่างชัดเจน ค่านั้นจะมีลำดับความสำคัญสูงสุด
- Per-agent config: `agents.list[].subagents.thinking`

พารามิเตอร์ของทูล:

- `task`
- `label`
- Spawn under a different agent id (must be allowed)
- โมเดล: ใช้การเลือกโมเดลปกติของเอเจนต์เป้าหมาย (เว้นแต่จะตั้งค่า `subagents.model`)
- การคิด: ไม่มีการ override สำหรับซับเอเจนต์ (เว้นแต่จะตั้งค่า `subagents.thinking`)
- Abort the sub-agent after N seconds
- `"delete"` \\

Allowlist:

- 5. {
     agents: {
     list: [
     {
     id: "orchestrator",
     subagents: {
     allowAgents: ["researcher", "coder"], // หรือ ["\*"] เพื่ออนุญาตทั้งหมด
     },
     },
     ],
     },
     } ค่าเริ่มต้น: เฉพาะเอเจนต์ผู้ร้องขอเท่านั้น

การค้นหา:

- ใช้เครื่องมือ `agents_list` เพื่อดูว่า agent id ใดบ้างที่อนุญาตสำหรับ `sessions_spawn` ในขณะนี้
</Tip>

Auto-Archive

- Sub-agent sessions are automatically archived after a configurable period:
- Archive renames the transcript to `*.deleted. ` (โฟลเดอร์เดียวกัน)
- `"delete"` archives immediately after announce
- \` (โฟลเดอร์เดียวกัน) — มีการเก็บบันทึกการถอดเสียงไว้ ไม่ได้ถูกลบ Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
- `runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
- การเก็บถาวรอัตโนมัติใช้เหมือนกันทั้งเซสชันระดับความลึก 1 และ 2

## Stopping Sub-Agents

- **No nested spawning:** Sub-agents cannot spawn their own sub-agents. คุณสามารถเปิดใช้งานการซ้อนกันได้หนึ่งระดับโดยตั้งค่า `maxSpawnDepth: 2` ซึ่งจะรองรับ **orchestrator pattern**: main → orchestrator sub-agent → worker sub-sub-agents

### วิธีเปิดใช้งาน

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### ระดับความลึก

| ความลึก | รูปแบบคีย์เซสชัน                                                                                                                                                                                         | บทบาท                                                                         | สามารถ spawn ได้หรือไม่?        |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------- |
| 0       | `agentId`                                                                                                                                                                                                | _(caller's agent)_                                         | เสมอ                            |
| 1       | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                         | Sub-agent (ทำหน้าที่เป็น orchestrator เมื่ออนุญาต depth 2) | เฉพาะเมื่อ `maxSpawnDepth >= 2` |
| 2       | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;archiveAfterMinutes: 120, // default: 60&#xA;},&#xA;},&#xA;},&#xA;} | Sub-sub-agent (worker ปลายทาง)                             | ไม่เคย                          |

### ลำดับการ announce

ผลลัพธ์จะไหลย้อนกลับขึ้นมาตามลำดับดังนี้:

1. Worker ระดับความลึก 2 ทำงานเสร็จ → announce ไปยัง parent ของตน (orchestrator ระดับความลึก 1)
2. Orchestrator ระดับความลึก 1 รับ announce สังเคราะห์ผลลัพธ์ แล้วเสร็จสิ้น → announce ไปยัง main
3. เอเจนต์ main รับ announce และส่งต่อให้ผู้ใช้

แต่ละระดับจะเห็นเฉพาะ announce จากลูกโดยตรงของตนเท่านั้น

### นโยบายทูลตามระดับความลึก

- **Depth 1 (orchestrator, เมื่อ `maxSpawnDepth >= 2`)**: ได้รับ `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` เพื่อให้สามารถจัดการลูกของตนได้ ทูล session/system อื่น ๆ ยังคงถูกปฏิเสธ
- **Depth 1 (leaf, เมื่อ `maxSpawnDepth == 1`)**: ไม่มีทูล session (พฤติกรรมเริ่มต้นปัจจุบัน)
- **Depth 2 (leaf worker)**: ไม่มีทูล session — `sessions_spawn` จะถูกปฏิเสธเสมอที่ depth 2 ไม่สามารถ spawn ลูกเพิ่มเติมได้

### Per-Agent Overrides

แต่ละเซสชันของเอเจนต์ (ในทุกระดับความลึก) สามารถมีลูกที่กำลังทำงานอยู่ได้สูงสุด `maxChildrenPerAgent` (ค่าเริ่มต้น: 5) ในเวลาเดียวกัน เพื่อป้องกันการแตกแขนงมากเกินไปจาก orchestrator เพียงตัวเดียว

### การหยุดแบบลูกโซ่

เมื่อหยุด orchestrator ระดับความลึก 1 จะหยุดลูกระดับความลึก 2 ทั้งหมดโดยอัตโนมัติ:

- พิมพ์ `/stop` ในแชทหลักเพื่อหยุดเอเจนต์ระดับความลึก 1 ทั้งหมด และจะหยุดลูกระดับความลึก 2 ของพวกเขาแบบลูกโซ่ด้วย
- `/subagents stop <id>`
- `/subagents kill all` จะหยุด sub-agent ทั้งหมดของผู้ร้องขอและหยุดแบบต่อเนื่องไปยังลำดับถัดไป

## การยืนยันตัวตน

การยืนยันตัวตนของซับเอเจนต์ถูกกำหนดโดย **เอเจนต์ไอดี** ไม่ใช่โดยประเภทเซสชัน:

- คีย์เซสชันของ sub-agent คือ `agent:<agentId>:subagent:<uuid>`.
- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)

The merge is additive — main profiles are always available as fallbacks
Fully isolated auth per sub-agent is not currently supported.

## ประกาศ

sub-agent จะรายงานกลับผ่านขั้นตอนประกาศ:

- ขั้นตอนประกาศจะทำงานภายในเซสชันของ sub-agent (ไม่ใช่เซสชันของผู้ร้องขอ)
- หาก sub-agent ตอบกลับว่า `ANNOUNCE_SKIP` ตรงตัว จะไม่มีการโพสต์ใด ๆ
- มิฉะนั้น ข้อความประกาศจะถูกโพสต์ไปยังแชทของผู้ร้องขอผ่านการเรียก `agent` แบบติดตามผล (`deliver=true`)
- คำตอบประกาศจะคงการกำหนดเส้นทางเธรด/หัวข้อเมื่อมีให้ (เธรด Slack หัวข้อ Telegram เธรด Matrix)
- ข้อความประกาศจะถูกปรับให้อยู่ในเทมเพลตมาตรฐานที่คงที่:
  - `Status:` ได้มาจากผลลัพธ์การทำงาน (`success`, `error`, `timeout` หรือ `unknown`)
  - `Result:` เนื้อหาสรุปจากขั้นตอนประกาศ (หรือ `(not available)` หากไม่มีข้อมูล)
  - `Notes:` รายละเอียดข้อผิดพลาดและบริบทที่เป็นประโยชน์อื่น ๆ
- ข้อความ announce จะมีสถานะที่ได้จากผลลัพธ์การทำงาน (ไม่ใช่จากเอาต์พุตของโมเดล):

แต่ละ announce จะมีบรรทัดสถิติซึ่งประกอบด้วย:

- ระยะเวลาการทำงาน (เช่น `runtime 5m12s`)
- การใช้โทเคน (อินพุต/เอาต์พุต/รวม)
- ค่าใช้จ่ายโดยประมาณ (เมื่อมีการตั้งค่าราคาของโมเดลผ่าน `models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` และพาธของทรานสคริปต์ (เพื่อให้เอเจนต์หลักสามารถดึงประวัติผ่าน `sessions_history` หรือเปิดดูไฟล์บนดิสก์ได้)

## นโยบายเครื่องมือ (Tool Policy)

โดยค่าเริ่มต้น ซับเอเจนต์จะได้รับ **เครื่องมือทั้งหมด ยกเว้น** ชุดเครื่องมือที่ถูกปฏิเสธซึ่งไม่ปลอดภัยหรือไม่จำเป็นสำหรับงานเบื้องหลัง:

- Explicit `thinking` parameter in the `sessions_spawn` call
- `runTimeoutSeconds`
- `sessions_send`
- The `sessions_spawn` Tool

เมื่อ `maxSpawnDepth >= 2` sub-agent ตัวประสานงานที่ระดับความลึก 1 จะได้รับ `sessions_spawn`, `subagents`, `sessions_list` และ `sessions_history` เพิ่มเติม เพื่อให้สามารถจัดการลูกของตนเองได้

การกำหนดค่า

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

## การทำงานพร้อมกัน

sub-agent ใช้คิวภายในโปรเซสที่แยกเฉพาะ:

- :subagent:
- Global default: `agents.defaults.subagents.model`

## การหยุดทำงาน

- การส่ง `/stop` ในแชทของผู้ร้องขอจะยกเลิกเซสชันของผู้ร้องขอ และหยุดการทำงานของ sub-agent ที่กำลังทำงานอยู่ซึ่งถูกสร้างจากเซสชันนั้น โดยจะหยุดแบบต่อเนื่องไปยังลูกที่ซ้อนกัน
- `) บนเลนคิว `subagent\` ที่กำหนดไว้โดยเฉพาะ

## ข้อจำกัด

- การประกาศของ sub-agent เป็นแบบ **best-effort** - **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- - **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- ```
  เอเจนต์หลักเรียก `sessions_spawn` พร้อมคำอธิบายงาน การเรียกนี้เป็นแบบ **ไม่บล็อก** — เอเจนต์หลักจะได้รับ `{ status: "accepted", runId, childSessionKey }` ทันที
  ```
- บริบทของ sub-agent จะใส่เฉพาะ `AGENTS.md` + `TOOLS.md` (ไม่มี `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` หรือ `BOOTSTRAP.md`)
- ความลึกสูงสุดของการซ้อนคือ 5 (`maxSpawnDepth` ช่วงค่า: 1–5) แนะนำให้ใช้ความลึกระดับ 2 สำหรับกรณีการใช้งานส่วนใหญ่
- `maxChildrenPerAgent` จำกัดจำนวนลูกที่ทำงานพร้อมกันต่อเซสชัน (ค่าเริ่มต้น: 5, ช่วงค่า: 1–20)

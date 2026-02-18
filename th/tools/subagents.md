---
title: "ซับเอเจนต์"
---

# ซับเอเจนต์

ซับเอเจนต์ช่วยให้คุณรันงานเบื้องหลังได้โดยไม่บล็อกการสนทนาหลัก เมื่อคุณสร้างซับเอเจนต์ มันจะรันในเซสชันที่แยกออกมาต่างหาก ทำงานของมัน และประกาศผลลัพธ์กลับมายังแชตเมื่อเสร็จสิ้น

**กรณีการใช้งาน:**

- ค้นคว้าหัวข้อหนึ่งในขณะที่เอเจนต์หลักยังคงตอบคำถามต่อไป
- รันงานยาวหลายงานพร้อมกัน (การสแครปเว็บ การวิเคราะห์โค้ด การประมวลผลไฟล์)
- มอบหมายงานให้เอเจนต์เฉพาะทางในสภาพแวดล้อมแบบหลายเอเจนต์

## เริ่มต้นอย่างรวดเร็ว

วิธีที่ง่ายที่สุดในการใช้ซับเอเจนต์คือการขอเอเจนต์ของคุณตามธรรมชาติ:

> "สร้างซับเอเจนต์เพื่อค้นคว้า release notes ล่าสุดของ Node.js"

เอเจนต์จะเรียกใช้เครื่องมือ `sessions_spawn` เบื้องหลัง เมื่อซับเอเจนต์เสร็จสิ้น มันจะประกาศสิ่งที่ค้นพบกลับเข้ามาในแชตของคุณ

คุณยังสามารถระบุตัวเลือกอย่างชัดเจนได้:

> "สร้างซับเอเจนต์เพื่อวิเคราะห์ล็อกเซิร์ฟเวอร์ของวันนี้ ใช้ gpt-5.2 และตั้งค่า timeout 5 นาที"

## ทำงานอย่างไร

<Steps>
  <Step title="Main agent spawns">
    เอเจนต์หลักเรียก `sessions_spawn` พร้อมคำอธิบายงาน การเรียกนี้เป็นแบบ **ไม่บล็อก** — เอเจนต์หลักจะได้รับ `{ status: "accepted", runId, childSessionKey }` ทันที
  </Step>
  <Step title="Sub-agent runs in the background">เซสชันที่แยกออกมาใหม่จะถูกสร้างขึ้น (`agent:<agentId>:subagent:<uuid>`) บนเลนคิว `subagent` ที่กำหนดไว้โดยเฉพาะ</Step>
  <Step title="Result is announced">
    เมื่อซับเอเจนต์เสร็จสิ้น มันจะประกาศสิ่งที่ค้นพบกลับไปยังแชตของผู้ร้องขอ เอเจนต์หลักจะโพสต์สรุปเป็นภาษาธรรมชาติ
  </Step>
  <Step title="Session is archived">
    เซสชันของซับเอเจนต์จะถูกเก็บถาวรอัตโนมัติหลังจาก 60 นาที (ปรับแต่งได้) มีการเก็บรักษาทรานสคริปต์ไว้
  </Step>
</Steps>

<Tip>
ซับเอเจนต์แต่ละตัวมีบริบทและการใช้โทเคนเป็นของ **ตนเอง** ตั้งค่าโมเดลที่ถูกกว่าให้ซับเอเจนต์เพื่อลดค่าใช้จ่าย — ดู [Setting a Default Model](#setting-a-default-model) ด้านล่าง
</Tip>

## การกำหนดค่า

ซับเอเจนต์ทำงานได้ทันทีโดยไม่ต้องตั้งค่าใด ๆ ค่าเริ่มต้น:

- โมเดล: ใช้การเลือกโมเดลปกติของเอเจนต์เป้าหมาย (เว้นแต่จะตั้งค่า `subagents.model`)
- การคิด: ไม่มีการ override สำหรับซับเอเจนต์ (เว้นแต่จะตั้งค่า `subagents.thinking`)
- จำนวนพร้อมกันสูงสุด: 8
- เก็บถาวรอัตโนมัติ: หลังจาก 60 นาที

### การตั้งค่าโมเดลเริ่มต้น

ใช้โมเดลที่ถูกกว่าสำหรับซับเอเจนต์เพื่อลดค่าใช้จ่ายโทเค็น:

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

### Setting a Default Thinking Level

```json5
{
  agents: {
    defaults: {
      subagents: {
        thinking: "low",
      },
    },
  },
}
```

### Per-Agent Overrides

In a multi-agent setup, you can set sub-agent defaults per agent:

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### การทำงานพร้อมกัน

Control how many sub-agents can run at the same time:

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

Sub-agents use a dedicated queue lane (`subagent`) separate from the main agent queue, so sub-agent runs don't block inbound replies.

### Auto-Archive

Sub-agent sessions are automatically archived after a configurable period:

```json5
{
  agents: {
    defaults: {
      subagents: {
        archiveAfterMinutes: 120, // default: 60
      },
    },
  },
}
```

<Note>
Archive renames the transcript to `*.deleted.<timestamp>` (โฟลเดอร์เดียวกัน) — มีการเก็บบันทึกการถอดเสียงไว้ ไม่ได้ถูกลบ Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
</Note>

## The `sessions_spawn` Tool

This is the tool the agent calls to create sub-agents.

### พารามิเตอร์

| Parameter           | Type                     | Default                               | Description                                                                                       |
| ------------------- | ------------------------ | ------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `task`              | string                   | _(required)_       | What the sub-agent should do                                                                      |
| `label`             | string                   | —                                     | Short label for identification                                                                    |
| `agentId`           | string                   | _(caller's agent)_ | Spawn under a different agent id (must be allowed)                             |
| `โมเดล`             | string                   | _(optional)_       | Override the model for this sub-agent                                                             |
| `thinking`          | string                   | _(optional)_       | Override thinking level (`off`, `low`, `medium`, `high`, etc.) |
| `runTimeoutSeconds` | number                   | `0` (no limit)     | Abort the sub-agent after N seconds                                                               |
| `การล้างค่า`        | `"delete"` \\| `"keep"` | `"keep"`                              | `"delete"` archives immediately after announce                                                    |

### Model Resolution Order

The sub-agent model is resolved in this order (first match wins):

1. Explicit `model` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.model`
3. Global default: `agents.defaults.subagents.model`
4. Target agent’s normal model resolution for that new session

Thinking level is resolved in this order:

1. Explicit `thinking` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.thinking`
3. Global default: `agents.defaults.subagents.thinking`
4. Otherwise no sub-agent-specific thinking override is applied

<Note>1.
ค่าของโมเดลที่ไม่ถูกต้องจะถูกข้ามไปโดยเงียบ — ซับเอเจนต์จะทำงานด้วยค่าเริ่มต้นที่ถูกต้องถัดไป พร้อมคำเตือนในผลลัพธ์ของเครื่องมือ</Note>

### 2. การสร้างเอเจนต์ข้ามกัน (Cross-Agent Spawning)

3. โดยค่าเริ่มต้น ซับเอเจนต์สามารถถูกสร้างได้ภายใต้ agent id ของตนเองเท่านั้น 4. เพื่ออนุญาตให้เอเจนต์สร้างซับเอเจนต์ภายใต้ agent id อื่น:

```json5
5. {
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // หรือ ["*"] เพื่ออนุญาตทั้งหมด
        },
      },
    ],
  },
}
```

<Tip>6.
ใช้เครื่องมือ `agents_list` เพื่อดูว่า agent id ใดบ้างที่อนุญาตสำหรับ `sessions_spawn` ในขณะนี้</Tip>

## 7. การจัดการซับเอเจนต์ (`/subagents`)

8. ใช้คำสั่งสแลช `/subagents` เพื่อตรวจสอบและควบคุมการทำงานของซับเอเจนต์สำหรับเซสชันปัจจุบัน:

| คำสั่ง                                     | คำอธิบาย                                                                                                           |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `/subagents list`                          | 9. แสดงรายการการทำงานของซับเอเจนต์ทั้งหมด (ที่กำลังทำงานและที่เสร็จแล้ว) |
| `/subagents stop <id\\|#\\|all>`         | 10. หยุดซับเอเจนต์ที่กำลังทำงาน                                                             |
| `/subagents log <id\\|#> [limit] [tools]` | 11. ดูบันทึกการสนทนาของซับเอเจนต์                                                           |
| `/subagents info <id\\|#>`                | 12. แสดงเมทาดาทาการรันโดยละเอียด                                                            |
| `/subagents send <id\\|#> <message>`      | 13. ส่งข้อความไปยังซับเอเจนต์ที่กำลังทำงาน                                                  |

14. คุณสามารถอ้างอิงซับเอเจนต์ได้ด้วยดัชนีในรายการ (`1`, `2`), คำนำหน้า run id, คีย์เซสชันแบบเต็ม หรือ `last`

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">15.
    ```
    /subagents list
    ```

    ````
    16. ```
    🧭 Subagents (current session)
    Active: 1 · Done: 2
    1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
    2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
    3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
    ```
    
    ```
    /subagents stop 3
    ```
    
    ```
    ⚙️ Stop requested for deploy staging.
    ```
    ````

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">17.
    ```
    /subagents info 1
    ```

    ````
    18. ```
    ℹ️ Subagent info
    Status: ✅
    Label: research logs
    Task: วิจัยบันทึกข้อผิดพลาดของเซิร์ฟเวอร์ล่าสุดและสรุปผลการค้นพบ
    Run: a1b2c3d4-...
    Session: agent:main:subagent:...
    Runtime: 2m31s
    Cleanup: keep
    Outcome: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">19.
    ```
    /subagents log 1 10
    ```

    ````
    20. แสดง 10 ข้อความล่าสุดจากบันทึกการสนทนาของซับเอเจนต์ เพิ่ม `tools` เพื่อรวมข้อความการเรียกใช้เครื่องมือ:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">21.
    ```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    22. ส่งข้อความเข้าไปยังเซสชันของซับเอเจนต์ที่กำลังทำงาน และรอการตอบกลับสูงสุด 30 วินาที
    ```

  </Accordion>
</AccordionGroup>

## 23. Announce (วิธีที่ผลลัพธ์ถูกส่งกลับ)

24. เมื่อซับเอเจนต์ทำงานเสร็จ จะผ่านขั้นตอน **announce**:

1. 25. คำตอบสุดท้ายของซับเอเจนต์จะถูกบันทึกไว้
2. 26. ข้อความสรุปจะถูกส่งไปยังเซสชันของเอเจนต์หลัก พร้อมผลลัพธ์ สถานะ และสถิติ
3. 27. เอเจนต์หลักจะโพสต์สรุปเป็นภาษาธรรมชาติลงในแชตของคุณ

คำตอบประกาศจะคงการกำหนดเส้นทางเธรด/หัวข้อเมื่อมีให้ (เธรด Slack หัวข้อ Telegram เธรด Matrix)

### 28. สถิติของ Announce

29. แต่ละ announce จะมีบรรทัดสถิติซึ่งประกอบด้วย:

- 30. ระยะเวลาการทำงาน
- การใช้โทเคน (อินพุต/เอาต์พุต/รวม)
- 31. ค่าใช้จ่ายโดยประมาณ (เมื่อมีการตั้งค่าราคาของโมเดลผ่าน `models.providers.*.models[].cost`)
- 32. คีย์เซสชัน, session id และเส้นทางบันทึกการสนทนา

### 33. สถานะของ Announce

34. ข้อความ announce จะมีสถานะที่ได้จากผลลัพธ์การทำงาน (ไม่ใช่จากเอาต์พุตของโมเดล):

- 35. **successful completion** (`ok`) — งานเสร็จสมบูรณ์ตามปกติ
- 36. **error** — งานล้มเหลว (รายละเอียดข้อผิดพลาดอยู่ในหมายเหตุ)
- 37. **timeout** — งานเกิน `runTimeoutSeconds`
- 38. **unknown** — ไม่สามารถระบุสถานะได้

<Tip>
39. หากไม่จำเป็นต้องมีการประกาศให้ผู้ใช้เห็น ขั้นตอนสรุปของเอเจนต์หลักสามารถคืนค่า `NO_REPLY` และจะไม่มีการโพสต์ใด ๆ
40. สิ่งนี้แตกต่างจาก `ANNOUNCE_SKIP` ซึ่งใช้ในกระบวนการ announce ระหว่างเอเจนต์ต่อเอเจนต์ (`sessions_send`)
</Tip>

## 41. นโยบายเครื่องมือ (Tool Policy)

42. โดยค่าเริ่มต้น ซับเอเจนต์จะได้รับ **เครื่องมือทั้งหมด ยกเว้น** ชุดเครื่องมือที่ถูกปฏิเสธซึ่งไม่ปลอดภัยหรือไม่จำเป็นสำหรับงานเบื้องหลัง:

<AccordionGroup>
  <Accordion title="Default denied tools">43.
    | เครื่องมือที่ถูกปฏิเสธ | เหตุผล |
    |-------------|--------|
    | `sessions_list` | การจัดการเซสชัน — เอเจนต์หลักเป็นผู้ควบคุม |
    | `sessions_history` | การจัดการเซสชัน — เอเจนต์หลักเป็นผู้ควบคุม |
    | `sessions_send` | การจัดการเซสชัน — เอเจนต์หลักเป็นผู้ควบคุม |
    | `sessions_spawn` | ไม่อนุญาตการแตกแขนงซ้อน (ซับเอเจนต์ไม่สามารถสร้างซับเอเจนต์ได้) |
    | `gateway` | ผู้ดูแลระบบ — อันตรายเมื่อใช้จากซับเอเจนต์ |
    | `agents_list` | ผู้ดูแลระบบ |
    | `whatsapp_login` | การตั้งค่าแบบโต้ตอบ — ไม่ใช่งาน |
    | `session_status` | สถานะ/การจัดตาราง — เอเจนต์หลักเป็นผู้ประสานงาน |
    | `cron` | สถานะ/การจัดตาราง — เอเจนต์หลักเป็นผู้ประสานงาน |
    | `memory_search` | ควรส่งข้อมูลที่เกี่ยวข้องไปในพรอมป์ต์ตอนสร้างแทน |
    | `memory_get` | ควรส่งข้อมูลที่เกี่ยวข้องไปในพรอมป์ต์ตอนสร้างแทน |</Accordion>
</AccordionGroup>

### 44. การปรับแต่งเครื่องมือของซับเอเจนต์

45. คุณสามารถจำกัดเครื่องมือของซับเอเจนต์เพิ่มเติมได้:

```json5
46. {
  tools: {
    subagents: {
      tools: {
        // deny จะมีผลเหนือ allow เสมอ
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

47. เพื่อจำกัดให้ซับเอเจนต์ใช้ได้ **เฉพาะ** เครื่องมือที่ระบุ:

```json5
48. {
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // หากตั้ง deny ไว้ deny จะยังคงมีผลเหนือ
      },
    },
  },
}
```

<Note>
49. รายการ deny ที่กำหนดเองจะถูก **เพิ่มเข้าไป** ในรายการ deny เริ่มต้น 50. หากตั้งค่า `allow` จะมีเฉพาะเครื่องมือเหล่านั้นที่ใช้งานได้ (โดยรายการ deny เริ่มต้นยังคงถูกนำมาพิจารณาทับซ้อน)
</Note>

## การยืนยันตัวตน

การยืนยันตัวตนของซับเอเจนต์ถูกกำหนดโดย **เอเจนต์ไอดี** ไม่ใช่โดยประเภทเซสชัน:

- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)
- The merge is additive — main profiles are always available as fallbacks

<Note>
Fully isolated auth per sub-agent is not currently supported.
</Note>

## Context and System Prompt

Sub-agents receive a reduced system prompt compared to the main agent:

- **Included:** Tooling, Workspace, Runtime sections, plus `AGENTS.md` and `TOOLS.md`
- **Not included:** `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`

The sub-agent also receives a task-focused system prompt that instructs it to stay focused on the assigned task, complete it, and not act as the main agent.

## Stopping Sub-Agents

| Method                 | Effect                                                                    |
| ---------------------- | ------------------------------------------------------------------------- |
| `/stop` in the chat    | Aborts the main session **and** all active sub-agent runs spawned from it |
| `/subagents stop <id>` | Stops a specific sub-agent without affecting the main session             |
| `runTimeoutSeconds`    | Automatically aborts the sub-agent run after the specified time           |

<Note>
`runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
</Note>

## Full Configuration Example

<Accordion title="Complete sub-agent configuration">
```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4" },
      subagents: {
        model: "minimax/MiniMax-M2.1",
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
      },
      {
        id: "ops",
        name: "Ops Agent",
        subagents: {
          model: "anthropic/claude-sonnet-4",
          allowAgents: ["main"], // ops can spawn sub-agents under "main"
        },
      },
    ],
  },
  tools: {
    subagents: {
      tools: {
        deny: ["browser"], // sub-agents can't use the browser
      },
    },
  },
}
```
</Accordion>

## ข้อจำกัด

<Warning>
- **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- **No nested spawning:** Sub-agents cannot spawn their own sub-agents.
- **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- **Auto-archive is best-effort:** Pending archive timers are lost on gateway restart.
</Warning>

## ดูเพิ่มเติม

- [Session Tools](/concepts/session-tool) — details on `sessions_spawn` and other session tools
- [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — per-agent tool restrictions and sandboxing
- [Configuration](/gateway/configuration) — `agents.defaults.subagents` reference
- [Queue](/concepts/queue) — how the `subagent` lane works

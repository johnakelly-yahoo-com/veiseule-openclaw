---
title: "การกำหนดค่า"
---

# การกำหนดค่า

OpenClaw อ่านคอนฟิกเสริมแบบ <Tooltip tip="JSON5 supports comments and trailing commas">**JSON5**</Tooltip> จาก `~/.openclaw/openclaw.json`

หากไม่มีไฟล์นี้ OpenClaw จะใช้ค่าเริ่มต้นที่ปลอดภัย โดยทั่วไปคุณจะเพิ่มคอนฟิกเมื่อ:

- เชื่อมต่อช่องทางต่าง ๆ และควบคุมว่าใครสามารถส่งข้อความถึงบอทได้
- ตั้งค่าโมเดล เครื่องมือ sandboxing หรือระบบอัตโนมัติ (cron, hooks)
- ปรับแต่งเซสชัน สื่อ เครือข่าย หรือ UI

ดู [full reference](/gateway/configuration-reference) สำหรับรายการฟิลด์ทั้งหมดที่มีให้ใช้งาน

<Tip>
**เพิ่งเริ่มใช้การกำหนดค่า?** เริ่มด้วย `openclaw onboard` สำหรับการตั้งค่าแบบโต้ตอบ หรือดูคู่มือ [Configuration Examples](/gateway/configuration-examples) สำหรับตัวอย่างคอนฟิกแบบคัดลอกไปใช้ได้ทันที
</Tip>

## คอนฟิกขั้นต่ำ

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## แก้ไขคอนฟิก

<Tabs>
  <Tab title="ตัวช่วยแบบโต้ตอบ">
    ```bash
    openclaw onboard       # ตัวช่วยตั้งค่าแบบเต็ม
    openclaw configure     # ตัวช่วยตั้งค่าคอนฟิก
    ```
  </Tab>
  <Tab title="CLI (คำสั่งบรรทัดเดียว)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    เปิด [http://127.0.0.1:18789](http://127.0.0.1:18789) แล้วใช้แท็บ **Config**  
    Control UI จะเรนเดอร์ฟอร์มจากสคีมาคอนฟิก พร้อมตัวแก้ไข **Raw JSON** สำหรับกรณีพิเศษ
  </Tab>
  <Tab title="แก้ไขไฟล์โดยตรง">
    แก้ไข `~/.openclaw/openclaw.json` โดยตรง Gateway จะเฝ้าดูไฟล์และนำการเปลี่ยนแปลงไปใช้โดยอัตโนมัติ (ดู [hot reload](#config-hot-reload))
  </Tab>
</Tabs>

## การตรวจสอบแบบเข้มงวด

<Warning>
OpenClaw ยอมรับเฉพาะคอนฟิกที่ตรงกับสคีมาเท่านั้น คีย์ที่ไม่รู้จัก ชนิดข้อมูลผิดรูปแบบ หรือค่าที่ไม่ถูกต้อง จะทำให้ Gateway **ปฏิเสธการเริ่มต้น** ข้อยกเว้นเดียวในระดับ root คือ `$schema` (string) เพื่อให้ตัวแก้ไขแนบ JSON Schema metadata ได้
</Warning>

เมื่อการตรวจสอบล้มเหลว:

- Gateway จะไม่บูต
- ใช้ได้เฉพาะคำสั่งวินิจฉัย (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- รัน `openclaw doctor` เพื่อดูปัญหาอย่างละเอียด
- รัน `openclaw doctor --fix` (หรือ `--yes`) เพื่อซ่อมแซมอัตโนมัติ

## งานที่พบบ่อย

<AccordionGroup>
  <Accordion title="ตั้งค่าช่องทาง (WhatsApp, Telegram, Discord ฯลฯ)">
    แต่ละช่องทางมีส่วนคอนฟิกของตนเองภายใต้ `channels.<provider>` ดูหน้าช่องทางเฉพาะสำหรับขั้นตอนการตั้งค่า:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    ทุกช่องทางใช้รูปแบบนโยบาย DM เหมือนกัน:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="เลือกและตั้งค่าโมเดล">
    ตั้งค่าโมเดลหลักและ fallback:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` กำหนดแคตตาล็อกโมเดลและทำหน้าที่เป็น allowlist สำหรับ `/model`
    - การอ้างอิงโมเดลใช้รูปแบบ `provider/model` (เช่น `anthropic/claude-opus-4-6`)
    - ดู [Models CLI](/concepts/models) สำหรับการสลับโมเดลในแชต และ [Model Failover](/concepts/model-failover) สำหรับพฤติกรรม fallback
    - สำหรับผู้ให้บริการแบบกำหนดเอง/self-hosted ดู [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls)

  </Accordion>

  <Accordion title="ควบคุมว่าใครสามารถส่งข้อความถึงบอทได้">
    การเข้าถึง DM ควบคุมต่อช่องทางผ่าน `dmPolicy`:

    - `"pairing"` (ค่าเริ่มต้น): ผู้ส่งที่ไม่รู้จักจะได้รับรหัสจับคู่แบบครั้งเดียว
    - `"allowlist"`: อนุญาตเฉพาะผู้ส่งใน `allowFrom`
    - `"open"`: อนุญาต DM ขาเข้าทั้งหมด (ต้องตั้ง `allowFrom: ["*"]`)
    - `"disabled"`: เพิกเฉย DM ทั้งหมด

    สำหรับกลุ่ม ใช้ `groupPolicy` + `groupAllowFrom` หรือ allowlist เฉพาะช่องทาง

    ดู [full reference](/gateway/configuration-reference#dm-and-group-access)

  </Accordion>

  <Accordion title="ตั้งค่า mention gating ในกลุ่ม">
    ข้อความกลุ่มตั้งค่าเริ่มต้นเป็น **ต้องมีการกล่าวถึง** กำหนดแพทเทิร์นต่อเอเจนต์:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Metadata mentions**: @-mentions แบบเนทีฟของแพลตฟอร์ม
    - **Text patterns**: regex ใน `mentionPatterns`
    - ดู [full reference](/gateway/configuration-reference#group-chat-mention-gating)

  </Accordion>

  <Accordion title="ตั้งค่าเซสชันและการรีเซ็ต">
    เซสชันควบคุมความต่อเนื่องของบทสนทนา:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - ดู [Session Management](/concepts/session) และ [full reference](/gateway/configuration-reference#session)

  </Accordion>

  <Accordion title="เปิดใช้งาน sandboxing">
    รันเซสชันใน Docker แบบแยกส่วน:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    สร้างอิมเมจครั้งแรกด้วย: `scripts/sandbox-setup.sh`

    ดู [Sandboxing](/gateway/sandboxing)

  </Accordion>

  <Accordion title="ตั้งค่า heartbeat (ตรวจสอบเป็นระยะ)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: เช่น `30m`, `2h` ตั้ง `0m` เพื่อปิด
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - ดู [Heartbeat](/gateway/heartbeat)

  </Accordion>

  <Accordion title="ตั้งค่า cron jobs">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    ดู [Cron jobs](/automation/cron-jobs)

  </Accordion>

  <Accordion title="ตั้งค่า webhooks (hooks)">
    เปิด endpoint webhook บน Gateway:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    ดู [full reference](/gateway/configuration-reference#hooks)

  </Accordion>

  <Accordion title="กำหนดเส้นทางหลายเอเจนต์">
    รันหลายเอเจนต์แบบแยก workspace/เซสชัน:

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    ดู [Multi-Agent](/concepts/multi-agent)

  </Accordion>

  <Accordion title="แยกคอนฟิกหลายไฟล์ ($include)">
    ใช้ `$include` เพื่อจัดระเบียบคอนฟิกขนาดใหญ่:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **ไฟล์เดียว**: แทนที่อ็อบเจ็กต์นั้น
    - **หลายไฟล์**: deep-merge ตามลำดับ
    - **คีย์พี่น้อง**: merge หลัง include
    - **Nested includes**: ลึกได้ 10 ระดับ
    - **เส้นทางสัมพัทธ์**: อ้างอิงจากไฟล์ที่ include
    - **การจัดการข้อผิดพลาด**: แจ้งชัดเจนเมื่อไฟล์หาย/พาร์สผิด/วนซ้ำ

  </Accordion>
</AccordionGroup>

## Config hot reload

Gateway จะเฝ้าดู `~/.openclaw/openclaw.json` และนำการเปลี่ยนแปลงไปใช้โดยอัตโนมัติ — ส่วนใหญ่ไม่ต้องรีสตาร์ตเอง

### โหมดการรีโหลด

| โหมด | พฤติกรรม |
|------|-----------|
| **`hybrid`** (ค่าเริ่มต้น) | ใช้การเปลี่ยนแปลงที่ปลอดภัยทันที และรีสตาร์ตอัตโนมัติเมื่อจำเป็น |
| **`hot`** | ใช้เฉพาะที่ปลอดภัย และแจ้งเตือนเมื่อจำเป็นต้องรีสตาร์ต |
| **`restart`** | รีสตาร์ต Gateway ทุกครั้งที่คอนฟิกเปลี่ยน |
| **`off`** | ปิดการเฝ้าดูไฟล์ ต้องรีสตาร์ตเอง |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

## ตัวแปรสภาพแวดล้อม

OpenClaw อ่าน env vars จากโปรเซสหลัก รวมถึง:

- `.env` ในไดเรกทอรีปัจจุบัน
- `~/.openclaw/.env` (fallback ส่วนกลาง)

ไฟล์เหล่านี้จะไม่เขียนทับ env ที่มีอยู่แล้ว คุณยังสามารถกำหนด inline env vars ในคอนฟิกได้:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (ไม่บังคับ)">
  หากเปิดใช้งาน และยังไม่มีคีย์ที่คาดหวัง OpenClaw จะรัน login shell ของคุณและนำเข้าเฉพาะคีย์ที่ขาด:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var เทียบเท่า: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="การแทนที่ env var ในคอนฟิก">
  อ้างอิงตัวแปรสภาพแวดล้อมในค่าสตริงด้วย `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

กฎ:

- จับคู่เฉพาะตัวพิมพ์ใหญ่: `[A-Z_][A-Z0-9_]*`
- ตัวแปรที่หายหรือว่างจะทำให้โหลดล้มเหลว
- ใช้ `$${VAR}` เพื่อ escape
- ใช้ได้ในไฟล์ `$include`
- รองรับ inline substitution เช่น `"${BASE}/v1"`

</Accordion>

ดู [Environment](/help/environment) สำหรับรายละเอียดทั้งหมด

## เอกสารอ้างอิงทั้งหมด

ดูรายละเอียดแบบฟิลด์ต่อฟิลด์ที่ **[Configuration Reference](/gateway/configuration-reference)**

---

_ที่เกี่ยวข้อง: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_

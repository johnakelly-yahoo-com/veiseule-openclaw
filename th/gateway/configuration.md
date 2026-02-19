---
summary: "ภาพรวมการตั้งค่า: งานที่พบบ่อย, การตั้งค่าแบบรวดเร็ว และลิงก์ไปยังเอกสารอ้างอิงฉบับเต็ม"
read_when:
  - การตั้งค่า OpenClaw ครั้งแรก
  - กำลังมองหารูปแบบการตั้งค่าที่ใช้บ่อย
  - ไปยังส่วนคอนฟิกที่ต้องการ
title: "การกำหนดค่า"
---

# การกำหนดค่า 🔧

OpenClaw อ่านไฟล์คอนฟิก **JSON5** แบบไม่บังคับจาก `~/.openclaw/openclaw.json` (รองรับคอมเมนต์และคอมมาท้ายบรรทัด)

หากไม่พบไฟล์ OpenClaw จะใช้ค่าเริ่มต้นที่ปลอดภัย เหตุผลทั่วไปในการเพิ่มคอนฟิก:

- เชื่อมต่อช่องทางและควบคุมว่าใครสามารถส่งข้อความถึงบอทได้
- ตั้งค่าโมเดล เครื่องมือ sandboxing หรือระบบอัตโนมัติ (cron, hooks)
- ปรับแต่งเซสชัน สื่อ เครือข่าย หรือ UI

ดู [เอกสารอ้างอิงฉบับเต็ม](/gateway/configuration-reference) สำหรับทุกฟิลด์ที่มีให้ใช้งาน

<Tip>
คำเตือน: `config.apply` จะเขียนทับ **คอนฟิกทั้งหมด** หากต้องการเปลี่ยนเพียงไม่กี่คีย์
  ให้ใช้ `config.patch` หรือ `openclaw config set` และควรสำรอง `~/.openclaw/openclaw.json` ไว้
</Tip>

## คอนฟิกแบบขั้นต่ำ

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## แก้ไขคอนฟิก

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # วิซาร์ดตั้งค่าแบบเต็ม
openclaw configure     # วิซาร์ดตั้งค่าคอนฟิก
```
</Tab>
  <Tab title="CLI (one-liners)">```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```
</Tab>
  <Tab title="Control UI">
    The Gateway exposes a JSON Schema representation of the config via `config.schema` for UI editors.
    Gateway เปิดเผยสคีมา JSON ของคอนฟิกผ่าน `config.schema` สำหรับตัวแก้ไข UI
Control UI จะเรนเดอร์ฟอร์มจากสคีมานี้ พร้อมตัวแก้ไข **Raw JSON** เป็นทางหนีทีไล่
  
</Tab>
  <Tab title="Direct edit">
    Keep a backup of `~/.openclaw/openclaw.json`. Gateway จะเฝ้าดูไฟล์และนำการเปลี่ยนแปลงไปใช้โดยอัตโนมัติ (ดู [hot reload](#config-hot-reload))
  
</Tab>
</Tabs>

## การตรวจสอบความถูกต้องแบบเข้มงวด

<Warning>
OpenClaw only accepts configurations that fully match the schema. OpenClaw ยอมรับเฉพาะคอนฟิกที่ตรงกับสคีมาทั้งหมดเท่านั้น
คีย์ที่ไม่รู้จัก ชนิดข้อมูลที่ผิดรูปแบบ หรือค่าที่ไม่ถูกต้อง จะทำให้ Gateway **ปฏิเสธการเริ่มต้น** เพื่อความปลอดภัย ข้อยกเว้นระดับ root เพียงรายการเดียวคือ `$schema` (string) เพื่อให้ตัวแก้ไขสามารถแนบข้อมูลเมตา JSON Schema ได้
</Warning>

เมื่อการตรวจสอบล้มเหลว:

- Gateway จะไม่บูต
- อนุญาตเฉพาะคำสั่งวินิจฉัยเท่านั้น (เช่น: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`)
- รัน `openclaw doctor` เพื่อดูปัญหาที่แน่ชัด
- รัน `openclaw doctor --fix` (หรือ `--yes`) เพื่อใช้การย้าย/ซ่อมแซม

## งานที่พบบ่อย

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    แต่ละ channel จะมีส่วนการตั้งค่าของตัวเองภายใต้ `channels.<provider>` ดูหน้าของ channel ที่เกี่ยวข้องสำหรับขั้นตอนการตั้งค่า:

    ```
    {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org.com"],
        },
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            GUILD_ID: {
              channels: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          channels: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">
    ตั้งค่าโมเดลหลักและตัวสำรอง (fallback) เพิ่มเติม:

- `"pairing"` (ค่าเริ่มต้น): ผู้ส่งที่ไม่รู้จักจะได้รับรหัสจับคู่แบบใช้ครั้งเดียวเพื่ออนุมัติ
- `"allowlist"`: อนุญาตเฉพาะผู้ส่งใน `allowFrom` (หรือใน paired allow store)
- `"open"`: อนุญาต DM ขาเข้าทั้งหมด (ต้องตั้งค่า `allowFrom: ["*"]`)
- `"disabled"`: เพิกเฉยต่อ DM ทั้งหมด

สำหรับกลุ่ม ให้ใช้ `groupPolicy` + `groupAllowFrom` หรือ allowlist เฉพาะของแต่ละ channel

ดู [ข้อมูลอ้างอิงฉบับเต็ม](/gateway/configuration-reference#dm-and-group-access) สำหรับรายละเอียดแยกตาม channel

    ```
    {
      agents: {
        defaults: {
          cliBackends: {
            "claude-cli": {
              command: "/opt/homebrew/bin/claude",
            },
            "my-cli": {
              command: "my-cli",
              args: ["--json"],
              output: "json",
              modelArg: "--model",
              sessionArg: "--session",
              sessionMode: "existing",
              systemPromptArg: "--system",
              systemPromptWhen: "first",
              imageArg: "--image",
              imageMode: "repeat",
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">DMs สาธารณะ: `channels.mattermost.dmPolicy="open"` พร้อมกับ `channels.mattermost.allowFrom=["*"]`

    ```
    ข้อความในกลุ่มค่าเริ่มต้นจะเป็น **ต้องมีการ mention**
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    ```json5
{
  session: {
    dmScope: "per-channel-peer",  // แนะนำสำหรับหลายผู้ใช้
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

- `dmScope`: `main` (ใช้ร่วมกัน) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
- ดู [Session Management](/concepts/session) สำหรับขอบเขต (scoping), การเชื่อมโยงตัวตน และนโยบายการส่งข้อความ
- ดู [ข้อมูลอ้างอิงฉบับเต็ม](/gateway/configuration-reference#session) สำหรับทุกฟิลด์ `agents.defaults.subagents` configures sub-agent defaults:

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">การแยกเซสชันเธรด:

    ```
    
        รันเซสชันของ agent ในคอนเทนเนอร์ Docker ที่แยกออกจากกัน:
    ```

  
</Accordion>

  <Accordion title="Enable sandboxing">
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

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">- `every`: สตริงระยะเวลา (`30m`, `2h`) ตั้งค่า `0m` เพื่อปิดใช้งาน
- `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
- ดู [Heartbeat](/gateway/heartbeat) สำหรับคำแนะนำฉบับเต็ม

    ```
    
        รันหลาย agent แบบแยกอิสระด้วย workspace และ session แยกกัน:
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">19. {
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    ดู [Cron jobs](/automation/cron-jobs) สำหรับภาพรวมฟีเจอร์และตัวอย่าง CLI
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">เปิดใช้งาน endpoint webhook HTTP แบบง่ายบนเซิร์ฟเวอร์ HTTP ของ Gateway

    ```
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        presets: ["gmail"],
        transformsDir: "~/.openclaw/hooks",
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            wakeMode: "now",
            name: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
            deliver: true,
            channel: "last",
            model: "openai/gpt-5.2-mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">  
</Accordion>

    ```
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
      channels: {
        whatsapp: {
          accounts: {
            personal: {},
            biz: {},
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">แยกคอนฟิกออกเป็นหลายไฟล์ด้วยคำสั่ง `$include` มีประโยชน์สำหรับ: This is useful for:

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (การรีโหลดคอนฟิกแบบ hot)

โหมดการรีโหลด

### **`hybrid`** (ค่าเริ่มต้น)

| Modes:                                                 | พฤติกรรมการผสาน                                                                                       |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| ปรับใช้การเปลี่ยนแปลงที่ปลอดภัยแบบทันที (hot-apply) | รีสตาร์ตโดยอัตโนมัติสำหรับการเปลี่ยนแปลงที่สำคัญ **`hot`**                                            |
| ปรับใช้เฉพาะการเปลี่ยนแปลงที่ปลอดภัยแบบทันที                           | บันทึกคำเตือนเมื่อจำเป็นต้องรีสตาร์ต — คุณต้องจัดการเอง **`restart`**                                 |
| **`off`**                                                              | `restart`: รีสตาร์ต Gateway เมื่อมีการเปลี่ยนแปลงคอนฟิกใด ๆ                           |
| ปิดการเฝ้าดูไฟล์                                                       | การเปลี่ยนแปลงจะมีผลเมื่อรีสตาร์ตด้วยตนเองครั้งถัดไป สิ่งที่ปรับใช้แบบทันทีได้ vs สิ่งที่ต้องรีสตาร์ต |

```json5
44. {
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### ฟิลด์ส่วนใหญ่ปรับใช้แบบทันทีได้โดยไม่ต้องหยุดระบบ

ในโหมด `hybrid` การเปลี่ยนแปลงที่ต้องรีสตาร์ตจะถูกจัดการโดยอัตโนมัติ หมวดหมู่

| ต้องรีสตาร์ตหรือไม่?                   | ฟิลด์:                                             | `channels.*`, `web` (WhatsApp) — ทุก channel ทั้งแบบในตัวและส่วนขยาย |
| -------------------------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| \`channels.&lt;provider&gt;  | ไม่                                                                | Agent และโมเดล                                                                          |
| `agent`, `agents`, `models`, `routing` | ไม่                                                                | ระบบอัตโนมัติ                                                                           |
| `hooks`, `cron`, `agent.heartbeat`     | ไม่                                                                | ไม่                                                                                     |
| messages                               | `messages`                                                         | เครื่องมือและสื่อ                                                                       |
| Tools & media      | `tools`, `browser`, `skills`, `audio`, `talk`                      | ไม่ใช่                                                                                  |
| UI และอื่นๆ                            | `ui`, `logging`, `identity`, `bindings`                            | ไม่ใช่                                                                                  |
| เซิร์ฟเวอร์ Gateway                    | `gateway` (พอร์ต/bind/auth/UI ควบคุม/tailscale) | **การแทนค่าแบบอินไลน์:**                                                |
| โครงสร้างพื้นฐาน                       | `discovery`, `canvasHost`, `plugins`                               | **ประเภทการกล่าวถึง:**                                                  |

<Note>
`gateway.reload` และ `gateway.remote` เป็นข้อยกเว้น — การเปลี่ยนค่า **จะไม่** ทำให้เกิดการรีสตาร์ท
</Note>

## อัปเดตบางส่วน (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">ใช้ `config.apply` เพื่อตรวจสอบ + เขียนคอนฟิกทั้งชุดและรีสตาร์ต Gateway ในขั้นตอนเดียว
ระบบจะเขียนสัญญาณรีสตาร์ตและส่งพิงไปยังเซสชันที่ใช้งานล่าสุดหลังจาก Gateway กลับมา
It writes a restart sentinel and pings the last active session after the Gateway comes back.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">ใช้ `config.patch` เพื่อผสานอัปเดตบางส่วนเข้ากับคอนฟิกเดิมโดยไม่เขียนทับคีย์ที่ไม่เกี่ยวข้อง
ใช้ความหมาย JSON merge patch: It applies JSON merge patch semantics:

    ````
    - อ็อบเจ็กต์จะถูกรวมแบบเรียกซ้ำ (recursive)
    - `null` ใช้ลบคีย์
    - อาร์เรย์จะถูกแทนที่ทั้งหมด
    
    พารามิเตอร์:
    
    - `raw` (string) — JSON5 ที่มีเฉพาะคีย์ที่ต้องการเปลี่ยน
    - `baseHash` (จำเป็น) — ค่าแฮชของคอนฟิกจาก `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — เหมือนกับใน `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    
    ````

  
</Accordion>
</AccordionGroup>

## ตัวแปร

OpenClaw อ่านตัวแปรสภาพแวดล้อม (env vars) จากโปรเซสหลัก รวมถึง:

- `.env` จากไดเรกทอรีทำงานปัจจุบัน (ถ้ามี)
- ค่า fallback ส่วนกลาง `.env` จาก `~/.openclaw/.env` (หรือ `$OPENCLAW_STATE_DIR/.env`)

ไฟล์ `.env` ใดๆ จะไม่เขียนทับตัวแปรสภาพแวดล้อมที่มีอยู่ You can also provide inline env vars in config.

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

<Accordion title="Shell env import (optional)">ความสะดวกแบบเลือกใช้: หากเปิดและยังไม่มีการตั้งค่าคีย์ที่คาดหวังใดๆ
OpenClaw จะรันเชลล์ล็อกอินของคุณและนำเข้าเฉพาะคีย์ที่ขาด (ไม่เขียนทับ)
ซึ่งเทียบเท่ากับการ source โปรไฟล์เชลล์ของคุณ
This effectively sources your shell profile.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">
  อ้างอิงตัวแปรสภาพแวดล้อมในค่าสตริงของคอนฟิกใดๆ ด้วย `${VAR_NAME}`:


```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**กฎ:**

- จับคู่เฉพาะชื่อตัวแปรตัวพิมพ์ใหญ่: `[A-Z_][A-Z0-9_]*`
- Missing or empty env vars throw an error at config load
- เอสเคปด้วย `$${VAR}` เพื่อพิมพ์ `${VAR}` แบบลิเทอรัล
- ใช้ได้กับ `$include` (ไฟล์ที่ถูกรวมก็ถูกแทนค่าด้วย)
- การแทนที่แบบอินไลน์: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

ดู [/environment](/help/environment) สำหรับลำดับความสำคัญและแหล่งที่มาอย่างครบถ้วน

## การอ้างอิงแบบเต็ม

**เพิ่งเริ่มใช้การกำหนดค่า?** ดูคู่มือ [Configuration Examples](/gateway/configuration-examples) สำหรับตัวอย่างครบถ้วนพร้อมคำอธิบายละเอียด!

---

Example (provider/model-specific allowlist):

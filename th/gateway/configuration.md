---
title: "การกำหนดค่า"
---

# การกำหนดค่า 🔧

OpenClaw อ่านไฟล์คอนฟิก **JSON5** แบบไม่บังคับจาก `~/.openclaw/openclaw.json` (รองรับคอมเมนต์และคอมมาท้ายบรรทัด)

หากไม่มีไฟล์นี้ OpenClaw จะใช้ค่าเริ่มต้นที่ค่อนข้างปลอดภัย (เอเจนต์ Pi แบบฝังตัว + เซสชันต่อผู้ส่ง + เวิร์กสเปซ `~/.openclaw/workspace`) โดยทั่วไปคุณจะต้องใช้คอนฟิกเมื่อ: You usually only need a config to:

- จำกัดว่าใครสามารถเรียกใช้บอตได้ (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` ฯลฯ)
- ควบคุม allowlist ของกลุ่มและพฤติกรรมการกล่าวถึง (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- ปรับแต่งพรีฟิกซ์ข้อความ (`messages`)
- ตั้งค่าเวิร์กสเปซของเอเจนต์ (`agents.defaults.workspace` หรือ `agents.list[].workspace`)
- ปรับจูนค่าเริ่มต้นของเอเจนต์แบบฝัง (`agents.defaults`) และพฤติกรรมเซสชัน (`session`)
- ตั้งค่าอัตลักษณ์ต่อเอเจนต์ (`agents.list[].identity`)

> **เพิ่งเริ่มใช้การกำหนดค่า?** ดูคู่มือ [Configuration Examples](/gateway/configuration-examples) สำหรับตัวอย่างครบถ้วนพร้อมคำอธิบายละเอียด!

## การตรวจสอบคอนฟิกแบบเข้มงวด

OpenClaw only accepts configurations that fully match the schema.
OpenClaw ยอมรับเฉพาะคอนฟิกที่ตรงกับสคีมาทั้งหมดเท่านั้น
คีย์ที่ไม่รู้จัก ชนิดข้อมูลที่ผิดรูปแบบ หรือค่าที่ไม่ถูกต้อง จะทำให้ Gateway **ปฏิเสธการเริ่มต้น** เพื่อความปลอดภัย

เมื่อการตรวจสอบล้มเหลว:

- Gateway จะไม่บูต
- อนุญาตเฉพาะคำสั่งวินิจฉัยเท่านั้น (เช่น: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`)
- รัน `openclaw doctor` เพื่อดูปัญหาที่แน่ชัด
- รัน `openclaw doctor --fix` (หรือ `--yes`) เพื่อใช้การย้าย/ซ่อมแซม

Doctor จะไม่เขียนการเปลี่ยนแปลงใดๆ เว้นแต่คุณจะเลือกเข้าร่วม `--fix`/`--yes` อย่างชัดเจน

## สคีมา + คำใบ้ UI

The Gateway exposes a JSON Schema representation of the config via `config.schema` for UI editors.
Gateway เปิดเผยสคีมา JSON ของคอนฟิกผ่าน `config.schema` สำหรับตัวแก้ไข UI
Control UI จะเรนเดอร์ฟอร์มจากสคีมานี้ พร้อมตัวแก้ไข **Raw JSON** เป็นทางหนีทีไล่

ปลั๊กอินช่องทางและส่วนขยายสามารถลงทะเบียนสคีมาและคำใบ้ UI สำหรับคอนฟิกของตนได้
เพื่อให้การตั้งค่าช่องทางยังคงขับเคลื่อนด้วยสคีมาข้ามแอป โดยไม่ต้องฮาร์ดโค้ดฟอร์ม

คำใบ้ (ป้ายกำกับ การจัดกลุ่ม ฟิลด์อ่อนไหว) มาพร้อมสคีมา
เพื่อให้ไคลเอนต์เรนเดอร์ฟอร์มได้ดีขึ้นโดยไม่ต้องฮาร์ดโค้ดความรู้คอนฟิก

## ใช้ + รีสตาร์ต (RPC)

ใช้ `config.apply` เพื่อตรวจสอบ + เขียนคอนฟิกทั้งชุดและรีสตาร์ต Gateway ในขั้นตอนเดียว
ระบบจะเขียนสัญญาณรีสตาร์ตและส่งพิงไปยังเซสชันที่ใช้งานล่าสุดหลังจาก Gateway กลับมา
It writes a restart sentinel and pings the last active session after the Gateway comes back.

Warning: `config.apply` replaces the **entire config**. If you want to change only a few keys,
use `config.patch` or `openclaw config set`. Keep a backup of `~/.openclaw/openclaw.json`.

Params:

- `raw` (string) — เพย์โหลด JSON5 สำหรับคอนฟิกทั้งหมด
- `baseHash` (ไม่บังคับ) — แฮชคอนฟิกจาก `config.get` (จำเป็นเมื่อมีคอนฟิกอยู่แล้ว)
- `sessionKey` (ไม่บังคับ) — คีย์เซสชันที่ใช้งานล่าสุดสำหรับการพิงปลุก
- `note` (ไม่บังคับ) — โน้ตที่จะรวมในสัญญาณรีสตาร์ต
- `restartDelayMs` (ไม่บังคับ) — หน่วงเวลาก่อนรีสตาร์ต (ค่าเริ่มต้น 2000)

ตัวอย่าง (ผ่าน `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## อัปเดตบางส่วน (RPC)

ใช้ `config.patch` เพื่อผสานอัปเดตบางส่วนเข้ากับคอนฟิกเดิมโดยไม่เขียนทับคีย์ที่ไม่เกี่ยวข้อง
ใช้ความหมาย JSON merge patch: It applies JSON merge patch semantics:

- อ็อบเจ็กต์ผสานแบบรีเคอร์ซีฟ
- `null` ใช้ลบคีย์
- อาเรย์ถูกแทนที่
  เช่นเดียวกับ `config.apply` ระบบจะตรวจสอบ เขียนคอนฟิก เก็บสัญญาณรีสตาร์ต และตั้งเวลาการรีสตาร์ต Gateway (พร้อมการปลุกเสริมเมื่อระบุ `sessionKey`)

Params:

- `raw` (string) — เพย์โหลด JSON5 ที่มีเฉพาะคีย์ที่ต้องการเปลี่ยน
- `baseHash` (จำเป็น) — แฮชคอนฟิกจาก `config.get`
- `sessionKey` (ไม่บังคับ) — คีย์เซสชันที่ใช้งานล่าสุดสำหรับการพิงปลุก
- `note` (ไม่บังคับ) — โน้ตที่จะรวมในสัญญาณรีสตาร์ต
- `restartDelayMs` (ไม่บังคับ) — หน่วงเวลาก่อนรีสตาร์ต (ค่าเริ่มต้น 2000)

ตัวอย่าง:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## คอนฟิกขั้นต่ำ (แนะนำเป็นจุดเริ่มต้น)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

สร้างอิมเมจค่าเริ่มต้นหนึ่งครั้งด้วย:

```bash
scripts/sandbox-setup.sh
```

## โหมดคุยกับตัวเอง (แนะนำสำหรับการควบคุมกลุ่ม)

เพื่อป้องกันไม่ให้บอตตอบ @-mentions ของ WhatsApp ในกลุ่ม (ตอบเฉพาะทริกเกอร์ข้อความที่กำหนด):

```json5
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

## การรวมคอนฟิก (`$include`)

แยกคอนฟิกออกเป็นหลายไฟล์ด้วยคำสั่ง `$include` มีประโยชน์สำหรับ: This is useful for:

- จัดระเบียบคอนฟิกขนาดใหญ่ (เช่น นิยามเอเจนต์ต่อไคลเอนต์)
- แชร์การตั้งค่าร่วมกันข้ามสภาพแวดล้อม
- แยกคอนฟิกที่อ่อนไหวออกต่างหาก

### การใช้งานพื้นฐาน

```json5
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

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### พฤติกรรมการผสาน

- **ไฟล์เดียว**: แทนที่อ็อบเจ็กต์ที่มี `$include`
- **อาเรย์ของไฟล์**: ผสานเชิงลึกตามลำดับ (ไฟล์หลังแทนที่ไฟล์ก่อน)
- **มีคีย์พี่น้อง**: คีย์พี่น้องจะผสานหลัง includes (แทนค่าที่รวมมา)
- **คีย์พี่น้อง + อาเรย์/พริมิทีฟ**: ไม่รองรับ (เนื้อหาที่รวมต้องเป็นอ็อบเจ็กต์)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### Nested includes

ไฟล์ที่ถูกรวมสามารถมีคำสั่ง `$include` ได้เอง (ลึกได้สูงสุด 10 ระดับ):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### การแก้เส้นทาง

- **เส้นทางสัมพัทธ์**: แก้จากไฟล์ที่อ้างถึง
- **เส้นทางสัมบูรณ์**: ใช้ตามที่ระบุ
- **ไดเรกทอรีแม่**: อ้างอิง `../` ทำงานตามคาด

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### การจัดการข้อผิดพลาด

- **ไฟล์หาย**: แสดงข้อผิดพลาดชัดเจนพร้อมเส้นทางที่แก้แล้ว
- **พาร์สผิดพลาด**: แสดงไฟล์ที่รวมแล้วล้มเหลว
- **รวมแบบวนซ้ำ**: ตรวจพบและรายงานพร้อมลำดับการรวม

### ตัวอย่าง: การตั้งค่ากฎหมายหลายไคลเอนต์

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## ตัวเลือกที่ใช้บ่อย

### Env vars + `.env`

OpenClaw อ่านตัวแปรสภาพแวดล้อมจากโปรเซสแม่ (เชลล์ launchd/systemd CI ฯลฯ)

นอกจากนี้ยังโหลด:

- `.env` จากไดเรกทอรีทำงานปัจจุบัน (ถ้ามี)
- ค่า fallback ส่วนกลาง `.env` จาก `~/.openclaw/.env` (หรือ `$OPENCLAW_STATE_DIR/.env`)

ไฟล์ `.env` ใดๆ จะไม่เขียนทับตัวแปรสภาพแวดล้อมที่มีอยู่

You can also provide inline env vars in config. These are only applied if the
process env is missing the key (same non-overriding rule):

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

ดู [/environment](/help/environment) สำหรับลำดับความสำคัญและแหล่งที่มาอย่างครบถ้วน

### `env.shellEnv` (ไม่บังคับ)

ความสะดวกแบบเลือกใช้: หากเปิดและยังไม่มีการตั้งค่าคีย์ที่คาดหวังใดๆ
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

Env var equivalent:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### การแทนที่env varในคอนฟิก

คุณสามารถอ้างอิงตัวแปรสภาพแวดล้อมได้โดยตรงในค่าสตริงของคอนฟิกด้วยไวยากรณ์
`${VAR_NAME}` การแทนค่าจะเกิดตอนโหลดคอนฟิก ก่อนการตรวจสอบ Variables are substituted at config load time, before validation.

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

**การแทนค่าแบบอินไลน์:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### ที่เก็บการยืนยันตัวตน (OAuth + คีย์API)

OpenClaw เก็บโปรไฟล์การยืนยันตัวตน **ต่อเอเจนต์** (OAuth + คีย์API) ไว้ที่:

- `<agentDir>/auth-profiles.json` (ค่าเริ่มต้น: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

ดูเพิ่มเติม: [/concepts/oauth](/concepts/oauth)

การนำเข้า OAuth แบบเดิม:

- `~/.openclaw/credentials/oauth.json` (หรือ `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

เอเจนต์ Pi แบบฝังจะดูแลแคชรันไทม์ที่:

- `<agentDir>/auth.json` (จัดการอัตโนมัติ; อย่าแก้ไขด้วยตนเอง)

ไดเรกทอรีเอเจนต์แบบเดิม (ก่อนหลายเอเจนต์):

- `~/.openclaw/agent/*` (ถูกย้ายโดย `openclaw doctor` ไปยัง `~/.openclaw/agents/<defaultAgentId>/agent/*`)

Overrides:

- ไดเรกทอรี OAuth (นำเข้าแบบเดิมเท่านั้น): `OPENCLAW_OAUTH_DIR`
- ไดเรกทอรีเอเจนต์ (แทนที่รากเอเจนต์เริ่มต้น): `OPENCLAW_AGENT_DIR` (แนะนำ), `PI_CODING_AGENT_DIR` (เดิม)

เมื่อใช้งานครั้งแรก OpenClaw จะนำเข้ารายการ `oauth.json` ไปยัง `auth-profiles.json`

### `auth`

Optional metadata for auth profiles. เมทาดาทาทางเลือกสำหรับโปรไฟล์การยืนยันตัวตน ไม่ได้เก็บความลับ
ใช้แมปโปรไฟล์กับผู้ให้บริการ + โหมด (และอีเมลถ้ามี) และกำหนดลำดับการสลับผู้ให้บริการสำหรับ failover

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

### `agents.list[].identity`

อัตลักษณ์ต่อเอเจนต์แบบทางเลือก ใช้สำหรับค่าเริ่มต้นและ UX
ถูกเขียนโดยผู้ช่วย onboarding บน macOS This is written by the macOS onboarding assistant.

หากตั้งค่า OpenClaw จะอนุมานค่าเริ่มต้น (เฉพาะเมื่อคุณยังไม่ได้ตั้งเอง):

- `messages.ackReaction` จาก `identity.emoji` ของ **เอเจนต์ที่ใช้งานอยู่** (ถอยกลับเป็น 👀)
- `agents.list[].groupChat.mentionPatterns` จาก `identity.name`/`identity.emoji` ของเอเจนต์ (เพื่อให้ “@Samantha” ใช้ได้ในกลุ่มข้าม Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
- `identity.avatar` รับพาธรูปภาพสัมพัทธ์เวิร์กสเปซหรือ URL ระยะไกล/data URL ไฟล์โลคัลต้องอยู่ภายในเวิร์กสเปซของเอเจนต์ Local files must live inside the agent workspace.

`identity.avatar` รับ:

- พาธสัมพัทธ์เวิร์กสเปซ (ต้องอยู่ภายในเวิร์กสเปซของเอเจนต์)
- URL `http(s)`
- URI `data:`

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

### `wizard`

เมทาดาทาที่เขียนโดยวิซาร์ด CLI (`onboard`, `configure`, `doctor`)

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

### `logging`

- ไฟล์ล็อกเริ่มต้น: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- หากต้องการพาธคงที่ ให้ตั้ง `logging.file` เป็น `/tmp/openclaw/openclaw.log`
- เอาต์พุตคอนโซลปรับแยกได้ผ่าน:
  - `logging.consoleLevel` (ค่าเริ่มต้น `info` เพิ่มเป็น `debug` เมื่อ `--verbose`)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- สรุปเครื่องมือสามารถปกปิดเพื่อหลีกเลี่ยงการรั่วไหลของความลับ:
  - `logging.redactSensitive` (`off` | `tools`, ค่าเริ่มต้น: `tools`)
  - `logging.redactPatterns` (อาเรย์ regex; เขียนทับค่าเริ่มต้น)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

ควบคุมการจัดการแชตตรง (DMs) ของ WhatsApp:

- `"pairing"` (ค่าเริ่มต้น): ผู้ส่งที่ไม่รู้จักจะได้รหัสจับคู่; เจ้าของต้องอนุมัติ
- `"allowlist"`: อนุญาตเฉพาะผู้ส่งใน `channels.whatsapp.allowFrom` (หรือร้าน allow ที่จับคู่แล้ว)
- `"open"`: อนุญาต DMs ขาเข้าทั้งหมด (**ต้องมี** `channels.whatsapp.allowFrom` รวม `"*"`)
- `"disabled"`: เพิกเฉย DMs ขาเข้าทั้งหมด

Pairing codes expire after 1 hour; the bot only sends a pairing code when a new request is created. รหัสจับคู่หมดอายุใน 1 ชั่วโมง บอตจะส่งรหัสเฉพาะเมื่อมีคำขอใหม่
คำขอจับคู่ DM ที่รอดำเนินการจำกัด **3 ต่อช่องทาง** โดยค่าเริ่มต้น

การอนุมัติการจับคู่:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

Allowlist หมายเลขโทรศัพท์ E.164 ที่สามารถกระตุ้นการตอบกลับอัตโนมัติของ WhatsApp (**เฉพาะ DMs**)
หากว่างและ `channels.whatsapp.dmPolicy="pairing"` ผู้ส่งที่ไม่รู้จักจะได้รับรหัสจับคู่
สำหรับกลุ่ม ใช้ `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`
If empty and `channels.whatsapp.dmPolicy="pairing"`, unknown senders will receive a pairing code.
For groups, use `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50, // optional inbound media cap (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

ควบคุมว่าข้อความ WhatsApp ขาเข้าจะถูกทำเครื่องหมายว่าอ่านแล้วหรือไม่ (ติ๊กน้ำเงิน)
ค่าเริ่มต้น: `true` Default: `true`.

โหมดคุยกับตัวเองจะข้ามการแจ้งอ่านเสมอ แม้จะเปิดใช้งาน

การแทนที่ต่อบัญชี: `channels.whatsapp.accounts.<id>.sendReadReceipts`

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (หลายบัญชี)

รันหลายบัญชี WhatsApp ใน Gateway เดียว:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

หมายเหตุ:

- คำสั่งขาออกจะใช้บัญชี `default` หากมี มิฉะนั้นใช้บัญชีแรกตามลำดับ
- ไดเรกทอรีการยืนยันตัวตน Baileys แบบบัญชีเดียวเดิมจะถูกย้ายโดย `openclaw doctor` ไปยัง `whatsapp/default`

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

รันหลายบัญชีต่อช่องทาง (แต่ละบัญชีมี `accountId` ของตนและ `name` แบบไม่บังคับ):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

หมายเหตุ:

- ใช้ `default` เมื่อไม่ระบุ `accountId` (CLI + การกำหนดเส้นทาง)
- โทเคน env ใช้กับบัญชี **ค่าเริ่มต้น** เท่านั้น
- การตั้งค่าช่องทางพื้นฐาน (นโยบายกลุ่ม การกั้นการกล่าวถึง ฯลฯ) ใช้กับทุกบัญชี เว้นแต่จะถูกแทนที่ต่อบัญชี apply to all accounts unless overridden per account.
- ใช้ `bindings[].match.accountId` เพื่อกำหนดเส้นทางแต่ละบัญชีไปยัง agents.defaults ต่างกัน

### การกั้นการกล่าวถึงในแชตกลุ่ม (`agents.list[].groupChat` + `messages.groupChat`)

ข้อความกลุ่มตั้งค่าเริ่มต้นเป็น **ต้องมีการกล่าวถึง** (ทั้งเมทาดาทาหรือแพทเทิร์น regex)
ใช้กับแชตกลุ่ม WhatsApp, Telegram, Discord, Google Chat และ iMessage Applies to WhatsApp, Telegram, Discord, Google Chat, and iMessage group chats.

**ประเภทการกล่าวถึง:**

- **Metadata mentions**: Native platform @-mentions (e.g., WhatsApp tap-to-mention). **เมทาดาทา**: @-mentions ตามแพลตฟอร์ม (เช่น แตะเพื่อกล่าวถึงใน WhatsApp) ถูกละเลยในโหมดคุยกับตัวเองของ WhatsApp (ดู `channels.whatsapp.allowFrom`)
- **แพทเทิร์นข้อความ**: regex ใน `agents.list[].groupChat.mentionPatterns` ตรวจเสมอไม่ขึ้นกับโหมดคุยกับตัวเอง Always checked regardless of self-chat mode.
- การกั้นการกล่าวถึงบังคับใช้เฉพาะเมื่อสามารถตรวจจับได้ (เมทาดาทาหรือมีอย่างน้อยหนึ่ง `mentionPattern`)

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` ตั้งค่าเริ่มต้นส่วนกลางสำหรับบริบทประวัติกลุ่ม ช่องทางสามารถแทนที่ด้วย `channels.<channel> Channels can override with `channels.<channel>.historyLimit`(หรือ`channels.<channel>.accounts.\*.historyLimit`สำหรับหลายบัญชี) ตั้ง`0`เพื่อปิดการห่อประวัติ Set`0\` to disable history wrapping.

#### ขีดจำกัดประวัติ DM

DM conversations use session-based history managed by the agent. การสนทนา DM ใช้ประวัติแบบเซสชันที่จัดการโดยเอเจนต์ คุณสามารถจำกัดจำนวนรอบผู้ใช้ที่เก็บต่อเซสชัน DM:

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30, // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

ลำดับการแก้ค่า:

1. แทนที่ต่อ DM: `channels.<provider>.dms[userId].historyLimit`
2. ค่าเริ่มต้นผู้ให้บริการ: `channels.<provider>.dmHistoryLimit`
3. ไม่จำกัด (เก็บทั้งหมด)

ผู้ให้บริการที่รองรับ: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`

การแทนที่ต่อเอเจนต์ (มีลำดับเหนือกว่าเมื่อกำหนด แม้ `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

Mention gating defaults live per channel (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). ค่าเริ่มต้นการกั้นการกล่าวถึงอยู่ต่อช่องทาง (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`) เมื่อกำหนด `*.groups` จะทำหน้าที่เป็น allowlist กลุ่มด้วย รวม `"*"` เพื่ออนุญาตทุกกลุ่ม

เพื่อตอบ **เฉพาะ** ทริกเกอร์ข้อความ (ไม่สนใจ @-mentions):

```json5
{
  channels: {
    whatsapp: {
      // Include your own number to enable self-chat mode (ignore native @-mentions).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### นโยบายกลุ่ม (ต่อช่องทาง)

ใช้ `channels.*.groupPolicy` เพื่อควบคุมว่าจะรับข้อความกลุ่ม/ห้องหรือไม่:

```json5
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

หมายเหตุ:

- `"open"`: กลุ่มข้าม allowlist; ยังใช้การกั้นการกล่าวถึง
- `"disabled"`: บล็อกข้อความกลุ่ม/ห้องทั้งหมด
- `"allowlist"`: อนุญาตเฉพาะกลุ่ม/ห้องที่ตรง allowlist
- `channels.defaults.groupPolicy` ตั้งค่าเริ่มต้นเมื่อ `groupPolicy` ของผู้ให้บริการไม่ถูกตั้ง
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams ใช้ `groupAllowFrom` (fallback: `allowFrom`)
- Discord/Slack ใช้ allowlist ช่องทาง (`channels.discord.guilds.*.channels`, `channels.slack.channels`)
- Group DMs (Discord/Slack) ยังถูกควบคุมโดย `dm.groupEnabled` + `dm.groupChannels`
- ค่าเริ่มต้นคือ `groupPolicy: "allowlist"` (เว้นแต่ถูกแทนที่โดย `channels.defaults.groupPolicy`) หากไม่มี allowlist ข้อความกลุ่มจะถูกบล็อก

### การกำหนดเส้นทางหลายเอเจนต์ (`agents.list` + `bindings`)

รันเอเจนต์ที่แยกจากกันหลายตัว (เวิร์กสเปซ, `agentDir`, เซสชันแยก)
ภายใน Gateway เดียว ข้อความขาเข้าจะถูกกำหนดเส้นทางไปยังเอเจนต์ผ่านการผูก
Inbound messages are routed to an agent via bindings.

- `agents.list[]`: การแทนที่ต่อเอเจนต์
  - `id`: id เอเจนต์คงที่ (จำเป็น)
  - `default`: ไม่บังคับ; หากตั้งหลายรายการ รายการแรกชนะและบันทึกคำเตือน หากไม่ตั้ง ค่าเริ่มต้นคือรายการแรก
    If none are set, the **first entry** in the list is the default agent.
  - `name`: ชื่อแสดงของเอเจนต์
  - `workspace`: ค่าเริ่มต้น `~/.openclaw/workspace-<agentId>` (สำหรับ `main` ถอยกลับ `agents.defaults.workspace`)
  - `agentDir`: ค่าเริ่มต้น `~/.openclaw/agents/<agentId>/agent`
  - `model`: โมเดลเริ่มต้นต่อเอเจนต์ แทนที่ `agents.defaults.model`
    - รูปแบบสตริง: `"provider/model"` แทนที่เฉพาะ `agents.defaults.model.primary`
    - รูปแบบอ็อบเจ็กต์: `{ primary, fallbacks }` (fallback แทนที่ `agents.defaults.model.fallbacks`; `[]` ปิด fallback ส่วนกลางสำหรับเอเจนต์นี้)
  - `identity`: ชื่อ/ธีม/อีโมจิต่อเอเจนต์ (ใช้กับแพทเทิร์นการกล่าวถึง + รีแอ็กชัน ack)
  - `groupChat`: การกั้นการกล่าวถึงต่อเอเจนต์ (`mentionPatterns`)
  - `sandbox`: คอนฟิก sandbox ต่อเอเจนต์ (แทนที่ `agents.defaults.sandbox`)
    - `mode`: `"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    - `scope`: `"session"` | `"agent"` | `"shared"`
    - `workspaceRoot`: รากเวิร์กสเปซ sandbox แบบกำหนดเอง
    - `docker`: การแทนที่ docker ต่อเอเจนต์ (เช่น `image`, `network`, `env`, `setupCommand`, limits; ถูกละเลยเมื่อ `scope: "shared"`)
    - `browser`: การแทนที่เบราว์เซอร์ sandbox ต่อเอเจนต์ (ถูกละเลยเมื่อ `scope: "shared"`)
    - `prune`: การแทนที่การตัดแต่ง sandbox ต่อเอเจนต์ (ถูกละเลยเมื่อ `scope: "shared"`)
  - `subagents`: ค่าเริ่มต้น sub-agent ต่อเอเจนต์
    - `allowAgents`: allowlist id เอเจนต์สำหรับ `sessions_spawn` จากเอเจนต์นี้ (`["*"]` = อนุญาตใดๆ ค่าเริ่มต้น: เฉพาะเอเจนต์เดียวกัน)
  - `tools`: ข้อจำกัดเครื่องมือต่อเอเจนต์ (ใช้ก่อนนโยบายเครื่องมือของ sandbox)
    - `profile`: โปรไฟล์เครื่องมือพื้นฐาน
    - `allow`: อาเรย์ชื่อเครื่องมือที่อนุญาต
    - `deny`: อาเรย์ชื่อเครื่องมือที่ห้าม (ห้ามชนะ)
- `agents.defaults`: ค่าเริ่มต้นเอเจนต์ที่แชร์
- `bindings[]`: กำหนดเส้นทางข้อความขาเข้าไปยัง `agentId`
  - `match.channel` (จำเป็น)
  - `match.accountId` (ไม่บังคับ; `*` = บัญชีใดก็ได้)
  - `match.peer` (ไม่บังคับ; `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (ไม่บังคับ; เฉพาะช่องทาง)

ลำดับการจับคู่แบบกำหนดแน่นอน:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`
5. `match.accountId: "*"`
6. เอเจนต์เริ่มต้น (`agents.list[].default` หรือรายการแรก หรือ `"main"`)

ภายในแต่ละชั้น การจับคู่รายการแรกใน `bindings` จะชนะ

#### โปรไฟล์การเข้าถึงต่อเอเจนต์ (หลายเอเจนต์)

แต่ละเอเจนต์สามารถมี sandbox + นโยบายเครื่องมือของตนเอง
ใช้เพื่อผสมระดับการเข้าถึงใน Gateway เดียว: Use this to mix access
levels in one gateway:

- **เข้าถึงเต็มรูปแบบ** (เอเจนต์ส่วนตัว)
- เครื่องมือ **อ่านอย่างเดียว** + เวิร์กสเปซ
- **ไม่เข้าถึงไฟล์ระบบ** (เฉพาะเครื่องมือข้อความ/เซสชัน)

ดู [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools)

ตัวอย่างการเข้าถึงเต็ม (ไม่มี sandbox):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

เครื่องมืออ่านอย่างเดียว + เวิร์กสเปซอ่านอย่างเดียว:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

ไม่เข้าถึงไฟล์ระบบ (เปิดเครื่องมือข้อความ/เซสชัน):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

ตัวอย่าง: สองบัญชี WhatsApp → สองเอเจนต์:

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

### `tools.agentToAgent` (ไม่บังคับ)

การส่งข้อความระหว่างเอเจนต์ต้องเลือกเปิดใช้:

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `messages.queue`

ควบคุมพฤติกรรมเมื่อมีเอเจนต์รันอยู่แล้วและมีข้อความขาเข้าใหม่

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

หน่วงรวมข้อความขาเข้าที่รวดเร็วจาก **ผู้ส่งเดียวกัน** ให้หลายข้อความติดกันกลายเป็นรอบเอเจนต์เดียว
การหน่วงใช้ขอบเขตต่อช่องทาง + การสนทนา และใช้ข้อความล่าสุดสำหรับการผูกเธรด/ID การตอบ Debouncing is scoped per channel + conversation
and uses the most recent message for reply threading/IDs.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

หมายเหตุ:

- หน่วงรวมเฉพาะข้อความ **ตัวอักษรล้วน**; สื่อ/ไฟล์แนบจะฟลัชทันที
- คำสั่งควบคุม (เช่น `/queue`, `/new`) ข้ามการหน่วง

### `commands` (การจัดการคำสั่งแชต)

ควบคุมการเปิดใช้คำสั่งแชตข้ามคอนเน็กเตอร์

```json5
{
  commands: {
    native: "auto", // register native commands when supported (auto)
    text: true, // parse slash commands in chat messages
    bash: false, // allow ! (alias: /bash) (host-only; requires tools.elevated allowlists)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false, // allow /config (writes to disk)
    debug: false, // allow /debug (runtime-only overrides)
    restart: false, // allow /restart + gateway restart tool
    useAccessGroups: true, // enforce access-group allowlists/policies for commands
  },
}
```

หมายเหตุ:

- คำสั่งข้อความต้องส่งเป็นข้อความ **เดี่ยว** และใช้พรีฟิกซ์นำหน้า `/`
- `commands.text: false` ปิดการพาร์สคำสั่งจากแชต
- `commands.native: "auto"` (ค่าเริ่มต้น) เปิดคำสั่งเนทีฟสำหรับ Discord/Telegram และปิด Slack
- ตั้ง `commands.native: true|false` เพื่อบังคับทั้งหมด หรือแทนที่ต่อช่องทางด้วย `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` `false` ล้างคำสั่งที่ลงทะเบียนก่อนหน้าบน Discord/Telegram ตอนเริ่ม
- `channels.telegram.customCommands` เพิ่มเมนูบอต Telegram เพิ่มเติม 2. ชื่อจะถูกทำให้เป็นมาตรฐาน; ความขัดแย้งกับคำสั่งเนทีฟจะถูกละเว้น
- `commands.bash: true` เปิด `! 3. <cmd>` เพื่อรันคำสั่งเชลล์ของโฮสต์ (`/bash <cmd>` ใช้เป็นนามแฝงได้เช่นกัน) <cmd>`เพื่อรันคำสั่งเชลล์โฮสต์ ต้องมี`tools.elevated.enabled`และ allowlist ผู้ส่งใน`tools.elevated.allowFrom.<channel>\`
- `commands.bashForegroundMs` ควบคุมเวลารอของ bash 4. ขณะงาน bash กำลังทำงาน คำขอ `! 5. <cmd>` ใหม่จะถูกปฏิเสธ (ครั้งละหนึ่งงาน) <cmd>\` requests are rejected (one at a time).
- `commands.config: true` เปิด `/config`
- 7. .configWrites`ใช้ควบคุมการเปลี่ยนแปลงคอนฟิกที่เริ่มจากช่องนั้น (ค่าเริ่มต้น: true)8. ใช้กับ`/config set|unset`รวมถึงการย้ายอัตโนมัติเฉพาะผู้ให้บริการ (การเปลี่ยน ID ซูเปอร์กรุ๊ปของ Telegram, การเปลี่ยน ID ช่องของ Slack) 9. การอนุญาตถูกกำหนดจาก
       allowlist/การจับคู่ของช่อง รวมถึง`commands.useAccessGroups\`
- `commands.debug: true` เปิด `/debug`
- `commands.restart: true` เปิด `/restart`
- `commands.useAccessGroups: false` อนุญาตคำสั่งข้าม allowlist/นโยบายกลุ่ม
- คำสั่งสแลชและไดเรกทีฟใช้ได้เฉพาะ **ผู้ส่งที่ได้รับอนุญาต** 10. WhatsApp ทำงานผ่านเว็บแชนแนลของเกตเวย์ (Baileys Web)

### `web` (รันไทม์ช่องทางเว็บ WhatsApp)

11. จะเริ่มทำงานอัตโนมัติเมื่อมีเซสชันที่เชื่อมโยงอยู่ 12. ตั้งค่า `web.enabled: false` เพื่อปิดเป็นค่าเริ่มต้น
12. OpenClaw จะเริ่ม Telegram เฉพาะเมื่อมีส่วนคอนฟิก `channels.telegram` อยู่

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

### `channels.telegram` (ทรานสปอร์ตบอต)

14. ตั้งค่า `channels.telegram.enabled: false` เพื่อปิดการเริ่มอัตโนมัติ OpenClaw จะเริ่ม Telegram เมื่อมีส่วนคอนฟิก `channels.telegram`
    โทเคนบอตแก้จาก `channels.telegram.botToken` (หรือ `channels.telegram.tokenFile`) โดยมี `TELEGRAM_BOT_TOKEN` เป็น fallback
    ตั้ง `channels.telegram.enabled: false` เพื่อปิดการเริ่มอัตโนมัติ
15. การรองรับหลายบัญชีอยู่ภายใต้ `channels.telegram.accounts` (ดูส่วนหลายบัญชีด้านบน)
16. โทเค็นจากตัวแปรแวดล้อมใช้ได้กับบัญชีเริ่มต้นเท่านั้น 17. ตั้งค่า `channels.telegram.configWrites: false` เพื่อบล็อกการเขียนคอนฟิกที่เริ่มจาก Telegram (รวมถึงการย้าย ID ซูเปอร์กรุ๊ปและ `/config set|unset`)
17. บันทึกการสตรีมแบบร่าง:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50, // include last N group messages as context (0 disables)
      replyToMode: "first", // off | first | all
      linkPreview: true, // toggle outbound link previews
      streamMode: "partial", // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {
        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        // transport overrides
        autoSelectFamily: false,
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook", // requires webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

19. ใช้ Telegram `sendMessageDraft` (ฟองร่าง ไม่ใช่ข้อความจริง)

- 20. ต้องใช้ **หัวข้อแชตส่วนตัว** (message_thread_id ใน DM; บอทเปิดใช้งานหัวข้อแล้ว)
- 21. `/reasoning stream` จะสตรีมเหตุผลลงในแบบร่าง จากนั้นส่งคำตอบสุดท้าย
- 22. ค่าเริ่มต้นและพฤติกรรมของนโยบายการลองใหม่ถูกอธิบายไว้ใน [Retry policy](/concepts/retry)
  23. `channels.discord` (การขนส่งบอท)

### 24. กำหนดค่าบอท Discord โดยตั้งค่าโทเค็นบอทและการจำกัดเพิ่มเติม (ถ้ามี):&#xA;การรองรับหลายบัญชีอยู่ภายใต้ `channels.discord.accounts` (ดูส่วนหลายบัญชีด้านบน)

25. โทเค็นจากตัวแปรแวดล้อมใช้ได้กับบัญชีเริ่มต้นเท่านั้น 26. {
    channels: {
    discord: {
    enabled: true,
    token: "your-bot-token",
    mediaMaxMb: 8, // จำกัดขนาดสื่อขาเข้า
    allowBots: false, // อนุญาตข้อความที่เขียนโดยบอท
    actions: {
    // ตัวควบคุมการกระทำของเครื่องมือ (false คือปิด)
    reactions: true,
    stickers: true,
    polls: true,
    permissions: true,
    messages: true,
    threads: true,
    pins: true,
    search: true,
    memberInfo: true,
    roleInfo: true,
    roles: false,
    channelInfo: true,
    voiceStatus: true,
    events: true,
    moderation: false,
    },
    replyToMode: "off", // off | first | all
    dm: {
    enabled: true, // ปิด DM ทั้งหมดเมื่อเป็น false
    policy: "pairing", // pairing | allowlist | open | disabled
    allowFrom: ["1234567890", "steipete"], // allowlist DM ตามตัวเลือก ("open" ต้องมี ["\*"])
    groupEnabled: false, // เปิดใช้งาน DM แบบกลุ่ม
    groupChannels: ["openclaw-dm"], // allowlist DM กลุ่มตามตัวเลือก
    },
    guilds: {
    "123456789012345678": {
    // guild id (แนะนำ) หรือ slug
    slug: "friends-of-openclaw",
    requireMention: false, // ค่าเริ่มต้นต่อกิลด์
    reactionNotifications: "own", // off | own | all | allowlist
    users: ["987654321098765432"], // allowlist ผู้ใช้ต่อกิลด์ (ตัวเลือก)
    channels: {
    general: { allow: true },
    help: {
    allow: true,
    requireMention: true,
    users: ["987654321098765432"],
    skills: ["docs"],
    systemPrompt: "Short answers only.",
    },
    },
    },
    },
    historyLimit: 20, // รวมข้อความกิลด์ล่าสุด N ข้อความเป็นบริบท
    textChunkLimit: 2000, // ขนาดการแบ่งข้อความขาออกตามตัวเลือก (ตัวอักษร)
    chunkMode: "length", // โหมดการแบ่งตามตัวเลือก (length | newline)
    maxLinesPerMessage: 17, // จำนวนบรรทัดสูงสุดต่อข้อความ (เพื่อหลีกเลี่ยงการตัดใน UI Discord)
    retry: {
    // นโยบายการลองส่งซ้ำขาออก
    attempts: 3,
    minDelayMs: 500,
    maxDelayMs: 30000,
    jitter: 0.1,
    },
    },
    },
    }

```json5
27. OpenClaw จะเริ่ม Discord เฉพาะเมื่อมีส่วนคอนฟิก `channels.discord` อยู่
```

28. โทเค็นจะถูกแก้จาก `channels.discord.token` โดยมี `DISCORD_BOT_TOKEN` เป็นค่า fallback สำหรับบัญชีเริ่มต้น (เว้นแต่ `channels.discord.enabled` จะเป็น `false`) 29. ใช้ `user:<id>` (DM) หรือ `channel:<id>` (ช่องกิลด์) เมื่อระบุเป้าหมายการส่งสำหรับคำสั่ง cron/CLI; ID ตัวเลขล้วนไม่ชัดเจนและจะถูกปฏิเสธ 30. slug ของกิลด์เป็นตัวพิมพ์เล็กและแทนที่ช่องว่างด้วย `-`; คีย์ช่องใช้ชื่อช่องที่ทำเป็น slug (ไม่มี `#` นำหน้า)
29. ควรใช้ guild id เป็นคีย์เพื่อหลีกเลี่ยงความกำกวมจากการเปลี่ยนชื่อ 32. ข้อความที่เขียนโดยบอทจะถูกละเว้นตามค่าเริ่มต้น
30. เปิดใช้งานด้วย `channels.discord.allowBots` (ข้อความของบอทเองยังคงถูกกรองเพื่อป้องกันลูปตอบตัวเอง) 34. โหมดการแจ้งเตือนรีแอคชัน:
31. ข้อความขาออกจะถูกแบ่งตาม `channels.discord.textChunkLimit` (ค่าเริ่มต้น 2000)

- `off`: ไม่มีอีเวนต์รีแอคชัน
- `own`: รีแอคชันบนข้อความของบอตเอง (ค่าเริ่มต้น)
- `all`: รีแอคชันทั้งหมดบนทุกข้อความ
- `allowlist`: รีแอคชันจาก `guilds.<id>.users` บนทุกข้อความ (รายการว่างจะปิดใช้งาน)
  36. ตั้งค่า `channels.discord.chunkMode="newline"` เพื่อแยกตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนการแบ่งตามความยาว 37. ไคลเอนต์ Discord อาจตัดข้อความที่ยาวมาก ดังนั้น `channels.discord.maxLinesPerMessage` (ค่าเริ่มต้น 17) จะตัดแบ่งคำตอบหลายบรรทัดที่ยาว แม้จะต่ำกว่า 2000 ตัวอักษร 38. ค่าเริ่มต้นและพฤติกรรมของนโยบายการลองใหม่ถูกอธิบายไว้ใน [Retry policy](/concepts/retry)
  39. `channels.googlechat` (เว็บฮุค Chat API)

### 40. Google Chat ทำงานผ่านเว็บฮุค HTTP พร้อมการยืนยันตัวตนระดับแอป (service account)

41. การรองรับหลายบัญชีอยู่ภายใต้ `channels.googlechat.accounts` (ดูส่วนหลายบัญชีด้านบน)
42. ตัวแปรแวดล้อมใช้ได้กับบัญชีเริ่มต้นเท่านั้น 43. {
    channels: {
    googlechat: {
    enabled: true,
    serviceAccountFile: "/path/to/service-account.json",
    audienceType: "app-url", // app-url | project-number
    audience: "https://gateway.example.com/googlechat",
    webhookPath: "/googlechat",
    botUser: "users/1234567890", // ตัวเลือก; ช่วยปรับปรุงการตรวจจับการกล่าวถึง
    dm: {
    enabled: true,
    policy: "pairing", // pairing | allowlist | open | disabled
    allowFrom: ["users/1234567890"], // ตัวเลือก; "open" ต้องมี ["\*"]
    },
    groupPolicy: "allowlist",
    groups: {
    "spaces/AAAA": { allow: true, requireMention: true },
    },
    actions: { reactions: true },
    typingIndicator: "message",
    mediaMaxMb: 20,
    },
    },
    }

```json5
44. JSON ของ Service account สามารถใส่แบบอินไลน์ (`serviceAccount`) หรือแบบไฟล์ (`serviceAccountFile`)
```

หมายเหตุ:

- 45. ค่า fallback จากตัวแปรแวดล้อมสำหรับบัญชีเริ่มต้น: `GOOGLE_CHAT_SERVICE_ACCOUNT` หรือ `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`
- 46. `audienceType` + `audience` ต้องตรงกับการตั้งค่าการยืนยันตัวตนของเว็บฮุคแอป Chat
- 47. ใช้ `spaces/<spaceId>` หรือ `users/<userId|email>` เมื่อกำหนดเป้าหมายการส่ง
- 48. `channels.slack` (โหมดซ็อกเก็ต)

### 49. Slack ทำงานในโหมด Socket และต้องใช้ทั้งโทเค็นบอทและโทเค็นแอป:

50. {
    channels: {
    slack: {
    enabled: true,
    botToken: "xoxb-...",
    appToken: "xapp-...",
    dm: {
    enabled: true,
    policy: "pairing", // pairing | allowlist | open | disabled
    allowFrom: ["U123", "U456", "_"], // ตัวเลือก; "open" ต้องมี ["_"]
    groupEnabled: false,
    groupChannels: ["G123"],
    },
    channels: {
    C123: { allow: true, requireMention: true, allowBots: false },
    "#general": {
    allow: true,
    requireMention: true,
    allowBots: false,
    users: ["U123"],
    skills: ["docs"],
    systemPrompt: "Short answers only.",
    },
    },
    historyLimit: 50, // รวมข้อความช่อง/กลุ่มล่าสุด N ข้อความเป็นบริบท (0 คือปิด)
    allowBots: false,
    reactionNotifications: "own", // off | own | all | allowlist
    reactionAllowlist: ["U123"],
    replyToMode: "off", // off | first | all
    thread: {
    historyScope: "thread", // thread | channel
    inheritParent: false,
    },
    actions: {
    reactions: true,
    messages: true,
    pins: true,
    memberInfo: true,
    emojiList: true,
    },
    slashCommand: {
    enabled: true,
    name: "openclaw",
    sessionPrefix: "slack:slash",
    ephemeral: true,
    },
    textChunkLimit: 4000,
    chunkMode: "length",
    mediaMaxMb: 20,
    },
    },
    }

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"],
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50, // include last N channel/group messages as context (0 disables)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

1. การรองรับหลายบัญชีอยู่ภายใต้ `channels.slack.accounts` (ดูส่วน multi-account ด้านบน) 2. โทเคนจากตัวแปรสภาพแวดล้อม (Env) ใช้ได้เฉพาะกับบัญชีค่าเริ่มต้นเท่านั้น

3. OpenClaw จะเริ่ม Slack เมื่อเปิดใช้งานผู้ให้บริการและตั้งค่าโทเคนทั้งสองแล้ว (ผ่านคอนฟิกหรือ `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`) 4. ใช้ `user:<id>` (DM) หรือ `channel:<id>` เมื่อระบุเป้าหมายการส่งสำหรับคำสั่ง cron/CLI
4. ตั้งค่า `channels.slack.configWrites: false` เพื่อบล็อกการเขียนคอนฟิกที่เริ่มจาก Slack (รวมถึงการย้าย channel ID และ `/config set|unset`)

6. ข้อความที่บอทเป็นผู้เขียนจะถูกละเว้นโดยค่าเริ่มต้น 7. เปิดใช้งานด้วย `channels.slack.allowBots` หรือ `channels.slack.channels.<id>.allowBots`

8. โหมดการแจ้งเตือนรีแอคชัน:

- `off`: ไม่มีอีเวนต์รีแอคชัน
- `own`: รีแอคชันบนข้อความของบอตเอง (ค่าเริ่มต้น)
- `all`: รีแอคชันทั้งหมดบนทุกข้อความ
- 9. `allowlist`: รีแอคชันจาก `channels.slack.reactionAllowlist` บนข้อความทั้งหมด (รายการว่างจะปิดการใช้งาน)

10. การแยกเซสชันเธรด:

- 11. `channels.slack.thread.historyScope` ควบคุมว่าประวัติเธรดจะเป็นต่อเธรด (`thread`, ค่าเริ่มต้น) หรือใช้ร่วมกันทั้งแชนเนล (`channel`)
- 12. `channels.slack.thread.inheritParent` ควบคุมว่าเซสชันเธรดใหม่จะสืบทอดทรานสคริปต์ของแชนเนลแม่หรือไม่ (ค่าเริ่มต้น: false)

13. กลุ่มแอ็กชันของ Slack (ปิด/เปิดการทำงานของเครื่องมือ `slack`):

| Action group | Default | Notes                                                 |
| ------------ | ------- | ----------------------------------------------------- |
| reactions    | enabled | 14. React + แสดงรายการรีแอคชัน |
| messages     | enabled | อ่าน/ส่ง/แก้ไข/ลบ                                     |
| pins         | enabled | ปักหมุด/ยกเลิก/แสดงรายการ                             |
| memberInfo   | enabled | ข้อมูลสมาชิก                                          |
| emojiList    | enabled | รายการอีโมจิแบบกำหนดเอง                               |

### 15. `channels.mattermost` (โทเคนบอท)

Mattermost ทำงานในรูปแบบปลั๊กอินและไม่ได้รวมมากับการติดตั้งแกนหลัก
16. ติดตั้งก่อน: `openclaw plugins install @openclaw/mattermost` (หรือ `./extensions/mattermost` จาก git checkout)

17. Mattermost ต้องใช้โทเคนบอทพร้อมกับ base URL ของเซิร์ฟเวอร์ของคุณ:

```json5
18. {
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

19. OpenClaw จะเริ่ม Mattermost เมื่อบัญชีถูกตั้งค่า (โทเคนบอท + base URL) และเปิดใช้งานแล้ว 20. โทเคน + base URL จะถูกแก้ไขค่าจาก `channels.mattermost.botToken` + `channels.mattermost.baseUrl` หรือ `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` สำหรับบัญชีค่าเริ่มต้น (เว้นแต่ `channels.mattermost.enabled` จะเป็น `false`)

21. โหมดแชต:

- 22. `oncall` (ค่าเริ่มต้น): ตอบข้อความในแชนเนลเฉพาะเมื่อถูก @mention
- `onmessage`: ตอบทุกข้อความในช่องทาง
- 23. `onchar`: ตอบเมื่อข้อความขึ้นต้นด้วยคำนำหน้าทริกเกอร์ (`channels.mattermost.oncharPrefixes`, ค่าเริ่มต้น `[">", "!"]`)

24. การควบคุมการเข้าถึง:

- 25. DM ค่าเริ่มต้น: `channels.mattermost.dmPolicy="pairing"` (ผู้ส่งที่ไม่รู้จักจะได้รับโค้ดจับคู่)
- DMs สาธารณะ: `channels.mattermost.dmPolicy="open"` พร้อมกับ `channels.mattermost.allowFrom=["*"]`
- 26. กลุ่ม: `channels.mattermost.groupPolicy="allowlist"` เป็นค่าเริ่มต้น (ต้องถูกกล่าวถึง) 27. ใช้ `channels.mattermost.groupAllowFrom` เพื่อจำกัดผู้ส่ง

28. การรองรับหลายบัญชีอยู่ภายใต้ `channels.mattermost.accounts` (ดูส่วน multi-account ด้านบน) 29. ตัวแปรสภาพแวดล้อมใช้ได้เฉพาะกับบัญชีค่าเริ่มต้นเท่านั้น
29. ใช้ `channel:<id>` หรือ `user:<id>` (หรือ `@username`) เมื่อระบุเป้าหมายการส่ง; id เปล่าจะถูกมองว่าเป็น id ของแชนเนล

### 31. `channels.signal` (signal-cli)

32. รีแอคชันของ Signal สามารถส่งอีเวนต์ระบบได้ (เครื่องมือรีแอคชันที่ใช้ร่วมกัน):

```json5
33. {
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // include last N group messages as context (0 disables)
    },
  },
}
```

34. โหมดการแจ้งเตือนรีแอคชัน:

- `off`: ไม่มีอีเวนต์รีแอคชัน
- `own`: รีแอคชันบนข้อความของบอตเอง (ค่าเริ่มต้น)
- `all`: รีแอคชันทั้งหมดบนทุกข้อความ
- 35. `allowlist`: รีแอคชันจาก `channels.signal.reactionAllowlist` บนข้อความทั้งหมด (รายการว่างจะปิดการใช้งาน)

### 36. `channels.imessage` (imsg CLI)

37. OpenClaw สร้างโปรเซส `imsg rpc` (JSON-RPC ผ่าน stdio) 38. ไม่ต้องใช้ดีมอนหรือพอร์ต

```json5
39. {
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SCP for remote attachments when using SSH wrapper
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50, // include last N group messages as context (0 disables)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

40. การรองรับหลายบัญชีอยู่ภายใต้ `channels.imessage.accounts` (ดูส่วน multi-account ด้านบน)

หมายเหตุ:

- 41. ต้องการสิทธิ์ Full Disk Access สำหรับฐานข้อมูล Messages
- 42. การส่งครั้งแรกจะมีพรอมต์ขอสิทธิ์การทำงานอัตโนมัติของ Messages
- 43. แนะนำให้ใช้เป้าหมายแบบ `chat_id:<id>` 44. ใช้ `imsg chats --limit 20` เพื่อแสดงรายการแชต
- 45. `channels.imessage.cliPath` สามารถชี้ไปที่สคริปต์ตัวห่อ (เช่น `ssh` ไปยัง Mac เครื่องอื่นที่รัน `imsg rpc`); ใช้ SSH keys เพื่อหลีกเลี่ยงการถามรหัสผ่าน
- 46. สำหรับตัวห่อ SSH ระยะไกล ให้ตั้งค่า `channels.imessage.remoteHost` เพื่อดึงไฟล์แนบผ่าน SCP เมื่อเปิด `includeAttachments`

ตัวอย่าง wrapper:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

47. ตั้งค่า **ไดเรกทอรีเวิร์กสเปซส่วนกลางเพียงหนึ่งเดียว** ที่เอเจนต์ใช้สำหรับการทำงานกับไฟล์

ค่าเริ่มต้น: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

48. หากเปิดใช้งาน `agents.defaults.sandbox` เซสชันที่ไม่ใช่หลักสามารถเขียนทับค่านี้ได้ด้วยเวิร์กสเปซต่อสโคปของตนเองภายใต้ `agents.defaults.sandbox.workspaceRoot`

### 49. `agents.defaults.repoRoot`

50. รูทรีโปซิทอรีเสริมเพื่อแสดงในบรรทัด Runtime ของ system prompt 1. หากไม่ได้ตั้งค่า OpenClaw
    จะพยายามตรวจหาไดเรกทอรี `.git` โดยไล่ขึ้นไปจากเวิร์กสเปซ (และไดเรกทอรีทำงานปัจจุบัน) 2. ต้องมีพาธนั้นอยู่จริงจึงจะนำมาใช้ได้

```json5
3. {
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### 4. `agents.defaults.skipBootstrap`

5. ปิดการสร้างไฟล์บูตสแตรปของเวิร์กสเปซโดยอัตโนมัติ (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` และ `BOOTSTRAP.md`)

6. ใช้สิ่งนี้สำหรับการดีพลอยที่มีการเตรียมไว้ล่วงหน้า ซึ่งไฟล์เวิร์กสเปซของคุณมาจากรีโป

```json5
7. {
  agents: { defaults: { skipBootstrap: true } },
}
```

### 8. `agents.defaults.bootstrapMaxChars`

9. จำนวนอักขระสูงสุดของไฟล์บูตสแตรปแต่ละไฟล์ของเวิร์กสเปซที่ถูกฉีดเข้าไปใน system prompt
   ก่อนถูกตัดทอน ค่าเริ่มต้น: `20000`.

10. เมื่อไฟล์มีขนาดเกินขีดจำกัดนี้ OpenClaw จะบันทึกคำเตือนและฉีดส่วนหัว/ส่วนท้ายที่ถูกตัดทอนพร้อมตัวบ่งชี้

```json5
11. {
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

12. ตั้งค่าเขตเวลาของผู้ใช้สำหรับ **บริบทของ system prompt** (ไม่ใช่สำหรับไทม์สแตมป์ในซองข้อความ) 13. หากไม่ได้ตั้งค่า OpenClaw จะใช้เขตเวลาของโฮสต์ขณะรันไทม์

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### 14. `agents.defaults.timeFormat`

15. ควบคุม **รูปแบบเวลา** ที่แสดงในส่วน Current Date & Time ของ system prompt
16. ค่าเริ่มต้น: `auto` (การตั้งค่าระบบปฏิบัติการ)

```json5
17. {
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `messages`

18. ควบคุมคำนำหน้าขาเข้า/ขาออก และปฏิกิริยาการยืนยัน (ack) แบบเลือกได้
19. ดู [Messages](/concepts/messages) สำหรับคิว เซสชัน และบริบทการสตรีม

```json5
20. {
  messages: {
    responsePrefix: "🦞", // หรือ "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

21. `responsePrefix` จะถูกใช้กับ **การตอบกลับขาออกทั้งหมด** (สรุปเครื่องมือ การสตรีมแบบบล็อก การตอบกลับสุดท้าย) ในทุกช่องทาง เว้นแต่จะมีอยู่แล้ว

22. สามารถกำหนด override แยกตามช่องทางและตามบัญชีได้:

- `channels.<provider>.configWrites` กั้นการเปลี่ยนคอนฟิกจากช่องทางนั้น
- `channels.<provider>.accounts.<id>.configWrites` กั้นการเปลี่ยนคอนฟิกจากช่องทางนั้น

ลำดับการตัดสิน(เฉพาะเจาะจงที่สุดชนะ):

1. `channels.<provider>.accounts.<id>.configWrites` กั้นการเปลี่ยนคอนฟิกจากช่องทางนั้น
2. `channels.<provider>.configWrites` กั้นการเปลี่ยนคอนฟิกจากช่องทางนั้น
3. 23. `messages.responsePrefix`

24) ความหมายเชิงพฤติกรรม:

- 25. `undefined` จะไหลต่อไปยังระดับถัดไป
- 26. `""` จะปิดการใช้คำนำหน้าอย่างชัดเจนและหยุดการไหลต่อ
- 27. `"auto"` จะสร้างค่า `[{identity.name}]` สำหรับเอเจนต์ที่ถูกกำหนดเส้นทาง

28. การ override มีผลกับทุกช่องทาง รวมถึงส่วนขยาย และกับการตอบกลับขาออกทุกประเภท

29. หากไม่ได้ตั้งค่า `messages.responsePrefix` จะไม่มีคำนำหน้าถูกใช้โดยค่าเริ่มต้น 30. การตอบกลับในการแชตตัวเองบน WhatsApp
    เป็นข้อยกเว้น: โดยค่าเริ่มต้นจะใช้ `[{identity.name}]` เมื่อมีการตั้งค่า มิฉะนั้นจะใช้
    `[openclaw]` เพื่อให้การสนทนาบนโทรศัพท์เครื่องเดียวกันอ่านเข้าใจได้
30. ตั้งค่าเป็น `"auto"` เพื่อสร้าง `[{identity.name}]` สำหรับเอเจนต์ที่ถูกกำหนดเส้นทาง (เมื่อมีการตั้งค่า)

#### 32. ตัวแปรเทมเพลต

33. สตริง `responsePrefix` สามารถรวมตัวแปรเทมเพลตที่แก้ค่าแบบไดนามิกได้:

| ตัวแปร                                       | Description                                        | Example                                                                |
| -------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------- |
| 34. `{model}`         | 35. ชื่อโมเดลแบบสั้น        | 36. `claude-opus-4-6`, `gpt-4o`                 |
| 37. `{modelFull}`     | Full model identifier                              | 39. `anthropic/claude-opus-4-6`                 |
| 40. `{provider}`      | 41. ชื่อผู้ให้บริการ        | 42. `anthropic`, `openai`                       |
| 43. `{thinkingLevel}` | 44. ระดับการคิดปัจจุบัน     | `high`, `low`, `off`                                                   |
| 46. `{identity.name}` | 47. ชื่ออัตลักษณ์ของเอเจนต์ | 48. (เหมือนกับโหมด `"auto"`) |

49. ตัวแปรไม่สนใจตัวพิมพ์เล็ก/ใหญ่ (`{MODEL}` = `{model}`) 50. `{think}` เป็นชื่อแฝงของ `{thinkingLevel}`
    Unresolved variables remain as literal text.

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]",
  },
}
```

Example output: `[claude-opus-4-6 | think:high] Here's my response...`

WhatsApp inbound prefix is configured via `channels.whatsapp.messagePrefix` (deprecated:
`messages.messagePrefix`). Default stays **unchanged**: `"[openclaw]"` when
`channels.whatsapp.allowFrom` is empty, otherwise `""` (no prefix). When using
`"[openclaw]"`, OpenClaw will instead use `[{identity.name}]` when the routed
agent has `identity.name` set.

`ackReaction` sends a best-effort emoji reaction to acknowledge inbound messages
on channels that support reactions (Slack/Discord/Telegram/Google Chat). Defaults to the
active agent’s `identity.emoji` when set, otherwise `"👀"`. Set it to `""` to disable.

`ackReactionScope` controls when reactions fire:

- `group-mentions` (default): only when a group/room requires mentions **and** the bot was mentioned
- `group-all`: all group/room messages
- `direct`: direct messages only
- `all`: all messages

`removeAckAfterReply` removes the bot’s ack reaction after a reply is sent
(Slack/Discord/Telegram/Google Chat only). Default: `false`.

#### `messages.tts`

Enable text-to-speech for outbound replies. When on, OpenClaw generates audio
using ElevenLabs or OpenAI and attaches it to responses. Telegram uses Opus
voice notes; other channels send MP3 audio.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

หมายเหตุ:

- `messages.tts.auto` controls auto‑TTS (`off`, `always`, `inbound`, `tagged`).
- `/tts off|always|inbound|tagged` sets the per‑session auto mode (overrides config).
- `messages.tts.enabled` is legacy; doctor migrates it to `messages.tts.auto`.
- `prefsPath` stores local overrides (provider/limit/summarize).
- `maxTextLength` is a hard cap for TTS input; summaries are truncated to fit.
- `summaryModel` overrides `agents.defaults.model.primary` for auto-summary.
  - Accepts `provider/model` or an alias from `agents.defaults.models`.
- `modelOverrides` enables model-driven overrides like `[[tts:...]]` tags (on by default).
- `/tts limit` and `/tts summary` control per-user summarization settings.
- `apiKey` values fall back to `ELEVENLABS_API_KEY`/`XI_API_KEY` and `OPENAI_API_KEY`.
- `elevenlabs.baseUrl` overrides the ElevenLabs API base URL.
- `elevenlabs.voiceSettings` supports `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost`, and `speed` (0.5..2.0).

### `talk`

Defaults for Talk mode (macOS/iOS/Android). Voice IDs fall back to `ELEVENLABS_VOICE_ID` or `SAG_VOICE_ID` when unset.
`apiKey` falls back to `ELEVENLABS_API_KEY` (or the gateway’s shell profile) when unset.
`voiceAliases` lets Talk directives use friendly names (e.g. `"voice":"Clawd"`).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

### `agents.defaults`

Controls the embedded agent runtime (model/thinking/verbose/timeouts).
`agents.defaults.models` defines the configured model catalog (and acts as the allowlist for `/model`).
`agents.defaults.model.primary` sets the default model; `agents.defaults.model.fallbacks` are global failovers.
`agents.defaults.imageModel` is optional and is **only used if the primary model lacks image input**.
Each `agents.defaults.models` entry can include:

- `alias` (optional model shortcut, e.g. `/opus`).
- `params` (optional provider-specific API params passed through to the model request).

`params` is also applied to streaming runs (embedded agent + compaction). Supported keys today: `temperature`, `maxTokens`. These merge with call-time options; caller-supplied values win. `temperature` เป็นตัวปรับขั้นสูง—ปล่อยว่างไว้ เว้นแต่คุณจะรู้ค่าเริ่มต้นของโมเดลและจำเป็นต้องเปลี่ยนแปลง

ตัวอย่าง:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

โมเดล Z.AI GLM-4.x จะเปิดโหมดคิด (thinking mode) โดยอัตโนมัติ เว้นแต่คุณจะ:

- ตั้งค่า `--thinking off` หรือ
- กำหนด `agents.defaults.models["zai/<model>"].params.thinking` ด้วยตัวคุณเอง

OpenClaw ยังมาพร้อมชื่อย่อ (alias shorthand) ที่มีมาให้ในตัวบางส่วน ค่าเริ่มต้นจะถูกนำไปใช้เฉพาะเมื่อโมเดล
มีอยู่แล้วใน `agents.defaults.models`:

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

หากคุณกำหนดชื่อ alias เดียวกัน (ไม่สนใจตัวพิมพ์เล็ก–ใหญ่) เอง ค่าของคุณจะมีผลเหนือกว่า (ค่าเริ่มต้นจะไม่เขียนทับ)

ตัวอย่าง: ใช้ Opus 4.6 เป็นหลัก และ MiniMax M2.1 เป็นตัวสำรอง (โฮสต์ MiniMax):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
    },
  },
}
```

การยืนยันตัวตน MiniMax: ตั้งค่า `MINIMAX_API_KEY` (ตัวแปรสภาพแวดล้อม) หรือกำหนด `models.providers.minimax`

#### `agents.defaults.cliBackends` (CLI fallback)

แบ็กเอนด์ CLI เสริมสำหรับการรันแบบข้อความล้วนเป็นตัวสำรอง (ไม่มีการเรียกใช้เครื่องมือ) สิ่งเหล่านี้มีประโยชน์เป็น
เส้นทางสำรองเมื่อผู้ให้บริการ API ล้มเหลว รองรับการส่งต่อภาพ (image pass-through) เมื่อคุณกำหนด
`imageArg` ที่รับพาธไฟล์

หมายเหตุ:

- แบ็กเอนด์ CLI เป็นแบบ **เน้นข้อความเป็นหลัก**; เครื่องมือจะถูกปิดใช้งานเสมอ
- รองรับเซสชันเมื่อกำหนด `sessionArg`; รหัสเซสชันจะถูกบันทึกต่อแบ็กเอนด์
- สำหรับ `claude-cli` มีการตั้งค่าเริ่มต้นเตรียมไว้ให้แล้ว เขียนทับพาธคำสั่งหาก PATH มีน้อย
  (launchd/systemd)

ตัวอย่าง:

```json5
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

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false,
            },
          },
        },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free",
        ],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last",
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000,
      },
      contextTokens: 200000,
    },
  },
}
```

#### `agents.defaults.contextPruning` (การตัดแต่งผลลัพธ์เครื่องมือ)

`agents.defaults.contextPruning` จะตัด **ผลลัพธ์เครื่องมือเก่า** ออกจากบริบทในหน่วยความจำ ทันที ก่อนส่งคำขอไปยัง LLM
จะ **ไม่** แก้ไขประวัติเซสชันบนดิสก์ (`*.jsonl` ยังคงสมบูรณ์)

ออกแบบมาเพื่อลดการใช้โทเค็นสำหรับเอเจนต์ที่ช่างคุยและสะสมผลลัพธ์เครื่องมือขนาดใหญ่ตามเวลา

ภาพรวมระดับสูง:

- ไม่แตะต้องข้อความของผู้ใช้/ผู้ช่วยเลย
- ปกป้องข้อความผู้ช่วยล่าสุดจำนวน `keepLastAssistants` (จะไม่ตัดผลลัพธ์เครื่องมือหลังจุดนั้น)
- ปกป้องส่วน bootstrap prefix (จะไม่ตัดสิ่งใดก่อนข้อความผู้ใช้แรก)
- โหมด:
  - `adaptive`: ตัดแบบนุ่ม (soft-trim) ผลลัพธ์เครื่องมือที่ใหญ่เกินไป (เก็บหัว/ท้าย) เมื่ออัตราส่วนบริบทโดยประมาณเกิน `softTrimRatio`
    จากนั้นจะล้างแบบแข็ง (hard-clear) ผลลัพธ์เครื่องมือที่เก่าที่สุดซึ่งเข้าเกณฑ์ เมื่ออัตราส่วนบริบทโดยประมาณเกิน `hardClearRatio` **และ**
    มีปริมาณผลลัพธ์เครื่องมือที่ตัดได้เพียงพอ (`minPrunableToolChars`)
  - `aggressive`: แทนที่ผลลัพธ์เครื่องมือที่เข้าเกณฑ์ทั้งหมดก่อนจุดตัดด้วย `hardClear.placeholder` เสมอ (ไม่ตรวจอัตราส่วน)

การตัดแบบนุ่ม vs แบบแข็ง (สิ่งที่เปลี่ยนในบริบทที่ส่งไปยัง LLM):

- **Soft-trim**: ใช้เฉพาะกับผลลัพธ์เครื่องมือที่ _ใหญ่เกินไป_ เก็บส่วนต้น + ส่วนท้าย และแทรก `...` ตรงกลาง
  - ก่อน: `toolResult("…very long output…")`
  - หลัง: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
- **Hard-clear**: แทนที่ผลลัพธ์เครื่องมือทั้งก้อนด้วย placeholder
  - ก่อน: `toolResult("…very long output…")`
  - หลัง: `toolResult("[Old tool result content cleared]")`

หมายเหตุ / ข้อจำกัดปัจจุบัน:

- ผลลัพธ์เครื่องมือที่มี **บล็อกภาพ** จะถูกข้าม (จะไม่ถูกตัด/ล้าง) ในขณะนี้
- การประเมิน “อัตราส่วนบริบท” อิงจาก **จำนวนตัวอักษร** (โดยประมาณ) ไม่ใช่จำนวนโทเค็นที่แน่นอน
- If the session doesn’t contain at least `keepLastAssistants` assistant messages yet, pruning is skipped.
- In `aggressive` mode, `hardClear.enabled` is ignored (eligible tool results are always replaced with `hardClear.placeholder`).

Default (adaptive):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

To disable:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

Defaults (when `mode` is `"adaptive"` or `"aggressive"`):

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (adaptive only)
- `hardClearRatio`: `0.5` (adaptive only)
- `minPrunableToolChars`: `50000` (adaptive only)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (adaptive only)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

Example (aggressive, minimal):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

Example (adaptive tuned):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Optional: restrict pruning to specific tools (deny wins; supports "*" wildcards)
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

See [/concepts/session-pruning](/concepts/session-pruning) for behavior details.

#### `agents.defaults.compaction` (reserve headroom + memory flush)

`agents.defaults.compaction.mode` selects the compaction summarization strategy. Defaults to `default`; set `safeguard` to enable chunked summarization for very long histories. See [/concepts/compaction](/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` enforces a minimum `reserveTokens`
value for Pi compaction (default: `20000`). Set it to `0` to disable the floor.

`agents.defaults.compaction.memoryFlush` runs a **silent** agentic turn before
auto-compaction, instructing the model to store durable memories on disk (e.g.
`memory/YYYY-MM-DD.md`). It triggers when the session token estimate crosses a
soft threshold below the compaction limit.

Legacy defaults:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: built-in defaults with `NO_REPLY`
- Note: memory flush is skipped when the session workspace is read-only
  (`agents.defaults.sandbox.workspaceAccess: "ro"` or `"none"`).

Example (tuned):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

Block streaming:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (ปิดเป็นค่าเริ่มต้น)

- Channel overrides: `*.blockStreaming` (and per-account variants) to force block streaming on/off.
  Non-Telegram channels require an explicit `*.blockStreaming: true` to enable block replies.

- `agents.defaults.blockStreamingBreak`: `"text_end"` or `"message_end"` (default: text_end).

- `agents.defaults.blockStreamingChunk`: soft chunking for streamed blocks. Defaults to
  800–1200 chars, prefers paragraph breaks (`\n\n`), then newlines, then sentences.
  ตัวอย่าง:

  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: merge streamed blocks before sending.
  Defaults to `{ idleMs: 1000 }` and inherits `minChars` from `blockStreamingChunk`
  with `maxChars` capped to the channel text limit. Signal/Slack/Discord/Google Chat default
  to `minChars: 1500` unless overridden.
  Channel overrides: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (and per-account variants).

- `agents.defaults.humanDelay`: randomized pause between **block replies** after the first.
  Modes: `off` (default), `natural` (800–2500ms), `custom` (use `minMs`/`maxMs`).
  Per-agent override: `agents.list[].humanDelay`.
  ตัวอย่าง:

  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } },
  }
  ```

  See [/concepts/streaming](/concepts/streaming) for behavior + chunking details.

Typing indicators:

- `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`. Defaults to
  `instant` for direct chats / mentions and `message` for unmentioned group chats.
- `session.typingMode`: per-session override for the mode.
- `agents.defaults.typingIntervalSeconds`: how often the typing signal is refreshed (default: 6s).
- `session.typingIntervalSeconds`: per-session override for the refresh interval.
  See [/concepts/typing-indicators](/concepts/typing-indicators) for behavior details.

`agents.defaults.model.primary` should be set as `provider/model` (e.g. `anthropic/claude-opus-4-6`).
Aliases come from `agents.defaults.models.*.alias` (e.g. `Opus`).
If you omit the provider, OpenClaw currently assumes `anthropic` as a temporary
deprecation fallback.
Z.AI models are available as `zai/<model>` (e.g. `zai/glm-4.7`) and require
`ZAI_API_KEY` (or legacy `Z_AI_API_KEY`) in the environment.

`agents.defaults.heartbeat` configures periodic heartbeat runs:

- `every`: duration string (`ms`, `s`, `m`, `h`); default unit minutes. Default:
  `30m`. Set `0m` to disable.
- `model`: optional override model for heartbeat runs (`provider/model`).
- `includeReasoning`: when `true`, heartbeats will also deliver the separate `Reasoning:` message when available (same shape as `/reasoning on`). Default: `false`.
- `session`: optional session key to control which session the heartbeat runs in. Default: `main`.
- `to`: optional recipient override (channel-specific id, e.g. E.164 for WhatsApp, chat id for Telegram).
- `target`: optional delivery channel (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Default: `last`.
- `prompt`: optional override for the heartbeat body (default: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Overrides are sent verbatim; include a `Read HEARTBEAT.md` line if you still want the file read.
- `ackMaxChars`: max chars allowed after `HEARTBEAT_OK` before delivery (default: 300).

Per-agent heartbeats:

- Set `agents.list[].heartbeat` to enable or override heartbeat settings for a specific agent.
- If any agent entry defines `heartbeat`, **only those agents** run heartbeats; defaults
  become the shared baseline for those agents.

Heartbeats run full agent turns. Shorter intervals burn more tokens; be mindful
of `every`, keep `HEARTBEAT.md` tiny, and/or choose a cheaper `model`.

`tools.exec` configures background exec defaults:

- `backgroundMs`: time before auto-background (ms, default 10000)
- `timeoutSec`: auto-kill after this runtime (seconds, default 1800)
- `cleanupMs`: how long to keep finished sessions in memory (ms, default 1800000)
- `notifyOnExit`: enqueue a system event + request heartbeat when backgrounded exec exits (default true)
- `applyPatch.enabled`: enable experimental `apply_patch` (OpenAI/OpenAI Codex only; default false)
- `applyPatch.allowModels`: optional allowlist of model ids (e.g. `gpt-5.2` or `openai/gpt-5.2`)
  Note: `applyPatch` is only under `tools.exec`.

`tools.web` configures web search + fetch tools:

- `tools.web.search.enabled` (default: true when key is present)
- `tools.web.search.apiKey` (recommended: set via `openclaw configure --section web`, or use `BRAVE_API_KEY` env var)
- `tools.web.search.maxResults` (1–10, default 5)
- `tools.web.search.timeoutSeconds` (ค่าเริ่มต้น 30)
- `tools.web.search.cacheTtlMinutes` (ค่าเริ่มต้น 15)
- `tools.web.fetch.enabled` (default true)
- `tools.web.fetch.maxChars` (ค่าเริ่มต้น 50000)
- `tools.web.fetch.maxCharsCap` (default 50000; clamps maxChars from config/tool calls)
- `tools.web.fetch.timeoutSeconds` (ค่าเริ่มต้น 30)
- `tools.web.fetch.cacheTtlMinutes` (ค่าเริ่มต้น 15)
- `tools.web.fetch.userAgent`(การแทนที่ไม่บังคับ)
- `tools.web.fetch.readability` (default true; disable to use basic HTML cleanup only)
- `tools.web.fetch.firecrawl.enabled` (default true when an API key is set)
- `tools.web.fetch.firecrawl.apiKey` (optional; defaults to `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (default [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (default true)
- `tools.web.fetch.firecrawl.maxAgeMs` (ไม่บังคับ)
- `tools.web.fetch.firecrawl.timeoutSeconds` (ไม่บังคับ)

`tools.media` configures inbound media understanding (image/audio/video):

- `tools.media.models`: shared model list (capability-tagged; used after per-cap lists).
- `tools.media.concurrency`: max concurrent capability runs (default 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `enabled`: opt-out switch (default true when models are configured).
  - `prompt`: optional prompt override (image/video append a `maxChars` hint automatically).
  - `maxChars`: max output characters (default 500 for image/video; unset for audio).
  - `maxBytes`: max media size to send (defaults: image 10MB, audio 20MB, video 50MB).
  - `timeoutSeconds`: request timeout (defaults: image 60s, audio 60s, video 120s).
  - `language`: optional audio hint.
  - `attachments`: attachment policy (`mode`, `maxAttachments`, `prefer`).
  - `scope`: optional gating (first match wins) with `match.channel`, `match.chatType`, or `match.keyPrefix`.
  - `models`: ordered list of model entries; failures or oversize media fall back to the next entry.
- Each `models[]` entry:
  - Provider entry (`type: "provider"` or omitted):
    - `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc).
    - `model`: model id override (required for image; defaults to `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` for audio providers, and `gemini-3-flash-preview` for video).
    - `profile` / `preferredProfile`: auth profile selection.
  - CLI entry (`type: "cli"`):
    - `command`: executable to run.
    - `args`: templated args (supports `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc).
  - `capabilities`: optional list (`image`, `audio`, `video`) to gate a shared entry. Defaults when omitted: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` can be overridden per entry.

If no models are configured (or `enabled: false`), understanding is skipped; the model still receives the original attachments.

Provider auth follows the standard model auth order (auth profiles, env vars like `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, or `models.providers.*.apiKey`).

ตัวอย่าง:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

`agents.defaults.subagents` configures sub-agent defaults:

- `model`: default model for spawned sub-agents (string or `{ primary, fallbacks }`). If omitted, sub-agents inherit the caller’s model unless overridden per agent or per call.
- `maxConcurrent`: max concurrent sub-agent runs (default 1)
- `archiveAfterMinutes`: auto-archive sub-agent sessions after N minutes (default 60; set `0` to disable)
- Per-subagent tool policy: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (deny wins)

`tools.profile` sets a **base tool allowlist** before `tools.allow`/`tools.deny`:

- `minimal`: `session_status` เท่านั้น
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: ไม่มีข้อจำกัด(เหมือนกับไม่ได้ตั้งค่า)

Per-agent override: `agents.list[].tools.profile`.

ตัวอย่าง(ค่าเริ่มต้นเฉพาะการส่งข้อความ และอนุญาตเครื่องมือ Slack + Discord เพิ่ม):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

ตัวอย่าง(โปรไฟล์การเขียนโค้ด แต่ปฏิเสธ exec/process ทุกที่):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` lets you **further restrict** tools for specific providers (or a single `provider/model`).
Per-agent override: `agents.list[].tools.byProvider`.

Order: base profile → provider profile → allow/deny policies.
Provider keys accept either `provider` (e.g. `google-antigravity`) or `provider/model`
(e.g. `openai/gpt-5.2`).

ตัวอย่าง(คงโปรไฟล์การเขียนโค้ดแบบส่วนกลาง แต่ใช้เครื่องมือขั้นต่ำสำหรับ Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Example (provider/model-specific allowlist):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

`tools.allow` / `tools.deny` configure a global tool allow/deny policy (deny wins).
Matching is case-insensitive and supports `*` wildcards (`"*"` means all tools).
This is applied even when the Docker sandbox is **off**.

Example (disable browser/canvas everywhere):

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

Tool groups (shorthands) work in **global** and **per-agent** tool policies:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: เครื่องมือ OpenClaw แบบบิลต์อินทั้งหมด(ไม่รวมปลั๊กอินผู้ให้บริการ)

`tools.elevated` controls elevated (host) exec access:

- `enabled`: allow elevated mode (default true)
- `allowFrom`: per-channel allowlists (empty = disabled)
  - `whatsapp`: E.164 numbers
  - `telegram`: chat ids or usernames
  - `discord`: user ids or usernames (falls back to `channels.discord.dm.allowFrom` if omitted)
  - `signal`: E.164 numbers
  - `imessage`: handles/chat ids
  - `webchat`: session ids or usernames

ตัวอย่าง:

```json5
1. {
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

2. การ override ต่อเอเจนต์ (จำกัดเพิ่มเติม):

```json5
3. {
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false },
        },
      },
    ],
  },
}
```

หมายเหตุ:

- 4. `tools.elevated` คือค่า baseline แบบ global 5. `agents.list[].tools.elevated` สามารถจำกัดเพิ่มเติมได้เท่านั้น (ทั้งสองต้องอนุญาต)
- 6. `/elevated on|off|ask|full` จะบันทึกสถานะต่อคีย์เซสชัน; inline directives มีผลกับข้อความเดียว
- 7. Elevated `exec` รันบนโฮสต์และข้ามการทำ sandboxing
- 8. นโยบายเครื่องมือยังคงมีผล; หาก `exec` ถูกปฏิเสธ จะไม่สามารถใช้ elevated ได้

9. `agents.defaults.maxConcurrent` กำหนดจำนวนสูงสุดของการรันเอเจนต์แบบฝังที่สามารถ
   รันพร้อมกันข้ามเซสชันได้ 10. แต่ละเซสชันยังคงถูกรันแบบลำดับ (หนึ่งการรัน
   ต่อคีย์เซสชันในแต่ละครั้ง) 11. ค่าเริ่มต้น: 1

### 12. `agents.defaults.sandbox`

13. **Docker sandboxing** แบบเลือกได้สำหรับเอเจนต์แบบฝัง 14. ออกแบบมาสำหรับเซสชันที่ไม่ใช่ main
    เพื่อไม่ให้เข้าถึงระบบโฮสต์ของคุณ

15. รายละเอียด: [Sandboxing](/gateway/sandboxing)

16. ค่าเริ่มต้น (เมื่อเปิดใช้งาน):

- 17. scope: `"agent"` (หนึ่งคอนเทนเนอร์ + เวิร์กสเปซต่อเอเจนต์)
- 18. อิมเมจพื้นฐาน Debian bookworm-slim
- 19. การเข้าถึงเวิร์กสเปซของเอเจนต์: `workspaceAccess: "none"` (ค่าเริ่มต้น)
  - `"none"`: use a per-scope sandbox workspace under `~/.openclaw/sandboxes`
- 21. `"ro"`: คงเวิร์กสเปซ sandbox ไว้ที่ `/workspace` และเมานต์เวิร์กสเปซของเอเจนต์แบบอ่านอย่างเดียวที่ `/agent` (ปิดการใช้งาน `write`/`edit`/`apply_patch`)
  - 22. `"rw"`: เมานต์เวิร์กสเปซของเอเจนต์แบบอ่าน/เขียนที่ `/workspace`
- ลบอัตโนมัติ: idle > 24 ชม. หรืออายุ > 7 วัน
- 23. นโยบายเครื่องมือ: อนุญาตเฉพาะ `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (deny ชนะ)
  - 24. กำหนดค่าผ่าน `tools.sandbox.tools`, override ต่อเอเจนต์ผ่าน `agents.list[].tools.sandbox.tools`
  - 25. รองรับ shorthand กลุ่มเครื่องมือในนโยบาย sandbox: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (ดู [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- 26. เบราว์เซอร์แบบ sandboxed (Chromium + CDP, ตัวดู noVNC) เป็นตัวเลือก
- 27. ตัวเลือกเสริมความแข็งแกร่ง: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

28. คำเตือน: `scope: "shared"` หมายถึงคอนเทนเนอร์และเวิร์กสเปซที่ใช้ร่วมกัน 29. ไม่มี
    การแยกข้ามเซสชัน 30. ใช้ `scope: "session"` เพื่อการแยกต่อเซสชัน

31. แบบเดิม: `perSession` ยังรองรับ (`true` → `scope: "session"`,
    `false` → `scope: "shared"`)

32. `setupCommand` จะรัน **ครั้งเดียว** หลังจากสร้างคอนเทนเนอร์ (ภายในคอนเทนเนอร์ผ่าน `sh -lc`)
33. สำหรับการติดตั้งแพ็กเกจ ตรวจสอบให้มี network egress, root FS ที่เขียนได้ และผู้ใช้ root

```json5
34. {
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Per-agent override (multi-agent): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7, // 0 disables max-age pruning
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

35. สร้างอิมเมจ sandbox ค่าเริ่มต้นหนึ่งครั้งด้วย:

```bash
scripts/sandbox-setup.sh
```

36. หมายเหตุ: คอนเทนเนอร์ sandbox มีค่าเริ่มต้นเป็น `network: "none"`; ตั้งค่า `agents.defaults.sandbox.docker.network`
    เป็น `"bridge"` (หรือเครือข่ายกำหนดเอง) หากเอเจนต์ต้องการการเชื่อมต่อขาออก

37. หมายเหตุ: ไฟล์แนบขาเข้าจะถูกจัดเตรียมไว้ในเวิร์กสเปซที่ใช้งานอยู่ที่ `media/inbound/*` 38. เมื่อใช้ `workspaceAccess: "rw"` หมายความว่าไฟล์จะถูกเขียนลงในเวิร์กสเปซของเอเจนต์

39. หมายเหตุ: `docker.binds` จะเมานต์ไดเรกทอรีโฮสต์เพิ่มเติม; การ bind แบบ global และต่อเอเจนต์จะถูกรวมกัน

40. สร้างอิมเมจเบราว์เซอร์เสริมด้วย:

```bash
scripts/sandbox-browser-setup.sh
```

41. เมื่อ `agents.defaults.sandbox.browser.enabled=true` เครื่องมือเบราว์เซอร์จะใช้
    Chromium แบบ sandboxed (CDP) 42. หากเปิดใช้ noVNC (ค่าเริ่มต้นเมื่อ headless=false),
    URL ของ noVNC จะถูกแทรกเข้าไปใน system prompt เพื่อให้เอเจนต์อ้างอิงได้
42. สิ่งนี้ไม่จำเป็นต้องตั้งค่า `browser.enabled` ในคอนฟิกหลัก; URL ควบคุม sandbox
    จะถูกแทรกต่อเซสชัน

44. `agents.defaults.sandbox.browser.allowHostControl` (ค่าเริ่มต้น: false) อนุญาตให้
    เซสชันที่ถูก sandbox ระบุเป้าหมายไปยังเซิร์ฟเวอร์ควบคุมเบราว์เซอร์ของ **โฮสต์**
    ผ่านเครื่องมือเบราว์เซอร์ (`target: "host"`) 45. ปิดตัวเลือกนี้ไว้หากต้องการ
    การแยก sandbox อย่างเคร่งครัด

46. รายการอนุญาต (allowlist) สำหรับการควบคุมระยะไกล:

- 47. `allowedControlUrls`: URL ควบคุมที่อนุญาตแบบตรงตัวสำหรับ `target: "custom"`
- 48. `allowedControlHosts`: ชื่อโฮสต์ที่อนุญาต (เฉพาะ hostname ไม่มีพอร์ต)
- 49. `allowedControlPorts`: พอร์ตที่อนุญาต (ค่าเริ่มต้น: http=80, https=443)
  50. ค่าเริ่มต้น: allowlist ทั้งหมดไม่ได้ตั้งค่า (ไม่มีข้อจำกัด) `allowHostControl` defaults to false.

### `models` (custom providers + base URLs)

OpenClaw uses the **pi-coding-agent** model catalog. You can add custom providers
(LiteLLM, local OpenAI-compatible servers, Anthropic proxies, etc.) by writing
`~/.openclaw/agents/<agentId>/agent/models.json` or by defining the same schema inside your
OpenClaw config under `models.providers`.
Provider-by-provider overview + examples: [/concepts/model-providers](/concepts/model-providers).

When `models.providers` is present, OpenClaw writes/merges a `models.json` into
`~/.openclaw/agents/<agentId>/agent/` on startup:

- default behavior: **merge** (keeps existing providers, overrides on name)
- set `models.mode: "replace"` to overwrite the file contents

Select the model via `agents.defaults.model.primary` (provider/model).

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {},
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

### OpenCode Zen (multi-model proxy)

OpenCode Zen is a multi-model gateway with per-model endpoints. OpenClaw uses
the built-in `opencode` provider from pi-ai; set `OPENCODE_API_KEY` (or
`OPENCODE_ZEN_API_KEY`) from [https://opencode.ai/auth](https://opencode.ai/auth).

หมายเหตุ:

- Model refs use `opencode/<modelId>` (example: `opencode/claude-opus-4-6`).
- If you enable an allowlist via `agents.defaults.models`, add each model you plan to use.
- Shortcut: `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

### Z.AI (GLM-4.7) — provider alias support

Z.AI models are available via the built-in `zai` provider. Set `ZAI_API_KEY`
in your environment and reference the model by provider/model.

Shortcut: `openclaw onboard --auth-choice zai-api-key`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

หมายเหตุ:

- `z.ai/*` and `z-ai/*` are accepted aliases and normalize to `zai/*`.
- If `ZAI_API_KEY` is missing, requests to `zai/*` will fail with an auth error at runtime.
- Example error: `No API key found for provider "zai".`
- Z.AI’s general API endpoint is `https://api.z.ai/api/paas/v4`. GLM coding
  requests use the dedicated Coding endpoint `https://api.z.ai/api/coding/paas/v4`.
  The built-in `zai` provider uses the Coding endpoint. If you need the general
  endpoint, define a custom provider in `models.providers` with the base URL
  override (see the custom providers section above).
- Use a fake placeholder in docs/configs; never commit real API keys.

### Moonshot AI (Kimi)

Use Moonshot's OpenAI-compatible endpoint:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

หมายเหตุ:

- Set `MOONSHOT_API_KEY` in the environment or use `openclaw onboard --auth-choice moonshot-api-key`.
- Model ref: `moonshot/kimi-k2.5`.
- For the China endpoint, either:
  - Run `openclaw onboard --auth-choice moonshot-api-key-cn` (wizard will set `https://api.moonshot.cn/v1`), or
  - Manually set `baseUrl: "https://api.moonshot.cn/v1"` in `models.providers.moonshot`.

### Kimi Coding

Use Moonshot AI's Kimi Coding endpoint (Anthropic-compatible, built-in provider):

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

หมายเหตุ:

- Set `KIMI_API_KEY` in the environment or use `openclaw onboard --auth-choice kimi-code-api-key`.
- Model ref: `kimi-coding/k2p5`.

### Synthetic (Anthropic-compatible)

Use Synthetic's Anthropic-compatible endpoint:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

หมายเหตุ:

- Set `SYNTHETIC_API_KEY` or use `openclaw onboard --auth-choice synthetic-api-key`.
- Model ref: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
- Base URL should omit `/v1` because the Anthropic client appends it.

### Local models (LM Studio) — recommended setup

See [/gateway/local-models](/gateway/local-models) for the current local guidance. TL;DR: run MiniMax M2.1 via LM Studio Responses API on serious hardware; keep hosted models merged for fallback.

### MiniMax M2.1

1. ใช้ MiniMax M2.1 โดยตรงโดยไม่ต้องใช้ LM Studio:

```json5
2. {
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Pricing: update in models.json if you need exact cost tracking.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

หมายเหตุ:

- 3. ตั้งค่าตัวแปรสภาพแวดล้อม `MINIMAX_API_KEY` หรือใช้ `openclaw onboard --auth-choice minimax-api`
- 4. โมเดลที่พร้อมใช้งาน: `MiniMax-M2.1` (ค่าเริ่มต้น)
- 5. อัปเดตราคาใน `models.json` หากคุณต้องการติดตามต้นทุนที่แม่นยำ

### 6. Cerebras (GLM 4.6 / 4.7)

7. ใช้ Cerebras ผ่านเอ็นด์พอยต์ที่เข้ากันได้กับ OpenAI:

```json5
8. {
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

หมายเหตุ:

- 9. ใช้ `cerebras/zai-glm-4.7` สำหรับ Cerebras; ใช้ `zai/glm-4.7` สำหรับ Z.AI โดยตรง
- 10. ตั้งค่า `CEREBRAS_API_KEY` ในสภาพแวดล้อมหรือไฟล์คอนฟิก

หมายเหตุ:

- 11. API ที่รองรับ: `openai-completions`, `openai-responses`, `anthropic-messages`,
      `google-generative-ai`
- 12. ใช้ `authHeader: true` ร่วมกับ `headers` สำหรับความต้องการการยืนยันตัวตนแบบกำหนดเอง
- 13. เขียนทับรากคอนฟิกของเอเจนต์ด้วย `OPENCLAW_AGENT_DIR` (หรือ `PI_CODING_AGENT_DIR`)
      หากคุณต้องการจัดเก็บ `models.json` ไว้ที่อื่น (ค่าเริ่มต้น: `~/.openclaw/agents/main/agent`).

### `session`

14. ควบคุมขอบเขตเซสชัน นโยบายการรีเซ็ต ตัวกระตุ้นการรีเซ็ต และตำแหน่งที่บันทึกที่เก็บเซสชัน

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    // ค่าเริ่มต้นเป็น per-agent อยู่แล้วภายใต้ ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // คุณสามารถแทนที่ด้วยการใช้เทมเพลต {agentId}:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // แชตแบบตรงจะถูกรวมเป็น agent:<agentId>:<mainKey> (ค่าเริ่มต้น: "main").
    mainKey: "main",
    agentToAgent: {
      // จำนวนรอบการตอบกลับไปมาสูงสุดระหว่างผู้ร้องขอ/เป้าหมาย (0–5)
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

ฟิลด์:

- 16. `mainKey`: คีย์บัคเก็ตของแชตโดยตรง (ค่าเริ่มต้น: `"main"`). 17. มีประโยชน์เมื่อคุณต้องการ “เปลี่ยนชื่อ” เธรด DM หลักโดยไม่ต้องเปลี่ยน `agentId`
  - 18. หมายเหตุเกี่ยวกับแซนด์บ็อกซ์: `agents.defaults.sandbox.mode: "non-main"` ใช้คีย์นี้เพื่อตรวจจับเซสชันหลัก 19. คีย์เซสชันใด ๆ ที่ไม่ตรงกับ `mainKey` (กลุ่ม/แชนเนล) จะถูกจัดเป็นแซนด์บ็อกซ์
- `dmScope`: how DM sessions are grouped (default: `"main"`).
  - 21. `main`: DM ทั้งหมดใช้เซสชันหลักร่วมกันเพื่อความต่อเนื่อง
  - 22. `per-peer`: แยก DM ตามรหัสผู้ส่งข้ามทุกแชนเนล
  - 23. `per-channel-peer`: แยก DM ต่อแชนเนล + ผู้ส่ง (แนะนำสำหรับกล่องข้อความหลายผู้ใช้)
  - 24. `per-account-channel-peer`: แยก DM ต่อบัญชี + แชนเนล + ผู้ส่ง (แนะนำสำหรับกล่องข้อความหลายบัญชี)
  - 25. โหมด DM ที่ปลอดภัย (แนะนำ): ตั้งค่า `session.dmScope: "per-channel-peer"` เมื่อมีหลายคนสามารถ DM บอทได้ (กล่องข้อความที่ใช้ร่วมกัน, allowlist หลายคน, หรือ `dmPolicy: "open"`).
- 26. `identityLinks`: จับคู่รหัสมาตรฐานกับ peer ที่มีคำนำหน้าผู้ให้บริการ เพื่อให้บุคคลเดียวกันใช้เซสชัน DM ร่วมกันข้ามแชนเนล เมื่อใช้ `per-peer`, `per-channel-peer` หรือ `per-account-channel-peer`
  - 27. ตัวอย่าง: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
- 28. `reset`: นโยบายการรีเซ็ตหลัก 29. ค่าเริ่มต้นคือรีเซ็ตรายวันเวลา 4:00 น. ตามเวลาท้องถิ่นของโฮสต์เกตเวย์
  - 30. `mode`: `daily` หรือ `idle` (ค่าเริ่มต้น: `daily` เมื่อมี `reset`).
  - 31. `atHour`: ชั่วโมงท้องถิ่น (0–23) สำหรับขอบเขตการรีเซ็ตรายวัน
  - 32. `idleMinutes`: หน้าต่างเวลาว่างแบบเลื่อน หน่วยเป็นนาที 33. เมื่อกำหนดทั้ง daily + idle ค่าใดหมดอายุก่อนจะถูกใช้
- `resetByType`: การแทนที่ระดับเซสชันสำหรับ `direct`, `group` และ `thread`. คีย์ `dm` แบบเดิม (Legacy) สามารถใช้เป็นชื่อแทนของ `direct` ได้
  - 35. หากคุณตั้งค่าเฉพาะ `session.idleMinutes` แบบเดิมโดยไม่มี `reset`/`resetByType`, OpenClaw จะคงโหมด idle-only เพื่อความเข้ากันได้ย้อนหลัง
- 36. `heartbeatIdleMinutes`: การเขียนทับ idle แบบเลือกได้สำหรับการตรวจ heartbeat (การรีเซ็ตรายวันยังคงมีผลเมื่อเปิดใช้งาน)
- 37. `agentToAgent.maxPingPongTurns`: จำนวนรอบการตอบกลับไปมา ระหว่างผู้ร้องขอ/เป้าหมาย สูงสุด (0–5, ค่าเริ่มต้น 5)
- 38. `sendPolicy.default`: ค่า fallback `allow` หรือ `deny` เมื่อไม่มีกฎใดตรง
- 39. `sendPolicy.rules[]`: จับคู่ตาม `channel`, `chatType` (`direct|group|room`) หรือ `keyPrefix` (เช่น `cron:`) 40. หากมี deny ก่อน ให้ถือว่า deny มิฉะนั้นอนุญาต

### 41. `skills` (การตั้งค่า skills)

42. ควบคุม allowlist แบบรวม การตั้งค่าการติดตั้ง โฟลเดอร์ skills เพิ่มเติม และการเขียนทับราย skill 43. ใช้กับ skills แบบ **bundled** และ `~/.openclaw/skills` (skills ในเวิร์กสเปซจะมีลำดับความสำคัญเหนือกว่าเมื่อชื่อซ้ำกัน)

ฟิลด์:

- 44. `allowBundled`: allowlist แบบเลือกได้สำหรับ skills แบบ **bundled** เท่านั้น 45. หากตั้งค่าไว้ จะมีสิทธิ์เฉพาะ bundled skills ที่ระบุ (managed/workspace skills ไม่ได้รับผลกระทบ)
- `load.extraDirs`: ไดเรกทอรีSkillsเพิ่มเติมที่จะสแกน (ลำดับความสำคัญต่ำสุด)
- `install.preferBrew`: เลือกใช้ตัวติดตั้งผ่าน brew เมื่อมีให้ใช้ (ค่าเริ่มต้น: true)
- 46. `install.nodeManager`: ตัวเลือกตัวจัดการติดตั้ง node (`npm` | `pnpm` | `yarn`, ค่าเริ่มต้น: npm)
- `entries.<skillKey>47. `:\` การเขียนทับคอนฟิกต่อ skill

ฟิลด์ต่อSkill:

- `enabled`: ตั้งค่า `false` เพื่อปิดการใช้งานSkill แม้ว่าจะถูกรวมมาหรือติดตั้งแล้ว
- `env`: ตัวแปรสภาพแวดล้อมที่ฉีดให้กับการรันเอเจนต์ (เฉพาะกรณีที่ยังไม่ได้ตั้งค่า)
- 48. `apiKey`: ตัวช่วยแบบเลือกได้สำหรับ skills ที่ประกาศตัวแปรสภาพแวดล้อมหลัก (เช่น `nano-banana-pro` → `GEMINI_API_KEY`)

ตัวอย่าง:

```json5
49. {
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### 50. `plugins` (ส่วนขยาย)

1. ควบคุมการค้นพบปลั๊กอิน การอนุญาต/ปฏิเสธ และการตั้งค่ารายปลั๊กอิน 2. ปลั๊กอินจะถูกโหลด
   จาก `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` รวมถึงรายการใด ๆ ใน
   `plugins.load.paths` 3. **การเปลี่ยนแปลงการตั้งค่าต้องรีสตาร์ทเกตเวย์**
   ดู [/plugin](/tools/plugin) สำหรับการใช้งานทั้งหมด

ฟิลด์:

- 4. `enabled`: สวิตช์หลักสำหรับการโหลดปลั๊กอิน (ค่าเริ่มต้น: true)
- 5. `allow`: รายการอนุญาตของรหัสปลั๊กอิน (ไม่บังคับ); เมื่อกำหนดแล้ว จะโหลดเฉพาะปลั๊กอินที่อยู่ในรายการ
- 6. `deny`: รายการปฏิเสธของรหัสปลั๊กอิน (ไม่บังคับ) (การปฏิเสธมีสิทธิ์เหนือกว่า)
- 7. `load.paths`: ไฟล์หรือไดเรกทอรีปลั๊กอินเพิ่มเติมที่จะโหลด (แบบ absolute หรือ `~`)
- 8. `entries.<pluginId>`9. \`: การตั้งค่า override รายปลั๊กอิน
  - 10. `enabled`: ตั้งค่า `false` เพื่อปิดการใช้งาน
  - 11. `config`: อ็อบเจ็กต์การตั้งค่าเฉพาะปลั๊กอิน (ปลั๊กอินจะตรวจสอบความถูกต้องหากมีให้)

ตัวอย่าง:

```json5
12. {
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio",
        },
      },
    },
  },
}
```

### 13. `browser` (เบราว์เซอร์ที่ OpenClaw จัดการ)

14. OpenClaw สามารถเริ่มอินสแตนซ์ Chrome/Brave/Edge/Chromium แบบ **เฉพาะและแยกส่วน** สำหรับ openclaw และเปิดบริการควบคุมแบบ loopback ขนาดเล็ก
15. โปรไฟล์สามารถชี้ไปยังเบราว์เซอร์ที่ใช้ Chromium แบบ **ระยะไกล** ผ่าน `profiles.<name>`16. `.cdpUrl` 17. โปรไฟล์ระยะไกลเป็นแบบแนบอย่างเดียว (ปิดการเริ่ม/หยุด/รีเซ็ต)

18. `browser.cdpUrl` ยังคงมีไว้สำหรับการตั้งค่าโปรไฟล์เดียวแบบเดิม และใช้เป็น scheme/host พื้นฐานสำหรับโปรไฟล์ที่ตั้งค่าเฉพาะ `cdpPort`

ค่าเริ่มต้น:

- 19. enabled: `true`
- 20. evaluateEnabled: `true` (ตั้งค่า `false` เพื่อปิด `act:evaluate` และ `wait --fn`)
- 21. บริการควบคุม: เฉพาะ loopback (พอร์ตคำนวณจาก `gateway.port`, ค่าเริ่มต้น `18791`)
- 22. URL ของ CDP: `http://127.0.0.1:18792` (บริการควบคุม + 1, โปรไฟล์เดียวแบบเดิม)
- 23. สีโปรไฟล์: `#FF4500` (lobster-orange)
- 24. หมายเหตุ: เซิร์ฟเวอร์ควบคุมจะถูกเริ่มโดยเกตเวย์ที่กำลังทำงาน (เมนูบาร์ OpenClaw.app หรือ `openclaw gateway`)
- 25. ลำดับการตรวจจับอัตโนมัติ: เบราว์เซอร์เริ่มต้นหากเป็น Chromium-based; มิฉะนั้น Chrome → Brave → Edge → Chromium → Chrome Canary

```json5
26. {
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // override โปรไฟล์เดียวแบบเดิม
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // ขั้นสูง:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // ตั้งค่า true เมื่อทำ tunneling CDP ระยะไกลไปยัง localhost
  },
}
```

### 27. `ui` (ลักษณะการแสดงผล)

28. สีเน้น (accent) ทางเลือกที่ใช้โดยแอปเนทีฟสำหรับส่วน UI (เช่น สีของบับเบิลโหมดพูดคุย)

29. หากไม่ตั้งค่า ไคลเอนต์จะใช้สีน้ำเงินอ่อนแบบหม่นเป็นค่าเริ่มต้น

```json5
30. {
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB หรือ #RRGGBB)
    // ตัวเลือก: override ตัวตนผู้ช่วยของ Control UI
    // หากไม่ตั้งค่า Control UI จะใช้ตัวตนของเอเจนต์ที่กำลังใช้งาน (config หรือ IDENTITY.md)
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // อีโมจิ ข้อความสั้น หรือ URL/URI ของรูปภาพ
    },
  },
}
```

### 31. `gateway` (โหมดเซิร์ฟเวอร์เกตเวย์ + การ bind)

32. ใช้ `gateway.mode` เพื่อประกาศอย่างชัดเจนว่าเครื่องนี้ควรรันเกตเวย์หรือไม่

ค่าเริ่มต้น:

- 33. mode: **ไม่ตั้งค่า** (ถือว่า “ไม่เริ่มอัตโนมัติ”)
- 34. bind: `loopback`
- 35. port: `18789` (พอร์ตเดียวสำหรับ WS + HTTP)

```json5
36. {
  gateway: {
    mode: "local", // หรือ "remote"
    port: 18789, // รวม WS + HTTP
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // ใช้โทเค็นควบคุมการเข้าถึง WS + Control UI
    // tailscale: { mode: "off" | "serve" | "funnel" }
  },
}
```

37. พาธฐานของ Control UI:

- 38. `gateway.controlUi.basePath` กำหนดคำนำหน้า URL ที่ใช้เสิร์ฟ Control UI
- 39. ตัวอย่าง: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`
- 40. ค่าเริ่มต้น: ราก (`/`) (ไม่เปลี่ยนแปลง)
- 41. `gateway.controlUi.root` กำหนดรากของไฟล์ระบบสำหรับแอสเซ็ต Control UI (ค่าเริ่มต้น: `dist/control-ui`)
- 42. `gateway.controlUi.allowInsecureAuth` อนุญาตการยืนยันตัวตนด้วยโทเค็นอย่างเดียวสำหรับ Control UI เมื่อไม่ระบุตัวตนอุปกรณ์ (โดยทั่วไปผ่าน HTTP) 43. ค่าเริ่มต้น: `false` 44. แนะนำให้ใช้ HTTPS
      (Tailscale Serve) หรือ `127.0.0.1`
- 45. `gateway.controlUi.dangerouslyDisableDeviceAuth` ปิดการตรวจสอบตัวตนอุปกรณ์สำหรับ Control UI (ใช้เฉพาะโทเค็น/รหัสผ่าน) 46. ค่าเริ่มต้น: `false` 47. ใช้เฉพาะกรณีฉุกเฉิน (Break-glass)

เอกสารที่เกี่ยวข้อง:

- [Control UI](/web/control-ui)
- 48. [ภาพรวมเว็บ](/web)
- [Tailscale](/gateway/tailscale)
- [Remote access](/gateway/remote)

Proxy ที่เชื่อถือได้:

- 49. `gateway.trustedProxies`: รายการ IP ของรีเวิร์สพร็อกซีที่ยุติ TLS หน้าเกตเวย์
- 50. เมื่อการเชื่อมต่อมาจากหนึ่งใน IP เหล่านี้ OpenClaw จะใช้ `x-forwarded-for` (หรือ `x-real-ip`) เพื่อระบุ IP ของไคลเอนต์สำหรับการตรวจสอบการจับคู่ในเครื่อง และการยืนยันตัวตน HTTP/การตรวจสอบในเครื่อง
- 1. แสดงรายการเฉพาะพร็อกซีที่คุณควบคุมได้อย่างสมบูรณ์เท่านั้น และต้องแน่ใจว่าพวกมัน **เขียนทับ (overwrite)** `x-forwarded-for` ที่เข้ามา

หมายเหตุ:

- 2. `openclaw gateway` จะปฏิเสธการเริ่มทำงาน เว้นแต่ `gateway.mode` จะถูกตั้งค่าเป็น `local` (หรือคุณส่งแฟล็ก override)
- `gateway.port` controls the single multiplexed port used for WebSocket + HTTP (control UI, hooks, A2UI).
- 4. OpenAI Chat Completions endpoint: **ปิดใช้งานโดยค่าเริ่มต้น**; เปิดใช้งานด้วย `gateway.http.endpoints.chatCompletions.enabled: true`
- 5. ลำดับความสำคัญ: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > ค่าเริ่มต้น `18789`
- 6. โดยค่าเริ่มต้น จำเป็นต้องมีการยืนยันตัวตนของ Gateway (โทเค็น/รหัสผ่าน หรือ Tailscale Serve identity) 7. การ bind ที่ไม่ใช่ loopback จำเป็นต้องใช้โทเค็น/รหัสผ่านที่ใช้ร่วมกัน
- 8. ตัวช่วย onboarding จะสร้างโทเค็นของ gateway ให้โดยค่าเริ่มต้น (แม้บน loopback)
- 9. `gateway.remote.token` ใช้ **เฉพาะ** สำหรับการเรียก CLI ระยะไกล; มันไม่เปิดใช้งานการยืนยันตัวตนของ gateway แบบ local 10. `gateway.token` จะถูกละเลย

11. การยืนยันตัวตนและ Tailscale:

- 12. `gateway.auth.mode` กำหนดข้อกำหนดของ handshake (`token` หรือ `password`) 13. เมื่อไม่ได้ตั้งค่า จะถือว่าใช้การยืนยันตัวตนแบบโทเค็น
- 14. `gateway.auth.token` เก็บโทเค็นที่ใช้ร่วมกันสำหรับการยืนยันตัวตนแบบโทเค็น (ใช้โดย CLI บนเครื่องเดียวกัน)
- 15. เมื่อมีการตั้งค่า `gateway.auth.mode` จะยอมรับเฉพาะวิธีนั้นเท่านั้น (รวมถึงส่วนหัว Tailscale แบบตัวเลือก)
- 16. `gateway.auth.password` สามารถตั้งค่าได้ที่นี่ หรือผ่าน `OPENCLAW_GATEWAY_PASSWORD` (แนะนำ)
- 17. `gateway.auth.allowTailscale` อนุญาตให้ส่วนหัว Tailscale Serve identity
      (`tailscale-user-login`) ใช้ผ่านการยืนยันตัวตนได้ เมื่อคำขอมาถึงบน loopback
      พร้อม `x-forwarded-for`, `x-forwarded-proto` และ `x-forwarded-host` 18. OpenClaw
      ตรวจสอบตัวตนโดยการ resolve ที่อยู่ `x-forwarded-for` ผ่าน
      `tailscale whois` ก่อนจะยอมรับ 19. เมื่อเป็น `true` คำขอจาก Serve จะไม่ต้องใช้
      โทเค็น/รหัสผ่าน; ตั้งค่าเป็น `false` เพื่อบังคับให้ต้องใช้ข้อมูลรับรองอย่างชัดเจน 20. ค่าเริ่มต้นเป็น
      `true` เมื่อ `tailscale.mode = "serve"` และโหมดการยืนยันตัวตนไม่ใช่ `password`
- 21. `gateway.tailscale.mode: "serve"` ใช้ Tailscale Serve (เฉพาะ tailnet, bind แบบ loopback)
- 22. `gateway.tailscale.mode: "funnel"` เปิดแดชบอร์ดสู่สาธารณะ; จำเป็นต้องมีการยืนยันตัวตน
- 23. `gateway.tailscale.resetOnExit` รีเซ็ตการตั้งค่า Serve/Funnel เมื่อปิดการทำงาน

24. ค่าเริ่มต้นของไคลเอนต์ระยะไกล (CLI):

- 25. `gateway.remote.url` ตั้งค่า Gateway WebSocket URL เริ่มต้นสำหรับการเรียก CLI เมื่อ `gateway.mode = "remote"`
- 26. `gateway.remote.transport` เลือก transport ระยะไกลบน macOS (`ssh` เป็นค่าเริ่มต้น, `direct` สำหรับ ws/wss) 27. เมื่อใช้ `direct`, `gateway.remote.url` ต้องเป็น `ws://` หรือ `wss://` 28. `ws://host` จะใช้พอร์ต `18789` โดยค่าเริ่มต้น
- 29. `gateway.remote.token` ระบุโทเค็นสำหรับการเรียกแบบ remote (ปล่อยว่างเพื่อไม่ใช้การยืนยันตัวตน)
- 30. `gateway.remote.password` ระบุรหัสผ่านสำหรับการเรียกแบบ remote (ปล่อยว่างเพื่อไม่ใช้การยืนยันตัวตน)

31. พฤติกรรมของแอป macOS:

- 32. OpenClaw.app เฝ้าดู `~/.openclaw/openclaw.json` และสลับโหมดแบบเรียลไทม์เมื่อ `gateway.mode` หรือ `gateway.remote.url` เปลี่ยนแปลง
- 33. หากไม่ได้ตั้งค่า `gateway.mode` แต่ตั้งค่า `gateway.remote.url` แอป macOS จะถือว่าเป็นโหมด remote
- 34. เมื่อคุณเปลี่ยนโหมดการเชื่อมต่อในแอป macOS แอปจะเขียน `gateway.mode` (และ `gateway.remote.url` + `gateway.remote.transport` ในโหมด remote) กลับไปยังไฟล์คอนฟิก

```json5
35. {
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

36. ตัวอย่าง transport แบบ direct (แอป macOS):

```json5
37. {
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### 38. `gateway.reload` (การรีโหลดคอนฟิกแบบ hot)

39. Gateway จะเฝ้าดู `~/.openclaw/openclaw.json` (หรือ `OPENCLAW_CONFIG_PATH`) และนำการเปลี่ยนแปลงไปใช้โดยอัตโนมัติ

Modes:

- 40. `hybrid` (ค่าเริ่มต้น): นำการเปลี่ยนแปลงที่ปลอดภัยมาใช้ทันที; รีสตาร์ต Gateway สำหรับการเปลี่ยนแปลงที่สำคัญ
- 41. `hot`: นำมาใช้เฉพาะการเปลี่ยนแปลงที่ปลอดภัยแบบ hot; บันทึก log เมื่อจำเป็นต้องรีสตาร์ต
- 42. `restart`: รีสตาร์ต Gateway เมื่อมีการเปลี่ยนแปลงคอนฟิกใด ๆ
- 43. `off`: ปิดการรีโหลดแบบ hot

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

#### 45. เมทริกซ์การรีโหลดแบบ hot (ไฟล์ + ผลกระทบ)

46. ไฟล์ที่ถูกเฝ้าดู:

- คำเตือน: `config.apply` จะเขียนทับ **คอนฟิกทั้งหมด** หากต้องการเปลี่ยนเพียงไม่กี่คีย์
  ให้ใช้ `config.patch` หรือ `openclaw config set` และควรสำรอง `~/.openclaw/openclaw.json` ไว้

47. นำมาใช้แบบ hot (ไม่ต้องรีสตาร์ต gateway ทั้งหมด):

- 48. `hooks` (การยืนยันตัวตน/พาธ/การแมปของ webhook) + `hooks.gmail` (รีสตาร์ต Gmail watcher)
- 49. `browser` (รีสตาร์ตเซิร์ฟเวอร์ควบคุมเบราว์เซอร์)
- 50. `cron` (รีสตาร์ตบริการ cron + อัปเดต concurrency)
- `agents.defaults.heartbeat` (การรีสตาร์ทรันเนอร์ของ heartbeat)
- `web` (การรีสตาร์ตช่องทาง WhatsApp web)
- `telegram`, `discord`, `signal`, `imessage` (การรีสตาร์ตช่องทาง)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (การอ่านแบบไดนามิก)

ต้องรีสตาร์ต Gateway ทั้งหมด:

- `gateway` (พอร์ต/bind/auth/UI ควบคุม/tailscale)
- `bridge` (ระบบเดิม)
- `discovery`
- `canvasHost`
- `ปลั๊กอิน`
- พาธคอนฟิกที่ไม่รู้จัก/ไม่รองรับ (ค่าเริ่มต้นคือรีสตาร์ตเพื่อความปลอดภัย)

### การแยกหลายอินสแตนซ์

เพื่อรันหลาย gateway บนโฮสต์เดียว (เพื่อความซ้ำซ้อนหรือบอทกู้ชีพ) ให้แยกสถานะ + คอนฟิกต่ออินสแตนซ์ และใช้พอร์ตที่ไม่ซ้ำกัน:

- `OPENCLAW_CONFIG_PATH` (คอนฟิกต่ออินสแตนซ์)
- `OPENCLAW_STATE_DIR` (เซสชัน/ข้อมูลรับรอง)
- `agents.defaults.workspace` (ความทรงจำ)
- `gateway.port` (ไม่ซ้ำกันต่ออินสแตนซ์)

แฟล็กอำนวยความสะดวก (CLI):

- `openclaw --dev …` → ใช้ `~/.openclaw-dev` + เลื่อนพอร์ตจากฐาน `19001`
- `openclaw --profile <name> …` → ใช้ `~/.openclaw-<name>` (พอร์ตผ่านคอนฟิก/env/แฟล็ก)

ดู [Gateway runbook](/gateway) สำหรับการแม็ปพอร์ตที่คำนวณได้ (gateway/browser/canvas)
ดู [Multiple gateways](/gateway/multiple-gateways) สำหรับรายละเอียดการแยกพอร์ต browser/CDP

ตัวอย่าง:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

### `hooks` (Webhook ของ Gateway)

เปิดใช้งาน endpoint webhook HTTP แบบง่ายบนเซิร์ฟเวอร์ HTTP ของ Gateway

ค่าเริ่มต้น:

- enabled: `false`
- path: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
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

คำขอต้องมีโทเคนของ hook:

- `Authorization: Bearer <token>` **หรือ**
- `x-openclaw-token: <token>`

Endpoints:

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` }\`
- `POST /hooks/<name>` → แก้ไขเส้นทางผ่าน `hooks.mappings`

`/hooks/agent` จะโพสต์สรุปไปยังเซสชันหลักเสมอ (และสามารถทริกเกอร์ heartbeat ทันทีได้ผ่าน `wakeMode: "now"`).

หมายเหตุเกี่ยวกับการแม็ป:

- `match.path` ตรงกับซับพาธหลัง `/hooks` (เช่น `/hooks/gmail` → `gmail`).
- `match.source` ตรงกับฟิลด์ใน payload (เช่น `{ source: "gmail" }`) เพื่อให้คุณใช้พาธทั่วไปอย่าง `/hooks/ingest` ได้
- เทมเพลตอย่าง `{{messages[0].subject}}` อ่านค่าจาก payload
- `transform` สามารถชี้ไปยังโมดูล JS/TS ที่คืนค่า hook action
- `deliver: true` จะส่งคำตอบสุดท้ายไปยังช่องทาง; `channel` ค่าเริ่มต้นคือ `last` (ย้อนกลับไปใช้ WhatsApp)
- หากไม่มีเส้นทางการส่งก่อนหน้า ให้ตั้งค่า `channel` + `to` โดยตรง (จำเป็นสำหรับ Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams)
- `model` ใช้แทน LLM สำหรับการรัน hook นี้ (`provider/model` หรือ alias; ต้องได้รับอนุญาตหากตั้งค่า `agents.defaults.models`)

คอนฟิกตัวช่วย Gmail (ใช้โดย `openclaw webhooks gmail setup` / `run`):

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Optional: use a cheaper model for Gmail hook processing
      // Falls back to agents.defaults.model.fallbacks, then primary, on auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    },
  },
}
```

การ override โมเดลสำหรับ Gmail hooks:

- `hooks.gmail.model` ระบุโมเดลที่จะใช้สำหรับการประมวลผล Gmail hook (ค่าเริ่มต้นคือ primary ของเซสชัน)
- รองรับการอ้างอิงแบบ `provider/model` หรือ alias จาก `agents.defaults.models`
- จะ fallback ไปที่ `agents.defaults.model.fallbacks` จากนั้น `agents.defaults.model.primary` เมื่อเกิดปัญหา auth/rate-limit/timeout
- หากตั้งค่า `agents.defaults.models` ต้องรวมโมเดลของ hooks ไว้ใน allowlist
- เมื่อเริ่มต้นระบบ จะเตือนหากโมเดลที่ตั้งค่าไว้ไม่อยู่ในแคตตาล็อกโมเดลหรือ allowlist
- `hooks.gmail.thinking` sets the default thinking level for Gmail hooks and is overridden by per-hook `thinking`.

Gateway auto-start:

- If `hooks.enabled=true` and `hooks.gmail.account` is set, the Gateway starts
  `gog gmail watch serve` on boot and auto-renews the watch.
- Set `OPENCLAW_SKIP_GMAIL_WATCHER=1` to disable the auto-start (for manual runs).
- Avoid running a separate `gog gmail watch serve` alongside the Gateway; it will
  fail with `listen tcp 127.0.0.1:8788: bind: address already in use`.

Note: when `tailscale.mode` is on, OpenClaw defaults `serve.path` to `/` so
Tailscale can proxy `/gmail-pubsub` correctly (it strips the set-path prefix).
If you need the backend to receive the prefixed path, set
`hooks.gmail.tailscale.target` to a full URL (and align `serve.path`).

### `canvasHost` (LAN/tailnet Canvas file server + live reload)

The Gateway serves a directory of HTML/CSS/JS over HTTP so iOS/Android nodes can simply `canvas.navigate` to it.

Default root: `~/.openclaw/workspace/canvas`  
Default port: `18793` (chosen to avoid the openclaw browser CDP port `18792`)  
The server listens on the **gateway bind host** (LAN or Tailnet) so nodes can reach it.

The server:

- serves files under `canvasHost.root`
- injects a tiny live-reload client into served HTML
- watches the directory and broadcasts reloads over a WebSocket endpoint at `/__openclaw__/ws`
- auto-creates a starter `index.html` when the directory is empty (so you see something immediately)
- also serves A2UI at `/__openclaw__/a2ui/` and is advertised to nodes as `canvasHostUrl`
  (always used by nodes for Canvas/A2UI)

Disable live reload (and file watching) if the directory is large or you hit `EMFILE`:

- config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

Changes to `canvasHost.*` require a gateway restart (config reload will restart).

ปิดใช้งานด้วย:

- config: `canvasHost: { enabled: false }`
- env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `bridge` (legacy TCP bridge, removed)

Current builds no longer include the TCP bridge listener; `bridge.*` config keys are ignored.
Nodes connect over the Gateway WebSocket. This section is kept for historical reference.

Legacy behavior:

- The Gateway could expose a simple TCP bridge for nodes (iOS/Android), typically on port `18790`.

ค่าเริ่มต้น:

- enabled: `true`
- port: `18790`
- bind: `lan` (binds to `0.0.0.0`)

Bind modes:

- `lan`: `0.0.0.0` (reachable on any interface, including LAN/Wi‑Fi and Tailscale)
- `tailnet`: bind only to the machine’s Tailscale IP (recommended for Vienna ⇄ London)
- `loopback`: `127.0.0.1` (local only)
- `auto`: prefer tailnet IP if present, else `lan`

TLS:

- `bridge.tls.enabled`: enable TLS for bridge connections (TLS-only when enabled).
- `bridge.tls.autoGenerate`: generate a self-signed cert when no cert/key are present (default: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: PEM paths for the bridge certificate + private key.
- `bridge.tls.caPath`: optional PEM CA bundle (custom roots or future mTLS).

When TLS is enabled, the Gateway advertises `bridgeTls=1` and `bridgeTlsSha256` in discovery TXT
records so nodes can pin the certificate. Manual connections use trust-on-first-use if no
fingerprint is stored yet.
Auto-generated certs require `openssl` on PATH; if generation fails, the bridge will not start.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Uses ~/.openclaw/bridge/tls/bridge-{cert,key}.pem when omitted.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS broadcast mode)

Controls LAN mDNS discovery broadcasts (`_openclaw-gw._tcp`).

- `minimal` (default): omit `cliPath` + `sshPort` from TXT records
- `full`: include `cliPath` + `sshPort` in TXT records
- `off`: disable mDNS broadcasts entirely
- 1. ชื่อโฮสต์: ค่าเริ่มต้นคือ `openclaw` (ประกาศเป็น `openclaw.local`). 2. สามารถกำหนดทับได้ด้วย `OPENCLAW_MDNS_HOSTNAME`.

```json5
3. {
  discovery: { mdns: { mode: "minimal" } },
}
```

### 4. `discovery.wideArea` (Wide-Area Bonjour / unicast DNS‑SD)

5. เมื่อเปิดใช้งาน Gateway จะเขียนโซน unicast DNS-SD สำหรับ `_openclaw-gw._tcp` ภายใต้ `~/.openclaw/dns/` โดยใช้โดเมนการค้นหาที่กำหนดค่าไว้ (ตัวอย่าง: `openclaw.internal.`).

6. เพื่อให้ iOS/Android ค้นพบข้ามเครือข่ายได้ (เวียนนา ⇄ ลอนดอน) ให้ใช้ร่วมกับ:

- 7. เซิร์ฟเวอร์ DNS บนโฮสต์ gateway ที่ให้บริการโดเมนที่คุณเลือก (แนะนำ CoreDNS)
- 8. Tailscale **split DNS** เพื่อให้ไคลเอนต์ resolve โดเมนนั้นผ่านเซิร์ฟเวอร์ DNS ของ gateway

One-time setup helper (gateway host):

```bash
openclaw dns setup --apply
```

```json5
10. {
  discovery: { wideArea: { enabled: true } },
}
```

## 11. ตัวแปรเทมเพลตของโมเดลสื่อ

12. ตัวแทนที่คั่นในเทมเพลตจะถูกขยายใน `tools.media.*.models[].args` และ `tools.media.models[].args` (และฟิลด์อาร์กิวเมนต์แบบเทมเพลตอื่น ๆ ในอนาคต).

13. \| ตัวแปร           | คำอธิบาย                                                                     |
    \| ------------------ | ------------------------------------------------------------------------------- | -------- | ------- | ---------- | ----- | ------ | -------- | ------- | ------- | --- |
    \| `{{Body}}`         | เนื้อหาข้อความขาเข้าทั้งหมด                                                   |
    \| `{{RawBody}}`      | เนื้อหาข้อความขาเข้าแบบดิบ (ไม่มี wrapper ประวัติ/ผู้ส่ง; เหมาะที่สุดสำหรับการพาร์สคำสั่ง) |
    \| `{{BodyStripped}}` | เนื้อหาที่ตัดการกล่าวถึงกลุ่มออกแล้ว (ค่าเริ่มต้นที่เหมาะสำหรับเอเจนต์)                     |
    \| `{{From}}`         | ตัวระบุผู้ส่ง (E.164 สำหรับ WhatsApp; อาจแตกต่างตามช่องทาง)                  |
    \| `{{To}}`           | ตัวระบุปลายทาง                                                              |
    \| `{{MessageSid}}`   | ID ข้อความของช่องทาง (เมื่อมี)                                               |
    \| `{{SessionId}}`    | UUID ของเซสชันปัจจุบัน                                                       |
    \| `{{IsNewSession}}` | "true" เมื่อมีการสร้างเซสชันใหม่                                         |
    \| `{{MediaUrl}}`     | pseudo-URL ของสื่อขาเข้า (ถ้ามี)                                              |
    \| `{{MediaPath}}`    | พาธสื่อภายในเครื่อง (ถ้าดาวน์โหลดแล้ว)                                      |
    \| `{{MediaType}}`    | ประเภทสื่อ (image/audio/document/…)                                             14. |
    \| `{{Transcript}}`   | ถอดเสียงเสียง (เมื่อเปิดใช้งาน)                                             |
    \| `{{Prompt}}`       | พรอมป์สื่อที่ถูก resolve สำหรับรายการ CLI                                   |
    \| `{{MaxChars}}`     | จำนวนอักขระเอาต์พุตสูงสุดที่ถูก resolve สำหรับรายการ CLI                     |
    \| `{{ChatType}}`     | "direct" หรือ "group"                                                   |
    \| `{{GroupSubject}}` | หัวข้อกลุ่ม (พยายามให้ได้ดีที่สุด)                                           |
    \| `{{GroupMembers}}` | ตัวอย่างสมาชิกกลุ่ม (พยายามให้ได้ดีที่สุด)                                  |
    \| `{{SenderName}}`   | ชื่อแสดงของผู้ส่ง (พยายามให้ได้ดีที่สุด)                                     |
    \| `{{SenderE164}}`   | หมายเลขโทรศัพท์ผู้ส่ง (พยายามให้ได้ดีที่สุด)                                 |
    \| `{{Provider}}`     | คำใบ้ผู้ให้บริการ (whatsapp                                                         | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  15. |

## 16. Cron (ตัวตั้งเวลาของ Gateway)

17. Cron เป็นตัวตั้งเวลาที่ Gateway เป็นเจ้าของ สำหรับการปลุกและงานที่ตั้งเวลาไว้ 18. ดู [Cron jobs](/automation/cron-jobs) สำหรับภาพรวมฟีเจอร์และตัวอย่าง CLI

```json5
19. {
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_ถัดไป: [Agent Runtime](/concepts/agent)_ 🦞

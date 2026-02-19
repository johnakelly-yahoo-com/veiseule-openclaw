---
summary: "เอกสารอ้างอิงCLIสำหรับ `openclaw agents` (list/add/delete/set identity)"
read_when:
  - คุณต้องการเอเจนต์หลายตัวที่แยกจากกัน (เวิร์กสเปซ + การกำหนดเส้นทาง + การยืนยันตัวตน)
title: "เอเจนต์"
---

# `openclaw agents`

จัดการเอเจนต์ที่แยกจากกัน (เวิร์กสเปซ + การยืนยันตัวตน + การกำหนดเส้นทาง)

เกี่ยวข้อง:

- การกำหนดเส้นทางแบบหลายเอเจนต์: [Multi-Agent Routing](/concepts/multi-agent)
- เวิร์กสเปซของเอเจนต์: [Agent workspace](/concepts/agent-workspace)

## ตัวอย่าง

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## ไฟล์ตัวตน

เวิร์กสเปซของเอเจนต์แต่ละตัวสามารถมี `IDENTITY.md` ที่รากของเวิร์กสเปซได้:

- พาธตัวอย่าง: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` อ่านจากรากของเวิร์กสเปซ (หรือจาก `--identity-file` ที่ระบุอย่างชัดเจน)

พาธของอวาตาร์จะถูกแก้ไขโดยอ้างอิงจากรากของเวิร์กสเปซ

## ตั้งค่าตัวตน

`set-identity` เขียนฟิลด์ลงใน `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (พาธที่อ้างอิงกับเวิร์กสเปซ, URL แบบ http(s) หรือ data URI)

โหลดจาก `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

เขียนทับฟิลด์โดยระบุโดยตรง:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

ตัวอย่างคอนฟิก:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```


---
summary: "เอกสารอ้างอิงCLIสำหรับ `openclaw onboard` (วิซาร์ดการเริ่มต้นใช้งานแบบโต้ตอบ)"
read_when:
  - คุณต้องการการตั้งค่าแบบมีคำแนะนำสำหรับGateway, เวิร์กสเปซ, การยืนยันตัวตน, ช่องทาง และSkills
title: "onboard"
---

# `openclaw onboard`

วิซาร์ดการเริ่มต้นใช้งานแบบโต้ตอบ(การตั้งค่าGatewayภายในเครื่องหรือระยะไกล)

## คู่มือที่เกี่ยวข้อง

- ศูนย์รวมการเริ่มต้นใช้งานCLI: [Onboarding Wizard (CLI)](/start/wizard)
- ภาพรวมการเริ่มต้นใช้งาน: [Onboarding Overview](/start/onboarding-overview)
- ระบบอัตโนมัติCLI: [CLI Automation](/start/wizard-cli-automation)
- เอกสารอ้างอิงการเริ่มต้นใช้งานCLI: [CLI Onboarding Reference](/start/wizard-cli-reference)
- การเริ่มต้นใช้งานบนmacOS: [Onboarding (macOS App)](/start/onboarding)

## Examples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

ผู้ให้บริการแบบกำหนดเองในโหมดไม่โต้ตอบ:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` ไม่บังคับในโหมดไม่โต้ตอบ หากไม่ระบุ การเริ่มต้นใช้งานจะตรวจสอบ `CUSTOM_API_KEY`

ตัวเลือก endpoint Z.AI แบบไม่โต้ตอบ:

หมายเหตุ: `--auth-choice zai-api-key` จะตรวจจับ endpoint Z.AI ที่เหมาะสมที่สุดสำหรับคีย์ของคุณโดยอัตโนมัติ (ให้ความสำคัญกับ API ทั่วไปที่ใช้ `zai/glm-5`)
หากคุณต้องการ endpoint ของ GLM Coding Plan โดยเฉพาะ ให้เลือก `zai-coding-global` หรือ `zai-coding-cn`

```bash
# การเลือก endpoint โดยไม่ต้องมีพรอมป์
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# ตัวเลือก endpoint Z.AI อื่นๆ:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow notes:

- `quickstart`: พรอมต์น้อยที่สุด สร้างโทเคนGatewayให้อัตโนมัติ
- `manual`: พรอมต์แบบครบถ้วนสำหรับพอร์ต/การผูก/การยืนยันตัวตน(นามแฝงของ `advanced`)
- การแชตครั้งแรกที่เร็วที่สุด: `openclaw dashboard` (Control UI ไม่ต้องตั้งค่าช่องทาง)
- Custom Provider: เชื่อมต่อกับ endpoint ใดก็ได้ที่รองรับ OpenAI หรือ Anthropic
  รวมถึงผู้ให้บริการแบบโฮสต์ที่ไม่ได้อยู่ในรายการ ใช้ Unknown เพื่อตรวจจับอัตโนมัติ

## Common follow-up commands

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` ไม่ได้หมายถึงโหมดไม่โต้ตอบ ใช้ `--non-interactive` สำหรับสคริปต์
 ใช้ `--non-interactive` สำหรับสคริปต์
</Note> ใช้ `--non-interactive` สำหรับสคริปต์
</Note>


---
title: "การสร้าง Skills"
---

# การสร้าง Skills แบบกำหนดเอง 🛠

OpenClaw is designed to be easily extensible. OpenClaw ถูกออกแบบมาให้ขยายความสามารถได้อย่างง่ายดาย โดย "Skills" คือวิธีหลักในการเพิ่มความสามารถใหม่ให้กับผู้ช่วยของคุณ

## Skill คืออะไร?

Skill คือไดเรกทอรีที่มีไฟล์ `SKILL.md` (ซึ่งให้คำสั่งและคำจำกัดความของเครื่องมือแก่ LLM) และอาจมีสคริปต์หรือทรัพยากรเพิ่มเติมตามต้องการ

## ทีละขั้นตอน: Skill แรกของคุณ

### 1. สร้างไดเรกทอรี

Skills จะอยู่ในเวิร์กสเปซของคุณ โดยปกติคือ `~/.openclaw/workspace/skills/` สร้างโฟลเดอร์ใหม่สำหรับ Skill ของคุณ: Create a new folder for your skill:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. กำหนด `SKILL.md`

Create a `SKILL.md` file in that directory. สร้างไฟล์ `SKILL.md` ในไดเรกทอรีนั้น ไฟล์นี้ใช้ YAML frontmatter สำหรับเมตาดาตา และใช้ Markdown สำหรับคำสั่งการทำงาน

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. เพิ่มเครื่องมือ (ไม่บังคับ)

คุณสามารถกำหนดเครื่องมือแบบกำหนดเองใน frontmatter หรือสั่งให้เอเจนต์ใช้เครื่องมือระบบที่มีอยู่แล้ว (เช่น `bash` หรือ `browser`)

### 4. รีเฟรช OpenClaw

Ask your agent to "refresh skills" or restart the gateway. ขอให้เอเจนต์ของคุณ "refresh skills" หรือรีสตาร์ท Gateway OpenClaw จะค้นหาไดเรกทอรีใหม่และทำดัชนีไฟล์ `SKILL.md`

## แนวปฏิบัติที่ดีที่สุด

- **กระชับชัดเจน**: บอกโมเดลว่าให้ทำ _อะไร_ ไม่ใช่สอนวิธีเป็น AI
- **ความปลอดภัยมาก่อน**: หาก Skill ของคุณใช้ `bash` ตรวจสอบให้แน่ใจว่าพรอมป์ต์ไม่เปิดโอกาสให้มีการฉีดคำสั่งโดยอำเภอใจจากอินพุตผู้ใช้ที่ไม่น่าเชื่อถือ
- **ทดสอบในเครื่อง**: ใช้ `openclaw agent --message "use my new skill"` เพื่อทดสอบ

## Skills ที่แชร์ร่วมกัน

คุณยังสามารถเรียกดูและร่วมแบ่งปัน Skills ได้ที่ [ClawHub](https://clawhub.com)


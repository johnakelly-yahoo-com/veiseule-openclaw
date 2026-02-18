---
title: "Ko‘nikmalarni yaratish"
---

# Maxsus Ko‘nikmalarni Yaratish 🛠

OpenClaw oson kengaytiriladigan qilib ishlab chiqilgan. "Ko‘nikmalar" yordamchingizga yangi imkoniyatlar qo‘shishning asosiy usulidir.

## Ko‘nikma nima?

Ko‘nikma — bu `SKILL.md` faylini (u LLM uchun ko‘rsatmalar va vosita ta’riflarini taqdim etadi) hamda ixtiyoriy ravishda ba’zi skriptlar yoki resurslarni o‘z ichiga olgan katalogdir.

## Bosqichma-bosqich: Birinchi Ko‘nikmangiz

### 1. Katalog yarating

Ko‘nikmalar ishchi muhitingizda, odatda `~/.openclaw/workspace/skills/` ichida joylashadi. Ko‘nikmangiz uchun yangi papka yarating:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. `SKILL.md` faylini belgilang

Ushbu katalog ichida `SKILL.md` faylini yarating. Bu fayl metama’lumotlar uchun YAML frontmatter va ko‘rsatmalar uchun Markdown’dan foydalanadi.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. Add Tools (Optional)

You can define custom tools in the frontmatter or instruct the agent to use existing system tools (like `bash` or `browser`).

### 4. Refresh OpenClaw

Ask your agent to "refresh skills" or restart the gateway. OpenClaw will discover the new directory and index the `SKILL.md`.

## Best Practices

- 1. **Qisqa bo‘ling**: Modelga _nima_ qilish kerakligini ayting, uni qanday qilib AI bo‘lishini emas.
- 2. **Xavfsizlik birinchi o‘rinda**: Agar sizning ko‘nikmangiz `bash` dan foydalansa, promptlar ishonchsiz foydalanuvchi kiritmalaridan ixtiyoriy buyruq injeksiyasiga yo‘l qo‘ymasligiga ishonch hosil qiling.
- 3. **Mahalliy sinovdan o‘tkazing**: Sinash uchun `openclaw agent --message "use my new skill"` dan foydalaning.

## 4. Umumiy ko‘nikmalar

5. Shuningdek, siz [ClawHub](https://clawhub.com) orqali ko‘nikmalarni ko‘rib chiqishingiz va ularga hissa qo‘shishingiz mumkin.


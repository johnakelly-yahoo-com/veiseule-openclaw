---
summary: "`openclaw logs` uchun CLI ma'lumotnomasi (Gateway loglarini RPC orqali real vaqtda kuzatish)"
read_when:
  - You need to tail Gateway logs remotely (without SSH)
  - You want JSON log lines for tooling
title: "logs"
---

# `openclaw logs`

Gateway fayl loglarini RPC orqali real vaqtda ko‘rish (masofaviy rejimda ishlaydi).

Bog‘liq:

- Loglash bo‘yicha umumiy ma’lumot: [Loglash](/logging)

## Misollar

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```

Vaqt belgilarini mahalliy vaqt mintaqangizda ko‘rsatish uchun `--local-time` dan foydalaning.


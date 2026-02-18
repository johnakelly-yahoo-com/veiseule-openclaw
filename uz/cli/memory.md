---
title: "memory"
---

# `openclaw memory`

Semantik xotirani indekslash va qidirishni boshqaradi.  
Faol xotira plagini tomonidan taqdim etiladi (standart: `memory-core`; o‘chirish uchun `plugins.slots.memory = "none"` ni o‘rnating).

Bog‘liq:

- Xotira tushunchasi: [Xotira](/concepts/memory)
- Plaginlar: [Plaginlar](/tools/plugin)

## Misollar

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## Parametrlar

Umumiy:

- `--agent <id>`: bitta agent doirasida ishlaydi (standart: barcha sozlangan agentlar).
- `--verbose`: tekshiruv va indekslash jarayonida batafsil loglarni chiqaradi.

Eslatmalar:

- `memory status --deep` vektor va embedding mavjudligini tekshiradi.
- `memory status --deep --index` agar saqlash joyi yangilanishni talab qilsa, qayta indekslashni ishga tushiradi.
- `memory index --verbose` har bir bosqich bo‘yicha tafsilotlarni (provider, model, manbalar, batch faoliyati) chiqaradi.
- `memory status` `memorySearch.extraPaths` orqali sozlangan qo‘shimcha yo‘llarni ham o‘z ichiga oladi.


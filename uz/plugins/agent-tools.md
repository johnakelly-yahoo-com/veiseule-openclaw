---
title: "Plugin Agent Vositalari"
---

# Plugin agent vositalari

OpenClaw pluginlari **agent vositalarini** (JSON‑schema funksiyalari) ro‘yxatdan o‘tkazishi mumkin, ular
agent ishga tushirilganda LLM’ga taqdim etiladi. Vositalar **majburiy** (har doim mavjud) yoki
**ixtiyoriy** (opt‑in) bo‘lishi mumkin.

Agent vositalari asosiy konfiguratsiyadagi `tools` bo‘limida yoki har bir agent uchun
`agents.list[].tools` ostida sozlanadi. Allowlist/denylist siyosati agent
qaysi vositalarni chaqira olishini boshqaradi.

## Oddiy vosita

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## Ixtiyoriy vosita (opt‑in)

Ixtiyoriy vositalar **hech qachon** avtomatik yoqilmaydi. Foydalanuvchilar ularni agent
allowlist’iga qo‘shishlari kerak.

```ts
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

Ixtiyoriy vositalarni `agents.list[].tools.allow` (yoki global `tools.allow`) orqali yoqing:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // aniq vosita nomi
            "workflow", // plugin id (ushbu plugindagi barcha vositalarni yoqadi)
            "group:plugins", // barcha plugin vositalari
          ],
        },
      },
    ],
  },
}
```

Vositalar mavjudligiga ta’sir qiluvchi boshqa konfiguratsiya sozlamalari:

- Faqat plugin vositalari ko‘rsatilgan allowlistlar plugin opt‑in sifatida qabul qilinadi; agar allowlistga core vositalar yoki guruhlar ham kiritilmasa, core vositalar yoqilganligicha qoladi.
- `tools.profile` / `agents.list[].tools.profile` (asosiy allowlist)
- `tools.byProvider` / `agents.list[].tools.byProvider` (provayderga xos allow/deny)
- `tools.sandbox.tools.*` (sandbox rejimida vositalar siyosati)

## Qoidalar + maslahatlar

- Vosita nomlari core vosita nomlari bilan **to‘qnash kelmasligi** kerak; zid kelgan vositalar o‘tkazib yuboriladi.
- Allowlistlarda ishlatiladigan plugin id’lar core vosita nomlari bilan to‘qnashmasligi kerak.
- Yon ta’sirlarni ishga tushiradigan yoki qo‘shimcha binary/fayllar yoki credential talab qiladigan vositalar uchun `optional: true` dan foydalanish tavsiya etiladi.

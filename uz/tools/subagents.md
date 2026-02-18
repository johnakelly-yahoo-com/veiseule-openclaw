---
title: "Sub-agentlar"
---

# Sub-agentlar

Sub-agentlar mavjud agent ishga tushirishidan yaratiladigan fon (background) agent ishga tushirishlaridir. Ular o‘zining alohida sessiyasida (`agent:<agentId>:subagent:<uuid>`) ishlaydi va yakunlangach, natijasini so‘rovchi chat kanaliga **e’lon qiladi**.

## Slash buyruq

Joriy sessiya uchun sub-agent ishga tushirishlarini ko‘rish yoki boshqarish uchun `/subagents` dan foydalaning:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` ishga tushirish metama’lumotlarini ko‘rsatadi (holat, vaqt belgilar, sessiya id, transkript yo‘li, tozalash).

Asosiy maqsadlar:

- Asosiy ishga tushirishni bloklamasdan "research / long task / sekin asbob" ishlarini parallellashtirish.
- Sub-agentlarni sukut bo‘yicha izolyatsiya qilish (sessiya ajratilishi + ixtiyoriy sandbox).
- Asbob yuzasini noto‘g‘ri ishlatishni qiyinlashtirish: sub-agentlar sukut bo‘yicha session asboblarini olmaydi.
- Orkestrator patternlari uchun sozlanadigan nesting chuqurligini qo‘llab-quvvatlash.

Xarajat eslatmasi: har bir sub-agent o‘zining **konteksti** va token sarfiga ega. Og‘ir yoki takroriy vazifalar uchun sub-agentlarga arzonroq model belgilang va asosiy agentni yuqori sifatli modelda qoldiring. Buni `agents.defaults.subagents.model` yoki har bir agent uchun override orqali sozlashingiz mumkin.

## Asbob

`sessions_spawn` dan foydalaning:

- Sub-agent ishga tushirishni boshlaydi (`deliver: false`, global lane: `subagent`)
- So‘ng announce bosqichini bajaradi va announce javobini so‘rovchi chat kanaliga joylaydi
- Standart model: chaqiruvchidan meros bo‘lib o‘tadi, agar siz `agents.defaults.subagents.model` (yoki per-agent `agents.list[].subagents.model`) ni sozlamasangiz; aniq `sessions_spawn.model` har doim ustun turadi.
- Standart thinking: chaqiruvchidan meros bo‘lib o‘tadi, agar siz `agents.defaults.subagents.thinking` (yoki per-agent `agents.list[].subagents.thinking`) ni sozlamasangiz; aniq `sessions_spawn.thinking` har doim ustun turadi.

Asbob parametrlari:

- `task` (majburiy)
- `label?` (ixtiyoriy)
- `agentId?` (ixtiyoriy; ruxsat berilgan bo‘lsa, boshqa agent id ostida ishga tushirish)
- `model?` (ixtiyoriy; sub-agent modeli ustidan override; noto‘g‘ri qiymatlar o‘tkazib yuboriladi va sub-agent standart modelda ishga tushadi, asbob natijasida ogohlantirish bilan)
- `thinking?` (ixtiyoriy; sub-agent ishga tushirish uchun thinking darajasini override qiladi)
- `runTimeoutSeconds?` (standart `0`; o‘rnatilsa, sub-agent N soniyadan keyin to‘xtatiladi)
- `cleanup?` (`delete|keep`, standart `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: `agentId` orqali nishonga olinishi mumkin bo‘lgan agent idlar ro‘yxati (`["*"]` — istalganiga ruxsat). Standart: faqat so‘rovchi agent.

Discovery:

- `sessions_spawn` uchun hozirda qaysi agent idlarga ruxsat berilganini ko‘rish uchun `agents_list` dan foydalaning.

Auto-archive:

- Sub-agent sessiyalari `agents.defaults.subagents.archiveAfterMinutes` dan so‘ng avtomatik arxivlanadi (standart: 60).
- Arxiv `sessions.delete` dan foydalanadi va transkriptni `*.deleted.<timestamp>` ga qayta nomlaydi (shu papkada).
- `cleanup: "delete"` announce’dan so‘ng darhol arxivlaydi (transkript rename orqali saqlanadi).
- Auto-archive best-effort; agar gateway qayta ishga tushsa, kutilayotgan timerlar yo‘qoladi.
- `runTimeoutSeconds` auto-archive qilmaydi; u faqat ishga tushirishni to‘xtatadi. Sessiya auto-archive’gacha saqlanadi.
- Auto-archive depth-1 va depth-2 sessiyalarga bir xil qo‘llanadi.

## Nested Sub-Agentlar

Sukut bo‘yicha sub-agentlar o‘z sub-agentlarini ishga tushira olmaydi (`maxSpawnDepth: 1`). Siz `maxSpawnDepth: 2` qilib, bir darajali nestingni yoqishingiz mumkin — bu **orchestrator pattern** ni imkon qiladi: main → orchestrator sub-agent → worker sub-sub-agentlar.

### Qanday yoqiladi

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // sub-agentlarga farzand yaratishga ruxsat (standart: 1)
        maxChildrenPerAgent: 5, // har bir agent sessiyasi uchun maksimal faol farzandlar (standart: 5)
        maxConcurrent: 8, // global concurrency lane chegarasi (standart: 8)
      },
    },
  },
}
```

### Chuqurlik darajalari

| Depth | Session key shakli                           | Rol                                             | Spawn qila oladimi?            |
| ----- | -------------------------------------------- | ----------------------------------------------- | ------------------------------- |
| 0     | `agent:<id>:main`                            | Asosiy agent                                   | Har doim                       |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agent (depth 2 ruxsat etilganda orchestrator) | Faqat `maxSpawnDepth >= 2` bo‘lsa |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                    | Hech qachon                    |

### Announce zanjiri

Natijalar zanjir bo‘ylab yuqoriga oqadi:

1. Depth-2 worker tugaydi → o‘z parentiga (depth-1 orchestrator) announce qiladi
2. Depth-1 orchestrator announce’ni oladi, natijalarni sintez qiladi, tugaydi → main’ga announce qiladi
3. Main agent announce’ni oladi va foydalanuvchiga yetkazadi

Har bir daraja faqat bevosita farzandlaridan kelgan announce’larni ko‘radi.

### Chuqurlik bo‘yicha asbob siyosati

- **Depth 1 (orchestrator, `maxSpawnDepth >= 2` bo‘lsa)**: `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` oladi, shunda o‘z farzandlarini boshqara oladi. Boshqa session/system asboblar rad etiladi.
- **Depth 1 (leaf, `maxSpawnDepth == 1` bo‘lsa)**: Session asboblari yo‘q (joriy standart xulq).
- **Depth 2 (leaf worker)**: Session asboblari yo‘q — depth 2 da `sessions_spawn` har doim rad etiladi. Yana farzand yarata olmaydi.

### Per-agent spawn limiti

Har bir agent sessiyasi (istalgan depth’da) bir vaqtning o‘zida maksimal `maxChildrenPerAgent` (standart: 5) faol farzandga ega bo‘lishi mumkin. Bu bitta orchestrator’dan nazoratsiz fan-out’ni oldini oladi.

### Cascade stop

Depth-1 orchestrator’ni to‘xtatish uning barcha depth-2 farzandlarini ham avtomatik to‘xtatadi:

- Main chat’da `/stop` barcha depth-1 agentlarni to‘xtatadi va ularning depth-2 farzandlariga kaskad qiladi.
- `/subagents kill <id>` aniq sub-agentni to‘xtatadi va uning farzandlariga kaskad qiladi.
- `/subagents kill all` so‘rovchi uchun barcha sub-agentlarni to‘xtatadi va kaskad qiladi.

## Autentifikatsiya

Sub-agent auth **sessiya turi** bo‘yicha emas, balki **agent id** bo‘yicha hal qilinadi:

- Sub-agent sessiya kaliti `agent:<agentId>:subagent:<uuid>`.
- Auth store shu agentning `agentDir` dan yuklanadi.
- Asosiy agentning auth profillari **fallback** sifatida qo‘shiladi; ziddiyatlarda agent profillari ustun turadi.

Eslatma: merge additive, shuning uchun asosiy profil har doim fallback sifatida mavjud. Har bir agent uchun to‘liq izolyatsiyalangan auth hozircha qo‘llab-quvvatlanmaydi.

## Announce

Sub-agentlar natijani announce bosqichi orqali qaytaradi:

- Announce bosqichi sub-agent sessiyasida ishlaydi (so‘rovchi sessiyasida emas).
- Agar sub-agent aniq `ANNOUNCE_SKIP` deb javob bersa, hech narsa joylanmaydi.
- Aks holda announce javobi so‘rovchi chat kanaliga follow-up `agent` chaqiruvi orqali (`deliver=true`) joylanadi.
- Announce javoblari mavjud bo‘lsa, thread/topic marshrutlashini saqlaydi (Slack threads, Telegram topics, Matrix threads).
- Announce xabarlari barqaror shablonga normallashtiriladi:
  - `Status:` ishga tushirish natijasidan olinadi (`success`, `error`, `timeout`, yoki `unknown`).
  - `Result:` announce bosqichidagi xulosa mazmuni (yoki mavjud bo‘lmasa `(not available)`).
  - `Notes:` xato tafsilotlari va boshqa foydali kontekst.
- `Status` model chiqishidan emas, runtime signal’laridan olinadi.

Announce payload’lari oxirida statistika qatorini o‘z ichiga oladi (hatto wrap qilinganda ham):

- Runtime (masalan, `runtime 5m12s`)
- Token sarfi (input/output/total)
- Model narxlari sozlangan bo‘lsa, taxminiy xarajat (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` va transkript yo‘li (shunda main agent `sessions_history` orqali tarixni olishi yoki diskdagi faylni ko‘rishi mumkin)

## Tool Policy (sub-agent asboblari)

Sukut bo‘yicha sub-agentlar **barcha asboblarni oladi, session asboblari** va system asboblaridan tashqari:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Agar `maxSpawnDepth >= 2` bo‘lsa, depth-1 orchestrator sub-agentlar qo‘shimcha ravishda `sessions_spawn`, `subagents`, `sessions_list`, va `sessions_history` oladi, shunda ular o‘z farzandlarini boshqara oladi.

Config orqali override:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // agar allow o‘rnatilsa, faqat allow-only bo‘ladi (deny baribir ustun)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

Sub-agentlar alohida in-process queue lane’dan foydalanadi:

- Lane nomi: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (standart `8`)

## To‘xtatish

- So‘rovchi chat’da `/stop` yuborish so‘rovchi sessiyasini bekor qiladi va undan yaratilgan barcha faol sub-agent ishga tushirishlarini, nested farzandlargacha, to‘xtatadi.
- `/subagents kill <id>` aniq sub-agentni to‘xtatadi va uning farzandlariga kaskad qiladi.

## Cheklovlar

- Sub-agent announce **best-effort**. Agar gateway qayta ishga tushsa, kutilayotgan "announce back" ishlari yo‘qoladi.
- Sub-agentlar bir xil gateway jarayon resurslarini bo‘lishadi; `maxConcurrent` ni xavfsizlik klapani sifatida ko‘ring.
- `sessions_spawn` har doim non-blocking: u darhol `{ status: "accepted", runId, childSessionKey }` qaytaradi.
- Sub-agent konteksti faqat `AGENTS.md` + `TOOLS.md` ni inject qiladi (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, yoki `BOOTSTRAP.md` yo‘q).
- Maksimal nesting chuqurligi 5 (`maxSpawnDepth` oralig‘i: 1–5). Ko‘pchilik holatlar uchun depth 2 tavsiya etiladi.
- `maxChildrenPerAgent` har bir sessiya uchun faol farzandlar sonini cheklaydi (standart: 5, oralig‘i: 1–20).

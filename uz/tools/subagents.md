---
summary: "Sub-agentlar: natijalarni so‘rovchi chatga e’lon qilib, izolyatsiyalangan agent ishlarini ishga tushirish"
read_when:
  - Agent orqali fon/parallel ishni xohlaysiz
  - Siz sessions_spawn yoki sub-agent tool siyosatini o‘zgartiryapsiz
title: "Sub-Agents"
---

# Sub-agentlar

Sub-agentlar — mavjud agent ishga tushirilishidan yaratiladigan fon (background) agent ishga tushirilishlaridir. Ular o‘z sessiyasida ishlaydi (`agent:<agentId>:subagent:<uuid>`) va yakunlangach, natijasini so‘rovchi chat kanaliga **e’lon qiladi**.

## Slash buyrug‘i

Joriy sessiya uchun sub-agent ishga tushirilishlarini ko‘rish yoki boshqarish uchun `/subagents` dan foydalaning:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

The simplest way to use sub-agents is to ask your agent naturally:

Asosiy maqsadlar:

- Asosiy ishga tushirilishni to‘smasdan “tadqiqot / uzoq vazifa / sekin vosita” ishlarini parallellashtirish.
- Sub-agentlarni sukut bo‘yicha izolyatsiya qilingan holda saqlash (sessiya ajratilishi + ixtiyoriy sandbox).
- Vositalar yuzasini noto‘g‘ri ishlatishga yo‘l qo‘ymaslik: sub-agentlar sukut bo‘yicha sessiya vositalariga ega bo‘lmaydi.
- Orkestrator andozalari uchun sozlanadigan ichma-ichlik (nesting) chuqurligini qo‘llab-quvvatlash.

Xarajat eslatmasi: har bir sub-agent o‘zining **alohida** konteksti va token sarfiga ega. Og‘ir yoki takroriy
vazifalar uchun sub-agentlarga arzonroq modelni belgilang va asosiy agentni yuqori sifatli modelda qoldiring.
Buni `agents.defaults.subagents.model` yoki har bir agent uchun alohida sozlamalar orqali sozlashingiz mumkin.

## Vosita

`sessions_spawn` dan foydalaning:

- Sub-agent ishga tushirilishini boshlaydi (`deliver: false`, global yo‘lak: `subagent`)
- So‘ng announce bosqichini bajaradi va announce javobini so‘rovchi chat kanaliga yuboradi
- Sukut bo‘yicha model: chaqiruvchidan meros oladi, agar `agents.defaults.subagents.model` (yoki har bir agent uchun `agents.list[].subagents.model`) ni sozlamagan bo‘lsangiz; aniq ko‘rsatilgan `sessions_spawn.model` ustunlik qiladi.
- Sukut bo‘yicha thinking: chaqiruvchidan meros oladi, agar `agents.defaults.subagents.thinking` (yoki har bir agent uchun `agents.list[].subagents.thinking`) ni sozlamagan bo‘lsangiz; aniq ko‘rsatilgan `sessions_spawn.thinking` ustunlik qiladi.

Vosita parametrlari:

- `task` (majburiy)
- `label?` (ixtiyoriy)
- `agentId?` (ixtiyoriy; ruxsat berilgan bo‘lsa, boshqa agent id ostida ishga tushirish)
- `model?` (ixtiyoriy; sub-agent modeli ustidan yozadi; noto‘g‘ri qiymatlar e’tiborsiz qoldiriladi va sub-agent sukut bo‘yicha modelda ishga tushadi, vosita natijasida ogohlantirish ko‘rsatiladi)
- `thinking?` (ixtiyoriy; sub-agent ishga tushirilishi uchun thinking darajasini ustidan yozadi)
- `runTimeoutSeconds?` (sukut bo‘yicha `0`; o‘rnatilsa, sub-agent ishga tushirilishi N soniyadan keyin to‘xtatiladi)
- `cleanup?` (`delete|keep`, sukut bo‘yicha `keep`)

Ruxsat ro‘yxati:

- `agents.list[].subagents.allowAgents`: `agentId` orqali nishonga olinishi mumkin bo‘lgan agent id lar ro‘yxati (`["*"]` — istalganiga ruxsat berish). Sukut bo‘yicha: faqat so‘rovchi agent.

Aniqlash:

- `sessions_spawn` uchun hozir ruxsat etilgan agent id larni ko‘rish uchun `agents_list` dan foydalaning.

Avtomatik arxivlash:

- Sub-agent sessiyalari `agents.defaults.subagents.archiveAfterMinutes` dan keyin (sukut bo‘yicha: 60) avtomatik arxivlanadi.
- Arxivlash `sessions.delete` dan foydalanadi va transkript nomini `*.deleted.<timestamp>` ga o‘zgartiradi\` (xuddi shu papka).
- `cleanup: "delete"` e’lon qilingandan so‘ng darhol arxivlaydi (transkript nomini o‘zgartirish orqali baribir saqlanadi).
- Avto-arxivlash best-effort asosida ishlaydi; agar gateway qayta ishga tushsa, kutilayotgan taymerlar yo‘qoladi.
- `runTimeoutSeconds` **avto-arxivlamaydi**; u faqat jarayonni to‘xtatadi. Sessiya avto-arxivlashgacha saqlanib qoladi.
- Avto-arxivlash depth-1 va depth-2 sessiyalariga bir xil qo‘llaniladi.

## Ichki Sub-Agentlar

Standart holatda, sub-agentlar o‘z sub-agentlarini yarata olmaydi (`maxSpawnDepth: 1`). `maxSpawnDepth: 2` qilib sozlash orqali bitta ichki darajani yoqishingiz mumkin, bu **orchestrator andozasi**ni qo‘llab-quvvatlaydi: main → orchestrator sub-agent → worker sub-sub-agentlar.

### Qanday yoqish kerak

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### Chuqurlik darajalari

| Chuqurlik | Sessiya kaliti shakli                        | Rol                                                                             | Yarata oladimi?                   |
| --------- | -------------------------------------------- | ------------------------------------------------------------------------------- | --------------------------------- |
| 0         | `agent:&lt;id&gt;:main`                            | Asosiy agent                                                                    | Har doim                          |
| 1         | `agent:&lt;id&gt;:subagent:<uuid>`                 | Sub-agent (agar depth 2 ruxsat etilgan bo‘lsa, orchestrator) | Faqat `maxSpawnDepth >= 2` bo‘lsa |
| 2         | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (yakuniy worker)                               | Hech qachon                       |

### E’lon zanjiri

Sub-agents use a dedicated queue lane (`subagent`) separate from the main agent queue, so sub-agent runs don't block inbound replies.

1. Depth-2 worker yakunlaydi → o‘z ota-agentiga (depth-1 orchestrator) e’lon yuboradi
2. Depth-1 orchestrator e’lonni qabul qiladi, natijalarni sintez qiladi, yakunlaydi → main ga e’lon yuboradi
3. Asosiy agent e’lonni qabul qiladi va foydalanuvchiga yetkazadi

Sub-agent sessions are automatically archived after a configurable period:

### Chuqurlik bo‘yicha tool siyosati

- **Depth 1 (orchestrator, `maxSpawnDepth >= 2` bo‘lganda)**: O‘z farzandlarini boshqarishi uchun `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` oladi. Boshqa sessiya/tizim toollari rad etiladi.
- **Depth 1 (leaf, `maxSpawnDepth == 1` bo‘lganda)**: Sessiya toollari yo‘q (joriy standart xatti-harakat).
- **Depth 2 (leaf worker)**: Sessiya toollari yo‘q — `sessions_spawn` depth 2 da har doim rad etiladi. Yana farzand agentlar yarata olmaydi.

### The `sessions_spawn` Tool

Har bir agent sessiyasi (istalgan chuqurlikda) bir vaqtning o‘zida ko‘pi bilan `maxChildrenPerAgent` (standart: 5) ta faol farzandga ega bo‘lishi mumkin. Bu bitta orchestrator’dan nazoratsiz ko‘payib ketishni oldini oladi.

### Parameters

Depth-1 orchestrator’ni to‘xtatish uning barcha depth-2 farzandlarini ham avtomatik ravishda to‘xtatadi:

- Asosiy chatdagi `/stop` barcha 1-darajali agentlarni to‘xtatadi va ularning 2-darajali farzand agentlariga ham ta’sir qiladi.
- `/subagents kill <id>` ma’lum bir sub-agentni to‘xtatadi va uning farzandlariga ham ta’sir qiladi.
- `/subagents kill all` so‘rov yuboruvchi uchun barcha sub-agentlarni to‘xtatadi va kaskad tarzda ta’sir qiladi.

## Autentifikatsiya

Sub-agent auth **sessiya turi** bo‘yicha emas, balki **agent id** bo‘yicha hal qilinadi:

- Sub-agent sessiya kaliti `agent:<agentId>:subagent:<uuid>` ko‘rinishida bo‘ladi.
- Auth store o‘sha agentning `agentDir` papkasidan yuklanadi.
- Asosiy agentning auth profillari **fallback** sifatida qo‘shib birlashtiriladi; ziddiyat yuzaga kelganda agent profillari asosiy profillardan ustun bo‘ladi.

Eslatma: birlashtirish qo‘shimcha (additive) tarzda amalga oshiriladi, shuning uchun asosiy profillar har doim fallback sifatida mavjud bo‘ladi. Har bir agent uchun to‘liq izolyatsiyalangan auth hozircha qo‘llab-quvvatlanmaydi.

## E’lon

Sub-agentlar natijani e’lon bosqichi orqali qaytaradi:

- E’lon bosqichi sub-agent sessiyasi ichida bajariladi (so‘rovchi sessiyasida emas).
- Agar sub-agent aynan `ANNOUNCE_SKIP` deb javob bersa, hech narsa joylanmaydi.
- Aks holda, e’lon javobi so‘rovchi chat kanaliga follow-up `agent` chaqiruvi (`deliver=true`) orqali yuboriladi.
- E’lon qilingan javoblar mavjud bo‘lsa, thread/mavzu marshrutlashini saqlab qoladi (Slack thread’lari, Telegram topic’lari, Matrix thread’lari).
- E’lon xabarlari barqaror shablonga keltiriladi:
  - `Status:` ishga tushirish natijasidan olinadi (`success`, `error`, `timeout` yoki `unknown`).
  - `Result:` e’lon bosqichidagi qisqacha mazmun (yoki mavjud bo‘lmasa `(not available)`).
  - `Notes:` xatolik tafsilotlari va boshqa foydali kontekst.
- `Status` model chiqishidan aniqlanmaydi; u runtime natija signallaridan olinadi.

E’lon payloadlari oxirida statistika qatorini o‘z ichiga oladi (hatto o‘ralgan bo‘lsa ham):

- Runtime (masalan, `runtime 5m12s`)
- Token usage (input/output/total)
- Model narxlari sozlangan bo‘lsa, taxminiy xarajat (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` va transcript yo‘li (asosiy agent `sessions_history` orqali tarixni olishi yoki diskdagi faylni tekshirishi uchun)

## Sub-agentlarni boshqarish (`/subagents`)

Joriy sessiya uchun sub-agent ishga tushirishlarini ko‘rish va boshqarish uchun `/subagents` slash-buyrug‘idan foydalaning:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Sub-agentlarga ro‘yxat indekslari (`1`, `2`), run id prefiksi, to‘liq sessiya kaliti yoki `last` orqali murojaat qilishingiz mumkin.

Config orqali o‘zgartirish:

`````json5
````
```
🧭 Subagents (current session)
Active: 1 · Done: 2
1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
```

```
/subagents stop 3
```

```
⚙️ Stop requested for deploy staging.
```
````
`````

## Concurrency

Sub-agentlar alohida in-process queue lane’dan foydalanadi:

- Lane nomi: `subagent`
- Bir vaqtda bajarilish soni: `agents.defaults.subagents.maxConcurrent` (standart `8`)

## To‘xtatish

- So‘rovchi chatida `/stop` yuborilganda, so‘rovchi sessiyasi bekor qilinadi va undan ishga tushirilgan barcha faol sub-agent jarayonlari, shu jumladan ichki farzandlari bilan birga to‘xtatiladi.
- `/subagents kill <id>` ma’lum bir sub-agentni to‘xtatadi va uning farzandlariga ham ta’sir qiladi.

## Limitations

- Sub-agent e’loni **best-effort** asosida amalga oshiriladi. Agar gateway qayta ishga tushsa, kutilayotgan "announce back" ishlari yo‘qoladi.
- Sub-agentlar hali ham bir xil gateway jarayon resurslaridan foydalanadi; `maxConcurrent` ni xavfsizlik cheklovi sifatida ko‘ring.
- `sessions_spawn` har doim non-blocking: u darhol `{ status: "accepted", runId, childSessionKey }` qaytaradi.
- Sub-agent konteksti faqat `AGENTS.md` + `TOOLS.md` ni qo‘shadi (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` yoki `BOOTSTRAP.md` yo‘q).
- Maksimal ichki joylashish chuqurligi 5 (`maxSpawnDepth` oralig‘i: 1–5). Ko‘pchilik holatlar uchun 2-daraja tavsiya etiladi.
- `maxChildrenPerAgent` har bir sessiya uchun faol childlar sonini cheklaydi (standart: 5, diapazon: 1–20).

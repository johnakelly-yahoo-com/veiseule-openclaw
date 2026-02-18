---
summary: "Chuqur tahlil: sessiya ombori + transkriptlar, hayotiy sikl va (avto)kompaktatsiya ichki mexanizmlari"
read_when:
  - Sessiya identifikatorlari, transcript JSONL yoki sessions.json maydonlarini nosozlikdan chiqarishingiz kerak bo‘lsa
  - Avto-kompaktatsiya xatti-harakatini o‘zgartirayotgan bo‘lsangiz yoki “pre-compaction” xizmat ishlarini qo‘shayotgan bo‘lsangiz
  - Xotira flush’larini yoki jim system turn’larni joriy qilmoqchi bo‘lsangiz
title: "Sessiyalarni Boshqarish: Chuqur Tahlil"
---

# Sessiyalarni Boshqarish & Kompaktatsiya (Chuqur Tahlil)

Ushbu hujjat OpenClaw sessiyalarni boshidan oxirigacha qanday boshqarishini tushuntiradi:

- **Sessiya marshrutizatsiyasi** (kiruvchi xabarlar `sessionKey` ga qanday moslanadi)
- **Sessiya ombori** (`sessions.json`) va u nimalarni kuzatadi
- **Transkriptni saqlash** (`*.jsonl`) va uning tuzilmasi
- **Transkript gigiyenasi** (ishga tushirishdan oldin provayderga xos tuzatishlar)
- **Kontekst cheklovlari** (kontekst oynasi va kuzatiladigan tokenlar)
- **Kompaktatsiya** (qo‘lda + avto-kompaktatsiya) va pre-kompaktatsiya ishlarini qayerga ulash mumkinligi
- **Yashirin xizmat ishlari** (masalan, foydalanuvchiga ko‘rinmas chiqishsiz xotira yozuvlari)

Agar avval yuqori darajadagi umumiy ko‘rinishni xohlasangiz, quyidagilardan boshlang:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## Haqiqat manbai: Gateway

OpenClaw yagona **Gateway jarayoni** atrofida loyihalangan bo‘lib, u sessiya holatini boshqaradi.

- UI’lar (macOS ilovasi, veb Control UI, TUI) sessiyalar ro‘yxati va token hisoblari uchun Gateway’dan so‘rov yuborishi kerak.
- Masofaviy rejimda sessiya fayllari masofaviy xostda bo‘ladi; “lokal Mac fayllarini tekshirish” Gateway foydalanayotgan ma’lumotni aks ettirmaydi.

---

## Ikki saqlash qatlami

OpenClaw sessiyalarni ikki qatlamda saqlaydi:

1. **Sessiya ombori (`sessions.json`)**
   - Kalit/qiymat xaritasi: `sessionKey -> SessionEntry`
   - Kichik, o‘zgaruvchan, tahrirlash (yoki yozuvlarni o‘chirish) xavfsiz
   - Sessiya metama’lumotlarini kuzatadi (joriy sessiya identifikatori, oxirgi faollik, toggle’lar, token hisoblagichlari va h.k.)

2. **Transkript (`<sessionId>.jsonl`)**
   - Daraxt tuzilmasiga ega append-only transkript (`id` + `parentId`)
   - Haqiqiy suhbat + tool chaqiruvlari + kompaktatsiya xulosalarini saqlaydi
   - Keyingi turn’lar uchun model kontekstini qayta tiklashda ishlatiladi

---

## Diskdagi joylashuv

Har bir agent uchun, Gateway xostida:

- Ombor: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transkriptlar: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Telegram mavzu sessiyalari: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw bularni `src/config/sessions.ts` orqali aniqlaydi.

---

## Sessiya kalitlari (`sessionKey`)

`sessionKey` siz qaysi _suhbat konteynerida_ ekaningizni aniqlaydi (marshrutizatsiya + izolyatsiya).

Keng tarqalgan andozalar:

- Asosiy/to‘g‘ridan-to‘g‘ri chat (har bir agent uchun): `agent:<agentId>:<mainKey>` (standart `main`)
- Guruh: `agent:<agentId>:<channel>:group:<id>`
- Xona/kanal (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` yoki `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (agar alohida belgilanmagan bo‘lsa)

Kanonik qoidalar [/concepts/session](/concepts/session) da hujjatlashtirilgan.

---

## Sessiya identifikatorlari (`sessionId`)

Har bir `sessionKey` joriy `sessionId` ga ishora qiladi (suhbat davom etadigan transkript fayli).

Asosiy qoidalar:

- **Reset** (`/new`, `/reset`) ushbu `sessionKey` uchun yangi `sessionId` yaratadi.
- **Kundalik reset** (standart: Gateway xostining lokal vaqti bilan 4:00) reset chegarasidan keyingi birinchi xabarda yangi `sessionId` yaratadi.
- **Idle muddati tugashi** (`session.reset.idleMinutes` yoki eski `session.idleMinutes`) idle oynasidan keyin xabar kelganda yangi `sessionId` yaratadi. Agar kundalik + idle ikkalasi ham sozlangan bo‘lsa, birinchi tugagan ustun bo‘ladi.

Implementatsiya tafsiloti: qaror `src/auto-reply/reply/session.ts` ichidagi `initSessionState()` da qabul qilinadi.

---

## Sessiya ombori sxemasi (`sessions.json`)

Ombordagi qiymat turi `src/config/sessions.ts` dagi `SessionEntry`.

Asosiy maydonlar (to‘liq emas):

- `sessionId`: joriy transkript identifikatori (agar `sessionFile` o‘rnatilmagan bo‘lsa, fayl nomi shundan olinadi)
- `updatedAt`: oxirgi faollik vaqti
- `sessionFile`: ixtiyoriy aniq transkript yo‘li
- `chatType`: `direct | group | room` (UI va yuborish siyosati uchun)
- `provider`, `subject`, `room`, `space`, `displayName`: guruh/kanal yorlig‘i uchun metama’lumot
- Toggle’lar:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (sessiya darajasida override)
- Model tanlash:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Token hisoblagichlari (eng yaxshi taxmin / provayderga bog‘liq):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: ushbu sessiya kaliti uchun avto-kompaktatsiya nechta bajarilgan
- `memoryFlushAt`: oxirgi pre-kompaktatsiya memory flush vaqti
- `memoryFlushCompactionCount`: oxirgi flush qaysi kompaktatsiya sonida bajarilgan

Omborni tahrirlash xavfsiz, ammo Gateway — asosiy manba: sessiyalar ishlayotganda yozuvlarni qayta yozishi yoki tiklashi mumkin.

---

## Transkript tuzilmasi (`*.jsonl`)

Transkriptlar `@mariozechner/pi-coding-agent` dagi `SessionManager` tomonidan boshqariladi.

Fayl JSONL formatida:

- Birinchi qatorda: sessiya sarlavhasi (`type: "session"`, `id`, `cwd`, `timestamp`, ixtiyoriy `parentSession`)
- Keyin: `id` + `parentId` ga ega sessiya yozuvlari (daraxt)

Muhim yozuv turlari:

- `message`: user/assistant/toolResult xabarlari
- `custom_message`: kengaytma kiritgan va model kontekstiga _kiradigan_ xabarlar (UI’da yashirilishi mumkin)
- `custom`: model kontekstiga _kirmaydigan_ kengaytma holati
- `compaction`: `firstKeptEntryId` va `tokensBefore` bilan saqlangan kompaktatsiya xulosasi
- `branch_summary`: daraxt shoxobchasida navigatsiya qilinganda saqlanadigan xulosa

OpenClaw ataylab transkriptlarni “tuzatmaydi”; Gateway ularni o‘qish/yozish uchun `SessionManager` dan foydalanadi.

---

## Kontekst oynasi va kuzatiladigan tokenlar

Ikki xil tushuncha muhim:

1. **Model kontekst oynasi**: model uchun qat’iy limit (model ko‘ra oladigan tokenlar soni)
2. **Sessiya ombori hisoblagichlari**: `sessions.json` ga yoziladigan aylanma statistika ( `/status` va dashboard’lar uchun)

Limitlarni sozlayotgan bo‘lsangiz:

- Kontekst oynasi model katalogidan olinadi (va config orqali override qilinishi mumkin).
- Ombordagi `contextTokens` — ish vaqtidagi taxmin/hisobot qiymati; uni qat’iy kafolat deb qabul qilmang.

Batafsil: [/token-use](/reference/token-use).

---

## Kompaktatsiya: bu nima

Kompaktatsiya eski suhbatni xulosa qilib, transkriptga saqlangan `compaction` yozuvini qo‘shadi va so‘nggi xabarlarni saqlab qoladi.

Kompaktatsiyadan keyin, keyingi turn’lar quyidagilarni ko‘radi:

- Kompaktatsiya xulosasi
- `firstKeptEntryId` dan keyingi xabarlar

Kompaktatsiya **doimiy** (sessiya pruning’dan farqli). Qarang: [/concepts/session-pruning](/concepts/session-pruning).

---

## Avto-kompaktatsiya qachon sodir bo‘ladi (Pi runtime)

O‘rnatilgan Pi agentida avto-kompaktatsiya ikki holatda ishga tushadi:

1. **Overflow tiklash**: model kontekst overflow xatosini qaytaradi → kompaktatsiya → qayta urinish.
2. **Threshold nazorati**: muvaffaqiyatli turn’dan keyin, quyidagi shart bajarilganda:

`contextTokens > contextWindow - reserveTokens`

Bu yerda:

- `contextWindow` — modelning kontekst oynasi
- `reserveTokens` — promptlar + keyingi model javobi uchun ajratilgan zahira

Bu Pi runtime semantikasi (OpenClaw hodisalarni qabul qiladi, lekin kompaktatsiya qachon bo‘lishini Pi hal qiladi).

---

## Kompaktatsiya sozlamalari (`reserveTokens`, `keepRecentTokens`)

Pi kompaktatsiya sozlamalari Pi settings’da joylashgan:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw o‘rnatilgan ishga tushirishlar uchun xavfsizlik minimumini ham qo‘llaydi:

- Agar `compaction.reserveTokens < reserveTokensFloor` bo‘lsa, OpenClaw uni oshiradi.
- Standart minimum: `20000` token.
- Minimumni o‘chirish uchun: `agents.defaults.compaction.reserveTokensFloor: 0`.
- Agar allaqachon yuqori bo‘lsa, OpenClaw o‘zgartirmaydi.

Sabab: kompaktatsiya muqarrar bo‘lishidan oldin ko‘p bosqichli “xizmat ishlari” (masalan, xotira yozuvlari) uchun yetarli zahira qoldirish.

Implementatsiya: `ensurePiCompactionReserveTokens()` (`src/agents/pi-settings.ts`)
(`src/agents/pi-embedded-runner.ts` dan chaqiriladi).

---

## Foydalanuvchiga ko‘rinadigan joylar

Kompaktatsiya va sessiya holatini quyidagilar orqali kuzatishingiz mumkin:

- `/status` (har qanday chat sessiyasida)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Verbose rejim: `🧹 Auto-compaction complete` + kompaktatsiya soni

---

## Yashirin xizmat turn’lari (`NO_REPLY`)

OpenClaw fon vazifalari uchun, foydalanuvchi oraliq chiqishni ko‘rmasligi kerak bo‘lgan “jim” turn’larni qo‘llab-quvvatlaydi.

Kelishuv:

- Assistant chiqishini `NO_REPLY` bilan boshlaydi — bu “foydalanuvchiga javob yuborilmasin” degani.
- OpenClaw yetkazish qatlamida buni olib tashlaydi/yashiradi.

`2026.1.10` dan boshlab, agar qisman streaming bo‘lagi `NO_REPLY` bilan boshlansa, OpenClaw **draft/typing streaming** ni ham bostiradi, shuning uchun jim operatsiyalar o‘rtada chiqib ketmaydi.

---

## Pre-kompaktatsiya “memory flush” (joriy qilingan)

Maqsad: avto-kompaktatsiyadan oldin, agent ish maydonida diskka doimiy
holat yozadigan (masalan, `memory/YYYY-MM-DD.md`) jim agentik turn ishga tushirish, shunda kompaktatsiya muhim kontekstni o‘chirib yubormaydi.

OpenClaw **pre-threshold flush** yondashuvidan foydalanadi:

1. Sessiya kontekst foydalanishini kuzatish.
2. U “yumshoq threshold” (Pi kompaktatsiya threshold’idan past) dan oshganda, agentga jim
   “xotirani hozir yoz” direktivasini yuborish.
3. Foydalanuvchi hech narsa ko‘rmasligi uchun `NO_REPLY` dan foydalanish.

Config (`agents.defaults.compaction.memoryFlush`):

- `enabled` (standart: `true`)
- `softThresholdTokens` (standart: `4000`)
- `prompt` (flush turn uchun user xabari)
- `systemPrompt` (flush turn uchun qo‘shimcha system prompt)

Izohlar:

- Standart prompt/system prompt yetkazishni bostirish uchun `NO_REPLY` ko‘rsatmasini o‘z ichiga oladi.
- Flush har bir kompaktatsiya siklida bir marta ishlaydi (`sessions.json` da kuzatiladi).
- Flush faqat o‘rnatilgan Pi sessiyalari uchun ishlaydi (CLI backend’lar o‘tkazib yuboradi).
- Agar sessiya workspace’i faqat o‘qish rejimida bo‘lsa (`workspaceAccess: "ro"` yoki `"none"`), flush o‘tkazib yuboriladi.
- Workspace fayl tuzilmasi va yozish andozalari uchun [Memory](/concepts/memory) ga qarang.

Pi extension API’da `session_before_compact` hook ham mavjud, ammo OpenClaw’ning
flush logikasi hozircha Gateway tomonida joylashgan.

---

## Muammolarni bartaraf etish ro‘yxati

- Sessiya kaliti noto‘g‘rimi? [/concepts/session](/concepts/session) dan boshlang va `/status` dagi `sessionKey` ni tekshiring.
- Ombor va transkript mos kelmayaptimi? Gateway xostini va `openclaw status` dagi ombor yo‘lini tekshiring.
- Kompaktatsiya juda tez-tezmi? Tekshiring:
  - model kontekst oynasi (juda kichik emasmi)
  - kompaktatsiya sozlamalari (`reserveTokens` model oynasiga nisbatan juda yuqori bo‘lsa, erta kompaktatsiya bo‘lishi mumkin)
  - tool-result hajmi: session pruning’ni yoqing/moslang
- Jim turn’lar chiqib ketayaptimi? Javob aynan `NO_REPLY` bilan boshlanishini va streaming bostirish tuzatmasi mavjud build’dan foydalanayotganingizni tekshiring.

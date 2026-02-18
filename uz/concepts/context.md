---
summary: "Kontekst: model nimani ko‘radi, u qanday yig‘iladi va uni qanday tekshirish mumkin"
read_when:
  - Siz OpenClaw’da “kontekst” nimani anglatishini tushunmoqchisiz
  - Model nima uchun nimanidir “bilishini” (yoki unutganini) nosozlikdan o‘tkazyapsiz
  - Kontekst yukini kamaytirmoqchisiz (/context, /status, /compact)
title: "Kontekst"
---

# Kontekst

“Kontekst” — bu **OpenClaw bitta ishga tushirish uchun modelga yuboradigan hamma narsa**. U modelning **kontekst oynasi** (token limiti) bilan cheklangan.

Boshlovchilar uchun mental model:

- **Tizim prompti** (OpenClaw tomonidan yaratilgan): qoidalar, asboblar, ko‘nikmalar ro‘yxati, vaqt/ijro muhiti va kiritilgan ishchi makon fayllari.
- **Suhbat tarixi**: ushbu sessiya uchun sizning xabarlaringiz + yordamchining xabarlari.
- **Asbob chaqiruvlari/natijalari + ilovalar**: buyruq chiqishi, fayl o‘qishlar, tasvirlar/audio va hokazo.

Kontekst “xotira” bilan _bir xil narsa emas_: xotira diskda saqlanib, keyinroq qayta yuklanishi mumkin; kontekst esa modelning joriy oynasi ichida bo‘lgan narsadir.

## Tezkor boshlash (kontekstni tekshirish)

- `/status` → “oynam qanchalik to‘ldi?” degan tezkor ko‘rinish + sessiya sozlamalari.
- `/context list` → nimalar kiritilgan + taxminiy hajmlar (har bir fayl + jami).
- `/context detail` → chuqurroq tafsilot: har bir fayl bo‘yicha, har bir asbob sxemasi hajmi, har bir ko‘nikma yozuvi hajmi va tizim prompti hajmi.
- `/usage tokens` → odatiy javoblarga har javob uchun foydalanish futerini qo‘shadi.
- `/compact` → eski tarixni ixcham yozuvga umumlashtirib, oyna joyini bo‘shatadi.

Shuningdek qarang: [Slash commands](/tools/slash-commands), [Token use & costs](/reference/token-use), [Compaction](/concepts/compaction).

## Namunaviy chiqish

Qiymatlar model, provayder, asbob siyosati va ishchi makoningizdagi tarkibga qarab farqlanadi.

### `/context list`

```
🧠 Kontekst taqsimoti
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Kontekst taqsimoti (batafsil)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Kontekst oynasiga nimalar hisoblanadi

Model qabul qiladigan hamma narsa hisobga olinadi, jumladan:

- System prompt (barcha bo‘limlar).
- Suhbat tarixi.
- Asbob chaqiruvlari + asbob natijalari.
- Biriktirmalar/transkriptlar (rasmlar/audio/fayllar).
- Kompaktlash xulosalari va pruning artefaktlari.
- Provider “wrapper”lari yoki yashirin sarlavhalar (ko‘rinmaydi, lekin baribir hisoblanadi).

## OpenClaw system promptni qanday quradi

System prompt **OpenClaw-ga tegishli** va har ishga tushirishda qayta tuziladi. U quyidagilarni o‘z ichiga oladi:

- Asboblar ro‘yxati + qisqa tavsiflar.
- Ko‘nikmalar ro‘yxati (faqat metadata; quyida qarang).
- Ishchi maydon joylashuvi.
- Vaqt (UTC + agar sozlangan bo‘lsa foydalanuvchi vaqti).
- Ish vaqti metama’lumotlari (host/OS/model/thinking).
- To‘liq tahlil: [System Prompt](/concepts/system-prompt).

`AGENTS.md`

## Kiritilgan ishchi maydon fayllari (Project Context)

Standart bo‘yicha, OpenClaw mavjud bo‘lsa ishchi maydonning qat’iy belgilangan fayllar to‘plamini kiritadi:

- System prompt ixcham **skills ro‘yxati**ni (nomi + tavsifi + joylashuvi) o‘z ichiga oladi.
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (faqat birinchi ishga tushirishda)

Katta fayllar har bir fayl bo‘yicha `agents.defaults.bootstrapMaxChars` (standart `20000` belgi) yordamida qisqartiriladi. `/context` **raw vs injected** o‘lchamlarini va qisqartirish bo‘lgan-bo‘lmaganini ko‘rsatadi.

## Ko‘nikmalar: qaysilari kiritiladi va qaysilari talab bo‘yicha yuklanadi

Mattermost va boshqalarni kengaytmalar bilan qo‘shing. Bu ro‘yxat sezilarli yuklama keltiradi.

Ko‘nikma ko‘rsatmalari standart bo‘yicha kiritilmaydi. Modeldan ko‘nikmaning `SKILL.md` faylini **faqat kerak bo‘lganda** `o‘qishi` kutiladi.

## Asboblar: ikkita xarajat turi mavjud

Asboblar kontekstga ikki yo‘l bilan ta’sir qiladi:

1. System promptdagi **asboblar ro‘yxati matni** (siz “Tooling” sifatida ko‘radigan narsa).
2. **Asbob sxemalari** (JSON). Bular model asboblarni chaqira olishi uchun unga yuboriladi. Ular oddiy matn sifatida ko‘rinmasa ham, kontekstga qo‘shiladi.

`/context detail` eng katta asbob sxemalarini tafsilotlab, nimasi ustun ekanini ko‘rish imkonini beradi.

## Buyruqlar, direktivalar va “inline shortcuts”

Slash buyruqlar Gateway tomonidan qayta ishlanadi. Bir nechta turli xatti-harakatlar mavjud:

- **Mustaqil buyruqlar**: faqat `/...` dan iborat xabar buyruq sifatida bajariladi.
- **Direktivalar**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` model xabarni ko‘rishidan oldin olib tashlanadi.
  - Faqat direktivadan iborat xabarlar sessiya sozlamalarini saqlab qoladi.
  - Oddiy xabardagi inline direktivalar har bir xabar uchun maslahat sifatida ishlaydi.
- **Inline shortcuts** (faqat ruxsat etilgan jo‘natuvchilar): oddiy xabar ichidagi ayrim `/...` tokenlar darhol ishga tushishi mumkin (masalan: “hey /status”), va qolgan matn modelga ko‘rsatilishidan oldin olib tashlanadi.

Tafsilotlar: [Slash commands](/tools/slash-commands).

## Sessiyalar, kompaktlash va pruning (nimalar saqlanib qoladi)

What persists across messages depends on the mechanism:

- **Normal history** persists in the session transcript until compacted/pruned by policy.
- **Compaction** persists a summary into the transcript and keeps recent messages intact.
- **Pruning** removes old tool results from the _in-memory_ prompt for a run, but does not rewrite the transcript.

Docs: [Session](/concepts/session), [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning).

## What `/context` actually reports

`/context` prefers the latest **run-built** system prompt report when available:

- `System prompt (run)` = captured from the last embedded (tool-capable) run and persisted in the session store.
- `System prompt (estimate)` = computed on the fly when no run report exists (or when running via a CLI backend that doesn’t generate the report).

Either way, it reports sizes and top contributors; it does **not** dump the full system prompt or tool schemas.

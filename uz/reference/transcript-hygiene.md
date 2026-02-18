---
title: "5. Transkript gigiyenasi"
---

# Transkript gigiyenasi (Provayder tuzatishlari)

Ushbu hujjat ishga tushirishdan oldin transkriptlarga qo‘llaniladigan **provayderga xos tuzatishlar**ni tavsiflaydi
(model kontekstini yaratish). 8. Bular qat’iy provayder talablarini qondirish uchun ishlatiladigan **xotira ichidagi** sozlashlardir. Bu gigiyena bosqichlari diskda saqlangan JSONL transkriptini **qayta yozmaydi**; biroq, alohida sessiya-faylni tuzatish jarayoni sessiya yuklanishidan oldin yaroqsiz satrlarni olib tashlash orqali noto‘g‘ri JSONL fayllarni qayta yozishi mumkin. Tuzatish amalga oshirilganda, asl fayl sessiya fayli bilan yonma-yon zaxiralab qo‘yiladi.

Qamrovga quyidagilar kiradi:

- 12. Tool call id sanitizatsiyasi
- Asbob chaqiruvi kiritmalarini tekshirish
- 14. Tool natijalarini juftlashni tuzatish
- Navbat (turn)ni tekshirish / tartiblash
- Fikr (thought) imzosini tozalash
- 17. Rasm payload sanitizatsiyasi

18. Agar transkriptni saqlash tafsilotlari kerak bo‘lsa, qarang:

- 19. [/reference/session-management-compaction](/reference/session-management-compaction)

---

## 20. Qayerda bajariladi

21. Barcha transkript gigiyenasi ichki (embedded) runner’da markazlashtirilgan:

- 22. Siyosat tanlash: `src/agents/transcript-policy.ts`
- 23. Sanitizatsiya/tuzatishni qo‘llash: `sanitizeSessionHistory` faylida `src/agents/pi-embedded-runner/google.ts`

24. Siyosat qaysi qo‘llanishini aniqlash uchun `provider`, `modelApi` va `modelId` dan foydalanadi.

25. Transkript gigiyenasidan alohida ravishda, sessiya fayllari yuklashdan oldin (kerak bo‘lsa) tuzatiladi:

- 26. `repairSessionFileIfNeeded` faylida `src/agents/session-file-repair.ts`
- 27. `run/attempt.ts` va `compact.ts` dan chaqiriladi (embedded runner)

---

## 28. Global qoida: rasm sanitizatsiyasi

29. Provayder tomonida o‘lcham cheklovlari sababli rad etilishining oldini olish uchun rasm payloadlari har doim sanitizatsiya qilinadi
    (haddan katta base64 rasmlarni kichraytirish/qayta siqish).

30. Amalga oshirish:

- 31. `sanitizeSessionMessagesImages` faylida `src/agents/pi-embedded-helpers/images.ts`
- 32. `sanitizeContentBlocksImages` faylida `src/agents/tool-images.ts`

---

## 33. Global qoida: noto‘g‘ri tool call’lar

34. `input` ham, `arguments` ham yetishmaydigan assistent tool-call bloklari
    model konteksti qurilishidan oldin olib tashlanadi. 35. Bu qisman saqlanib qolgan tool call’lar (masalan,
    rate limit xatosidan keyin) sababli provayder rad etilishining oldini oladi.

36. Amalga oshirish:

- 37. `sanitizeToolCallInputs` faylida `src/agents/session-transcript-repair.ts`
- 38. `sanitizeSessionHistory` da qo‘llaniladi (`src/agents/pi-embedded-runner/google.ts`)

---

## 39. Provayderlar matriksi (joriy xatti-harakat)

40. **OpenAI / OpenAI Codex**

- 41. Faqat rasm sanitizatsiyasi.
- 42. OpenAI Responses/Codex’ga model almashtirilganda, yetim qolgan reasoning imzolari (keyingi kontent bloki bo‘lmagan mustaqil reasoning elementlari) olib tashlanadi.
- 43. Tool call id sanitizatsiyasi yo‘q.
- 44. Tool natijalarini juftlashni tuzatish yo‘q.
- 45. Navbat (turn) validatsiyasi yoki qayta tartiblash yo‘q.
- 46. Sintetik tool natijalari yo‘q.
- 47. Fikr (thought) imzolarini olib tashlash yo‘q.

48. **Google (Generative AI / Gemini CLI / Antigravity)**

- 49. Tool call id sanitizatsiyasi: qat’iy alfanumerik.
- 50. Tool natijalarini juftlashni tuzatish va sintetik tool natijalari.
- Turn validation (Gemini-style turn alternation).
- Google turn ordering fixup (prepend a tiny user bootstrap if history starts with assistant).
- Antigravity Claude: normalize thinking signatures; drop unsigned thinking blocks.

**Anthropic / Minimax (Anthropic-compatible)**

- Asbob natijalarini juftlashni tuzatish va sintetik asbob natijalari.
- Turn validation (merge consecutive user turns to satisfy strict alternation).

**Mistral (including model-id based detection)**

- Tool call id sanitization: strict9 (alphanumeric length 9).

**OpenRouter Gemini**

- Thought signature cleanup: strip non-base64 `thought_signature` values (keep base64).

**Everything else**

- Image sanitization only.

---

## Historical behavior (pre-2026.1.22)

Before the 2026.1.22 release, OpenClaw applied multiple layers of transcript hygiene:

- A **transcript-sanitize extension** ran on every context build and could:
  - Repair tool use/result pairing.
  - Sanitize tool call ids (including a non-strict mode that preserved `_`/`-`).
- The runner also performed provider-specific sanitization, which duplicated work.
- Additional mutations occurred outside the provider policy, including:
  - Stripping `<final>` tags from assistant text before persistence.
  - Dropping empty assistant error turns.
  - Asbob chaqiruvlaridan keyin yordamchi kontentni qisqartirish.

This complexity caused cross-provider regressions (notably `openai-responses`
`call_id|fc_id` pairing). The 2026.1.22 cleanup removed the extension, centralized
logic in the runner, and made OpenAI **no-touch** beyond image sanitization.


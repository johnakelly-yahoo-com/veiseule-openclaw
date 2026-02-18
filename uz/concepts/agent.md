---
summary: "6. Agent runtime (ichki pi-mono), ish maydoni shartnomasi va sessiyani ishga tushirish"
read_when:
  - 7. Agent runtime’ini, ish maydonini ishga tushirishni yoki sessiya xatti-harakatini o‘zgartirish
title: "8. Agent Runtime"
---

# 9. Agent Runtime 🤖

Workspace (majburiy)

## `SOUL.md` — persona, chegaralar, ohang

12. OpenClaw agent uchun **yagona** ishchi katalog (`cwd`) sifatida asboblar va kontekst uchun bitta agent ish maydoni katalogidan (`agents.defaults.workspace`) foydalanadi.

13. Tavsiya etiladi: agar mavjud bo‘lmasa, `~/.openclaw/openclaw.json` ni yaratish va ish maydoni fayllarini boshlang‘ich holatga keltirish uchun `openclaw setup` dan foydalaning.

14. Ish maydonining to‘liq tuzilishi va zaxira nusxa qo‘llanmasi: [Agent workspace](/concepts/agent-workspace)

15. Agar `agents.defaults.sandbox` yoqilgan bo‘lsa, asosiy bo‘lmagan sessiyalar buni `agents.defaults.sandbox.workspaceRoot` ostidagi sessiya-bo‘yicha ish maydonlari bilan almashtirishi mumkin (qarang: [Gateway configuration](/gateway/configuration)).

## 16. Bootstrap fayllari (kiritiladi)

17. `agents.defaults.workspace` ichida OpenClaw quyidagi foydalanuvchi tahrirlashi mumkin bo‘lgan fayllarni kutadi:

- 18. `AGENTS.md` — ishlash bo‘yicha ko‘rsatmalar + “xotira”
- Queue rejimi `steer` bo‘lganda, kiruvchi xabarlar joriy run ichiga kiritiladi.
- 20. `TOOLS.md` — foydalanuvchi tomonidan yuritiladigan asboblar bo‘yicha eslatmalar (masalan, `imsg`, `sag`, konventsiyalar)
- 21. `BOOTSTRAP.md` — bir martalik birinchi ishga tushirish marosimi (yakunlangach o‘chiriladi)
- 22. `IDENTITY.md` — agent nomi / vibe / emoji
- 23. `USER.md` — foydalanuvchi profili + afzal murojaat shakli

24. Yangi sessiyaning birinchi navbatida OpenClaw ushbu fayllarning mazmunini to‘g‘ridan-to‘g‘ri agent kontekstiga kiritadi.

25. Bo‘sh fayllar o‘tkazib yuboriladi. 26. Katta fayllar qisqartiriladi va belgi bilan kesib tashlanadi, shunda promptlar ixcham bo‘lib qoladi (to‘liq mazmun uchun faylni o‘qing).

27. Agar fayl mavjud bo‘lmasa, OpenClaw bitta “fayl yo‘q” belgisi qatorini kiritadi (va `openclaw setup` xavfsiz standart shablonni yaratadi).

28. `BOOTSTRAP.md` faqat **mutlaqo yangi ish maydoni** uchun yaratiladi (boshqa bootstrap fayllari mavjud bo‘lmaganida). 29. Marosimni tugatgach uni o‘chirib tashlasangiz, keyingi qayta ishga tushirishlarda u qayta yaratilmasligi kerak.

30. Bootstrap fayllarini yaratishni butunlay o‘chirish uchun (oldindan to‘ldirilgan ish maydonlari uchun), quyidagini sozlang:

```json5
31. { agent: { skipBootstrap: true } }
```

## 32. Ichki asboblar

33. Asosiy asboblar (o‘qish/ijro/tahrirlash/yozish va tegishli tizim asboblari) asboblar siyosatiga bo‘ysungan holda har doim mavjud. 34. `apply_patch` ixtiyoriy va
    `tools.exec.applyPatch` orqali cheklanadi. 35. `TOOLS.md` qaysi asboblar mavjudligini **boshqarmaydi**; u ularni _siz_ qanday ishlatilishini xohlayotganingiz bo‘yicha ko‘rsatma beradi.

## 36. Ko‘nikmalar

37. OpenClaw ko‘nikmalarni uchta joydan yuklaydi (nomlar to‘qnashganda ish maydoni ustun):

- 38. Paketlangan (o‘rnatish bilan birga yetkaziladi)
- 39. Boshqariladigan/mahalliy: `~/.openclaw/skills`
- 40. Ish maydoni: `<workspace>/skills`

41. Ko‘nikmalar konfiguratsiya/muhit orqali cheklanishi mumkin (qarang: [Gateway configuration](/gateway/configuration) dagi `skills`).

## 42. pi-mono integratsiyasi

43. OpenClaw pi-mono kod bazasining ayrim qismlaridan (modelllar/asboblar) qayta foydalanadi, ammo **sessiyalarni boshqarish, aniqlash va asboblarni ulash OpenClaw’ga tegishli**.

- 44. pi-coding agent runtime yo‘q.
- 45. `~/.pi/agent` yoki `<workspace>/.pi` sozlamalari hisobga olinmaydi.

## 46. Sessiyalar

47. Sessiya transkriptlari JSONL formatida quyida saqlanadi:

- 48. `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

49. Sessiya ID barqaror bo‘lib, OpenClaw tomonidan tanlanadi.
50. Meros bo‘lib qolgan Pi/Tau sessiya papkalari **o‘qilmaydi**.

## Steering while streaming

Ulanish hayotiy sikli (bitta mijoz)
The queue is checked **after each tool call**; if a queued message is present,
remaining tool calls from the current assistant message are skipped (error tool
results with "Skipped due to queued user message."), then the queued user
message is injected before the next assistant response.

When queue mode is `followup` or `collect`, inbound messages are held until the
current turn ends, then a new agent turn starts with the queued payloads. See
[Queue](/concepts/queue) for mode + debounce/cap behavior.

Block streaming sends completed assistant blocks as soon as they finish; it is
**off by default** (`agents.defaults.blockStreamingDefault: "off"`).
Tune the boundary via `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; defaults to text_end).
Control soft block chunking with `agents.defaults.blockStreamingChunk` (defaults to
800–1200 chars; prefers paragraph breaks, then newlines; sentences last).
Coalesce streamed chunks with `agents.defaults.blockStreamingCoalesce` to reduce
single-line spam (idle-based merging before send). Non-Telegram channels require
explicit `*.blockStreaming: true` to enable block replies.
Verbose tool summaries are emitted at tool start (no debounce); Control UI
streams tool output via agent events when available.
More details: [Streaming + chunking](/concepts/streaming).

## Model refs

Model refs in config (for example `agents.defaults.model` and `agents.defaults.models`) are parsed by splitting on the **first** `/`.

- Use `provider/model` when configuring models.
- If the model ID itself contains `/` (OpenRouter-style), include the provider prefix (example: `openrouter/moonshotai/kimi-k2`).
- If you omit the provider, OpenClaw treats the input as an alias or a model for the **default provider** (only works when there is no `/` in the model ID).

## Configuration (minimal)

At minimum, set:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (strongly recommended)

---

_Keyingi: [Guruh chatlari](/channels/group-messages)_ 🦞

---
summary: "36. Xabarlar oqimi, sessiyalar, navbatga qo‘yish va mulohaza (reasoning) ko‘rinishi"
read_when:
  - 37. Kiruvchi xabarlar qanday qilib javoblarga aylanishini tushuntirish
  - 38. Sessiyalar, navbatga qo‘yish rejimlari yoki streaming xatti-harakatlarini aniqlashtirish
  - 39. Mulohaza (reasoning) ko‘rinishi va undan foydalanish oqibatlarini hujjatlashtirish
title: "40. Xabarlar"
---

# 41. Xabarlar

42. Ushbu sahifa OpenClaw kiruvchi xabarlar, sessiyalar, navbatga qo‘yish,
    streaming va mulohaza ko‘rinishini qanday boshqarishini birlashtirib tushuntiradi.

## 43. Xabar oqimi (yuqori daraja)

```
44. Kiruvchi xabar
  -> marshrutlash/bog‘lash -> sessiya kaliti
  -> navbat (agar run faol bo‘lsa)
  -> agent runi (streaming + vositalar)
  -> chiquvchi javoblar (kanal cheklovlari + bo‘laklash)
```

45. Asosiy sozlamalar konfiguratsiyada joylashgan:

- 46. Prefikslar, navbatga qo‘yish va guruh xatti-harakati uchun `messages.*`.
- 47. Blokli streaming va bo‘laklash sukut bo‘yicha sozlamalari uchun `agents.defaults.*`.
- 48. Kanal bo‘yicha override’lar (`channels.whatsapp.*`, `channels.telegram.*`, va h.k.) 49. limitlar va streaming yoqib/o‘chirishlari uchun.

50. To‘liq sxema uchun [Configuration](/gateway/configuration) sahifasiga qarang.

## 1. Kiruvchi deduplikatsiya

2. Qayta ulanishlardan so‘ng kanallar bir xil xabarni qayta yetkazishi mumkin. 3. OpenClaw kanal/hisob/peer/sessiya/xabar identifikatori bo‘yicha kalitlangan qisqa muddatli keshni saqlaydi, shuning uchun takroriy yetkazib berishlar agentni yana ishga tushirmaydi.

## 4. Kiruvchi debouncing

5. **Bir xil yuboruvchidan** tez-tez ketma-ket kelgan xabarlar `messages.inbound` orqali bitta agent navbatiga birlashtirilishi mumkin. 6. Debouncing kanal + suhbat bo‘yicha chegaralanadi va javob ipi/IDlar uchun eng so‘nggi xabardan foydalanadi.

7. Konfiguratsiya (global sukut bo‘yicha + kanal bo‘yicha o‘zgartirishlar):

```json5
8. {
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

9. Eslatmalar:

- 10. Debounce **faqat matnli** xabarlarga qo‘llanadi; media/ilovalar darhol yuboriladi.
- 11. Boshqaruv buyruqlari debouncingni chetlab o‘tadi, shuning uchun ular alohida qoladi.

## 12. Sessiyalar va qurilmalar

13. Sessiyalar mijozlar tomonidan emas, balki shlyuz tomonidan boshqariladi.

- 14. To‘g‘ridan-to‘g‘ri chatlar agentning asosiy sessiya kalitiga birlashtiriladi.
- Tafsilotlar: [Session management](/concepts/session).
- 16. Sessiya ombori va transkriptlar shlyuz xostida joylashadi.

17. Bir nechta qurilmalar/kanallar bir xil sessiyaga mos kelishi mumkin, ammo tarix har bir mijozga to‘liq sinxronlanmaydi. 18. Tavsiya: kontekstning ajralib ketishidan qochish uchun uzoq suhbatlarda bitta asosiy qurilmadan foydalaning. 19. Control UI va TUI har doim shlyuzga tayangan sessiya transkriptini ko‘rsatadi, shuning uchun ular haqiqat manbai hisoblanadi.

20. Tafsilotlar: [Session management](/concepts/session).

## 21. Kiruvchi bodylar va tarix konteksti

22. OpenClaw **prompt body**ni **command body**dan ajratadi:

- 23. `Body`: agentga yuboriladigan prompt matni. 24. Bu kanal konvertlari va ixtiyoriy tarix o‘ramlarini o‘z ichiga olishi mumkin.
- 25. `CommandBody`: direktiva/buyruqlarni tahlil qilish uchun foydalanuvchining xom matni.
- 26. `RawBody`: `CommandBody` uchun eski alias (moslik uchun saqlangan).

27. Kanal tarixni taqdim etsa, u umumiy o‘ramdan foydalanadi:

- `[Chat messages since your last reply - for context]`
- `[Current message - respond to this]`

30. **To‘g‘ridan-to‘g‘ri bo‘lmagan chatlar** (guruhlar/kanallar/xonalar) uchun **joriy xabar body** yuboruvchi yorlig‘i bilan prefiks qilinadi (tarix yozuvlari uchun ishlatiladigan uslub bilan bir xil). 31. Bu real vaqt va navbatdagi/tarixiy xabarlarni agent promptida izchil saqlaydi.

32. Tarix buferlari **faqat kutilayotgan** holatdagi: ular ishga tushirishni qo‘zg‘atmagan guruh xabarlarini (masalan, eslatma bilan cheklangan xabarlar) o‘z ichiga oladi va sessiya transkriptida allaqachon mavjud xabarlarni **istisno qiladi**.

33. Direktivani olib tashlash faqat **joriy xabar** bo‘limiga qo‘llanadi, shuning uchun tarix saqlanib qoladi. 34. Tarixni o‘raydigan kanallar `CommandBody`ni (yoki `RawBody`) asl xabar matniga o‘rnatishi va `Body`ni birlashtirilgan prompt sifatida qoldirishi kerak.
34. Tarix buferlari `messages.groupChat.historyLimit` (global sukut bo‘yicha) va `channels.slack.historyLimit` yoki `channels.telegram.accounts.<id>` kabi kanal bo‘yicha o‘zgartirishlar orqali sozlanadi36. `.historyLimit` (o‘chirish uchun `0` ga o‘rnating).

## 37. Navbatga qo‘yish va followup’lar

38. Agar ishga tushirish allaqachon faol bo‘lsa, kiruvchi xabarlar navbatga qo‘yilishi, joriy ishga yo‘naltirilishi yoki keyingi navbat uchun yig‘ilishi mumkin.

- 39. `messages.queue` (va `messages.queue.byChannel`) orqali sozlang.
- 40. Rejimlar: `interrupt`, `steer`, `followup`, `collect`, shuningdek backlog variantlari.

41. Tafsilotlar: [Queueing](/concepts/queue).

## 42. Striming, bo‘laklash va batchlash

43. Blok strimingi model matn bloklarini ishlab chiqargan sari qisman javoblarni yuboradi.
44. Bo‘laklash kanal matn cheklovlariga amal qiladi va fenced code’ni bo‘lib yuborishdan qochadi.

45. Asosiy sozlamalar:

- 46. `agents.defaults.blockStreamingDefault` (`on|off`, sukut bo‘yicha off)
- `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
- `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
- 49. `agents.defaults.blockStreamingCoalesce` (bo‘sh turishga asoslangan batchlash)
- 50. `agents.defaults.humanDelay` (blok javoblar orasidagi insoniyga o‘xshash tanaffus)
- Channel overrides: `*.blockStreaming` and `*.blockStreamingCoalesce` (non-Telegram channels require explicit `*.blockStreaming: true`)

Details: [Streaming + chunking](/concepts/streaming).

## Reasoning visibility and tokens

OpenClaw can expose or hide model reasoning:

- `/reasoning on|off|stream` controls visibility.
- Reasoning content still counts toward token usage when produced by the model.
- Telegram supports reasoning stream into the draft bubble.

Details: [Thinking + reasoning directives](/tools/thinking) and [Token use](/reference/token-use).

## Prefixes, threading, and replies

Outbound message formatting is centralized in `messages`:

- `messages.responsePrefix`, `channels.<channel>.responsePrefix`, and `channels.<channel>.accounts.<id>.responsePrefix` (outbound prefix cascade), plus `channels.whatsapp.messagePrefix` (WhatsApp inbound prefix)
- Reply threading via `replyToMode` and per-channel defaults

Details: [Configuration](/gateway/configuration#messages) and channel docs.

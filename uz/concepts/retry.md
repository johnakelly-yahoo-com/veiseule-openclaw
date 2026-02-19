---
summary: "33. Chiquvchi provayder chaqiruvlari uchun qayta urinish siyosati"
read_when:
  - 34. Provayderning qayta urinish xatti-harakatini yoki standartlarini yangilash
  - 35. Provayder yuborish xatolari yoki tezlik cheklovlarini nosozlikdan chiqarish
title: "36. Qayta urinish siyosati"
---

# 37. Qayta urinish siyosati

## 38. Maqsadlar

- 39. Har bir HTTP so‘rovi bo‘yicha qayta urinish, ko‘p bosqichli oqim bo‘yicha emas.
- 40. Faqat joriy bosqichni qayta urinib, tartibni saqlash.
- 41. Idempotent bo‘lmagan amallarni takrorlab yuborishdan qochish.

## 42. Standartlar

- 43. Urinishlar: 3
- 44. Maksimal kechikish chegarasi: 30000 ms
- 45. Jitter: 0.1 (10 foiz)
- 46. Provayder standartlari:
  - 47. Telegram minimal kechikish: 400 ms
  - 48. Discord minimal kechikish: 500 ms

## 49. Xatti-harakat

### 50. Discord

- Faqat rate-limit xatolarida (HTTP 429) qayta urinadi.
- Mavjud bo‘lsa, Discord `retry_after` dan foydalanadi, aks holda eksponensial backoff qo‘llanadi.

### Telegram

- Vaqtinchalik xatolarda (429, timeout, connect/reset/closed, temporarily unavailable) qayta urinadi.
- Mavjud bo‘lsa `retry_after` dan foydalanadi, aks holda eksponensial backoff qo‘llanadi.
- Markdown parse xatolari uchun qayta urinilmaydi; ular oddiy matnga o‘tkaziladi.

## Konfiguratsiya

Har bir provayder uchun qayta urinish siyosatini `~/.openclaw/openclaw.json` faylida sozlang:

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## Eslatmalar

- Qayta urinishlar har bir so‘rov uchun qo‘llanadi (xabar yuborish, media yuklash, reaksiya, so‘rovnoma, stiker).
- Composite flows do not retry completed steps.


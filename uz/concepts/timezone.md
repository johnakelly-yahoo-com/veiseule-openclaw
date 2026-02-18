---
title: "Vaqt mintaqalari"
---

# Vaqt mintaqalari

OpenClaw vaqt belgilarini standartlashtiradi, shunda model **bitta yagona mos yozuv vaqtiga** tayanadi.

## Xabar konvertlari (standart bo‘yicha mahalliy)

Kiruvchi xabarlar quyidagi kabi konvertga o‘raladi:

```
[Provider ... 2026-01-05 16:26 PST] message text
```

Konvertdagi vaqt belgisi standart bo‘yicha **hostning mahalliy vaqtida**, daqiqa aniqligida bo‘ladi.

Buni quyidagicha o‘zgartirishingiz mumkin:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA timezone
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

- `envelopeTimezone: "utc"` UTC’dan foydalanadi.
- `envelopeTimezone: "user"` `agents.defaults.userTimezone` dan foydalanadi (agar mavjud bo‘lmasa, host vaqt mintaqasiga qaytadi).
- Telegram forum mavzulari izolyatsiya uchun guruh identifikatoriga `:topic:<threadId>` ni qo‘shadi.
- `envelopeTimestamp: "off"` removes absolute timestamps from envelope headers.
- `envelopeElapsed: "off"` removes elapsed time suffixes (the `+2m` style).

### Examples

**Local (default):**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**Fixed timezone:**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**Elapsed time:**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

## Tool payloads (raw provider data + normalized fields)

Tool calls (`channels.discord.readMessages`, `channels.slack.readMessages`, etc.) return **raw provider timestamps**.
We also attach normalized fields for consistency:

- `timestampMs` (UTC epoch milliseconds)
- `timestampUtc` (ISO 8601 UTC string)

Raw provider fields are preserved.

## User timezone for the system prompt

1. Modelga foydalanuvchining mahalliy vaqt mintaqasini bildirish uchun `agents.defaults.userTimezone` ni sozlang. 2. Agar u o‘rnatilmagan bo‘lsa, OpenClaw **ishga tushirish vaqtida xost vaqt mintaqasini** aniqlaydi (konfiguratsiya yozilmaydi).

```json5
3. {
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

4. Tizim prompti quyidagilarni o‘z ichiga oladi:

- 5. Mahalliy vaqt va vaqt mintaqasini ko‘rsatadigan `Current Date & Time` bo‘limi
- 6. `Time format: 12-hour` yoki `24-hour`

7. Prompt formatini `agents.defaults.timeFormat` (`auto` | `12` | `24`) orqali boshqarishingiz mumkin.

8. To‘liq xulq-atvor va misollar uchun [Date & Time](/date-time) bo‘limiga qarang.


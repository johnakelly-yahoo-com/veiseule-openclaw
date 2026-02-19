---
summary: "Global ovozli uyg‘otish so‘zlari (Gateway tomonidan boshqariladi) va ular tugunlar bo‘ylab qanday sinxronlanadi"
read_when:
  - Changing voice wake words behavior or defaults
  - Adding new node platforms that need wake word sync
title: "Ovozli uyg‘otish"
---

# Ovozli uyg‘otish (Global uyg‘otish so‘zlari)

OpenClaw **uyg‘otish so‘zlarini yagona global ro‘yxat** sifatida ko‘radi va u **Gateway** tomonidan boshqariladi.

- **Har bir tugun uchun alohida maxsus uyg‘otish so‘zlari yo‘q**.
- **Istalgan tugun/ilova UI ro‘yxatni tahrirlashi mumkin**; o‘zgarishlar Gateway tomonidan saqlanadi va hammaga uzatiladi.
- Har bir qurilma baribir o‘zining **Ovozli uyg‘otish yoqilgan/o‘chirilgan** tugmasiga ega (mahalliy UX va ruxsatlar farq qiladi).

## Saqlash (Gateway xosti)

Uyg‘otish so‘zlari gateway qurilmasida quyidagi manzilda saqlanadi:

- `~/.openclaw/settings/voicewake.json`

Tuzilishi:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## Protocol

### Methods

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` with params `{ triggers: string[] }` → `{ triggers: string[] }`

Notes:

- Triggers are normalized (trimmed, empties dropped). Bo‘sh ro‘yxatlar sukut bo‘yicha qiymatlarga qaytadi.
- Xavfsizlik uchun cheklovlar qo‘llaniladi (son/uzunlik limitlari).

### Hodisalar

- `voicewake.changed` payload `{ triggers: string[] }`

Kimlar qabul qiladi:

- Barcha WebSocket mijozlari (macOS ilovasi, WebChat va boshqalar)
- Barcha ulangan tugunlar (iOS/Android), shuningdek tugun ulanganda boshlang‘ich “joriy holat” sifatida ham yuboriladi.

## Mijoz xulqi

### macOS app

- Global ro‘yxatdan `VoiceWakeRuntime` triggerlarini boshqarish uchun foydalanadi.
- Voice Wake sozlamalarida “Trigger words”ni tahrirlash `voicewake.set`ni chaqiradi va boshqa mijozlarni sinxron holatda ushlab turish uchun translyatsiyaga tayanadi.

### iOS tuguni

- `VoiceWakeManager` triggerlarini aniqlash uchun global ro‘yxatdan foydalanadi.
- Editing Wake Words in Settings calls `voicewake.set` (over the Gateway WS) and also keeps local wake-word detection responsive.

### Android tuguni

- Exposes a Wake Words editor in Settings.
- Tahrirlar hamma joyda sinxron bo‘lishi uchun Gateway WS orqali `voicewake.set`ni chaqiradi.

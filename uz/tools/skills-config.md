---
title: "Skills Konfiguratsiyasi"
---

# Skills Konfiguratsiyasi

Skills bilan bog‘liq barcha sozlamalar `~/.openclaw/openclaw.json` ichidagi `skills` bo‘limida joylashgan.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## Maydonlar

- `allowBundled`: faqat **bundled** skills uchun ixtiyoriy allowlist. Agar o‘rnatilgan bo‘lsa, faqat ro‘yxatdagi bundled skills ishlatiladi (managed/workspace skills ta’sir qilmaydi).
- `load.extraDirs`: skanerlash uchun qo‘shimcha skill kataloglari (eng past ustuvorlik).
- `load.watch`: skill papkalarini kuzatish va skills snapshot’ini yangilash (standart: true).
- `load.watchDebounceMs`: skill watcher hodisalari uchun millisekundlardagi debounce (standart: 250).
- `install.preferBrew`: mavjud bo‘lsa, brew o‘rnatuvchilarini afzal ko‘rish (standart: true).
- `install.nodeManager`: node o‘rnatuvchi tanlovi (`npm` | `pnpm` | `yarn` | `bun`, standart: npm).
  Bu faqat **skill o‘rnatishlariga** ta’sir qiladi; Gateway runtime baribir Node bo‘lishi kerak
  (WhatsApp/Telegram uchun Bun tavsiya etilmaydi).
- `entries.<skillKey>`: har bir skill uchun alohida sozlamalar.

Har bir skill uchun maydonlar:

- `enabled`: agar skill bundled/o‘rnatilgan bo‘lsa ham, uni o‘chirish uchun `false` qilib qo‘ying.
- `env`: agent ishga tushirilganda qo‘shiladigan muhit o‘zgaruvchilari (faqat avvaldan o‘rnatilmagan bo‘lsa).
- `apiKey`: asosiy env o‘zgaruvchini belgilaydigan skills uchun ixtiyoriy qulay parametr.

## Eslatmalar

- `entries` ostidagi kalitlar odatda skill nomiga mos keladi. Agar skill
  `metadata.openclaw.skillKey` ni belgilagan bo‘lsa, o‘sha kalitdan foydalaning.
- Watcher yoqilgan bo‘lsa, skills’dagi o‘zgarishlar keyingi agent bosqichida qo‘llaniladi.

### Sandboxed skills + env o‘zgaruvchilar

Sessiya **sandboxed** bo‘lsa, skill jarayonlari Docker ichida ishlaydi. Sandbox
host `process.env` ni **meros qilib olmaydi**.

Quyidagilardan birini ishlating:

- `agents.defaults.sandbox.docker.env` (yoki har bir agent uchun `agents.list[].sandbox.docker.env`)
- env’ni o‘zingizning maxsus sandbox image’ingizga qo‘shib yuboring

Global `env` va `skills.entries.<skill>.env/apiKey` faqat **host** ishga tushirishlari uchun amal qiladi.


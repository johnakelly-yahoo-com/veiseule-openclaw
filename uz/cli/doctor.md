---
title: "doctor"
---

# `openclaw doctor`

Gateway va kanallar uchun sog‘liq tekshiruvlari + tezkor tuzatishlar.

Bog‘liq:

- Nosozliklarni bartaraf etish: [Nosozliklarni bartaraf etish](/gateway/troubleshooting)
- Xavfsizlik auditi: [Xavfsizlik](/gateway/security)

## Misollar

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Eslatmalar:

- Interaktiv so‘rovlar (masalan, keychain/OAuth tuzatishlari) faqat stdin TTY bo‘lganda va `--non-interactive` o‘rnatilmaganida ishga tushadi. Headless runs (cron, Telegram, no terminal) will skip prompts.
- `--fix` (`--repair` uchun alias) `~/.openclaw/openclaw.json.bak` ga zaxira nusxa yozadi va noma’lum konfiguratsiya kalitlarini o‘chiradi, har bir olib tashlashni ro‘yxatlab.

## macOS: `launchctl` env o‘zgaruvchilarini ustidan yozish

If you previously ran `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (or `...PASSWORD`), that value overrides your config file and can cause persistent “unauthorized” errors.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```


---
summary: "Menyu paneli holati logikasi va foydalanuvchilarga nimalar ko‘rsatilishi"
read_when:
  - Mac menyu UI yoki holat logikasini sozlash
title: "Menyu paneli"
---

# Menyu paneli holati logikasi

## Nimalar ko‘rsatiladi

- Joriy agent ish holatini menyu paneli ikonkasida va menyuning birinchi holat qatorida ko‘rsatamiz.
- Ish faol bo‘lganda sog‘liq holati yashiriladi; barcha sessiyalar bo‘sh turganda u qaytadi.
- Menyudagi “Nodes” bloki faqat **qurilmalar**ni ( `node.list` orqali juftlangan tugunlar) ko‘rsatadi, klient/presence yozuvlarini emas.
- Provider foydalanish snapshotlari mavjud bo‘lganda Context ostida “Usage” bo‘limi paydo bo‘ladi.

## Holat modeli

- Sessiyalar: hodisalar payload ichida `runId` (har bir ishga tushirish uchun) va `sessionKey` bilan keladi. “Asosiy” sessiya kaliti `main`; agar u bo‘lmasa, eng yaqinda yangilangan sessiyaga qaytamiz.
- Ustuvorlik: main har doim yutadi. Agar main faol bo‘lsa, uning holati darhol ko‘rsatiladi. Agar main bo‘sh turgan bo‘lsa, eng yaqinda faol bo‘lgan main bo‘lmagan sessiya ko‘rsatiladi. Faoliyat o‘rtasida almashib ketmaymiz; faqat joriy sessiya bo‘sh holatga o‘tganda yoki main faol bo‘lganda almashtiramiz.
- Faoliyat turlari:
  - `job`: yuqori darajadagi buyruq bajarilishi (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` bilan `toolName` va `meta/args`.

## IconState enumeratsiyasi (Swift)

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (debug override)

### ActivityKind → glif

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- default → 🛠️

### Vizual moslashtirish

- `idle`: oddiy jonzot.
- `workingMain`: glifli belgi, to‘liq rang, oyoqlarning “working” animatsiyasi.
- `workingOther`: glifli belgi, xiralashtirilgan rang, yugurishsiz.
- `overridden`: faoliyatdan qat’i nazar tanlangan glif/rangdan foydalanadi.

## Holat qatori matni (menyu)

- Ish faol bo‘lganda: `<Session role> · <activity label>`
  - Misollar: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Bo‘sh turganda: sog‘liq xulosasiga qaytadi.

## Hodisalarni qabul qilish

- Source: control‑channel `agent` events (`ControlChannel.handleAgentEvent`).
- Parsed fields:
  - `stream: "job"` with `data.state` for start/stop.
  - `stream: "tool"` with `data.phase`, `name`, optional `meta`/`args`.
- Labels:
  - `exec`: first line of `args.command`.
  - `read`/`write`: shortened path.
  - `edit`: path plus inferred change kind from `meta`/diff counts.
  - fallback: tool name.

## Debug override

- Settings ▸ Debug ▸ “Icon override” picker:
  - `System (auto)` (default)
  - `Working: main` (per tool kind)
  - `Working: other` (per tool kind)
  - `Idle`
- Stored via `@AppStorage("iconOverride")`; mapped to `IconState.overridden`.

## Testing checklist

- Trigger main session job: verify icon switches immediately and status row shows main label.
- Trigger non‑main session job while main idle: icon/status shows non‑main; stays stable until it finishes.
- Start main while other active: icon flips to main instantly.
- Rapid tool bursts: ensure badge does not flicker (TTL grace on tool results).
- Health row reappears once all sessions idle.

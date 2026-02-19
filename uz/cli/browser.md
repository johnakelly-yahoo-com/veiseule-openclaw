---
summary: "`openclaw browser` uchun CLI ma’lumotnomasi (profilar, tablar, amallar, kengaytma relesi)"
read_when:
  - Siz `openclaw browser` dan foydalanasiz va umumiy vazifalar uchun misollarni xohlaysiz
  - Tugun xosti orqali boshqa mashinada ishlayotgan brauzerni boshqarishni xohlaysiz
  - Chrome kengaytma relesidan foydalanmoqchisiz (asboblar paneli tugmasi orqali ulash/ajratish)
title: "browser"
---

# `openclaw browser`

OpenClaw’ning brauzer boshqaruv serverini boshqarish va brauzer amallarini bajarish (tablar, snapshotlar, skrinshotlar, navigatsiya, bosishlar, yozish).

Bog‘liq:

- Brauzer vositasi + API: [Browser tool](/tools/browser)
- Chrome kengaytma relesi: [Chrome extension](/tools/chrome-extension)

## Umumiy flaglar

- `--url <gatewayWsUrl>`: Shlyuz WebSocket URL’i (konfiguratsiyadan standart).
- `--token <token>`: Shlyuz tokeni (agar talab qilinsa).
- `--timeout <ms>`: so‘rov taymauti (ms).
- `--browser-profile <name>`: brauzer profilini tanlash (standart — konfiguratsiyadan).
- `--json`: mashina o‘qiy oladigan chiqish (qo‘llab-quvvatlangan joylarda).

## Tezkor boshlash (lokal)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## Profilar

Profilar — nomlangan brauzer marshrutlash konfiguratsiyalari. Amalda:

- `openclaw`: OpenClaw tomonidan boshqariladigan alohida Chrome instansiyasini ishga tushiradi/unga ulanadi (ajratilgan foydalanuvchi ma’lumotlari katalogi).
- `chrome`: Chrome kengaytma relesi orqali mavjud Chrome tab(lar)ingizni boshqaradi.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Muayyan profilni ishlating:

```bash
openclaw browser --browser-profile work tabs
```

## Yorliqlar

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## Snapshot / skrinshot / amallar

Snapshot:

```bash
openclaw browser snapshot
```

Skrinshot:

```bash
openclaw browser screenshot
```

Navigatsiya/bosish/yozish (ref-based UI avtomatlashtirish):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Chrome kengaytma releyi (asboblar paneli tugmasi orqali ulanish)

Bu rejim agentga siz qo‘lda biriktirgan mavjud Chrome yorlig‘ini boshqarishga imkon beradi (u avtomatik biriktirilmaydi).

Unpacked kengaytmani barqaror joylashuvga o‘rnating:

```bash
openclaw browser extension install
openclaw browser extension path
```

So‘ng Chrome → `chrome://extensions` → “Developer mode”ni yoqing → “Load unpacked” → chop etilgan papkani tanlang.

To‘liq qo‘llanma: [Chrome extension](/tools/chrome-extension)

## Masofaviy brauzer boshqaruvi (node host proksi)

If the Gateway runs on a different machine than the browser, run a **node host** on the machine that has Chrome/Brave/Edge/Chromium. The Gateway will proxy browser actions to that node (no separate browser control server required).

Avtomatik yo‘naltirishni boshqarish uchun `gateway.nodes.browser.mode`dan va bir nechta node ulangan bo‘lsa, aniq node’ni belgilash uchun `gateway.nodes.browser.node`dan foydalaning.

Xavfsizlik + masofaviy sozlash: [Browser tool](/tools/browser), [Remote access](/gateway/remote), [Tailscale](/gateway/tailscale), [Security](/gateway/security)

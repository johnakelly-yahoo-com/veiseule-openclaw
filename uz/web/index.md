---
title: "Veb"
---

# Veb (Gateway)

Gateway Gateway WebSocket bilan bir xil portda kichik **brauzer Boshqaruv UI** (Vite + Lit) ni taqdim etadi:

- standart: `http://<host>:18789/`
- ixtiyoriy prefiks: `gateway.controlUi.basePath` ni sozlang (masalan, `/openclaw`)

Imkoniyatlar [Control UI](/web/control-ui) sahifasida keltirilgan.
Bu sahifa ulanish rejimlari, xavfsizlik va vebga ochiq yuzalarga e’tibor qaratadi.

## Webhook’lar

`hooks.enabled=true` bo‘lsa, Gateway shu HTTP serverda kichik webhook endpoint’ni ham ochadi.
Autentifikatsiya va payload’lar uchun [Gateway configuration](/gateway/configuration) → `hooks` bo‘limiga qarang.

## Konfiguratsiya (standart bo‘yicha yoqilgan)

Agar asset’lar mavjud bo‘lsa (`dist/control-ui`), Boshqaruv UI **standart bo‘yicha yoqilgan**.
Uni konfiguratsiya orqali boshqarishingiz mumkin:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath ixtiyoriy
  },
}
```

## Tailscale orqali kirish

### Integrated Serve (tavsiya etiladi)

Gateway’ni loopback’da qoldiring va Tailscale Serve orqali proksi qiling:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

So‘ng gateway’ni ishga tushiring:

```bash
openclaw gateway
```

Ochish:

- `https://<magicdns>/` (yoki sozlangan `gateway.controlUi.basePath`)

### Tailnet’ni bog‘lash + token

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

So‘ng gateway’ni ishga tushiring (loopback’dan tashqari bind’lar uchun token talab qilinadi):

```bash
openclaw gateway
```

Ochish:

- `http://<tailscale-ip>:18789/` (yoki sozlangan `gateway.controlUi.basePath`)

### Ochiq internet (Funnel)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // yoki OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## Xavfsizlik bo‘yicha eslatmalar

- Standart bo‘yicha Gateway autentifikatsiyasi talab qilinadi (token/parol yoki Tailscale identifikatsiya sarlavhalari).
- Loopback’dan tashqari bind’lar ham **majburiy ravishda** umumiy token/parol talab qiladi (`gateway.auth` yoki env).
- Wizard standart bo‘yicha gateway token’ini yaratadi (hatto loopback’da ham).
- UI `connect.params.auth.token` yoki `connect.params.auth.password` yuboradi.
- Boshqaruv UI anti-clickjacking sarlavhalarini yuboradi va faqat bir xil origin’dan
  websocket brauzer ulanishlarini qabul qiladi, agar `gateway.controlUi.allowedOrigins` o‘rnatilmagan bo‘lsa.
- Serve bilan ishlaganda, agar `gateway.auth.allowTailscale` `true` bo‘lsa,
  Tailscale identifikatsiya sarlavhalari autentifikatsiyani ta’minlashi mumkin (token/parol talab qilinmaydi). 
  Aniq login ma’lumotlarini talab qilish uchun `gateway.auth.allowTailscale: false` qilib sozlang. Qarang:
  [Tailscale](/gateway/tailscale) va [Security](/gateway/security).
- `gateway.tailscale.mode: "funnel"` uchun `gateway.auth.mode: "password"` (umumiy parol) talab qilinadi.

## UI’ni build qilish

Gateway statik fayllarni `dist/control-ui` dan taqdim etadi. Ularni quyidagicha build qiling:

```bash
pnpm ui:build # birinchi ishga tushirishda UI dependency’larini avtomatik o‘rnatadi
```


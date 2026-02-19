---
summary: "Gateway veb yuzalari: Boshqaruv UI, ulanish rejimlari va xavfsizlik"
read_when:
  - Gateway’ga Tailscale orqali kirishni xohlaysiz
  - Brauzer Boshqaruv UI va konfiguratsiyani tahrirlash kerak
title: "Veb"
---

# Veb (Gateway)

Gateway Gateway WebSocket bilan bir xil portda kichik **brauzer Boshqaruv UI** (Vite + Lit) ni taqdim etadi:

- standart: `http://<host>:18789/`
- ixtiyoriy prefiks: `gateway.controlUi.basePath` ni sozlang (masalan, `/openclaw`)

Capabilities live in [Control UI](/web/control-ui).
This page focuses on bind modes, security, and web-facing surfaces.

## Webhook’lar

When `hooks.enabled=true`, the Gateway also exposes a small webhook endpoint on the same HTTP server.
See [Gateway configuration](/gateway/configuration) → `hooks` for auth + payloads.

## Konfiguratsiya (standart bo‘yicha yoqilgan)

The Control UI is **enabled by default** when assets are present (`dist/control-ui`).
You can control it via config:

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
- With Serve, Tailscale identity headers can satisfy auth when
  `gateway.auth.allowTailscale` is `true` (no token/password required). Set
  `gateway.auth.allowTailscale: false` to require explicit credentials. See
  [Tailscale](/gateway/tailscale) and [Security](/gateway/security).
- `gateway.tailscale.mode: "funnel"` uchun `gateway.auth.mode: "password"` (umumiy parol) talab qilinadi.

## UI’ni build qilish

The Gateway serves static files from `dist/control-ui`. Build them with:

```bash
pnpm ui:build # birinchi ishga tushirishda UI dependency’larini avtomatik o‘rnatadi
```

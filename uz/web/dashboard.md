---
title: "Boshqaruv paneli"
---

# Boshqaruv paneli (Control UI)

Gateway boshqaruv paneli — bu brauzer orqali ishlaydigan Control UI bo‘lib, odatda `/` manzilida xizmat ko‘rsatadi  
(`gateway.controlUi.basePath` orqali o‘zgartiriladi).

Tez ochish (mahalliy Gateway):

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (yoki [http://localhost:18789/](http://localhost:18789/))

Asosiy havolalar:

- Foydalanish va interfeys imkoniyatlari uchun [Control UI](/web/control-ui).
- Serve/Funnel avtomatlashtirish uchun [Tailscale](/gateway/tailscale).
- Bog‘lanish rejimlari va xavfsizlik eslatmalari uchun [Web surfaces](/web).

Autentifikatsiya WebSocket qo‘l siqish bosqichida `connect.params.auth`  
(token yoki parol) orqali amalga oshiriladi. `gateway.auth` bo‘limiga qarang: [Gateway configuration](/gateway/configuration).

Xavfsizlik eslatmasi: Control UI — bu **administrator interfeysi** (chat, konfiguratsiya, exec tasdiqlari).  
Uni ommaga ochmang. Interfeys birinchi yuklangandan so‘ng tokenni `localStorage`da saqlaydi.  
localhost, Tailscale Serve yoki SSH tunnelidan foydalanish tavsiya etiladi.

## Tezkor yo‘l (tavsiya etiladi)

- Onboardingdan so‘ng, CLI boshqaruv panelini avtomatik ochadi va toza (tokensiz) havolani chiqaradi.
- Istalgan vaqtda qayta ochish: `openclaw dashboard` (havolani nusxalaydi, imkon bo‘lsa brauzerni ochadi, headless rejimda bo‘lsa SSH bo‘yicha ko‘rsatma beradi).
- Agar UI autentifikatsiya so‘rasa, `gateway.auth.token` (yoki `OPENCLAW_GATEWAY_TOKEN`) qiymatini Control UI sozlamalariga joylashtiring.

## Token asoslari (mahalliy va masofaviy)

- **Localhost**: `http://127.0.0.1:18789/` manzilini oching.
- **Token manbai**: `gateway.auth.token` (yoki `OPENCLAW_GATEWAY_TOKEN`); ulanganingizdan so‘ng UI uning nusxasini localStorage’da saqlaydi.
- **Localhost emas**: Tailscale Serve’dan foydalaning (agar `gateway.auth.allowTailscale: true` bo‘lsa tokensiz), tailnet orqali token bilan ulaning yoki SSH tunnel ishlating. Qarang: [Web surfaces](/web).

## Agar “unauthorized” / 1008 xatosini ko‘rsangiz

- Gateway mavjudligini tekshiring (mahalliy: `openclaw status`; masofaviy: SSH tunnel `ssh -N -L 18789:127.0.0.1:18789 user@host` va so‘ng `http://127.0.0.1:18789/` ni oching).
- Gateway joylashgan hostdan tokenni oling: `openclaw config get gateway.auth.token` (yoki yangisini yarating: `openclaw doctor --generate-gateway-token`).
- Boshqaruv paneli sozlamalarida tokenni autentifikatsiya maydoniga joylashtiring va ulang.
---
title: "Boshqaruv interfeysi"
---

# Boshqaruv interfeysi (brauzer)

Boshqaruv interfeysi — bu Gateway tomonidan taqdim etiladigan kichik **Vite + Lit** asosidagi bitta sahifali ilova:

- standart: `http://<host>:18789/`
- ixtiyoriy prefiks: `gateway.controlUi.basePath` ni sozlang (masalan, `/openclaw`)

U **to‘g‘ridan-to‘g‘ri Gateway WebSocket’iga** shu port orqali ulanadi.

## Tez ochish (lokal)

Agar Gateway shu kompyuterda ishlayotgan bo‘lsa, quyidagini oching:

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (yoki [http://localhost:18789/](http://localhost:18789/))

Agar sahifa yuklanmasa, avval Gateway’ni ishga tushiring: `openclaw gateway`.

Autentifikatsiya WebSocket qo‘l siqish (handshake) vaqtida quyidagilar orqali uzatiladi:

- `connect.params.auth.token`
- `connect.params.auth.password`  
  Dashboard sozlamalari paneli tokenni saqlash imkonini beradi; parollar saqlanmaydi.  
  Onboarding ustasi (wizard) odatda gateway tokenini avtomatik yaratadi, shuning uchun birinchi ulanishda uni shu yerga joylashtiring.

## Qurilmani juftlash (birinchi ulanish)

Boshqaruv interfeysiga yangi brauzer yoki qurilmadan ulanganda, Gateway  
**bir martalik juftlash tasdig‘ini** talab qiladi — hatto siz `gateway.auth.allowTailscale: true` bilan bir xil Tailnet’da bo‘lsangiz ham. Bu ruxsatsiz kirishni oldini olish uchun xavfsizlik chorasi.

**Ko‘rinadigan xabar:** "disconnected (1008): pairing required"

**Qurilmani tasdiqlash uchun:**

```bash
# Kutilayotgan so‘rovlarni ko‘rish
openclaw devices list

# So‘rov ID bo‘yicha tasdiqlash
openclaw devices approve <requestId>
```

Tasdiqlangandan so‘ng, qurilma eslab qolinadi va qayta tasdiq talab qilinmaydi,  
faqat `openclaw devices revoke --device <id> --role <role>` orqali bekor qilmasangiz.  
Tokenni yangilash va bekor qilish haqida batafsil: [Devices CLI](/cli/devices).

**Eslatma:**

- Lokal ulanishlar (`127.0.0.1`) avtomatik tasdiqlanadi.
- Masofaviy ulanishlar (LAN, Tailnet va boshqalar) qo‘lda tasdiqlashni talab qiladi.
- Har bir brauzer profili noyob qurilma ID yaratadi, shuning uchun brauzerni almashtirish yoki brauzer ma’lumotlarini tozalash qayta juftlashni talab qiladi.

## Hozirgi imkoniyatlar

- Gateway WS orqali model bilan chat (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
- Chat’da tool chaqiruvlarini va jonli tool natijalarini oqim ko‘rinishida ko‘rsatish (agent hodisalari)
- Kanallar: WhatsApp/Telegram/Discord/Slack + plagin kanallari (Mattermost va boshqalar) holati + QR login + har bir kanal uchun sozlamalar (`channels.status`, `web.login.*`, `config.patch`)
- Instansiyalar: mavjudlik ro‘yxati + yangilash (`system-presence`)
- Sessiyalar: ro‘yxat + har bir sessiya uchun thinking/verbose sozlamalarini o‘zgartirish (`sessions.list`, `sessions.patch`)
- Cron vazifalar: ro‘yxat/qoshish/ishga tushirish/yoqish/o‘chirish + ishga tushirish tarixi (`cron.*`)
- Skills: holat, yoqish/o‘chirish, o‘rnatish, API kalitlarini yangilash (`skills.*`)
- Tugunlar: ro‘yxat + imkoniyatlar (`node.list`)
- Exec tasdiqlari: gateway yoki tugun allowlist’larini tahrirlash + `exec host=gateway/node` uchun siyosat so‘rash (`exec.approvals.*`)
- Config: `~/.openclaw/openclaw.json` ni ko‘rish/tahrirlash (`config.get`, `config.set`)
- Config: tekshiruv bilan qo‘llash + qayta ishga tushirish (`config.apply`) va oxirgi faol sessiyani uyg‘otish
- Config yozuvlari bir vaqtda tahrirlarni ustma-ust yozib yubormaslik uchun base-hash himoyasiga ega
- Config sxemasi + forma orqali render qilish (`config.schema`, plagin + kanal sxemalarini ham o‘z ichiga oladi); Raw JSON muharriri ham mavjud
- Debug: holat/health/models snapshot’lari + hodisalar logi + qo‘lda RPC chaqiruvlari (`status`, `health`, `models.list`)
- Loglar: gateway fayl loglarini jonli kuzatish + filtrlash/eksport (`logs.tail`)
- Yangilash: paket/git orqali yangilash + qayta ishga tushirish (`update.run`) va qayta ishga tushirish hisoboti

Cron paneli eslatmalari:

- Izolyatsiyalangan vazifalar uchun yetkazish odatda qisqa xulosa e’lon qilishga sozlangan. Agar faqat ichki ishga tushirish kerak bo‘lsa, none’ga o‘zgartirishingiz mumkin.
- E’lon tanlanganda kanal/target maydonlari paydo bo‘ladi.

## Chat xatti-harakati

- `chat.send` **bloklanmaydi**: darhol `{ runId, status: "started" }` bilan javob beradi va natija `chat` hodisalari orqali oqimda keladi.
- Xuddi shu `idempotencyKey` bilan qayta yuborilganda, jarayon davomida `{ status: "in_flight" }`, tugagandan so‘ng `{ status: "ok" }` qaytariladi.
- `chat.inject` sessiya transkriptiga assistant eslatmasini qo‘shadi va UI yangilanishi uchun `chat` hodisasini yuboradi (agent ishga tushirilmaydi, kanalga yetkazilmaydi).
- To‘xtatish:
  - **Stop** tugmasini bosing (`chat.abort` chaqiriladi)
  - `/stop` yozing (yoki `stop|esc|abort|wait|exit|interrupt`) — tashqi tarzda to‘xtatish
  - `chat.abort` `{ sessionKey }` ( `runId` siz) bilan shu sessiyadagi barcha faol jarayonlarni to‘xtatadi

## Tailnet orqali kirish (tavsiya etiladi)

### Integratsiyalashgan Tailscale Serve (afzal)

Gateway’ni loopback’da qoldiring va Tailscale Serve orqali HTTPS bilan proksi qiling:

```bash
openclaw gateway --tailscale serve
```

Ochish:

- `https://<magicdns>/` (yoki sozlangan `gateway.controlUi.basePath`)

Standart bo‘yicha, Serve so‘rovlari `gateway.auth.allowTailscale: true` bo‘lsa,  
Tailscale identifikatsiya sarlavhalari (`tailscale-user-login`) orqali autentifikatsiya qilinishi mumkin. OpenClaw identifikatsiyani `x-forwarded-for` manzilini `tailscale whois` orqali tekshirib va sarlavha bilan mosligini solishtirib tasdiqlaydi, va bularni faqat so‘rov loopback’ka Tailscale’ning `x-forwarded-*` sarlavhalari bilan kelganda qabul qiladi.  
Agar Serve trafigi uchun ham token/parol talab qilmoqchi bo‘lsangiz, `gateway.auth.allowTailscale: false` (yoki `gateway.auth.mode: "password"`) ni sozlang.

### Tailnet’ga bind qilish + token

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

So‘ng oching:

- `http://<tailscale-ip>:18789/` (yoki sozlangan `gateway.controlUi.basePath`)

Tokenni UI sozlamalariga joylashtiring ( `connect.params.auth.token` sifatida yuboriladi).

## Xavfsiz bo‘lmagan HTTP

Agar dashboard’ni oddiy HTTP orqali ochsangiz (`http://<lan-ip>` yoki `http://<tailscale-ip>`),  
brauzer **xavfsiz bo‘lmagan kontekst**da ishlaydi va WebCrypto’ni bloklaydi. Standart bo‘yicha,  
OpenClaw qurilma identifikatsiyasisiz Control UI ulanishlarini **bloklaydi**.

**Tavsiya etilgan yechim:** HTTPS (Tailscale Serve) dan foydalaning yoki UI’ni lokal oching:

- `https://<magicdns>/` (Serve)
- `http://127.0.0.1:18789/` (gateway joylashgan host’da)

**Pasaytirilgan xavfsizlik misoli (HTTP ustida faqat token):**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

Bu Control UI uchun qurilma identifikatsiyasi + juftlashni o‘chiradi (hatto HTTPS’da ham).  
Faqat tarmoqqa ishonchingiz komil bo‘lsa foydalaning.

HTTPS sozlash bo‘yicha ko‘rsatmalar: [Tailscale](/gateway/tailscale).

## UI’ni build qilish

Gateway statik fayllarni `dist/control-ui` dan taqdim etadi. Ularni build qilish:

```bash
pnpm ui:build # birinchi ishga tushirishda UI bog‘liqliklarini avtomatik o‘rnatadi
```

Ixtiyoriy absolut base (asset URL’lari qat’iy bo‘lishi kerak bo‘lsa):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Lokal ishlab chiqish uchun (alohida dev server):

```bash
pnpm ui:dev # birinchi ishga tushirishda UI bog‘liqliklarini avtomatik o‘rnatadi
```

So‘ng UI’ni Gateway WS manziliga yo‘naltiring (masalan, `ws://127.0.0.1:18789`).

## Debug/test: dev server + masofaviy Gateway

Control UI — statik fayllar to‘plami; WebSocket manzili sozlanadi va HTTP origin’dan farq qilishi mumkin. Bu Vite dev server lokal ishlaganda, Gateway boshqa joyda bo‘lsa qulay.

1. UI dev serverni ishga tushiring: `pnpm ui:dev`
2. Quyidagi kabi URL oching:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Ixtiyoriy bir martalik autentifikatsiya (kerak bo‘lsa):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Eslatma:

- `gatewayUrl` yuklangandan so‘ng localStorage’da saqlanadi va URL’dan olib tashlanadi.
- `token` localStorage’da saqlanadi; `password` faqat xotirada saqlanadi.
- `gatewayUrl` o‘rnatilganda, UI config yoki muhitdagi credential’larga qaytmaydi.  
  `token` (yoki `password`) ni aniq ko‘rsating. Aks holda xatolik yuz beradi.
- Gateway TLS ortida bo‘lsa (`Tailscale Serve`, HTTPS proksi va boshqalar), `wss://` dan foydalaning.
- `gatewayUrl` faqat yuqori darajadagi oynada (embed qilinmagan) qabul qilinadi — clickjacking’ni oldini olish uchun.
- Cross-origin dev sozlamalari uchun (masalan, `pnpm ui:dev` dan masofaviy Gateway’ga), UI origin’ni `gateway.controlUi.allowedOrigins` ga qo‘shing.

Misol:

```json5
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

Masofaviy kirish sozlamalari tafsilotlari: [Remote access](/gateway/remote).

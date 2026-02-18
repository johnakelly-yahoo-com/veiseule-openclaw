---
title: "Clawnet Refaktori"
---

# Clawnet refaktori (protokol + autentifikatsiyani birlashtirish)

## Salom

Salom Peter — a’lo yo‘nalish; bu soddaroq UX + kuchliroq xavfsizlikni ta’minlaydi.

## Maqsad

Quyidagilar uchun yagona, puxta hujjat:

- Joriy holat: protokollar, oqimlar, ishonch chegaralari.
- Og‘riqli nuqtalar: tasdiqlashlar, ko‘p bosqichli marshrutlash, UI takrorlanishi.
- Taklif etilayotgan yangi holat: bitta protokol, chegaralangan rollar, yagona autentifikatsiya/juftlash, TLS pinning.
- Identifikatsiya modeli: barqaror ID + yoqimli sluglar.
- Migratsiya rejasi, xavflar, ochiq savollar.

## Maqsadlar (muhokamadan)

- Barcha mijozlar uchun bitta protokol (mac ilova, CLI, iOS, Android, headless tugun).
- Har bir tarmoq ishtirokchisi autentifikatsiyadan o‘tgan + juftlangan bo‘lishi.
- Rollar aniqligi: tugunlar va operatorlar.
- Markazlashgan tasdiqlashlar foydalanuvchi qayerda bo‘lsa, o‘sha yerga yetkaziladi.
- Barcha masofaviy trafik uchun TLS shifrlash + ixtiyoriy pinning.
- Minimal kod takrorlanishi.
- Bitta qurilma interfeysda bir marta ko‘rinishi (UI/tugun dublsiz).

## Maqsad bo‘lmaganlar (aniq)

- Imkoniyatlar ajratilishini olib tashlash (kam imtiyoz prinsipi saqlanadi).
- Scope tekshiruvlarisiz to‘liq gateway boshqaruvini ochish.
- Autentifikatsiyani insoniy yorliqlarga bog‘lash (sluglar xavfsizlik uchun ishlatilmaydi).

---

# Joriy holat (hozirgi)

## Ikki protokol

### 1) Gateway WebSocket (boshqaruv tekisligi)

- To‘liq API: config, channels, models, sessions, agent run’lar, loglar, tugunlar va boshqalar.
- Standart bind: loopback. Masofaviy kirish SSH/Tailscale orqali.
- Auth: `connect` orqali token/parol.
- TLS pinning yo‘q (loopback/tunnel’ga tayanadi).
- Kod:
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

### 2) Bridge (tugun transporti)

- Cheklangan allowlist yuzasi, tugun identifikatsiyasi + juftlash.
- TCP ustida JSONL; ixtiyoriy TLS + sertifikat fingerprint pinning.
- TLS discovery TXT’da fingerprint’ni e’lon qiladi.
- Kod:
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

## Hozirgi boshqaruv tekisligi mijozlari

- CLI → Gateway WS orqali `callGateway` (`src/gateway/call.ts`).
- macOS ilova UI → Gateway WS (`GatewayConnection`).
- Veb boshqaruv interfeysi → Gateway WS.
- ACP → Gateway WS.
- Brauzer boshqaruvi o‘zining HTTP control server’idan foydalanadi.

## Hozirgi tugunlar

- Tugun rejimidagi macOS ilova Gateway bridge’ga ulanadi (`MacNodeBridgeSession`).
- iOS/Android ilovalari Gateway bridge’ga ulanadi.
- Juftlash + har bir tugun uchun token gateway’da saqlanadi.

## Hozirgi tasdiqlash oqimi (exec)

- Agent Gateway orqali `system.run` ishlatadi.
- Gateway bridge orqali tugunni chaqiradi.
- Tugun runtime tasdiqlashni hal qiladi.
- UI so‘rovi mac ilovada ko‘rsatiladi (agar tugun == mac ilova bo‘lsa).
- Tugun `invoke-res`ni Gateway’ga qaytaradi.
- Ko‘p bosqichli, UI tugun xostiga bog‘langan.

## Hozirgi mavjudlik + identifikatsiya

- Gateway mavjudlik yozuvlari WS mijozlaridan.
- Tugun mavjudlik yozuvlari bridge’dan.
- mac ilova bir xil qurilma uchun ikkita yozuv ko‘rsatishi mumkin (UI + tugun).
- Tugun identifikatsiyasi pairing store’da; UI identifikatsiyasi alohida.

---

# Muammolar / og‘riqli nuqtalar

- Ikkita protokol stekini qo‘llab‑quvvatlash (WS + Bridge).
- Masofaviy tugunlarda tasdiqlash: so‘rov foydalanuvchi turgan joyda emas, tugun xostida chiqadi.
- TLS pinning faqat bridge’da; WS esa SSH/Tailscale’ga bog‘liq.
- Identifikatsiya dublikatlari: bir xil qurilma bir nechta instansiya sifatida ko‘rinadi.
- Noaniq rollar: UI + tugun + CLI imkoniyatlari aniq ajratilmagan.

---

# Taklif etilayotgan yangi holat (Clawnet)

## Bitta protokol, ikkita rol

Role + scope bilan yagona WS protokoli.

- **Rol: node** (imkoniyatlar xosti)
- **Rol: operator** (boshqaruv tekisligi)
- Operator uchun ixtiyoriy **scope**:
  - `operator.read` (holat + ko‘rish)
  - `operator.write` (agent run, yuborishlar)
  - `operator.admin` (sozlamalar, kanallar, modellar)

### Rol xatti‑harakatlari

**Node**

- Imkoniyatlarni ro‘yxatdan o‘tkazadi (`caps`, `commands`, ruxsatlar).
- `invoke` buyruqlarini qabul qiladi (`system.run`, `camera.*`, `canvas.*`, `screen.record` va boshqalar).
- Hodisalarni yuboradi: `voice.transcript`, `agent.request`, `chat.subscribe`.
- Config/models/channels/sessions/agent boshqaruv API’larini chaqira olmaydi.

**Operator**

- Scope bilan cheklangan to‘liq boshqaruv API.
- Barcha tasdiqlashlarni qabul qiladi.
- OS amallarini bevosita bajarmaydi; tugunlarga yo‘naltiradi.

### Asosiy qoida

Rol har bir ulanish uchun belgilanadi, qurilma bo‘yicha emas. Bitta qurilma alohida ulanishlar orqali ikkala rolni ham ochishi mumkin.

---

# Yagona autentifikatsiya + juftlash

## Mijoz identifikatsiyasi

Har bir mijoz quyidagilarni taqdim etadi:

- `deviceId` (barqaror, qurilma kalitidan hosil qilingan).
- `displayName` (insoniy nom).
- `role` + `scope` + `caps` + `commands`.

## Juftlash jarayoni (yagona)

- Mijoz autentifikatsiyasiz ulanadi.
- Gateway ushbu `deviceId` uchun **pairing request** yaratadi.
- Operator so‘rov oladi; tasdiqlaydi yoki rad etadi.
- Gateway quyidagilarga bog‘langan credential’larni beradi:
  - qurilma public key
  - rol(lar)
  - scope(lar)
  - imkoniyatlar/buyruqlar
- Mijoz token’ni saqlaydi va autentifikatsiya bilan qayta ulanadi.

## Qurilmaga bog‘langan auth (bearer token replay’dan qochish)

Afzal: qurilma keypair’lari.

- Qurilma bir marta keypair yaratadi.
- `deviceId = fingerprint(publicKey)`.
- Gateway nonce yuboradi; qurilma imzolaydi; gateway tekshiradi.
- Tokenlar satrga emas, public key’ga (proof‑of‑possession) beriladi.

Muqobillar:

- mTLS (client sertifikatlari): eng kuchli, ammo operatsion murakkabroq.
- Qisqa muddatli bearer tokenlar faqat vaqtinchalik bosqich sifatida (erta aylantirish + bekor qilish).

## Sokin tasdiqlash (SSH evristikasi)

Zaif bo‘g‘in bo‘lmasligi uchun aniq belgilang. Quyidagilardan biri afzal:

- **Faqat lokal**: mijoz loopback/Unix socket orqali ulanganida auto‑pair.
- **SSH orqali challenge**: gateway nonce beradi; mijoz SSH orqali uni olib isbotlaydi.
- **Jismoniy ishtirok oynasi**: gateway xosti UI’da lokal tasdiqlashdan so‘ng qisqa muddat (masalan, 10 daqiqa) auto‑pair ruxsati.

Har doim auto‑approval’larni log qiling + yozib boring.

---

# TLS hamma joyda (dev + prod)

## Mavjud bridge TLS’dan foydalanish

Joriy TLS runtime + fingerprint pinning’dan foydalanish:

- `src/infra/bridge/server/tls.ts`
- `src/node-host/bridge-client.ts` dagi fingerprint tekshiruvi

## WS’ga qo‘llash

- WS server xuddi shu cert/key + fingerprint bilan TLS’ni qo‘llab‑quvvatlaydi.
- WS mijozlari fingerprint’ni pin qilishi mumkin (ixtiyoriy).
- Discovery barcha endpoint’lar uchun TLS + fingerprint’ni e’lon qiladi.
  - Discovery faqat lokator ko‘rsatmalar; hech qachon ishonch ankori emas.

## Nega

- Maxfiylik uchun SSH/Tailscale’ga bog‘liqlikni kamaytirish.
- Masofaviy mobil ulanishlarni standart bo‘yicha xavfsiz qilish.

---

# Tasdiqlashlarni qayta loyihalash (markazlashgan)

## Hozir

Tasdiqlash tugun xostida amalga oshadi (mac ilova tugun runtime). So‘rov tugun ishlayotgan joyda chiqadi.

## Taklif

Tasdiqlash **gateway’da joylashgan**, UI operator mijozlariga yetkaziladi.

### Yangi oqim

1. Gateway `system.run` intent’ini oladi (agent).
2. Gateway tasdiqlash yozuvini yaratadi: `approval.requested`.
3. Operator UI(lar)i so‘rovni ko‘rsatadi.
4. Tasdiqlash qarori gateway’ga yuboriladi: `approval.resolve`.
5. Tasdiqlansa, Gateway tugun buyrug‘ini chaqiradi.
6. Tugun bajaradi va `invoke-res` qaytaradi.

### Tasdiqlash semantikasi (kuchaytirish)

- Barcha operatorlarga yuboriladi; faqat faol UI modal ko‘rsatadi (boshqalarda toast).
- Birinchi qaror ustun; keyingilari allaqachon hal qilingan deb rad etiladi.
- Standart timeout: N soniyadan so‘ng rad (masalan, 60s), sabab log qilinadi.
- Qaror uchun `operator.approvals` scope talab qilinadi.

## Afzalliklar

- So‘rov foydalanuvchi turgan joyda chiqadi (mac/telefon).
- Masofaviy tugunlar uchun bir xil tasdiqlash mexanizmi.
- Tugun runtime headless qoladi; UI’ga bog‘liq emas.

---

# Rol aniqligi misollari

## iPhone ilovasi

- **Node roli**: mic, camera, voice chat, location, push‑to‑talk.
- Ixtiyoriy **operator.read**: holat va chat ko‘rish.
- **operator.write/admin** faqat aniq yoqilganda.

## macOS ilovasi

- Standart bo‘yicha operator roli (boshqaruv UI).
- “Mac node” yoqilganda node roli (system.run, screen, camera).
- Ikkala ulanish uchun bir xil deviceId → UI’da birlashtirilgan yozuv.

## CLI

- Har doim operator roli.
- Scope subcommand’dan kelib chiqadi:
  - `status`, `logs` → read
  - `agent`, `message` → write
  - `config`, `channels` → admin
  - tasdiqlash + juftlash → `operator.approvals` / `operator.pairing`

---

# Identifikatsiya + sluglar

## Barqaror ID

Auth uchun talab qilinadi; hech qachon o‘zgarmaydi.  
Afzal:

- Keypair fingerprint’i (public key hash).

## Yoqimli slug (lobster mavzusida)

Faqat insoniy yorliq.

- Misol: `scarlet-claw`, `saltwave`, `mantis-pinch`.
- Gateway registrida saqlanadi, tahrirlanadi.
- Kolliziya: `-2`, `-3`.

## UI guruhlash

Bir xil `deviceId` turli rollarda → bitta “Instance” qatori:

- Badge: `operator`, `node`.
- Imkoniyatlar + oxirgi ko‘rilgan vaqt.

---

# Migratsiya strategiyasi

## 0‑bosqich: Hujjatlash + moslash

- Ushbu hujjatni e’lon qilish.
- Barcha protokol chaqiruvlari + tasdiqlash oqimlarini inventarizatsiya qilish.

## 1‑bosqich: WS’ga role/scope qo‘shish

- `connect` parametrlarini `role`, `scope`, `deviceId` bilan kengaytirish.
- Node roli uchun allowlist cheklovlarini qo‘shish.

## 2‑bosqich: Bridge mosligi

- Bridge’ni ishlashda qoldirish.
- Parallel ravishda WS node qo‘llab‑quvvatlashini qo‘shish.
- Feature’larni config flag orqali boshqarish.

## 3‑bosqich: Markazlashgan tasdiqlashlar

- WS’da approval request + resolve hodisalarini qo‘shish.
- mac ilova UI’ni so‘rov ko‘rsatish + javob berishga yangilash.
- Tugun runtime UI ko‘rsatishni to‘xtatadi.

## 4‑bosqich: TLS’ni birlashtirish

- Bridge TLS runtime’dan foydalangan holda WS uchun TLS config qo‘shish.
- Mijozlarga pinning qo‘shish.

## 5‑bosqich: Bridge’ni bekor qilish

- iOS/Android/mac tugunni WS’ga migratsiya qilish.
- Bridge fallback sifatida qoldiriladi; barqarorlashgach olib tashlanadi.

## 6‑bosqich: Qurilmaga bog‘langan auth

- Barcha nolokal ulanishlar uchun key‑based identifikatsiyani majburiy qilish.
- Bekor qilish + aylantirish UI’sini qo‘shish.

---

# Xavfsizlik eslatmalari

- Role/allowlist gateway chegarasida majburiy tekshiriladi.
- Operator scope’siz hech bir mijoz “to‘liq” API olmaydi.
- _Barcha_ ulanishlar uchun juftlash talab qilinadi.
- TLS + pinning mobil uchun MITM xavfini kamaytiradi.
- SSH sokin tasdiqlash qulaylik; baribir yozib olinadi + bekor qilinadi.
- Discovery hech qachon ishonch ankori emas.
- Imkoniyat da’volari platforma/tur bo‘yicha server allowlist’lariga qarshi tekshiriladi.

# Streaming + katta payload’lar (tugun media)

WS boshqaruv tekisligi kichik xabarlar uchun yetarli, ammo tugunlar quyidagilarni ham bajaradi:

- kamera kliplari
- ekran yozuvlari
- audio oqimlar

Variantlar:

1. WS binary frame’lar + chunking + backpressure qoidalari.
2. Alohida streaming endpoint (baribir TLS + auth bilan).
3. Media og‘ir buyruqlar uchun bridge’ni uzoqroq saqlash, oxirida migratsiya qilish.

Drift bo‘lmasligi uchun implementatsiyadan oldin birini tanlang.

# Imkoniyat + buyruq siyosati

- Tugun bildirgan caps/commands **da’vo** sifatida qaraladi.
- Gateway platforma bo‘yicha allowlist’larni majburiy qo‘llaydi.
- Har qanday yangi buyruq operator tasdiqlashi yoki aniq allowlist o‘zgarishini talab qiladi.
- O‘zgarishlar vaqt tamg‘asi bilan audit qilinadi.

# Audit + rate limiting

- Log qilish: juftlash so‘rovlari, tasdiqlash/rad etishlar, token berish/aylantirish/bekor qilish.
- Juftlash spam’i va tasdiqlash so‘rovlarini rate‑limit qilish.

# Protokol gigiyenasi

- Aniq protokol versiyasi + xato kodlari.
- Qayta ulanish qoidalari + heartbeat siyosati.
- Presence TTL va last‑seen semantikasi.

---

# Ochiq savollar

1. Ikkala rolni ham ishlatadigan bitta qurilma: token modeli
   - Tavsiya: har bir rol uchun alohida token (node va operator).
   - Bir xil deviceId; turli scope; aniqroq bekor qilish.

2. Operator scope granularligi
   - read/write/admin + approvals + pairing (minimal variant).
   - Keyinroq feature‑bo‘yicha scope ko‘rib chiqish.

3. Token aylantirish + bekor qilish UX
   - Rol o‘zgarganda avtomatik aylantirish.
   - deviceId + rol bo‘yicha bekor qilish UI’si.

4. Discovery
   - Joriy Bonjour TXT’ni WS TLS fingerprint + rol hint’lari bilan kengaytirish.
   - Faqat lokator ko‘rsatma sifatida qarash.

5. Tarmoqlararo tasdiqlash
   - Barcha operator mijozlariga yuborish; faol UI modal ko‘rsatadi.
   - Birinchi javob ustun; gateway atomiklikni ta’minlaydi.

---

# Xulosa (TL;DR)

- Bugun: WS boshqaruv tekisligi + Bridge tugun transporti.
- Muammo: tasdiqlashlar + dublikatlar + ikki stek.
- Taklif: aniq role + scope’li yagona WS protokoli, yagona juftlash + TLS pinning, gateway’da joylashgan tasdiqlashlar, barqaror device ID + yoqimli sluglar.
- Natija: soddaroq UX, kuchliroq xavfsizlik, kamroq takrorlanish, mobil uchun yaxshiroq marshrutlash.



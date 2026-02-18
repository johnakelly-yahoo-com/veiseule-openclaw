---
title: "Aniqlash va transportlar"
---

# Aniqlash va transportlar

OpenClaw’da tashqi tomondan o‘xshash ko‘rinsa ham, aslida ikki xil muammo mavjud:

1. **Operatorning masofaviy boshqaruvi**: macOS menyu paneli ilovasi boshqa joyda ishlayotgan gateway’ni boshqaradi.
2. **Tugunni juftlash**: iOS/Android (va kelajakdagi tugunlar) gateway’ni topib, xavfsiz juftlashadi.

Dizayn maqsadi — barcha tarmoq aniqlash/e’lon qilish ishlarini **Node Gateway** (`openclaw gateway`) ichida saqlash va mijozlarni (mac ilovasi, iOS) faqat iste’molchi sifatida qoldirish.

## Atamalar

- **Gateway**: holatni (sessiyalar, juftlash, tugunlar reyestri) boshqaradigan va kanallarni ishga tushiradigan yagona uzoq ishlovchi gateway jarayoni. Odatda har bir host uchun bittadan bo‘ladi; ajratilgan ko‘p-gateway sozlamalari ham mumkin.
- **Gateway WS (control plane)**: odatda `127.0.0.1:18789` dagi WebSocket endpoint; `gateway.bind` orqali LAN/tailnet’ga bog‘lanishi mumkin.
- **Direct WS transport**: LAN/tailnet’ga ochiq Gateway WS endpoint (SSH’siz).
- **SSH transport (fallback)**: `127.0.0.1:18789` ni SSH orqali forward qilib masofadan boshqarish.
- **Legacy TCP bridge (deprecated/removed)**: eski tugun transporti (qarang: [Bridge protocol](/gateway/bridge-protocol)); endi aniqlash uchun e’lon qilinmaydi.

Protokol tafsilotlari:

- [Gateway protocol](/gateway/protocol)
- [Bridge protocol (legacy)](/gateway/bridge-protocol)

## Nega “direct” va SSH ikkalasini ham saqlaymiz

- **Direct WS** bir xil tarmoqda va tailnet ichida eng yaxshi UX’ni beradi:
  - LAN’da Bonjour orqali avtomatik aniqlash
  - juftlash tokenlari + ACL’lar gateway tomonidan boshqariladi
  - shell kirishi talab qilinmaydi; protokol yuzasi ixcham va audit qilinadigan bo‘lib qoladi
- **SSH** esa universal fallback bo‘lib qoladi:
  - SSH mavjud bo‘lgan istalgan joyda ishlaydi (hatto aloqasiz tarmoqlar orasida ham)
  - multicast/mDNS muammolarida ham ishlaydi
  - SSH’dan tashqari yangi kiruvchi portlar talab qilmaydi

## Aniqlash manbalari (mijozlar gateway’ni qanday topadi)

### 1) Bonjour / mDNS (faqat LAN)

Bonjour best-effort asosida ishlaydi va tarmoqlar orasidan o‘tmaydi. U faqat “bir xil LAN” qulayligi uchun ishlatiladi.

Maqsad yo‘nalishi:

- **Gateway** o‘zining WS endpoint’ini Bonjour orqali e’lon qiladi.
- Mijozlar ko‘rib chiqadi va “gateway tanlash” ro‘yxatini ko‘rsatadi, so‘ng tanlangan endpoint’ni saqlaydi.

Nosozliklarni aniqlash va beacon tafsilotlari: [Bonjour](/gateway/bonjour).

#### Service beacon tafsilotlari

- Service turlari:
  - `_openclaw-gw._tcp` (gateway transport beacon)
- TXT kalitlari (maxfiy emas):
  - `role=gateway`
  - `lanHost=<hostname>.local`
  - `sshPort=22` (yoki e’lon qilingan boshqa port)
  - `gatewayPort=18789` (Gateway WS + HTTP)
  - `gatewayTls=1` (faqat TLS yoqilganda)
  - `gatewayTlsSha256=<sha256>` (faqat TLS yoqilganda va fingerprint mavjud bo‘lsa)
  - `canvasPort=<port>` (canvas host porti; hozircha canvas host yoqilganda `gatewayPort` bilan bir xil)
  - `cliPath=<path>` (ixtiyoriy; ishga tushiriladigan `openclaw` entrypoint yoki binary’ning to‘liq yo‘li)
  - `tailnetDns=<magicdns>` (ixtiyoriy ishora; Tailscale mavjud bo‘lsa avtomatik aniqlanadi)

Xavfsizlik eslatmalari:

- Bonjour/mDNS TXT yozuvlari **autentifikatsiyalanmagan**. Mijozlar TXT qiymatlarini faqat UX uchun ishora sifatida qabul qilishi kerak.
- Marshrutlash (host/port) TXT’dagi `lanHost`, `tailnetDns` yoki `gatewayPort` o‘rniga **aniqlangan service endpoint** (SRV + A/AAAA) ni afzal ko‘rishi kerak.
- TLS pinlash avval saqlangan pin’ni reklama qilingan `gatewayTlsSha256` bilan hech qachon almashtirishga yo‘l qo‘ymasligi kerak.
- iOS/Android tugunlari aniqlash asosidagi direct ulanishlarni **faqat TLS orqali** amalga oshirishi va birinchi marta pin saqlashdan oldin aniq “ushbu fingerprint’ga ishonish” tasdig‘ini (out-of-band tekshiruv) talab qilishi kerak.

O‘chirish/override:

- `OPENCLAW_DISABLE_BONJOUR=1` e’lon qilishni o‘chiradi.
- `~/.openclaw/openclaw.json` ichidagi `gateway.bind` Gateway’ning bind rejimini boshqaradi.
- `OPENCLAW_SSH_PORT` TXT’da e’lon qilinadigan SSH portini o‘zgartiradi (standart 22).
- `OPENCLAW_TAILNET_DNS` `tailnetDns` ishorasini (MagicDNS) e’lon qiladi.
- `OPENCLAW_CLI_PATH` e’lon qilinadigan CLI yo‘lini o‘zgartiradi.

### 2) Tailnet (tarmoqlararo)

London/Vienna uslubidagi sozlamalarda Bonjour yordam bermaydi. Tavsiya etilgan “direct” manzil:

- Tailscale MagicDNS nomi (afzal) yoki barqaror tailnet IP.

Agar gateway Tailscale ostida ishlayotganini aniqlay olsa, mijozlar uchun (shu jumladan keng hududli beacon’lar uchun) ixtiyoriy ishora sifatida `tailnetDns` ni e’lon qiladi.

### 3) Qo‘lda / SSH manzil

To‘g‘ridan-to‘g‘ri yo‘l bo‘lmaganda (yoki direct o‘chirilgan bo‘lsa), mijozlar har doim loopback gateway portini SSH orqali forward qilib ulanishi mumkin.

Qarang: [Remote access](/gateway/remote).

## Transport tanlash (mijoz siyosati)

Tavsiya etilgan mijoz xatti-harakati:

1. Agar juftlangan direct endpoint sozlangan va mavjud bo‘lsa, undan foydalanish.
2. Aks holda, agar Bonjour LAN’da gateway topsa, bir bosishda “Ushbu gateway’dan foydalanish” variantini taklif qilish va uni direct endpoint sifatida saqlash.
3. Aks holda, agar tailnet DNS/IP sozlangan bo‘lsa, direct ulanishni sinab ko‘rish.
4. Aks holda, SSH’ga fallback qilish.

## Juftlash + autentifikatsiya (direct transport)

Gateway — tugun/mijoz qabul qilish bo‘yicha yagona haqiqat manbai.

- Juftlash so‘rovlari gateway’da yaratiladi/tasdiqlanadi/rad etiladi (qarang: [Gateway pairing](/gateway/pairing)).
- Gateway quyidagilarni majburiy qiladi:
  - autentifikatsiya (token / kalit juftligi)
  - scope’lar/ACL’lar (gateway har bir metodga xom proksi emas)
  - tezlik cheklovlari (rate limits)

## Komponentlar bo‘yicha mas’uliyat

- **Gateway**: aniqlash beacon’larini e’lon qiladi, juftlash qarorlarini boshqaradi va WS endpoint’ni host qiladi.
- **macOS ilovasi**: gateway tanlashda yordam beradi, juftlash so‘rovlarini ko‘rsatadi va faqat fallback sifatida SSH’dan foydalanadi.
- **iOS/Android tugunlari**: qulaylik uchun Bonjour orqali ko‘rib chiqadi va juftlangan Gateway WS’ga ulanadi.



---
summary: "Terminal UI (TUI): istalgan qurilmadan Gateway’ga ulanish"
read_when:
  - Sizga TUI bo‘yicha boshlovchilar uchun qulay qo‘llanma kerak
  - Sizga TUI funksiyalari, buyruqlari va klavish yorliqlarining to‘liq ro‘yxati kerak
title: "TUI"
---

# TUI (Terminal interfeysi)

## Tezkor boshlash

1. Gateway’ni ishga tushiring.

```bash
openclaw gateway
```

2. TUI’ni oching.

```bash
openclaw tui
```

3. Xabar yozing va Enter tugmasini bosing.

Masofaviy Gateway:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Agar Gateway parol orqali autentifikatsiyadan foydalansa, `--password` dan foydalaning.

## Nimalarni ko‘rasiz

- Sarlavha: ulanish URL’i, joriy agent, joriy sessiya.
- Chat jurnali: foydalanuvchi xabarlari, assistent javoblari, tizim bildirishnomalari, vosita kartalari.
- Holat qatori: ulanish/ishlash holati (connecting, running, streaming, idle, error).
- Pastki panel: ulanish holati + agent + sessiya + model + think/verbose/reasoning + tokenlar soni + deliver.
- Kiritish maydoni: avtoto‘ldirishga ega matn muharriri.

## Asosiy tushuncha: agentlar + sessiyalar

- Agentlar — noyob identifikatorlar (masalan, `main`, `research`). Gateway ularning ro‘yxatini taqdim etadi.
- Sessiyalar joriy agentga tegishli bo‘ladi.
- Sessiya kalitlari `agent:<agentId>:<sessionKey>` ko‘rinishida saqlanadi.
  - Agar siz `/session main` deb yozsangiz, TUI uni `agent:<currentAgent>:main` ga kengaytiradi.
  - Agar `/session agent:other:main` deb yozsangiz, siz aniq o‘sha agent sessiyasiga o‘tasiz.
- Sessiya doirasi:
  - `per-sender` (standart): har bir agentda bir nechta sessiya bo‘ladi.
  - `global`: TUI doimo `global` sessiyasidan foydalanadi (tanlash oynasi bo‘sh bo‘lishi mumkin).
- Joriy agent + sessiya har doim pastki panelda ko‘rinadi.

## Yuborish + yetkazish

- Xabarlar Gateway’ga yuboriladi; provayderga yetkazish standart bo‘yicha o‘chiq.
- Yetkazishni yoqing:
  - `/deliver on`
  - yoki Settings paneli orqali
  - yoki `openclaw tui --deliver` bilan ishga tushiring

## Tanlash oynalari + overlay’lar

- Model tanlash: mavjud modellar ro‘yxatini ko‘rsatadi va sessiya uchun override o‘rnatadi.
- Agent tanlash: boshqa agentni tanlash.
- Sessiya tanlash: faqat joriy agentga tegishli sessiyalarni ko‘rsatadi.
- Sozlamalar: deliver, vosita natijalarini kengaytirish va thinking ko‘rinishini yoqish/o‘chirish.

## Klavish yorliqlari

- Enter: xabar yuborish
- Esc: faol jarayonni to‘xtatish
- Ctrl+C: kiritishni tozalash (chiqish uchun ikki marta bosing)
- Ctrl+D: chiqish
- Ctrl+L: model tanlash oynasi
- Ctrl+G: agent tanlash oynasi
- Ctrl+P: sessiya tanlash oynasi
- Ctrl+O: vosita natijalarini kengaytirish/yig‘ish
- Ctrl+T: thinking ko‘rinishini yoqish/o‘chirish (tarix qayta yuklanadi)

## Slash buyruqlar

Asosiy:

- `/help`
- `/status`
- `/agent <id>` (yoki `/agents`)
- `/session <key>` (yoki `/sessions`)
- `/model <provider/model>` (yoki `/models`)

Sessiya boshqaruvi:

- `/think <off|minimal|low|medium|high>`
- `/verbose <on|full|off>`
- `/reasoning <on|off|stream>`
- `/usage <off|tokens|full>`
- `/elevated <on|off|ask|full>` (alias: `/elev`)
- `/activation <mention|always>`
- `/deliver <on|off>`

Sessiya hayot sikli:

- `/new` yoki `/reset` (sessiyani qayta boshlash)
- `/abort` (faol jarayonni to‘xtatish)
- `/settings`
- `/exit`

Boshqa Gateway slash buyruqlari (masalan, `/context`) Gateway’ga uzatiladi va tizim chiqishi sifatida ko‘rsatiladi. Qarang: [Slash commands](/tools/slash-commands).

## Lokal shell buyruqlari

- Qator boshiga `!` qo‘yib, TUI ishlayotgan qurilmada lokal shell buyrug‘ini bajarish mumkin.
- TUI har bir sessiyada lokal bajarishga bir marta ruxsat so‘raydi; rad etilsa, `!` o‘sha sessiya uchun o‘chiriladi.
- Buyruqlar TUI ishchi katalogida yangi, interaktiv bo‘lmagan shell’da bajariladi (`cd`/env saqlanmaydi).
- Yolg‘iz `!` oddiy xabar sifatida yuboriladi; boshidagi bo‘sh joylar lokal bajarishni ishga tushirmaydi.

## Vosita natijalari

- Vosita chaqiruvlari argumentlar + natijalar bilan kartalar ko‘rinishida chiqadi.
- Ctrl+O yig‘ilgan/kengaytirilgan ko‘rinish o‘rtasida almashtiradi.
- Vositalar ishlayotgan paytda oraliq yangilanishlar shu kartaning o‘zida oqim sifatida ko‘rsatiladi.

## Tarix + oqimli javoblar

- Ulanishda TUI so‘nggi tarixni yuklaydi (standart 200 ta xabar).
- Oqimli javoblar yakunlangunga qadar joyida yangilanadi.
- TUI boyroq vosita kartalari uchun agent vosita hodisalarini ham tinglaydi.

## Ulanish tafsilotlari

- TUI Gateway’da `mode: "tui"` sifatida ro‘yxatdan o‘tadi.
- Qayta ulanishlar tizim xabari bilan ko‘rsatiladi; hodisalar uzilishi jurnalga chiqariladi.

## Parametrlar

- `--url <url>`: Gateway WebSocket URL’i (standart: konfiguratsiya yoki `ws://127.0.0.1:<port>`)
- `--token <token>`: Gateway tokeni (agar talab qilinsa)
- `--password <password>`: Gateway paroli (agar talab qilinsa)
- `--session <key>`: Sessiya kaliti (standart: `main`, yoki scope global bo‘lsa `global`)
- `--deliver`: Assistent javoblarini provayderga yetkazish (standart o‘chiq)
- `--thinking <level>`: Yuborishda thinking darajasini override qilish
- `--timeout-ms <ms>`: Agent timeout vaqti (ms) (`agents.defaults.timeoutSeconds` standart)

Eslatma: `--url` o‘rnatilganda, TUI konfiguratsiya yoki muhit (environment) ma’lumotlariga qaytmaydi.
`--token` yoki `--password` ni aniq ko‘rsating. Aniq ko‘rsatilmagan autentifikatsiya ma’lumotlari xatoga olib keladi.

## Muammolarni bartaraf etish

Xabar yuborilgandan so‘ng chiqish yo‘q:

- Gateway ulangan va idle/busy holatda ekanini tasdiqlash uchun TUI’da `/status` ni ishga tushiring.
- Gateway loglarini tekshiring: `openclaw logs --follow`.
- Agent ishlay olishiga ishonch hosil qiling: `openclaw status` va `openclaw models status`.
- Agar xabarlar chat kanaliga yetib borishini kutsangiz, yetkazishni yoqing (`/deliver on` yoki `--deliver`).
- `--history-limit <n>`: Yuklanadigan tarix yozuvlari soni (standart 200)

## Ulanish bilan bog‘liq muammolar

- `disconnected`: Gateway ishlayotganini va `--url/--token/--password` to‘g‘ri ekanini tekshiring.
- Tanlash oynasida agentlar yo‘q: `openclaw agents list` va routing konfiguratsiyangizni tekshiring.
- Bo‘sh sessiya tanlash oynasi: siz global scope’da bo‘lishingiz yoki hali sessiyalar bo‘lmasligi mumkin.

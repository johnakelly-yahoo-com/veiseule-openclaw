---
summary: "Gateway, kanallar, avtomatlashtirish, tugunlar va brauzer uchun chuqur nosozliklarni bartaraf etish qoʻllanmasi"
read_when:
  - The troubleshooting hub pointed you here for deeper diagnosis
  - 15. Sizga aniq buyruqlar bilan barqaror, simptomga asoslangan runbook bo‘limlari kerak
title: "Nosozliklarni bartaraf etish"
---

# Gateway nosozliklarini bartaraf etish

Bu sahifa chuqurlashtirilgan qoʻllanmadir.
Start at [/help/troubleshooting](/help/troubleshooting) if you want the fast triage flow first.

## Buyruqlar ketma-ketligi

Avval quyidagilarni, aynan shu tartibda ishga tushiring:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Kutilayotgan sogʻlom holat belgilar:

- `openclaw gateway status` `Runtime: running` va `RPC probe: ok` holatini koʻrsatadi.
- `openclaw doctor` konfiguratsiya/xizmat bilan bogʻliq toʻsib qoʻyuvchi muammolar yoʻqligini bildiradi.
- `openclaw channels status --probe` ulangan/tayyor kanallarni koʻrsatadi.

## No replies

If channels are up but nothing answers, check routing and policy before reconnecting anything.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list <channel>
openclaw config get channels
openclaw logs --follow
```

Look for:

- 1. DM yuboruvchilar uchun juftlash kutilmoqda.
- 2. Guruh eslatmasini cheklash (`requireMention`, `mentionPatterns`).
- 3. Kanal/guruh ruxsat ro‘yxati nomuvofiqliklari.

4. Keng tarqalgan belgilar:

- 5. `drop guild message (mention required` → eslatma bo‘lmaguncha guruh xabari e’tiborsiz qoldiriladi.
- 6. `pairing request` → yuboruvchi tasdiqlanishi kerak.
- 7. `blocked` / `allowlist` → yuboruvchi/kanal siyosat bo‘yicha filtrlangan.

8. Bog‘liq:

- 9. [/channels/troubleshooting](/channels/troubleshooting)
- 10. [/channels/pairing](/channels/pairing)
- 11. [/channels/groups](/channels/groups)

## 12. Boshqaruv paneli boshqaruv UI ulanishi

13. Boshqaruv paneli/boshqaruv UI ulanmasa, URL, autentifikatsiya rejimi va xavfsiz kontekst taxminlarini tekshiring.

```bash
14. openclaw shlyuzi holati
```

openclaw status

- openclaw logs --follow
- openclaw doctor
- openclaw gateway status --json

15. Qidiring:

- 16. To‘g‘ri probe URL va boshqaruv paneli URL.
- 17. Mijoz va shlyuz o‘rtasida autentifikatsiya rejimi/token nomuvofiqligi.
- 18. Qurilma identifikatori talab qilinadigan joyda HTTP’dan foydalanish.

19. Keng tarqalgan belgilar:

- 20. `device identity required` → xavfsiz bo‘lmagan kontekst yoki qurilma autentifikatsiyasi yetishmaydi.
- 21. `unauthorized` / qayta ulanish sikli → token/parol nomuvofiqligi.
- 22. `gateway connect failed:` → noto‘g‘ri host/port/url manzili.

## 23. Bog‘liq:

24. [/web/control-ui](/web/control-ui)

```bash
25. [/gateway/authentication](/gateway/authentication)
```

26. [/gateway/remote](/gateway/remote)

- 27. Shlyuz xizmati ishga tushmagan
- 28. Xizmat o‘rnatilgan, lekin jarayon ishlashda qolmasa, shundan foydalaning.
- openclaw gateway status

openclaw status

- openclaw logs --follow
- openclaw doctor openclaw gateway status --deep
- 30. Qidiring:

31. `Runtime: stopped` va chiqish bo‘yicha maslahatlar.

- 32. Xizmat konfiguratsiyasi nomuvofiqligi (`Config (cli)` vs `Config (service)`).
- 33. Port/listener ziddiyatlari.
- 34. Keng tarqalgan belgilar:

## 35. `Gateway start blocked: set gateway.mode=local` → lokal shlyuz rejimi yoqilmagan.

36. \`refusing to bind gateway ...

```bash
37. without auth` → token/parolsiz loopback bo‘lmagan bind.
```

38. `another gateway instance is already listening` / `EADDRINUSE` → port ziddiyati.

- 39. Bog‘liq:
- 40. [/gateway/background-process](/gateway/background-process)
- 41. [/gateway/configuration](/gateway/configuration)

42. [/gateway/doctor](/gateway/doctor)

- `mention required` → guruh eslatish siyosati sababli xabar e’tiborsiz qoldiriladi.
- `pairing` / tasdiqlash kutilayotgan izlar → yuboruvchi tasdiqlanmagan.
- `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → kanal autentifikatsiyasi/ruxsatlari muammosi.

Bog‘liq:

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/whatsapp](/channels/whatsapp)
- [/channels/telegram](/channels/telegram)
- [/channels/discord](/channels/discord)

## Cron va heartbeat yetkazib berilishi

Agar cron yoki heartbeat ishga tushmagan yoki yetkazilmagan bo‘lsa, avval rejalashtiruvchi holatini, so‘ng yetkazib berish manzilini tekshiring.

```bash
openclaw cron holati
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Quyidagilarga e’tibor bering:

- Cron yoqilgan va keyingi uyg‘onish vaqti mavjud.
- Ish bajarilish tarixi holati (`ok`, `skipped`, `error`).
- Heartbeat o‘tkazib yuborish sabablari (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Keng tarqalgan belgilar:

- `cron: scheduler disabled; jobs will not run automatically` → cron o‘chirilgan.
- `cron: timer tick failed` → rejalashtiruvchi tik muvaffaqiyatsiz; fayl/log/runtime xatolarini tekshiring.
- `heartbeat skipped` va `reason=quiet-hours` → faol soatlar oynasidan tashqarida.
- `heartbeat: unknown accountId` → heartbeat yetkazib berish manzili uchun yaroqsiz accountId.

Bog‘liq:

- [/automation/troubleshooting](/automation/troubleshooting)
- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)

## Ulangan tugun vositasi ishlamaydi

Tugun ulangan bo‘lsa-yu, vositalar ishlamasa, oldingi rejim, ruxsat va tasdiqlash holatini ajrating.

```bash
openclaw nodes holati
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Quyidagilarga e’tibor bering:

- Tugun kutilgan imkoniyatlar bilan onlayn.
- Kamera/mikrofon/joylashuv/ekran uchun OS ruxsatlari berilgan.
- Exec tasdiqlashlari va allowlist holati.

Keng tarqalgan belgilar:

- `NODE_BACKGROUND_UNAVAILABLE` → tugun ilovasi oldingi rejimda bo‘lishi kerak.
- `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → OS ruxsati yetishmaydi.
- `SYSTEM_RUN_DENIED: approval required` → exec tasdiqlashi kutilmoqda.
- `SYSTEM_RUN_DENIED: allowlist miss` → buyruq allowlist tomonidan bloklangan.

Bog‘liq:

- [/nodes/troubleshooting](/nodes/troubleshooting)
- [/nodes/index](/nodes/index)
- [/tools/exec-approvals](/tools/exec-approvals)

## Brauzer vositasi ishlamaydi

Gateway sog‘lom bo‘lsa ham brauzer vositasi amallari muvaffaqiyatsiz bo‘lganda foydalaning.

```bash
openclaw browser holati
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

Quyidagilarga e’tibor bering:

- Yaroqli brauzer bajariluvchi fayli yo‘li.
- CDP profili yetib borilishi.
- `profile="chrome"` uchun kengaytma relay yorlig‘i biriktirilgan.

Keng tarqalgan belgilar:

- `Failed to start Chrome CDP on port` → brauzer jarayoni ishga tushmadi.
- `browser.executablePath not found` → sozlangan yo‘l yaroqsiz.
- `Chrome extension relay is running, but no tab is connected` → kengaytma relesi ulangan emas.
- `Browser attachOnly is enabled ... `not reachable\` → faqat ulash (attach-only) profili uchun yetib boriladigan maqsad yo‘q.

Bog‘liq:

- [/tools/browser-linux-troubleshooting](/tools/browser-linux-troubleshooting)
- [/tools/chrome-extension](/tools/chrome-extension)
- [/tools/browser](/tools/browser)

## Agar yangilashdan keyin birdan nimadir buzilib qolgan bo‘lsa

Yangilashdan keyingi ko‘p nosozliklar konfiguratsiya siljishi yoki endi qat’iyroq sukut bo‘yicha sozlamalar qo‘llanilayotganidan kelib chiqadi.

### 1. Autentifikatsiya va URLni almashtirish xatti-harakati o‘zgardi

```bash
openclaw gateway holati
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

Nimani tekshirish kerak:

- Agar `gateway.mode=remote` bo‘lsa, CLI chaqiruvlari masofaviy manzilga yo‘naltirilayotgan bo‘lishi mumkin, lokal xizmatingiz esa sog‘lom bo‘lsa ham.
- Aniq `--url` bilan qilingan chaqiruvlar saqlangan hisob ma’lumotlariga qaytmaydi.

Keng tarqalgan belgilar:

- `gateway connect failed:` → noto‘g‘ri URL manzili.
- `unauthorized` → endpoint yetib boriladi, ammo autentifikatsiya noto‘g‘ri.

### 2. Bind va autentifikatsiya himoya cheklovlari qat’iylashdi

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

Nimani tekshirish kerak:

- Loopback bo‘lmagan bindlar (`lan`, `tailnet`, `custom`) uchun autentifikatsiya sozlangan bo‘lishi kerak.
- `gateway.token` kabi eski kalitlar `gateway.auth.token`ni almashtirmaydi.

Keng tarqalgan belgilar:

- `refusing to bind gateway ... without auth` → bind + auth mos kelmasligi.
- `RPC probe: failed` runtime ishlayotgan paytda → gateway tirik, ammo joriy auth/url bilan kirish imkonsiz.

### 3. Juftlash va qurilma identifikatsiyasi holati o‘zgardi

```bash
openclaw devices list
openclaw pairing list <channel>
openclaw logs --follow
openclaw doctor
```

Nimani tekshirish kerak:

- Dashboard/nodlar uchun kutilayotgan qurilma tasdiqlashlari.
- Siyosat yoki identifikatsiya o‘zgarishlaridan keyin kutilayotgan DM juftlash tasdiqlashlari.

Keng tarqalgan belgilar:

- `device identity required` → qurilma autentifikatsiyasi qondirilmagan.
- `pairing required` → jo‘natuvchi/qurilma tasdiqlanishi kerak.

Tekshiruvlardan keyin ham xizmat konfiguratsiyasi va runtime kelishmasa, bir xil profil/holat katalogidan xizmat metama’lumotlarini qayta o‘rnating:

```bash
openclaw gateway install --force
openclaw gateway restart
```

Bog‘liq:

- [/gateway/pairing](/gateway/pairing)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/background-process](/gateway/background-process)

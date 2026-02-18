---
summary: "Agent foydalanishi uchun kamera tasvirga olish (iOS tuguni + macOS ilovasi): fotosuratlar (jpg) va qisqa video roliklar (mp4)"
read_when:
  - iOS tugunlari yoki macOS’da kamera tasvirga olishni qo‘shish yoki o‘zgartirishda
  - Agent kira oladigan MEDIA vaqtinchalik fayl jarayonlarini kengaytirishda
title: "Kamera tasvirga olish"
---

# Kamera tasvirga olish (agent)

OpenClaw agent ish jarayonlari uchun **kamera tasvirga olish**ni qo‘llab-quvvatlaydi:

- **iOS tuguni** (Gateway orqali ulangan): `node.invoke` orqali **fotosurat** (`jpg`) yoki **qisqa video rolik** (`mp4`, ixtiyoriy audio bilan) olish.
- **Android tuguni** (Gateway orqali ulangan): `node.invoke` orqali **fotosurat** (`jpg`) yoki **qisqa video rolik** (`mp4`, ixtiyoriy audio bilan) olish.
- **macOS ilovasi** (Gateway orqali tugun): `node.invoke` orqali **fotosurat** (`jpg`) yoki **qisqa video rolik** (`mp4`, ixtiyoriy audio bilan) olish.

Barcha kamera kirishlari **foydalanuvchi boshqaradigan sozlamalar** orqali nazorat qilinadi.

## iOS tuguni

### Foydalanuvchi sozlamasi (standart yoqilgan)

- iOS Settings yorlig‘i → **Camera** → **Allow Camera** (`camera.enabled`)
  - Standart: **yoqilgan** (kalit mavjud bo‘lmasa ham yoqilgan deb hisoblanadi).
  - O‘chirilganda: `camera.*` buyruqlari `CAMERA_DISABLED` qaytaradi.

### Buyruqlar (Gateway orqali `node.invoke`)

- `camera.list`
  - Javob payload:
    - `devices`: `{ id, name, position, deviceType }` obyektlar massivi

- `camera.snap`
  - Parametrlar:
    - `facing`: `front|back` (standart: `front`)
    - `maxWidth`: son (ixtiyoriy; iOS tugunida standart `1600`)
    - `quality`: `0..1` (ixtiyoriy; standart `0.9`)
    - `format`: hozircha `jpg`
    - `delayMs`: son (ixtiyoriy; standart `0`)
    - `deviceId`: satr (ixtiyoriy; `camera.list` dan)
  - Javob payload:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Payload himoyasi: fotosuratlar base64 payload hajmi 5 MB dan oshmasligi uchun qayta siqiladi.

- `camera.clip`
  - Parametrlar:
    - `facing`: `front|back` (standart: `front`)
    - `durationMs`: son (standart `3000`, maksimal `60000` gacha cheklanadi)
    - `includeAudio`: boolean (standart `true`)
    - `format`: hozircha `mp4`
    - `deviceId`: satr (ixtiyoriy; `camera.list` dan)
  - Javob payload:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

### Old fon talabi

`canvas.*` kabi, iOS tuguni `camera.*` buyruqlariga faqat **old fon** holatida ruxsat beradi. Orqa fondagi chaqiruvlar `NODE_BACKGROUND_UNAVAILABLE` qaytaradi.

### CLI yordamchisi (vaqtinchalik fayllar + MEDIA)

Biriktirmalarni olishning eng oson yo‘li — dekodlangan mediani vaqtinchalik faylga yozib, `MEDIA:<path>` chiqaradigan CLI yordamchisidan foydalanish.

Misollar:

```bash
openclaw nodes camera snap --node <id>               # standart: front + back (2 ta MEDIA qatori)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Eslatmalar:

- `nodes camera snap` standart bo‘yicha agentga ikkala ko‘rinishni berish uchun **ikkala** tomonni suratga oladi.
- Agar o‘zingizning wrapper’ingizni yaratmasangiz, chiqish fayllari vaqtinchalik (OT vaqtinchalik katalogida) bo‘ladi.

## Android tuguni

### Android foydalanuvchi sozlamasi (standart yoqilgan)

- Android Settings oynasi → **Camera** → **Allow Camera** (`camera.enabled`)
  - Standart: **yoqilgan** (kalit mavjud bo‘lmasa ham yoqilgan deb hisoblanadi).
  - O‘chirilganda: `camera.*` buyruqlari `CAMERA_DISABLED` qaytaradi.

### Ruxsatlar

- Android ish vaqtida ruxsatlarni talab qiladi:
  - `CAMERA` — `camera.snap` va `camera.clip` uchun.
  - `RECORD_AUDIO` — `camera.clip` da `includeAudio=true` bo‘lsa.

Agar ruxsatlar mavjud bo‘lmasa, ilova imkon qadar so‘rov yuboradi; rad etilsa, `camera.*` so‘rovlari  
`*_PERMISSION_REQUIRED` xatosi bilan yakunlanadi.

### Android old fon talabi

`canvas.*` kabi, Android tuguni ham `camera.*` buyruqlariga faqat **old fon** holatida ruxsat beradi. Orqa fondagi chaqiruvlar `NODE_BACKGROUND_UNAVAILABLE` qaytaradi.

### Payload himoyasi

Fotosuratlar base64 payload hajmi 5 MB dan oshmasligi uchun qayta siqiladi.

## macOS ilovasi

### Foydalanuvchi sozlamasi (standart o‘chirilgan)

macOS hamroh ilovasi quyidagi belgilash katagini taqdim etadi:

- **Sozlamalar → Umumiy → Kameraga ruxsat berish** (`openclaw.cameraEnabled`)
  - Standart: **o‘chirilgan**
  - O‘chirilgan bo‘lsa: kamera so‘rovlari “Camera disabled by user” qaytaradi.

### CLI yordamchisi (node invoke)

macOS tugunida kamera buyruqlarini chaqirish uchun asosiy `openclaw` CLI’dan foydalaning.

Misollar:

```bash
openclaw nodes camera list --node <id>            # kamera id larini ko‘rsatadi
openclaw nodes camera snap --node <id>            # MEDIA:<path> chiqaradi
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # MEDIA:<path> chiqaradi
openclaw nodes camera clip --node <id> --duration-ms 3000      # MEDIA:<path> chiqaradi (eski flag)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Eslatmalar:

- `openclaw nodes camera snap` standart bo‘yicha, agar o‘zgartirilmasa, `maxWidth=1600` dan foydalanadi.
- macOS’da `camera.snap` suratga olishdan oldin qizish/ekspozitsiya barqarorlashishi uchun `delayMs` (standart 2000ms) kutadi.
- Foto payloadlari base64 hajmi 5 MB dan oshmasligi uchun qayta siqiladi.

## Xavfsizlik + amaliy cheklovlar

- Kamera va mikrofon kirishi odatiy OT ruxsat so‘rovlarini ishga tushiradi (hamda Info.plist’da usage satrlarini talab qiladi).
- Video roliklar tugun payloadi haddan tashqari kattalashib ketmasligi uchun (base64 overhead + xabar cheklovlari) cheklangan (hozircha `<= 60s`).

## macOS ekran videosi (OT darajasida)

_Kamera emas_, balki ekran videosi uchun macOS hamrohidan foydalaning:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # MEDIA:<path> chiqaradi
```

Eslatmalar:

- macOS’da **Screen Recording** ruxsati (TCC) talab qilinadi.

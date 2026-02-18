---
title: "Mavjudlik"
---

# Mavjudlik

OpenClaw “presence” — bu yengil, best‑effort ko‘rinish bo‘lib, quyidagilarni aks ettiradi:

- **Gateway**’ning o‘zi, va
- **Gateway’ga ulangan klientlar** (mac ilovasi, WebChat, CLI va boshqalar)

Presence asosan macOS ilovasining **Instances** tab’ini chizish va operatorga tezkor ko‘rinish berish uchun ishlatiladi.

## Presence maydonlari (nimalar ko‘rinadi)

Presence yozuvlari quyidagi kabi maydonlarga ega strukturali obyektlardir:

- `instanceId` (ixtiyoriy, lekin qat’iy tavsiya etiladi): barqaror klient identifikatori (odatda `connect.client.instanceId`)
- `host`: odamga qulay host nomi
- `ip`: best‑effort IP manzil
- `version`: klient versiyasi satri
- `deviceFamily` / `modelIdentifier`: apparat bo‘yicha ishoralar
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: “oxirgi foydalanuvchi kiritishidan beri soniyalar” (agar ma’lum bo‘lsa)
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: oxirgi yangilanish vaqti (epoch’dan boshlab ms)

## Prodyuserlar (presence qayerdan keladi)

Presence yozuvlari bir nechta manbalar tomonidan yaratiladi va **birlashtiriladi**.

### 1. Gateway’ning o‘z yozuvi

Gateway ishga tushishda har doim “self” yozuvini yaratadi, shunda hech qanday klient ulanmagan bo‘lsa ham UI’larda gateway host ko‘rinadi.

### 2. WebSocket ulanishi

Har bir WS klient `connect` so‘rovi bilan boshlaydi. Handshake muvaffaqiyatli bo‘lgach, Gateway ushbu ulanish uchun presence yozuvini upsert qiladi.

#### 1. Nega bir martalik CLI buyruqlari ko‘rinmaydi

2. CLI ko‘pincha qisqa, bir martalik buyruqlar uchun ulanadi. 3. Instances ro‘yxatini spam qilmaslik uchun, `client.mode === "cli"` **presence** yozuviga aylantirilmaydi.

### 4. 3. `system-event` mayoqchalari

5. Mijozlar `system-event` metodi orqali boyroq davriy mayoqchalarni yuborishi mumkin. 6. macOS ilovasi bundan xost nomi, IP va `lastInputSeconds` ni hisobot qilish uchun foydalanadi.

### 7. 4. Node ulanadi (rol: node)

8. Node Gateway WebSocket orqali `role: node` bilan ulanganda, Gateway ushbu node uchun presence yozuvini upsert qiladi (boshqa WS mijozlari bilan bir xil oqim).

## 9. Birlashtirish + dublikatlarni olib tashlash qoidalari (`instanceId` nega muhim)

10. Presence yozuvlari bitta xotiradagi xaritada saqlanadi:

- 11. Yozuvlar **presence kaliti** bilan kalitlanadi.
- 12. Eng yaxshi kalit — qayta ishga tushirishlardan omon qoladigan barqaror `instanceId` (`connect.client.instanceId` dan).
- 13. Kalitlar katta-kichik harfga sezgir emas.

14. Agar mijoz barqaror `instanceId` siz qayta ulansa, u **dublikat** qator sifatida ko‘rinishi mumkin.

## 15. TTL va cheklangan hajm

16. Presence ataylab efemer:

- 17. **TTL:** 5 daqiqadan eski yozuvlar olib tashlanadi
- 18. **Maks. yozuvlar:** 200 (eng eskilari birinchi bo‘lib olib tashlanadi)

19. Bu ro‘yxatni yangicha saqlaydi va xotiraning cheksiz o‘sishidan qochadi.

## 20. Masofaviy/tunnel ogohlantirishi (loopback IP-lar)

21. Mijoz SSH tunnel / lokal port forward orqali ulanganda, Gateway masofaviy manzilni `127.0.0.1` sifatida ko‘rishi mumkin. 22. Yaxshi mijoz tomonidan xabar qilingan IP ustiga yozib yubormaslik uchun, loopback masofaviy manzillar e’tiborga olinmaydi.

## 23. Iste’molchilar

### 24. macOS Instances yorlig‘i

25. macOS ilovasi `system-presence` chiqishini render qiladi va oxirgi yangilanish yoshiga qarab kichik holat indikatori (Active/Idle/Stale) ni qo‘llaydi.

## 26. Nosozliklarni tuzatish bo‘yicha maslahatlar

- 27. Xom ro‘yxatni ko‘rish uchun Gateway ga `system-presence` chaqiruvini yuboring.
- 28. Agar dublikatlarni ko‘rsangiz:
  - 29. mijozlar handshake paytida barqaror `client.instanceId` yuborayotganini tasdiqlang
  - 30. davriy mayoqchalar xuddi shu `instanceId` dan foydalanayotganini tasdiqlang
  - 31. ulanishdan kelib chiqqan yozuvda `instanceId` yo‘qligini tekshiring (bunday holatda dublikatlar kutiladi)

---
summary: "OpenClaw qanday qilib auth profillarini aylantiradi va modellar o‘rtasida fallback qiladi"
read_when:
  - Auth profili aylanishi, cooldown’lar yoki model fallback xatti-harakatlarini diagnostika qilayotganda
  - Auth profillari yoki modellar uchun failover qoidalarini yangilayotganda
title: "Model Fallback"
---

# Model fallback

OpenClaw xatolarni ikki bosqichda boshqaradi:

1. Joriy provayder ichida **auth profillarini aylantirish**.
2. `agents.defaults.model.fallbacks` dagi keyingi modelga **model fallback** qilish.

Ushbu hujjat ish vaqtida qo‘llaniladigan qoidalar va ularni qo‘llab-quvvatlovchi ma’lumotlarni tushuntiradi.

## Auth saqlash (kalitlar + OAuth)

OpenClaw API kalitlari hamda OAuth tokenlari uchun **auth profillaridan** foydalanadi.

- Maxfiy ma’lumotlar `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` faylida saqlanadi (eski joylashuv: `~/.openclaw/agent/auth-profiles.json`).
- `auth.profiles` / `auth.order` konfiguratsiyasi faqat **metadata + marshrutlash** uchun (maxfiy ma’lumotlarsiz).
- Faqat import uchun mo‘ljallangan eski OAuth fayli: `~/.openclaw/credentials/oauth.json` (birinchi foydalanishda `auth-profiles.json` ga import qilinadi).

Batafsil: [/concepts/oauth](/concepts/oauth)

Credential turlari:

- `type: "api_key"` → `{ provider, key }`
- `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ ayrim provayderlar uchun `projectId`/`enterpriseUrl`)

## Profil IDlari

OAuth orqali kirish har bir akkaunt birga mavjud bo‘lishi uchun alohida profil yaratadi.

- Standart: agar email mavjud bo‘lmasa `provider:default`.
- Email bilan OAuth: `provider:<email>` (masalan `google-antigravity:user@gmail.com`).

Profillar `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` faylida `profiles` bo‘limida saqlanadi.

## Aylanish tartibi

Agar provayderda bir nechta profil bo‘lsa, OpenClaw quyidagi tartibda tanlaydi:

1. **Aniq konfiguratsiya**: `auth.order[provider]` (agar o‘rnatilgan bo‘lsa).
2. **Konfiguratsiyalangan profillar**: provayder bo‘yicha filtrlab olingan `auth.profiles`.
3. **Saqlangan profillar**: `auth-profiles.json` dagi shu provayderga tegishli yozuvlar.

Agar aniq tartib sozlanmagan bo‘lsa, OpenClaw round‑robin tartibidan foydalanadi:

- **Asosiy kalit:** profil turi (**API kalitlaridan oldin OAuth**).
- **Ikkinchi kalit:** `usageStats.lastUsed` (har bir tur ichida eng eski birinchi).
- **Cooldown/disabled profillar** oxiriga o‘tkaziladi va muddati eng yaqin tugaydigan birinchi bo‘lib joylashtiriladi.

### Sessiya bo‘yicha biriktirish (kesh uchun qulay)

OpenClaw **tanlangan auth profilini sessiya bo‘yicha biriktiradi**, provayder keshlarini “issiq” saqlash uchun.  
Har bir so‘rovda aylantirilmaydi. Biriktirilgan profil quyidagi holatlargacha qayta ishlatiladi:

- sessiya tiklanganda (`/new` / `/reset`)
- compaction yakunlanganda (compaction soni oshganda)
- profil cooldown yoki disabled holatida bo‘lsa

`/model …@<profileId>` orqali qo‘lda tanlash ushbu sessiya uchun **foydalanuvchi override** ni o‘rnatadi  
va yangi sessiya boshlanmaguncha avtomatik aylantirilmaydi.

Avtomatik biriktirilgan profillar (sessiya router tomonidan tanlangan) **afzallik** sifatida ko‘riladi:  
ular birinchi sinab ko‘riladi, ammo rate limit yoki timeout bo‘lsa OpenClaw boshqa profilga o‘tishi mumkin.  
Foydalanuvchi biriktirgan profillar esa o‘sha profilga qat’iy bog‘lanadi; agar u ishlamasa va model fallback  
sozlangan bo‘lsa, OpenClaw profilni almashtirish o‘rniga keyingi modelga o‘tadi.

### Nega OAuth “yo‘qolib qolgandek” ko‘rinishi mumkin

Agar bir provayder uchun ham OAuth, ham API kalit profillari bo‘lsa, round‑robin xabarlar orasida ularni almashtirishi mumkin (agar biriktirilmagan bo‘lsa). Bitta profilni majburan ishlatish uchun:

- `auth.order[provider] = ["provider:profileId"]` bilan biriktiring, yoki
- UI/chat interfeysingiz qo‘llab-quvvatlasa, `/model …` orqali sessiya bo‘yicha override ishlating.

## Cooldown’lar

Agar profil auth/rate‑limit xatolari (yoki rate limitga o‘xshash timeout) sababli muvaffaqiyatsiz tugasa, OpenClaw uni cooldown holatiga o‘tkazadi va keyingi profilga o‘tadi. Format/invalid‑request xatolari (masalan, Cloud Code Assist tool call ID tekshiruv xatolari) ham failover talab qiluvchi deb hisoblanadi va xuddi shu cooldown mexanizmidan foydalanadi.

Cooldown’lar eksponensial backoff asosida:

- 1 daqiqa
- 5 daqiqa
- 25 daqiqa
- 1 soat (maksimal)

Holat `auth-profiles.json` faylida `usageStats` ostida saqlanadi:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## Billing sababli o‘chirish

Billing/kredit xatolari (masalan, “insufficient credits” / “credit balance too low”) ham failover talab qiladi, ammo ular odatda vaqtinchalik emas. Qisqa cooldown o‘rniga OpenClaw profilni **disabled** deb belgilaydi (uzoqroq backoff bilan) va keyingi profil/provayderga o‘tadi.

Holat `auth-profiles.json` faylida saqlanadi:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Standart sozlamalar:

- Billing backoff **5 soatdan** boshlanadi, har bir billing xatosida ikki baravar oshadi va **24 soat** bilan cheklanadi.
- Agar profil **24 soat** davomida xatoga uchramasa (sozlanishi mumkin), backoff hisoblagichlari tiklanadi.

## Modelning zaxira rejimi

Agar provayderning barcha profillari muvaffaqiyatsiz tugasa, OpenClaw  
`agents.defaults.model.fallbacks` dagi keyingi modelga o‘tadi. Bu auth xatolari, rate limit va  
profil aylanishi tugagan timeout holatlariga tegishli (boshqa xatolar fallback’ni davom ettirmaydi).

Agar ishga tushirish model override (hook yoki CLI orqali) bilan boshlangan bo‘lsa ham, fallback’lar  
sozlangan fallback’lar sinab ko‘rilgach, baribir `agents.defaults.model.primary` da yakunlanadi.

## Tegishli konfiguratsiya

Qarang: [Gateway configuration](/gateway/configuration):

- `auth.profiles` / `auth.order`
- `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
- `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
- `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel` marshrutlash

Kengroq model tanlash va fallback haqida umumiy ma’lumot uchun [Models](/concepts/models) sahifasiga qarang.

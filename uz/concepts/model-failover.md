---
summary: "OpenClaw qanday qilib auth profillarini aylantiradi va modellar o‘rtasida fallback qiladi"
read_when:
  - Auth profili aylanishi, cooldown’lar yoki model fallback xatti-harakatlarini diagnostika qilayotganda
  - Auth profillari yoki modellar uchun failover qoidalarini yangilayotganda
title: "Model Fallback"
---

# Model failover

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
- `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` for some providers)

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

1. OpenClaw **tanlangan auth profilini sessiya davomida mahkamlaydi** — bu provayder keshlarini “issiq” holatda saqlash uchun.
2. U har bir so‘rovda **aylantirilmaydi**. 3. Mahkamlangan profil quyidagilargacha qayta ishlatiladi:

- sessiya tiklanganda (`/new` / `/reset`)
- compaction yakunlanganda (compaction soni oshganda)
- profil cooldown yoki disabled holatida bo‘lsa

`/model …@<profileId>` orqali qo‘lda tanlash ushbu sessiya uchun **foydalanuvchi override** ni o‘rnatadi  
va yangi sessiya boshlanmaguncha avtomatik aylantirilmaydi.

8. Avtomatik mahkamlangan profillar (sessiya routeri tomonidan tanlangan) **afzallik** sifatida qaraladi:
   ular avval sinab ko‘riladi, ammo rate limitlar/timeoutlar bo‘lsa, OpenClaw boshqa profilga o‘tishi mumkin.
9. Foydalanuvchi mahkamlagan profillar o‘sha profilga qulflanib qoladi; agar u ishlamasa va model fallbacklar
   sozlangan bo‘lsa, OpenClaw profilni almashtirish o‘rniga keyingi modelga o‘tadi.

### Nega OAuth “yo‘qolib qolgandek” ko‘rinishi mumkin

11. Agar bir provayder uchun ham OAuth profili, ham API key profili bo‘lsa, round‑robin mahkamlashsiz xabarlar orasida ular o‘rtasida almashishi mumkin. 12. Bitta profilni majburlash uchun:

- `auth.order[provider] = ["provider:profileId"]` bilan biriktiring, yoki
- UI/chat interfeysingiz qo‘llab-quvvatlasa, `/model …` orqali sessiya bo‘yicha override ishlating.

## Cooldown’lar

16. Profil auth/rate‑limit xatolari (yoki rate limitingga o‘xshash timeout) sababli ishlamay qolsa, OpenClaw uni cooldown holatiga o‘tkazadi va keyingi profilga o‘tadi.
17. Format/yaroqsiz so‘rov xatolari (masalan, Cloud Code Assist tool call ID tekshiruv xatolari) ham failoverga loyiq deb qaraladi va xuddi shu cooldownlardan foydalanadi.

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

26. Billing/kredit xatolari (masalan, “insufficient credits” / “credit balance too low”) failoverga loyiq deb qaraladi, ammo ular odatda vaqtinchalik bo‘lmaydi. 27. Qisqa cooldown o‘rniga, OpenClaw profilni **o‘chirilgan** deb belgilaydi (uzoqroq backoff bilan) va keyingi profil/provayderga o‘tadi.

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

## Model fallback

34. Agar provayder uchun barcha profillar ishlamasa, OpenClaw `agents.defaults.model.fallbacks` dagi keyingi modelga o‘tadi. 35. Bu auth xatolari, rate limitlar va
    profil aylantirish tugagan timeoutlarga taalluqli (boshqa xatolar fallbackni oldinga siljitmaydi).

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


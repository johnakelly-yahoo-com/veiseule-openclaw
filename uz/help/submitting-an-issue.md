---
title: "Muammo yuborish"
---

## Muammo yuborish

Aniq va lo‘nda yozilgan muammolar tashxis va tuzatish jarayonini tezlashtiradi. Xatoliklar, regressiyalar yoki funksional bo‘shliqlar uchun quyidagilarni kiriting:

### Nimalarni kiritish kerak

- [ ] Sarlavha: soha va alomat
- [ ] Minimal takrorlash (repro) qadamlari
- [ ] Kutilgan natija va amaldagi natija
- [ ] Ta’siri va jiddiylik darajasi
- [ ] Muhit: OT, runtime, versiyalar, konfiguratsiya
- [ ] Dalillar: maxfiy ma’lumotlari olib tashlangan loglar, skrinshotlar (PIIsiz)
- [ ] Qamrov: yangi, regressiya yoki uzoq vaqtdan beri mavjud
- [ ] Kod so‘zi: muammo matniga lobster-biscuit kiriting
- [ ] Mavjud muammo bor-yo‘qligini tekshirish uchun kod bazasi va GitHub’da qidirildi
- [ ] Yaqinda tuzatilmagan/hal qilinmaganligi tasdiqlandi (ayniqsa xavfsizlik bo‘yicha)
- [ ] Da’volar dalil yoki takrorlash qadamlari bilan tasdiqlangan

Qisqa yozing. Mukammal grammatikadan ko‘ra lo‘ndalik muhim.

Tekshiruv (PR yuborishdan oldin ishga tushiring/tuzating):

- `pnpm lint`
- `pnpm check`
- `pnpm build`
- `pnpm test`
- Agar protokol kodi bo‘lsa: `pnpm protocol:check`

### Shablonlar

#### Xatolik hisoboti

```md
- [ ] Minimal repro
- [ ] Expected vs actual
- [ ] Environment
- [ ] Affected channels, where not seen
- [ ] Logs/screenshots (redacted)
- [ ] Impact/severity
- [ ] Workarounds

### Summary

### Repro Steps

### Expected

### Actual

### Environment

### Logs/Evidence

### Impact

### Workarounds
```

#### Xavfsizlik muammosi

```md
### Summary

### Impact

### Versions

### Repro Steps (safe to share)

### Mitigation/workaround

### Evidence (redacted)
```

_Ochiq joyda maxfiy ma’lumotlar yoki eksploit tafsilotlarini keltirmang. Nozik muammolar uchun tafsilotlarni minimallashtiring va yopiq tarzda oshkor qilishni so‘rang._

#### Regressiya hisoboti

```md
### Summary

### Last Known Good

### First Known Bad

### Repro Steps

### Expected

### Actual

### Environment

### Logs/Evidence

### Impact
```

#### Funksiya so‘rovi

```md
### Summary

### Problem

### Proposed Solution

### Alternatives

### Impact

### Evidence/examples
```

#### Yaxshilash (Enhancement)

```md
### Summary

### Current vs Desired Behavior

### Rationale

### Alternatives

### Evidence/examples
```

#### Tekshiruv (Investigation)

```md
### Summary

### Symptoms

### What Was Tried

### Environment

### Logs/Evidence

### Impact
```

### Tuzatish uchun PR yuborish

PR’dan oldin muammo ochish majburiy emas. Agar o‘tkazib yuborsangiz, PR ichida batafsil ma’lumot bering. PR’ni aniq bir vazifaga yo‘naltiring, muammo raqamini ko‘rsating, testlar qo‘shing yoki nima uchun yo‘qligini tushuntiring, xulq-atvor o‘zgarishlari/xavflarni hujjatlashtiring, dalil sifatida maxfiy ma’lumotlari olib tashlangan loglar/skrinshotlarni qo‘shing va yuborishdan oldin tegishli tekshiruvlarni ishga tushiring.

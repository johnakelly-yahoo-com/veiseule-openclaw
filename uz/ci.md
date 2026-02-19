---
title: CI Pipeline
description: OpenClaw CI pipeline qanday ishlaydi
---

# CI Pipeline

CI `main` ga har bir push qilinganda va har bir pull requestda ishga tushadi. U faqat hujjatlar yoki native kod o‘zgarganda qimmat jarayonlarni o‘tkazib yuborish uchun aqlli skouplardan foydalanadi.

## Vazifalar umumiy ko‘rinishi

| Vazifa            | Maqsad                                                                      | Qachon ishga tushadi                               |
| ----------------- | --------------------------------------------------------------------------- | -------------------------------------------------- |
| `docs-scope`      | Faqat hujjatlardagi o‘zgarishlarni aniqlash                                 | Har doim                                           |
| `changed-scope`   | Qaysi qismlar o‘zgarganini aniqlash (node/macos/android) | Hujjatlardan tashqari PRlar                        |
| `check`           | TypeScript turlari, lint, format                                            | Hujjatlardan tashqari o‘zgarishlar                 |
| `check-docs`      | Markdown lint + buzilgan havolalarni tekshirish                             | Hujjatlar o‘zgarganda                              |
| `code-analysis`   | LOC chegarasini tekshirish (1000 qator)                  | Faqat PRlar                                        |
| `secrets`         | Oshkor qilingan maxfiy ma’lumotlarni aniqlash                               | Har doim                                           |
| `build-artifacts` | Dist ni bir marta build qilish va boshqa vazifalar bilan ulashish           | Hujjatlardan tashqari, node o‘zgarishlari          |
| `release-check`   | npm pack tarkibini tekshirish                                               | Builddan keyin                                     |
| `checks`          | Node/Bun testlari + protokol tekshiruvi                                     | Hujjatlardan tashqari, node o‘zgarishlari          |
| `checks-windows`  | Windows-ga xos testlar                                                      | Hujjatlardan tashqari, node o‘zgarishlari          |
| `macos`           | Swift lint/build/test + TS testlari                                         | macos o‘zgarishlari bo‘lgan PRlar                  |
| `android`         | Gradle build + testlar                                                      | Docs bo‘lmagan o‘zgarishlar, android o‘zgarishlari |

## Fail-Fast tartibi

Ishlar shunday tartiblangan-ki, arzon tekshiruvlar qimmatlari ishga tushishidan oldin xatolik bilan to‘xtaydi:

1. `docs-scope` + `code-analysis` + `check` (parallel, ~1-2 daqiqa)
2. `build-artifacts` (yuqoridagilarga bog‘liq)
3. `checks`, `checks-windows`, `macos`, `android` (build’ga bog‘liq)

## Runner

| Ishlar                                     | `blacksmith-4vcpu-ubuntu-2404`  |
| ------------------------------------------ | ------------------------------- |
| Aksariyat Linux ishlari                    | `blacksmith-4vcpu-windows-2025` |
| `checks-windows`                           | `macos-latest`                  |
| `macos`, `ios`                             | `ubuntu-latest`                 |
| Scope aniqlash (yengil) | Mahalliy ekvivalentlar          |

pnpm check          # turlar + lint + format
pnpm test           # vitest testlari
pnpm check:docs     # docs format + lint + buzilgan havolalar
pnpm release:check  # npm pack tekshiruvi
---------------------------------------------------------

```bash
**📎 bootstrap-extra-files**: `agent:bootstrap` jarayonida sozlangan glob/path shablonlari orqali qo‘shimcha workspace bootstrap fayllarini qo‘shadi
```


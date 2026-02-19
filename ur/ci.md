---
title: CI پائپ لائن
description: OpenClaw CI پائپ لائن کیسے کام کرتی ہے
---

# CI پائپ لائن

CI ہر بار `main` پر push اور ہر pull request پر چلتی ہے۔ یہ اسمارٹ اسکوپنگ استعمال کرتی ہے تاکہ جب صرف ڈاکس یا نیٹو کوڈ تبدیل ہو تو مہنگے جابز کو چھوڑ دیا جائے۔

## جاب کا جائزہ

| جاب               | مقصد                                                                          | یہ کب چلتا ہے               |
| ----------------- | ----------------------------------------------------------------------------- | --------------------------- |
| `docs-scope`      | صرف ڈاکس میں تبدیلیوں کا پتہ لگانا                                            | ہمیشہ                       |
| `changed-scope`   | کن حصوں میں تبدیلی ہوئی ہے معلوم کرنا (node/macos/android) | غیر ڈاکس PRs                |
| `check`           | TypeScript ٹائپس، lint، فارمیٹ                                                | غیر ڈاکس تبدیلیاں           |
| `check-docs`      | Markdown lint + ٹوٹی ہوئی لنکس کی جانچ                                        | ڈاکس میں تبدیلی             |
| `code-analysis`   | LOC حد کی جانچ (1000 لائنیں)                               | صرف PRs                     |
| `secrets`         | لیک شدہ سیکرٹس کا پتہ لگانا                                                   | ہمیشہ                       |
| `build-artifacts` | dist ایک بار بنائیں، دیگر جابز کے ساتھ شیئر کریں                              | غیر ڈاکس، node میں تبدیلیاں |
| `release-check`   | npm pack کے مواد کی توثیق                                                     | بلڈ کے بعد                  |
| `checks`          | Node/Bun ٹیسٹس + پروٹوکول چیک                                                 | غیر ڈاکس، node میں تبدیلیاں |
| `checks-windows`  | Windows کے مخصوص ٹیسٹس                                                        | غیر ڈاکس، node میں تبدیلیاں |
| `macos`           | Swift lint/build/test + TS ٹیسٹس                                              | macos تبدیلیوں والے PRs     |
| `android`         | Gradle build + ٹیسٹس                                                          | Non-docs، android تبدیلیاں  |

## Fail-Fast ترتیب

Jobs کو اس طرح ترتیب دیا گیا ہے کہ سستے چیکس مہنگے چیکس سے پہلے فیل ہو جائیں:

1. `docs-scope` + `code-analysis` + `check` (متوازی، ~1-2 منٹ)
2. `build-artifacts` (اوپر والے مراحل پر منحصر)
3. `checks`, `checks-windows`, `macos`, `android` (build پر منحصر)

## رنرز

| رنر                             | جابز                                           |
| ------------------------------- | ---------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | زیادہ تر Linux جابز                            |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                               |
| `macos-latest`                  | `macos`, `ios`                                 |
| `ubuntu-latest`                 | اسکوپ کی شناخت (ہلکا پھلکا) |

## مقامی متبادل

```bash
pnpm check          # اقسام + لنٹ + فارمیٹ
pnpm test           # vitest ٹیسٹس
pnpm check:docs     # ڈاکس فارمیٹ + لنٹ + خراب لنکس
pnpm release:check  # npm pack کی توثیق
```


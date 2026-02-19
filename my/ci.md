---
title: CI Pipeline
description: OpenClaw CI pipeline အလုပ်လုပ်ပုံ
---

# CI Pipeline

CI သည် `main` သို့ push လုပ်တိုင်းနှင့် pull request တိုင်းတွင် အလုပ်လုပ်ဆောင်ပါသည်။ docs သို့မဟုတ် native code များသာ ပြောင်းလဲခဲ့ပါက ကုန်ကျစရိတ်မြင့်မားသော job များကို ကျော်ရန် smart scoping ကို အသုံးပြုပါသည်။

## Job အကျဉ်းချုပ်

| Job               | ရည်ရွယ်ချက်                                                                                | မည်သည့်အချိန်တွင် လည်ပတ်သနည်း           |
| ----------------- | ------------------------------------------------------------------------------------------ | --------------------------------------- |
| `docs-scope`      | docs သာ ပြောင်းလဲမှုများကို ရှာဖွေခြင်း                                                    | အမြဲတမ်း                                |
| `changed-scope`   | မည်သည့် အပိုင်းများ ပြောင်းလဲခဲ့သည်ကို ရှာဖွေခြင်း (node/macos/android) | Docs မဟုတ်သော PR များ                   |
| `check`           | TypeScript types, lint နှင့် format စစ်ဆေးခြင်း                                            | Docs မဟုတ်သော ပြောင်းလဲမှုများ          |
| `check-docs`      | Markdown lint နှင့် ပျက်နေသော link များကို စစ်ဆေးခြင်း                                     | Docs ပြောင်းလဲခဲ့သည်                    |
| `code-analysis`   | LOC အကန့်အသတ် စစ်ဆေးခြင်း (စာကြောင်း 1000)                              | PR များတွင်သာ                           |
| `secrets`         | ပေါက်ကြားသွားသော secrets များကို ရှာဖွေခြင်း                                               | အမြဲတမ်း                                |
| `build-artifacts` | dist ကို တစ်ကြိမ်တည်း build လုပ်ပြီး အခြား job များနှင့် မျှဝေခြင်း                        | Docs မဟုတ်သော၊ node ပြောင်းလဲမှုများ    |
| `release-check`   | npm pack အတွင်းပါဝင်သည့် အကြောင်းအရာများကို စစ်ဆေးအတည်ပြုခြင်း                             | build ပြီးနောက်                         |
| `checks`          | Node/Bun tests နှင့် protocol စစ်ဆေးခြင်း                                                  | Docs မဟုတ်သော၊ node ပြောင်းလဲမှုများ    |
| `checks-windows`  | Windows အထူးပြု tests များ                                                                 | Docs မဟုတ်သော၊ node ပြောင်းလဲမှုများ    |
| `macos`           | Swift lint/build/test နှင့် TS tests                                                       | macos ပြောင်းလဲမှုများပါဝင်သော PR များ  |
| `android`         | Gradle build + စမ်းသပ်မှုများ                                                              | docs မဟုတ်သော၊ android ပြောင်းလဲမှုများ |

## Fail-Fast အစီအစဉ်

စျေးသက်သာသော စစ်ဆေးမှုများ မအောင်မြင်ပါက စျေးကြီးသော အလုပ်များ မလုပ်မီ အလုပ်များကို အစီအစဉ်တကျ စီထားသည်:

1. `docs-scope` + `code-analysis` + `check` (ပြိုင်တူ, ~1-2 မိနစ်)
2. `build-artifacts` (အထက်ပါများ အောင်မြင်မှသာ လုပ်ဆောင်မည်)
3. `checks`, `checks-windows`, `macos`, `android` (build အပေါ် မူတည်)

## Runners

| Runner                          | Jobs                                           |
| ------------------------------- | ---------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | Linux အလုပ်များ အများစု                        |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                               |
| `macos-latest`                  | `macos`, `ios`                                 |
| `ubuntu-latest`                 | Scope စစ်ဆေးခြင်း (ပေါ့ပါး) |

## Local Equivalents

```bash
pnpm check          # types + lint + format ကို စစ်ဆေးခြင်း
pnpm test           # vitest စမ်းသပ်မှုများ
pnpm check:docs     # docs format + lint + လင့်ခ်ပျက်များ စစ်ဆေးခြင်း
pnpm release:check  # npm pack ကို အတည်ပြုခြင်း
```


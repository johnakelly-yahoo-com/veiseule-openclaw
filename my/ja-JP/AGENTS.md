# AGENTS.md - ja-JP စာရွက်စာတမ်း ဘာသာပြန် workspace

## ဖတ်ရှုရန် အချိန်

- `docs/ja-JP/**` ကို ထိန်းသိမ်းစဉ်
- ဂျပန်ဘာသာပြန် pipeline (glossary/TM/prompt) ကို အပ်ဒိတ်လုပ်စဉ်
- ဂျပန်ဘာသာပြန်နှင့် ပတ်သက်သော feedback သို့မဟုတ် regression များကို ကိုင်တွယ်စဉ်

## Pipeline (docs-i18n)

- မူရင်း စာရွက်စာတမ်းများ: `docs/**/*.md`
- ပစ်မှတ် စာရွက်စာတမ်းများ: `docs/ja-JP/**/*.md`
- Glossary: `docs/.i18n/glossary.ja-JP.json`
- Translation memory: `docs/.i18n/ja-JP.tm.jsonl`
- Prompt စည်းမျဉ်းများ: `scripts/docs-i18n/prompt.go`

အသုံးများသော လည်ပတ်မှုများ:

````bash
```# Bulk (doc mode; parallel OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Single file
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Small patches (segment mode; uses TM; no parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```
````

မှတ်ချက်များ:

- စာမျက်နှာတစ်ခုလုံး ဘာသာပြန်ရန် `doc` mode ကို အသုံးပြုပါ; အသေးစား ပြင်ဆင်မှုများအတွက် `segment` mode ကို အသုံးပြုပါ။
- အလွန်ကြီးမားသော ဖိုင်တစ်ခုသည် အချိန်ကုန်သွားပါက ရည်ရွယ်ထားသော ပြင်ဆင်မှုများ ပြုလုပ်ပါ သို့မဟုတ် စာမျက်နှာကို ခွဲပြီး ပြန်လည် လည်ပတ်ပါ။
- ဘာသာပြန်ပြီးနောက် စစ်ဆေးပါ: code span/block များ မပြောင်းလဲရ၊ link/anchor များ မပြောင်းလဲရ၊ placeholder များ ထိန်းသိမ်းထားရ။

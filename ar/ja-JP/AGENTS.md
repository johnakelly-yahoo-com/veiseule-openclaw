# AGENTS.md - مساحة عمل ترجمة مستندات ja-JP

## اقرأ عند

- صيانة `docs/ja-JP/**`
- تحديث مسار ترجمة اللغة اليابانية (glossary/TM/prompt)
- التعامل مع ملاحظات أو تراجعات ترجمة اللغة اليابانية

## المسار (docs-i18n)

- المستندات المصدر: `docs/**/*.md`
- الملفات المستهدفة: `docs/ja-JP/**/*.md`
- مسرد المصطلحات: `docs/.i18n/glossary.ja-JP.json`
- ذاكرة الترجمة: `docs/.i18n/ja-JP.tm.jsonl`
- قواعد الموجه: `scripts/docs-i18n/prompt.go`

عمليات التشغيل الشائعة:

```bash
# Bulk (doc mode; parallel OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Single file
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Small patches (segment mode; uses TM; no parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

ملاحظات:

- يفضَّل استخدام وضع `doc` لترجمة الصفحة كاملة؛ واستخدام وضع `segment` للإصلاحات الصغيرة.
- إذا انتهت مهلة ملف كبير جدًا، فأجرِ تعديلات موجهة أو قسّم الصفحة قبل إعادة التشغيل.
- بعد الترجمة، تحقّق سريعًا من: عدم تغيير مقاطع/كتل الكود، وعدم تغيير الروابط/المراسي، والحفاظ على العناصر النائبة.

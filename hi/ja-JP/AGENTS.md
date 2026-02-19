# AGENTS.md - ja-JP दस्तावेज़ अनुवाद कार्यक्षेत्र

## कब पढ़ें

- `docs/ja-JP/**` का रखरखाव करते समय
- जापानी अनुवाद पाइपलाइन (glossary/TM/prompt) को अपडेट करते समय
- जापानी अनुवाद से संबंधित प्रतिक्रिया या रिग्रेशन को संभालते समय

## पाइपलाइन (docs-i18n)

- स्रोत दस्तावेज़: `docs/**/*.md`
- लक्ष्य दस्तावेज़: `docs/ja-JP/**/*.md`
- शब्दावली: `docs/.i18n/glossary.ja-JP.json`
- अनुवाद मेमोरी: `docs/.i18n/ja-JP.tm.jsonl`
- प्रॉम्प्ट नियम: `scripts/docs-i18n/prompt.go`

सामान्य रन:

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

नोट्स:

- पूरे पृष्ठ के अनुवाद के लिए `doc` मोड को प्राथमिकता दें; छोटे सुधारों के लिए `segment` मोड का उपयोग करें।
- यदि कोई बहुत बड़ी फ़ाइल टाइमआउट हो जाए, तो दोबारा चलाने से पहले लक्षित संपादन करें या पृष्ठ को विभाजित करें।
- अनुवाद के बाद जाँच करें: कोड स्पैन/ब्लॉक अपरिवर्तित हों, लिंक/एंकर अपरिवर्तित हों, प्लेसहोल्डर सुरक्षित हों।

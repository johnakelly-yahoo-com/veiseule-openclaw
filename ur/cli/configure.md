---
summary: "CLI کے لیے حوالہ: `openclaw configure` (انٹرایکٹو کنفیگریشن پرامپٹس)"
read_when:
  - آپ اس وقت جب اسناد، ڈیوائسز، یا ایجنٹ کی ڈیفالٹس کو انٹرایکٹو طور پر ایڈجسٹ کرنا چاہتے ہوں
title: "کنفیگر"
---

# `openclaw configure`

اسناد، ڈیوائسز، اور ایجنٹ کی ڈیفالٹس سیٹ کرنے کے لیے انٹرایکٹو پرامپٹ۔

نوٹ: **Model** سیکشن میں اب `agents.defaults.models` اجازت فہرست کے لیے ملٹی-سلیکٹ شامل ہے (جو `/model` اور ماڈل پکر میں ظاہر ہوتا ہے)۔

مشورہ: `openclaw config` کو بغیر کسی ذیلی کمانڈ کے چلانے سے وہی وزرڈ کھلتا ہے۔ استعمال کریں
`openclaw config get|set|unset` for non-interactive edits.

متعلقہ:

- Gateway کنفیگریشن حوالہ: [Configuration](/gateway/configuration)
- کنفیگ CLI: [Config](/cli/config)

نوٹس:

- Gateway کہاں چلتا ہے اس کا انتخاب کرنے سے ہمیشہ `gateway.mode` اپ ڈیٹ ہوتا ہے۔ اگر آپ کو صرف یہی درکار ہے تو آپ دیگر حصوں کے بغیر "Continue" منتخب کر سکتے ہیں۔
- چینل پر مبنی سروسز (Slack/Discord/Matrix/Microsoft Teams) سیٹ اپ کے دوران چینل/روم الاؤ لسٹس کے لیے اشارہ کرتی ہیں۔ آپ نام یا IDs درج کر سکتے ہیں؛ وزرڈ جہاں ممکن ہو ناموں کو IDs میں تبدیل کر دیتا ہے۔

## مثالیں

```bash
openclaw configure
openclaw configure --section models --section channels
```

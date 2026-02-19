---
summary: "Together AI سیٹ اپ (auth + ماڈل کا انتخاب)"
read_when:
  - آپ OpenClaw کے ساتھ Together AI استعمال کرنا چاہتے ہیں
  - آپ کو API کلید کے env ویری ایبل یا CLI auth انتخاب کی ضرورت ہے
---

# Together AI

[Together AI](https://together.ai) ایک متحدہ API کے ذریعے معروف اوپن سورس ماڈلز بشمول Llama، DeepSeek، Kimi اور دیگر تک رسائی فراہم کرتا ہے۔

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-مطابقت رکھنے والا

## کوئیک اسٹارٹ

1. API کلید سیٹ کریں (تجویز کردہ: اسے Gateway کے لیے محفوظ کریں):

```bash
openclaw onboard --auth-choice together-api-key
```

2. ایک ڈیفالٹ ماڈل سیٹ کریں:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## نان-انٹرایکٹو مثال

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

یہ `together/moonshotai/Kimi-K2.5` کو بطور ڈیفالٹ ماڈل سیٹ کر دے گا۔

## ماحولیاتی نوٹ

اگر Gateway بطور ڈیمَن (launchd/systemd) چل رہا ہو تو یقینی بنائیں کہ `TOGETHER_API_KEY`
اس پراسیس کے لیے دستیاب ہو (مثال کے طور پر، `~/.clawdbot/.env` میں یا
`env.shellEnv` کے ذریعے)۔

## دستیاب ماڈلز

Together AI بہت سے مقبول اوپن سورس ماڈلز تک رسائی فراہم کرتا ہے:

- **GLM 4.7 Fp8** - 200K کانٹیکسٹ ونڈو کے ساتھ ڈیفالٹ ماڈل
- **Llama 3.3 70B Instruct Turbo** - تیز اور مؤثر ہدایات پر عمل کرنے والا ماڈل
- **Llama 4 Scout** - امیج سمجھنے کی صلاحیت کے ساتھ وژن ماڈل
- **Llama 4 Maverick** - جدید وژن اور ریزننگ کی صلاحیتیں
- **DeepSeek V3.1** - طاقتور کوڈنگ اور ریزننگ ماڈل
- **DeepSeek R1** - جدید ریزننگ ماڈل
- **Kimi K2 Instruct** - 262K کانٹیکسٹ ونڈو کے ساتھ ہائی پرفارمنس ماڈل

تمام ماڈلز معیاری چیٹ کمپلیشنز کو سپورٹ کرتے ہیں اور OpenAI API کے ساتھ مطابقت رکھتے ہیں۔

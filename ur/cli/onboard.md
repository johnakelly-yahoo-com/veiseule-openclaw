---
summary: "CLI کے لیے `openclaw onboard` کا حوالہ (انٹرایکٹو آن بورڈنگ وزارڈ)"
read_when:
  - آپ گیٹ وے، ورک اسپیس، تصدیق، چینلز اور Skills کے لیے رہنمائی شدہ سیٹ اپ چاہتے ہیں
title: "آن بورڈ"
---

# `openclaw onboard`

انٹرایکٹو آن بورڈنگ وزارڈ (لوکل یا ریموٹ Gateway سیٹ اپ)۔

## متعلقہ رہنما

- CLI آن بورڈنگ ہب: [Onboarding Wizard (CLI)](/start/wizard)
- CLI آن بورڈنگ حوالہ: [CLI Onboarding Reference](/start/wizard-cli-reference)
- CLI آٹومیشن: [CLI Automation](/start/wizard-cli-automation)
- macOS آن بورڈنگ: [Onboarding (macOS App)](/start/onboarding)
- macOS آن بورڈنگ: [Onboarding (macOS App)](/start/onboarding)

## مثالیں

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

فلو نوٹس:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` غیر تعاملی موڈ میں اختیاری ہے۔ اگر فراہم نہ کیا جائے تو آن بورڈنگ `CUSTOM_API_KEY` کو چیک کرتی ہے۔

غیر تعاملی Z.AI اینڈپوائنٹ کے اختیارات:

نوٹ: `--auth-choice zai-api-key` اب آپ کی key کے لیے بہترین Z.AI اینڈپوائنٹ خودکار طور پر منتخب کرتا ہے (عمومی API کو `zai/glm-5` کے ساتھ ترجیح دیتا ہے)۔
اگر آپ خاص طور پر GLM Coding Plan اینڈپوائنٹس چاہتے ہیں تو `zai-coding-global` یا `zai-coding-cn` منتخب کریں۔

```bash
# بغیر پرامپٹ کے اینڈپوائنٹ کا انتخاب
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# دیگر Z.AI اینڈپوائنٹ کے اختیارات:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

فلو نوٹس:

- `quickstart`: کم سے کم پرامپٹس، خودکار طور پر گیٹ وے ٹوکن تیار کرتا ہے۔
- `manual`: پورٹ/بائنڈ/تصدیق کے لیے مکمل پرامپٹس ( `advanced` کا عرف)۔
- سب سے تیز پہلی چیٹ: `openclaw dashboard` (کنٹرول UI، کوئی چینل سیٹ اپ نہیں)۔
- Custom Provider: کسی بھی OpenAI یا Anthropic مطابقت رکھنے والے اینڈپوائنٹ سے منسلک کریں،
  بشمول وہ ہوسٹڈ پرووائیڈرز جو فہرست میں شامل نہیں ہیں۔ خودکار شناخت کے لیے Unknown استعمال کریں۔

## عام فالو اپ کمانڈز

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` نان اِنٹریکٹو موڈ کا مطلب نہیں ہوتا۔ 8. اسکرپٹس کے لیے `--non-interactive` استعمال کریں۔
</Note>

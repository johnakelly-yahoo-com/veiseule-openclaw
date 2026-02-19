---
read_when:
  - آن بورڈنگ وزرڈ چلانے یا ترتیب دینے کے وقت
  - نئی مشین کا سیٹ اپ کرتے وقت
sidebarTitle: Wizard (CLI)
summary: "CLI آن بورڈنگ وزرڈ: Gateway، ورک اسپیس، چینلز اور Skills کی انٹرایکٹو سیٹ اپ"
title: آن بورڈنگ وزرڈ (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# آن بورڈنگ وزرڈ (CLI)

CLI آن بورڈنگ وزرڈ macOS، Linux، اور Windows (WSL2 کے ذریعے) پر OpenClaw سیٹ اپ کرنے کے لیے تجویز کردہ طریقہ ہے۔ یہ مقامی Gateway یا ریموٹ Gateway کنکشن کے ساتھ ساتھ ورک اسپیس کی ڈیفالٹ سیٹنگز، چینلز اور Skills کو بھی کنفیگر کرتا ہے۔

```bash
openclaw onboard
```

<Info>
پہلی چیٹ تیزی سے شروع کرنے کا آسان ترین طریقہ: Control UI کھولیں (چینل سیٹ اپ درکار نہیں)۔ `openclaw dashboard` چلائیں اور براؤزر میں چیٹ کریں۔ دستاویزات: [Dashboard](/web/dashboard).
</Info>

## کوئیک اسٹارٹ بمقابلہ تفصیلی کنفیگریشن

وزرڈ آغاز میں آپ کو **کوئیک اسٹارٹ** (ڈیفالٹ سیٹنگز) یا **تفصیلی کنفیگریشن** (مکمل کنٹرول) میں سے ایک منتخب کرنے دیتا ہے۔

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - loopback پر مقامی Gateway
    - موجودہ ورک اسپیس یا ڈیفالٹ ورک اسپیس
    - Gateway پورٹ `18789`
    - Gateway تصدیقی ٹوکن خودکار طور پر تیار ہوتا ہے (loopback پر بھی تیار کیا جاتا ہے)
    - Tailscale پبلک ایکسپوژر بند
    - Telegram اور WhatsApp کے DM ڈیفالٹ طور پر اجازت فہرست میں شامل (آپ سے فون نمبر درج کرنے کو کہا جا سکتا ہے)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - موڈ، ورک اسپیس، Gateway، چینلز، ڈیمن، اور Skills کے لیے مکمل پرامپٹ فلو دکھائیں
  
</Tab>
</Tabs>

## CLI آن بورڈنگ کی تفصیلات

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    مقامی اور ریموٹ فلو کی مکمل وضاحت، تصدیق اور ماڈل میٹرکس، کنفیگریشن آؤٹ پٹ، وزرڈ RPC، اور signal-cli کا برتاؤ۔
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    غیر انٹرایکٹو آن بورڈنگ کے طریقے اور خودکار `agents add` کی مثالیں۔
  
</Card>
</Columns>

## اکثر استعمال ہونے والے فالو اپ کمانڈز

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` کا مطلب غیر انٹرایکٹو موڈ نہیں ہے۔ اسکرپٹس میں `--non-interactive` استعمال کریں۔
</Note>

<Tip>
تجویز: ایجنٹ کو `web_search` استعمال کرنے کے قابل بنانے کے لیے Brave Search API کلید سیٹ کریں (`web_fetch` کلید کے بغیر بھی کام کرتا ہے)۔ سب سے آسان طریقہ: `openclaw configure --section web` چلائیں، اس سے `tools.web.search.apiKey` محفوظ ہو جائے گا۔ دستاویزات: [Web ٹولز](/tools/web).
</Tip>

## متعلقہ دستاویزات

- CLI کمانڈ ریفرنس: [`openclaw onboard`](/cli/onboard)
- macOS ایپ کی آن بورڈنگ: [آن بورڈنگ](/start/onboarding)
- ایجنٹ کے پہلے آغاز کے مراحل: [ایجنٹ بوٹ اسٹرپنگ](/start/bootstrapping)

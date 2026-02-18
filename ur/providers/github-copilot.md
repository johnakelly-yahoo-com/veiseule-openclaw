---
title: "GitHub Copilot"
---

# GitHub Copilot

## GitHub Copilot کیا ہے؟

GitHub Copilot، GitHub کا AI کوڈنگ اسسٹنٹ ہے۔ It provides access to Copilot
models for your GitHub account and plan. OpenClaw can use Copilot as a model
provider in two different ways.

## OpenClaw میں Copilot استعمال کرنے کے دو طریقے

### 1. بلٹ اِن GitHub Copilot فراہم کنندہ (`github-copilot`)

GitHub ٹوکن حاصل کرنے کے لیے مقامی ڈیوائس لاگ اِن فلو استعمال کریں، پھر اسے ایکسچینج کریں
Copilot API tokens when OpenClaw runs. This is the **default** and simplest path
because it does not require VS Code.

### 2. Copilot Proxy پلگ اِن (`copilot-proxy`)

مقامی برج کے طور پر **Copilot Proxy** VS Code ایکسٹینشن استعمال کریں۔ OpenClaw اس سے بات کرتا ہے
the proxy’s `/v1` endpoint and uses the model list you configure there. Choose
this when you already run Copilot Proxy in VS Code or need to route through it.
You must enable the plugin and keep the VS Code extension running.

GitHub Copilot کو بطور ماڈل فراہم کنندہ (`github-copilot`) استعمال کریں۔ لاگ اِن کمانڈ چلتی ہے
the GitHub device flow, saves an auth profile, and updates your config to use that
profile.

## CLI سیٹ اپ

```bash
openclaw models auth login-github-copilot
```

آپ سے ایک URL پر جانے اور ایک وقتی کوڈ درج کرنے کے لیے کہا جائے گا۔ ٹرمینل کو کھلا رکھیں
open until it completes.

### اختیاری فلیگز

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## بطورِ طے شدہ ماڈل سیٹ کریں

```bash
openclaw models set github-copilot/gpt-4o
```

### کنفیگ ٹکڑا

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## نوٹس

- انٹرایکٹو TTY درکار ہے؛ اسے براہِ راست ٹرمینل میں چلائیں۔
- Copilot ماڈلز کی دستیابی آپ کے پلان پر منحصر ہے؛ اگر کوئی ماڈل مسترد ہو جائے تو
  کسی دوسرے ID کو آزمائیں (مثال کے طور پر `github-copilot/gpt-4.1`)۔
- لاگ اِن، auth پروفائل اسٹور میں GitHub ٹوکن محفوظ کرتا ہے اور جب OpenClaw چلتا ہے تو اسے
  Copilot API ٹوکن کے ساتھ ایکسچینج کرتا ہے۔

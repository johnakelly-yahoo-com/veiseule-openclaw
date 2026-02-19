---
summary: "CLI کے لیے `openclaw plugins` کا حوالہ (list، install، enable/disable، doctor)"
read_when:
  - آپ in-process Gateway پلگ اِنز انسٹال یا منظم کرنا چاہتے ہیں
  - آپ پلگ اِن لوڈ کی ناکامیوں کی خرابیوں کا ازالہ کرنا چاہتے ہیں
title: "plugins"
---

# `openclaw plugins`

Gateway (گیٹ وے) پلگ اِنز/ایکسٹینشنز کا انتظام کریں (جو in-process لوڈ ہوتے ہیں)۔

متعلقہ:

- پلگ اِن سسٹم: [Plugins](/tools/plugin)
- پلگ اِن منشور + اسکیما: [Plugin manifest](/plugins/manifest)
- سکیورٹی مضبوطی: [Security](/gateway/security)

## کمانڈز

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

بنڈل شدہ پلگ اِنز OpenClaw کے ساتھ آتے ہیں لیکن ابتدا میں غیر فعال ہوتے ہیں۔ انہیں فعال کرنے کے لیے `plugins enable` استعمال کریں۔

تمام پلگ اِنز کو ایک `openclaw.plugin.json` فائل فراہم کرنا لازم ہے جس میں اِن لائن JSON Schema ہو (`configSchema`، چاہے خالی ہی کیوں نہ ہو)۔ غائب/غلط مینی فیسٹس یا اسکیما پلگ اِن کے لوڈ ہونے سے روکتے ہیں اور کنفیگ ویلیڈیشن ناکام ہو جاتی ہے۔

### تنصیب

```bash
openclaw plugins install <path-or-spec>
```

سیکیورٹی نوٹ: پلگ اِن انسٹالز کو کوڈ چلانے کے مترادف سمجھیں۔ پن کی گئی ورژنز کو ترجیح دیں۔

Npm specs صرف **registry-only** ہیں (پیکیج کا نام + اختیاری ورژن/ٹیگ)۔ Git/URL/file
specs مسترد کر دی جاتی ہیں۔ Dependency installs حفاظتی مقصد کے لیے `--ignore-scripts` کے ساتھ چلائے جاتے ہیں۔

مقامی ڈائریکٹری کی کاپی سے بچنے کے لیے `--link` استعمال کریں (یہ `plugins.load.paths` میں شامل کرتا ہے):

مقامی ڈائریکٹری کی کاپی سے بچنے کے لیے `--link` استعمال کریں (یہ `plugins.load.paths` میں شامل کرتا ہے):

```bash
openclaw plugins install -l ./my-plugin
```

### Uninstall

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` قابل اطلاق ہونے پر `plugins.entries`، `plugins.installs`،
plugin allowlist، اور منسلک `plugins.load.paths` اندراجات سے plugin ریکارڈز کو ہٹا دیتا ہے۔
فعال memory plugins کے لیے، memory slot دوبارہ `memory-core` پر سیٹ ہو جاتا ہے۔

بطور ڈیفالٹ، uninstall فعال state dir extensions روٹ (`$OPENCLAW_STATE_DIR/extensions/<id>`) کے تحت plugin کی انسٹال ڈائریکٹری کو بھی ہٹا دیتا ہے۔ ڈسک پر فائلیں برقرار رکھنے کے لیے
`--keep-files` استعمال کریں۔

`--keep-config` کو `--keep-files` کے ایک فرسودہ متبادل کے طور پر سپورٹ کیا جاتا ہے۔

### Update

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

اپ ڈیٹس صرف اُن پلگ اِنز پر لاگو ہوتی ہیں جو npm سے انسٹال کیے گئے ہوں (جو `plugins.installs` میں ٹریک ہوتے ہیں)۔

---
title: "لاگنگ"
---

# لاگنگ

صارف کے لیے جامع جائزہ (CLI + کنٹرول UI + کنفیگ) کے لیے [/logging](/logging) دیکھیں۔

OpenClaw میں لاگز کی دو “سطحیں” ہیں:

- **کنسول آؤٹ پٹ** (جو آپ ٹرمینل / Debug UI میں دیکھتے ہیں)۔
- **فائل لاگز** (JSON لائنیں) جو gateway logger کے ذریعے لکھی جاتی ہیں۔

## فائل پر مبنی logger

- بطورِ طے شدہ رولنگ لاگ فائل `/tmp/openclaw/` کے تحت ہوتی ہے (ہر دن کے لیے ایک فائل): `openclaw-YYYY-MM-DD.log`
  - تاریخ gateway ہوسٹ کے مقامی ٹائم زون کے مطابق ہوتی ہے۔
- لاگ فائل کا راستہ اور لیول `~/.openclaw/openclaw.json` کے ذریعے کنفیگر کیے جا سکتے ہیں:
  - `logging.file`
  - `logging.level`

فائل کا فارمیٹ ہر لائن پر ایک JSON آبجیکٹ ہوتا ہے۔

30. کنٹرول UI کے Logs ٹیب میں گیٹ وے کے ذریعے اس فائل کو ٹیل کیا جاتا ہے (`logs.tail`)۔
31. CLI بھی یہی کر سکتا ہے:

```bash
openclaw logs --follow
```

**Verbose بمقابلہ لاگ لیولز**

- **فائل لاگز** مکمل طور پر `logging.level` کے ذریعے کنٹرول ہوتے ہیں۔
- `--verbose` صرف **کنسول verbosity** (اور WS لاگ اسٹائل) کو متاثر کرتا ہے؛ یہ فائل لاگ لیول کو **بڑھاتا نہیں**۔
- verbose-only تفصیلات کو فائل لاگز میں محفوظ کرنے کے لیے، `logging.level` کو `debug` یا
  `trace` پر سیٹ کریں۔

## کنسول کیپچر

CLI، `console.log/info/warn/error/debug/trace` کو کیپچر کرتا ہے اور انہیں فائل لاگز میں لکھتا ہے،
جبکہ stdout/stderr پر پرنٹ بھی جاری رکھتا ہے۔

آپ کنسول verbosity کو الگ سے درج ذیل کے ذریعے ایڈجسٹ کر سکتے ہیں:

- `logging.consoleLevel` (بطورِ طے شدہ `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)

## ٹول سمری ریڈیکشن

32. تفصیلی ٹول سمریز (مثلاً `🛠️ Exec: ...`) کنسول اسٹریم تک پہنچنے سے پہلے حساس ٹوکنز کو ماسک کر سکتی ہیں۔ 33. یہ **صرف ٹولز** کے لیے ہے اور فائل لاگز میں تبدیلی نہیں کرتا۔

- `logging.redactSensitive`: `off` | `tools` (بطورِ طے شدہ: `tools`)
- `logging.redactPatterns`: regex اسٹرنگز کی array (ڈیفالٹس کو اووررائیڈ کرتی ہے)
  - خام regex اسٹرنگز استعمال کریں (خودکار `gi`)، یا اگر کسٹم flags درکار ہوں تو `/pattern/flags`۔
  - میچز کو پہلے 6 + آخری 4 حروف برقرار رکھ کر ماسک کیا جاتا ہے (لمبائی >= 18)، ورنہ `***`۔
  - ڈیفالٹس عام key assignments، CLI flags، JSON فیلڈز، bearer headers، PEM blocks، اور مقبول ٹوکن prefixes کا احاطہ کرتے ہیں۔

## Gateway WebSocket لاگز

gateway، WebSocket پروٹوکول لاگز کو دو موڈز میں پرنٹ کرتا ہے:

- **نارمل موڈ (بغیر `--verbose`)**: صرف “دلچسپ” RPC نتائج پرنٹ ہوتے ہیں:
  - غلطیاں (`ok=false`)
  - سست کالز (بطورِ طے شدہ حد: `>= 50ms`)
  - پارس کی غلطیاں
- **Verbose موڈ (`--verbose`)**: تمام WS درخواست/جواب ٹریفک پرنٹ کرتا ہے۔

### WS لاگ اسٹائل

`openclaw gateway` ہر gateway کے لیے اسٹائل سوئچ کی سہولت دیتا ہے:

- `--ws-log auto` (بطورِ طے شدہ): نارمل موڈ آپٹمائزڈ؛ verbose موڈ میں کمپیکٹ آؤٹ پٹ
- `--ws-log compact`: verbose میں کمپیکٹ آؤٹ پٹ (جوڑی شدہ درخواست/جواب)
- `--ws-log full`: verbose میں مکمل فی-فریم آؤٹ پٹ
- `--compact`: `--ws-log compact` کے لیے عرف

مثالیں:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# show all WS traffic (full meta)
openclaw gateway --verbose --ws-log full
```

## کنسول فارمیٹنگ (سب سسٹم لاگنگ)

34. کنسول فارمیٹر **TTY-aware** ہے اور یکساں، پری فکسڈ لائنیں پرنٹ کرتا ہے۔
35. سب سسٹم لاگرز آؤٹ پٹ کو گروپ میں اور اسکین کے قابل رکھتے ہیں۔

رویّہ:

- ہر لائن پر **سب سسٹم prefixes** (مثلاً `[gateway]`, `[canvas]`, `[tailscale]`)
- **سب سسٹم رنگ** (ہر سب سسٹم کے لیے مستقل) اور لیول کلرنگ
- **جب آؤٹ پٹ TTY ہو یا ماحول ایک بھرپور ٹرمینل جیسا ہو تو رنگ** (`TERM`/`COLORTERM`/`TERM_PROGRAM`)، `NO_COLOR` کا احترام
- **مختصر کیے گئے سب سسٹم prefixes**: ابتدائی `gateway/` + `channels/` کو ہٹا کر آخری 2 حصے رکھتا ہے (مثلاً `whatsapp/outbound`)
- **سب سسٹم کے لحاظ سے سب-لاگرز** (خودکار prefix + ساختی فیلڈ `{ subsystem }`)
- **QR/UX آؤٹ پٹ کے لیے `logRaw()`** (نہ prefix، نہ فارمیٹنگ)
- **کنسول اسٹائلز** (مثلاً `pretty | compact | json`)
- **کنسول لاگ لیول** فائل لاگ لیول سے الگ (جب `logging.level` کو `debug`/`trace` پر سیٹ کیا جائے تو فائل مکمل تفصیل رکھتی ہے)
- **WhatsApp پیغام کے متن** `debug` پر لاگ ہوتے ہیں (انہیں دیکھنے کے لیے `--verbose` استعمال کریں)

یہ طریقہ موجودہ فائل لاگز کو مستحکم رکھتا ہے جبکہ انٹرایکٹو آؤٹ پٹ کو پڑھنے میں آسان بناتا ہے۔

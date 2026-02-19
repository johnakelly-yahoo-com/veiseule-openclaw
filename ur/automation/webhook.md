---
summary: "جاگنے اور علیحدہ ایجنٹ رنز کے لیے Webhook اِن گریس"
read_when:
  - Webhook اینڈپوائنٹس شامل یا تبدیل کرتے وقت
  - بیرونی نظاموں کو OpenClaw سے جوڑتے وقت
title: "Webhooks"
---

# Webhooks

Gateway بیرونی ٹرگرز کے لیے ایک چھوٹا HTTP webhook اینڈپوائنٹ فراہم کر سکتا ہے۔

## فعال کریں

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

نوٹس:

- `hooks.token` درکار ہے جب `hooks.enabled=true`۔
- `hooks.path` بطورِ طے شدہ `/hooks` ہوتا ہے۔

## Auth

40. ہر درخواست میں hook ٹوکن شامل ہونا ضروری ہے۔ 41. headers کو ترجیح دیں:

- `Authorization: Bearer <token>` (سفارش کردہ)
- `x-openclaw-token: <token>`
- `?token=<token>` (متروک؛ وارننگ لاگ کرتا ہے اور مستقبل کی کسی بڑی ریلیز میں ہٹا دیا جائے گا)

## Endpoints

### `POST /hooks/wake`

پے لوڈ:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **required** (string): ایونٹ کی وضاحت (مثلاً، "نیا ای میل موصول ہوا")۔
- `mode` optional (`now` | `next-heartbeat`): فوری ہارٹ بیٹ ٹرگر کرنا ہے یا اگلے دورانی چیک کا انتظار کرنا ہے (بطورِ طے شدہ `now`)۔

Effect:

- **main** سیشن کے لیے ایک سسٹم ایونٹ قطار میں شامل کرتا ہے
- اگر `mode=now` ہو تو فوری ہارٹ بیٹ ٹرگر کرتا ہے

### `POST /hooks/agent`

پے لوڈ:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **required** (string): ایجنٹ کے لیے پروسیس کرنے کا پرامپٹ یا پیغام۔
- `name` optional (string): hook کے لیے انسانی قابلِ فہم نام (مثلاً، "GitHub")، جو سیشن خلاصوں میں بطور سابقہ استعمال ہوتا ہے۔
- `agentId` اختیاری (string): اس ہک کو کسی مخصوص agent کی طرف روٹ کریں۔ نامعلوم IDs ڈیفالٹ ایجنٹ پر واپس چلے جاتے ہیں۔ سیٹ ہونے پر، ہُک ریزولو شدہ ایجنٹ کے ورک اسپیس اور کنفیگریشن کو استعمال کرتے ہوئے چلتا ہے۔
- 42. `sessionKey` اختیاری (string): ایجنٹ کے سیشن کی شناخت کے لیے استعمال ہونے والی کلید۔ ڈیفالٹ طور پر یہ فیلڈ مسترد کر دی جاتی ہے جب تک کہ `hooks.allowRequestSessionKey=true` نہ ہو۔
- `wakeMode` optional (`now` | `next-heartbeat`): فوری ہارٹ بیٹ ٹرگر کرنا ہے یا اگلے دورانی چیک کا انتظار کرنا ہے (بطورِ طے شدہ `now`)۔
- 45. `deliver` اختیاری (boolean): اگر `true` ہو تو ایجنٹ کا جواب میسجنگ چینل پر بھیج دیا جائے گا۔ 46. ڈیفالٹ `true` ہے۔ 47. وہ جوابات جو صرف heartbeat acknowledgments ہوں، خودکار طور پر چھوڑ دیے جاتے ہیں۔
- 48. `channel` اختیاری (string): ترسیل کے لیے میسجنگ چینل۔ 49. ان میں سے ایک: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`۔ 50. ڈیفالٹ `last` ہے۔
- `to` optional (string): The recipient identifier for the channel (e.g., phone number for WhatsApp/Signal, chat ID for Telegram, channel ID for Discord/Slack/Mattermost (plugin), conversation ID for MS Teams). Defaults to the last recipient in the main session.
- `model` optional (string): Model override (e.g., `anthropic/claude-3-5-sonnet` or an alias). Must be in the allowed model list if restricted.
- `timeoutSeconds` optional (number): ایجنٹ رن کے لیے زیادہ سے زیادہ دورانیہ سیکنڈز میں۔
- `timeoutSeconds` optional (number): ایجنٹ رن کے لیے زیادہ سے زیادہ دورانیہ سیکنڈز میں۔

Effect:

- ایک **isolated** ایجنٹ ٹرن چلاتا ہے (اپنی سیشن کلید کے ساتھ)
- ہمیشہ **main** سیشن میں ایک خلاصہ پوسٹ کرتا ہے
- اگر `wakeMode=now` ہو تو فوری ہارٹ بیٹ ٹرگر کرتا ہے

## سیشن کی پالیسی (بریکنگ تبدیلی)

`/hooks/agent` پے لوڈ میں `sessionKey` اووررائیڈز ڈیفالٹ طور پر غیر فعال ہیں۔

- تجویز کردہ: ایک مستقل `hooks.defaultSessionKey` سیٹ کریں اور ریکویسٹ اووررائیڈز کو بند رکھیں۔
- اختیاری: ریکویسٹ اووررائیڈز صرف ضرورت کے وقت اجازت دیں، اور پری فکسز کو محدود کریں۔

تجویز کردہ کنفیگریشن:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

مطابقتی کنفیگریشن (لیگیسی رویہ):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // بھرپور طور پر تجویز کردہ
  },
}
```

### `POST /hooks/<name>` (mapped)

Custom hook names are resolved via `hooks.mappings` (see configuration). A mapping can
turn arbitrary payloads into `wake` or `agent` actions, with optional templates or
code transforms.

Mapping options (summary):

- `hooks.presets: ["gmail"]` بلٹ اِن Gmail میپنگ کو فعال کرتا ہے۔
- `hooks.mappings` آپ کو کنفیگ میں `match`, `action`، اور ٹیمپلیٹس متعین کرنے دیتا ہے۔
- `hooks.transformsDir` + `transform.module` حسبِ ضرورت لاجک کے لیے JS/TS ماڈیول لوڈ کرتا ہے۔
  - `hooks.transformsDir` (اگر سیٹ کیا گیا ہو) آپ کی OpenClaw کنفیگ ڈائریکٹری کے اندر transforms روٹ کے اندر ہی رہنا چاہیے (عام طور پر `~/.openclaw/hooks/transforms`)۔
  - `transform.module` کو مؤثر transforms ڈائریکٹری کے اندر ہی ریزولو ہونا چاہیے (traversal/escape راستے مسترد کر دیے جاتے ہیں)۔
- `match.source` استعمال کریں تاکہ ایک عام ingest اینڈپوائنٹ برقرار رہے (payload پر مبنی راؤٹنگ)۔
- TS ٹرانسفارمز کے لیے TS لوڈر درکار ہوتا ہے (مثلاً `bun` یا `tsx`) یا رَن ٹائم پر پہلے سے کمپائل شدہ `.js`۔
- جوابات کو چیٹ سطح پر راؤٹ کرنے کے لیے میپنگز پر `deliver: true` + `channel`/`to` سیٹ کریں
  (`channel` بطورِ طے شدہ `last` ہے اور WhatsApp پر فال بیک کرتا ہے)۔
- `agentId` ہُک کو مخصوص ایجنٹ کی طرف روٹ کرتا ہے؛ نامعلوم IDs ڈیفالٹ ایجنٹ پر واپس چلے جاتے ہیں۔
- `hooks.allowedAgentIds` واضح `agentId` روٹنگ کو محدود کرتا ہے۔ اسے چھوڑ دیں (یا `*` شامل کریں) تاکہ کسی بھی ایجنٹ کی اجازت ہو۔ واضح `agentId` روٹنگ کو مسترد کرنے کے لیے `[]` سیٹ کریں۔
- `hooks.defaultSessionKey` ہُک ایجنٹ رنز کے لیے ڈیفالٹ سیشن سیٹ کرتا ہے جب کوئی واضح key فراہم نہ کی گئی ہو۔
- `hooks.allowRequestSessionKey` یہ کنٹرول کرتا ہے کہ آیا `/hooks/agent` پے لوڈز `sessionKey` سیٹ کر سکتے ہیں (ڈیفالٹ: `false`)۔
- `hooks.allowedSessionKeyPrefixes` اختیاری طور پر ریکویسٹ پے لوڈز اور میپنگز سے واضح `sessionKey` ویلیوز کو محدود کرتا ہے۔
- `allowUnsafeExternalContent: true` اس hook کے لیے بیرونی مواد کی حفاظت کے ریپر کو غیر فعال کرتا ہے
  (خطرناک؛ صرف قابلِ اعتماد اندرونی ذرائع کے لیے)۔
- `openclaw webhooks gmail setup` writes `hooks.gmail` config for `openclaw webhooks gmail run`.
  See [Gmail Pub/Sub](/automation/gmail-pubsub) for the full Gmail watch flow.

## Responses

- `200` برائے `/hooks/wake`
- `202` برائے `/hooks/agent` (async رن شروع ہو گیا)
- تصدیق کی ناکامی پر `401`
- اسی کلائنٹ سے بار بار auth ناکامیوں کے بعد `429` ( `Retry-After` چیک کریں )
- غلط payload پر `400`
- حد سے بڑے payloads پر `413`

## Examples

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Use a different model

اس رن کے لیے ماڈل اووررائیڈ کرنے کو ایجنٹ payload (یا میپنگ) میں `model` شامل کریں:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

اگر آپ `agents.defaults.models` نافذ کرتے ہیں تو یقینی بنائیں کہ اووررائیڈ ماڈل اس میں شامل ہو۔

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Security

- hook اینڈپوائنٹس کو loopback، tailnet، یا قابلِ اعتماد ریورس پراکسی کے پیچھے رکھیں۔
- ایک مخصوص hook ٹوکن استعمال کریں؛ gateway کی auth ٹوکنز دوبارہ استعمال نہ کریں۔
- brute-force کوششوں کو سست کرنے کے لیے بار بار auth ناکامیوں پر فی کلائنٹ ایڈریس ریٹ لمٹ لاگو کی جاتی ہے۔
- اگر آپ multi-agent روٹنگ استعمال کرتے ہیں تو واضح `agentId` کے انتخاب کو محدود کرنے کے لیے `hooks.allowedAgentIds` سیٹ کریں۔
- جب تک آپ کو کالر کے منتخب کردہ سیشنز درکار نہ ہوں `hooks.allowRequestSessionKey=false` رکھیں۔
- اگر آپ ریکویسٹ `sessionKey` فعال کرتے ہیں تو `hooks.allowedSessionKeyPrefixes` کو محدود کریں (مثال کے طور پر، `["hook:"]`)۔
- webhook لاگز میں حساس خام payloads شامل کرنے سے گریز کریں۔
- Hook payloads are treated as untrusted and wrapped with safety boundaries by default.
  If you must disable this for a specific hook, set `allowUnsafeExternalContent: true`
  in that hook's mapping (dangerous).


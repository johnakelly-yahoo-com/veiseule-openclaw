---
summary: "signal-cli (JSON-RPC + SSE) کے ذریعے Signal سپورٹ، سیٹ اپ، اور نمبر ماڈل"
read_when:
  - Signal سپورٹ سیٹ اپ کرنا
  - Signal بھیجنے/موصول کرنے کی خرابیوں کی جانچ
title: "Signal"
---

# Signal (signal-cli)

Status: external CLI integration. Gateway talks to `signal-cli` over HTTP JSON-RPC + SSE.

## فوری سیٹ اپ (مبتدی)

- بوٹ کے لیے **علیحدہ Signal نمبر** استعمال کریں (سفارش کردہ)۔
- `signal-cli` انسٹال کریں (Java درکار ہے)۔
- بوٹ ڈیوائس کو لنک کریں اور ڈیمَن شروع کریں:
- OpenClaw کنفیگر کریں اور گیٹ وے شروع کریں۔

## فوری سیٹ اپ (مبتدی)

1. بوٹ کے لیے **علیحدہ Signal نمبر** استعمال کریں (سفارش کردہ)۔
2. `signal-cli` انسٹال کریں (اگر آپ JVM build استعمال کرتے ہیں تو Java درکار ہے)۔
3. ایک setup راستہ منتخب کریں:
   - **Path A (QR link):** `signal-cli link -n "OpenClaw"` چلائیں اور Signal سے scan کریں۔
   - **Path B (SMS register):** captcha + SMS verification کے ساتھ ایک مخصوص نمبر رجسٹر کریں۔
4. OpenClaw کو configure کریں اور gateway کو دوبارہ شروع کریں۔
5. پہلا DM بھیجیں اور pairing منظور کریں (`openclaw pairing approve signal <CODE>`)۔

کم از کم کنفیگ:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

فیلڈ حوالہ:

| فیلڈ        | وضاحت                                                                                |
| ----------- | ------------------------------------------------------------------------------------ |
| `account`   | E.164 فارمیٹ میں بوٹ کا فون نمبر (`+15551234567`) |
| `cliPath`   | `signal-cli` کا راستہ (`PATH` میں ہو تو `signal-cli`)             |
| `dmPolicy`  | DM رسائی پالیسی (`pairing` تجویز کردہ)                            |
| `allowFrom` | وہ فون نمبرز یا `uuid:&lt;id&gt;` ویلیوز جنہیں DM کی اجازت ہے                              |

## یہ کیا ہے

- `signal-cli` کے ذریعے Signal چینل (ایمبیڈڈ libsignal نہیں)۔
- متعین روٹنگ: جوابات ہمیشہ Signal پر ہی واپس جاتے ہیں۔
- DMs ایجنٹ کے مرکزی سیشن کو شیئر کرتے ہیں؛ گروپس الگ تھلگ ہوتے ہیں (`agent:<agentId>:signal:group:<groupId>`)۔

## نمبر ماڈل (اہم)

بطورِ طے شدہ، Signal کو `/config set|unset` کے ذریعے متحرک ہونے والی کنفیگ اپڈیٹس لکھنے کی اجازت ہے ( `commands.config: true` درکار ہے)۔

اسے بند کرنے کے لیے:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## نمبر ماڈل (اہم)

- گیٹ وے ایک **Signal ڈیوائس** سے جڑتا ہے ( `signal-cli` اکاؤنٹ)۔
- اگر آپ بوٹ کو **اپنے ذاتی Signal اکاؤنٹ** پر چلاتے ہیں تو یہ آپ کے اپنے پیغامات کو نظرانداز کرے گا (لوپ پروٹیکشن)۔
- “میں بوٹ کو میسج کروں اور وہ جواب دے” کے لیے **علیحدہ بوٹ نمبر** استعمال کریں۔

## سیٹ اپ راستہ A: موجودہ Signal اکاؤنٹ لنک کریں (QR)

1. `signal-cli` انسٹال کریں (JVM یا native build).
2. بوٹ اکاؤنٹ لنک کریں:
   - `signal-cli link -n "OpenClaw"` پھر Signal میں QR اسکین کریں۔
3. Signal کنفیگر کریں اور گیٹ وے شروع کریں۔

اگر آپ `signal-cli` کو خود منیج کرنا چاہتے ہیں (سست JVM کولڈ اسٹارٹس، کنٹینر انِٹ، یا مشترکہ CPUs)، تو ڈیمَن الگ سے چلائیں اور OpenClaw کو اس کی طرف پوائنٹ کریں:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Multi-account support: use `channels.signal.accounts` with per-account config and optional `name`. See [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) for the shared pattern.

## رسائی کا کنٹرول (DMs + گروپس)

یہ اس وقت استعمال کریں جب آپ موجودہ Signal ایپ اکاؤنٹ لنک کرنے کے بجائے ایک مخصوص بوٹ نمبر چاہتے ہوں۔

1. بطورِ طے شدہ: `channels.signal.dmPolicy = "pairing"`۔
   - اکاؤنٹ/سیشن تنازعات سے بچنے کے لیے ایک مخصوص بوٹ نمبر استعمال کریں۔
2. نامعلوم ارسال کنندگان کو ایک پیئرنگ کوڈ ملتا ہے؛ منظوری تک پیغامات نظرانداز کیے جاتے ہیں (کوڈز 1 گھنٹے بعد ختم ہو جاتے ہیں)۔

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

اگر آپ JVM build (`signal-cli-${VERSION}.tar.gz`) استعمال کرتے ہیں تو پہلے JRE 25+ انسٹال کریں۔
`signal-cli` کو اپڈیٹ رکھیں؛ upstream کے مطابق پرانے ریلیزز Signal سرور API میں تبدیلیوں کے باعث کام کرنا بند کر سکتے ہیں۔

3. نمبر رجسٹر اور ویریفائی کریں:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

اگر captcha درکار ہو:

1. آؤٹ باؤنڈ متن کو `channels.signal.textChunkLimit` تک حصوں میں توڑا جاتا ہے (ڈیفالٹ 4000)۔
2. اختیاری نئی لائن چنکنگ: خالی لائنوں (پیراگراف کی حدیں) پر تقسیم کے لیے `channels.signal.chunkMode="newline"` سیٹ کریں، پھر لمبائی کے مطابق چنکنگ ہوگی۔
3. اٹیچمنٹس سپورٹڈ ہیں (base64، `signal-cli` سے حاصل شدہ)۔
4. ڈیفالٹ میڈیا حد: `channels.signal.mediaMaxMb` (ڈیفالٹ 8)۔

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. **ٹائپنگ اشارے**: OpenClaw `signal-cli sendTyping` کے ذریعے ٹائپنگ سگنلز بھیجتا ہے اور جواب کے دوران انہیں ریفریش کرتا ہے۔

```bash
# اگر آپ gateway کو user systemd سروس کے طور پر چلا رہے ہیں:
systemctl --user restart openclaw-gateway

# پھر تصدیق کریں:
openclaw doctor
openclaw channels status --probe
```

5. `channel=signal` کے ساتھ `message action=react` استعمال کریں۔
   - بوٹ نمبر پر کوئی بھی پیغام بھیجیں۔
   - سرور پر کوڈ کی منظوری دیں: `openclaw pairing approve signal <PAIRING_CODE>`.
   - "Unknown contact" سے بچنے کے لیے اپنے فون میں بوٹ نمبر کو بطور کانٹیکٹ محفوظ کریں۔

اہم: `signal-cli` کے ساتھ فون نمبر اکاؤنٹ رجسٹر کرنے سے اس نمبر کے لیے مرکزی Signal ایپ سیشن ڈی آتھنٹیکیٹ ہو سکتا ہے۔ ایک مخصوص بوٹ نمبر کو ترجیح دیں، یا اگر آپ اپنی موجودہ فون ایپ سیٹ اپ برقرار رکھنا چاہتے ہیں تو QR لنک موڈ استعمال کریں۔

Upstream حوالہ جات:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Captcha فلو: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Linking فلو: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## بیرونی ڈیمَن موڈ (httpUrl)

اگر آپ `signal-cli` کو خود منیج کرنا چاہتے ہیں (سست JVM کولڈ اسٹارٹس، کنٹینر انِٹ، یا مشترکہ CPUs)، تو ڈیمَن الگ سے چلائیں اور OpenClaw کو اس کی طرف پوائنٹ کریں:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

This skips auto-spawn and the startup wait inside OpenClaw. For slow starts when auto-spawning, set `channels.signal.startupTimeoutMs`.

## رسائی کا کنٹرول (DMs + گروپس)

DMs:

- بطورِ طے شدہ: `channels.signal.dmPolicy = "pairing"`۔
- نامعلوم ارسال کنندگان کو ایک پیئرنگ کوڈ ملتا ہے؛ منظوری تک پیغامات نظرانداز کیے جاتے ہیں (کوڈز 1 گھنٹے بعد ختم ہو جاتے ہیں)۔
- منظوری کے طریقے:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Pairing is the default token exchange for Signal DMs. Details: [Pairing](/channels/pairing)
- صرف UUID والے ارسال کنندگان (`sourceUuid` سے) `channels.signal.allowFrom` میں `uuid:<id>` کے طور پر محفوظ کیے جاتے ہیں۔

گروپس:

- `channels.signal.groupPolicy = open | allowlist | disabled`۔
- `channels.signal.groupAllowFrom` یہ کنٹرول کرتا ہے کہ جب `allowlist` سیٹ ہو تو گروپس میں کون ٹرگر کر سکتا ہے۔

## یہ کیسے کام کرتا ہے (رویّہ)

- `signal-cli` بطور ڈیمَن چلتا ہے؛ گیٹ وے SSE کے ذریعے واقعات پڑھتا ہے۔
- آنے والے پیغامات کو مشترکہ چینل لفافے میں نارملائز کیا جاتا ہے۔
- جوابات ہمیشہ اسی نمبر یا گروپ کی طرف روٹ ہوتے ہیں۔

## کنفیگریشن حوالہ (Signal)

- آؤٹ باؤنڈ متن کو `channels.signal.textChunkLimit` تک حصوں میں توڑا جاتا ہے (ڈیفالٹ 4000)۔
- اختیاری نئی لائن چنکنگ: خالی لائنوں (پیراگراف کی حدیں) پر تقسیم کے لیے `channels.signal.chunkMode="newline"` سیٹ کریں، پھر لمبائی کے مطابق چنکنگ ہوگی۔
- اٹیچمنٹس سپورٹڈ ہیں (base64، `signal-cli` سے حاصل شدہ)۔
- ڈیفالٹ میڈیا حد: `channels.signal.mediaMaxMb` (ڈیفالٹ 8)۔
- میڈیا ڈاؤن لوڈ چھوڑنے کے لیے `channels.signal.ignoreAttachments` استعمال کریں۔
- Group history context uses `channels.signal.historyLimit` (or `channels.signal.accounts.*.historyLimit`), falling back to `messages.groupChat.historyLimit`. Set `0` to disable (default 50).

## ٹائپنگ + ریڈ رسیدیں

- `channels.signal.enabled`: چینل اسٹارٹ اپ فعال/غیرفعال کریں۔
- `channels.signal.account`: بوٹ اکاؤنٹ کے لیے E.164۔
- `channels.signal.cliPath`: `signal-cli` کا راستہ۔

## ری ایکشنز (میسج ٹول)

- `agents.list[].groupChat.mentionPatterns` (Signal مقامی مینشنز سپورٹ نہیں کرتا)۔
- `messages.groupChat.mentionPatterns` (عالمی فال بیک)۔
- `messages.responsePrefix`۔
- گروپ ری ایکشنز کے لیے `targetAuthor` یا `targetAuthorUuid` درکار ہے۔

مثالیں:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

کنفیگ:

- `channels.signal.actions.reactions`: ری ایکشن ایکشنز فعال/غیرفعال کریں (ڈیفالٹ true)۔
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`۔
  - `off`/`ack` ایجنٹ ری ایکشنز کو بند کرتا ہے (میسج ٹول `react` ایرر دے گا)۔
  - `minimal`/`extensive` ایجنٹ ری ایکشنز فعال کرتا ہے اور رہنمائی کی سطح سیٹ کرتا ہے۔
- Per-account overrides: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## ڈیلیوری اہداف (CLI/cron)

- DMs: `signal:+15551234567` (یا سادہ E.164)۔
- UUID DMs: `uuid:<id>` (یا سادہ UUID)۔
- گروپس: `signal:group:<groupId>`۔
- یوزرنیمز: `username:<name>` (اگر آپ کے Signal اکاؤنٹ میں سپورٹ ہو)۔

## خرابیوں کا ازالہ

سب سے پہلے یہ سیڑھی چلائیں:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

پھر ضرورت ہو تو DM پیئرنگ اسٹیٹ کی تصدیق کریں:

```bash
openclaw pairing list signal
```

عام ناکامیاں:

- ڈیمَن قابلِ رسائی ہے مگر جوابات نہیں: اکاؤنٹ/ڈیمَن سیٹنگز (`httpUrl`, `account`) اور رِسیو موڈ کی تصدیق کریں۔
- DMs نظرانداز: ارسال کنندہ پیئرنگ منظوری کا منتظر ہے۔
- گروپ پیغامات نظرانداز: گروپ بھیجنے والے/مینشن گیٹنگ ڈیلیوری روکتی ہے۔
- ترمیم کے بعد config ویلیڈیشن کی غلطیاں: `openclaw doctor --fix` چلائیں۔
- اگر diagnostics میں Signal نظر نہیں آ رہا: تصدیق کریں `channels.signal.enabled: true`.

اضافی جانچیں:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

ٹریاج فلو کے لیے: [/channels/troubleshooting](/channels/troubleshooting)۔

## سیکیورٹی نوٹس

- `signal-cli` اکاؤنٹ کیز مقامی طور پر محفوظ کرتا ہے (عام طور پر `~/.local/share/signal-cli/data/`).
- سرور مائیگریشن یا دوبارہ تعمیر سے پہلے Signal اکاؤنٹ کی حالت کا بیک اپ لیں۔
- جب تک آپ واضح طور پر وسیع DM رسائی نہیں چاہتے، `channels.signal.dmPolicy: "pairing"` برقرار رکھیں۔
- SMS کی تصدیق صرف رجسٹریشن یا ریکوری کے عمل کے لیے درکار ہوتی ہے، لیکن نمبر/اکاؤنٹ پر کنٹرول کھو دینے سے دوبارہ رجسٹریشن پیچیدہ ہو سکتی ہے۔

## کنفیگریشن حوالہ (Signal)

مکمل کنفیگریشن: [Configuration](/gateway/configuration)

فراہم کنندہ کے اختیارات:

- `channels.signal.enabled`: چینل اسٹارٹ اپ فعال/غیرفعال کریں۔
- `channels.signal.account`: بوٹ اکاؤنٹ کے لیے E.164۔
- `channels.signal.cliPath`: `signal-cli` کا راستہ۔
- `channels.signal.httpUrl`: مکمل ڈیمَن URL (ہوسٹ/پورٹ کو اوور رائیڈ کرتا ہے)۔
- `channels.signal.httpHost`, `channels.signal.httpPort`: ڈیمَن بائنڈ (ڈیفالٹ 127.0.0.1:8080)۔
- `channels.signal.autoStart`: آٹو-اسپان ڈیمَن (اگر `httpUrl` غیر سیٹ ہو تو ڈیفالٹ true)۔
- `channels.signal.startupTimeoutMs`: اسٹارٹ اپ ویٹ ٹائم آؤٹ (ms) (حد 120000)۔
- `channels.signal.receiveMode`: `on-start | manual`۔
- `channels.signal.ignoreAttachments`: اٹیچمنٹ ڈاؤن لوڈ چھوڑیں۔
- `channels.signal.ignoreStories`: ڈیمَن سے اسٹوریز نظرانداز کریں۔
- `channels.signal.sendReadReceipts`: ریڈ رسیدیں فارورڈ کریں۔
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (ڈیفالٹ: پیئرنگ)۔
- `channels.signal.allowFrom`: DM allowlist (E.164 or `uuid:<id>`). `open` requires `"*"`. Signal میں صارف نام نہیں ہوتے؛ فون/UUID آئی ڈیز استعمال کریں۔
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (ڈیفالٹ: اجازت فہرست)۔
- `channels.signal.groupAllowFrom`: گروپ ارسال کنندہ اجازت فہرست۔
- `channels.signal.historyLimit`: سیاق کے طور پر شامل کرنے کے لیے زیادہ سے زیادہ گروپ پیغامات (0 بند کرتا ہے)۔
- `channels.signal.dmHistoryLimit`: صارف کے ٹرنز میں DM ہسٹری کی حد۔ فی صارف اووررائیڈز: `channels.signal.dms["<phone_or_uuid>"].historyLimit`۔
- `channels.signal.textChunkLimit`: آؤٹ باؤنڈ چنک سائز (حروف)۔
- `channels.signal.chunkMode`: `length` (ڈیفالٹ) یا `newline` تاکہ لمبائی چنکنگ سے پہلے خالی لائنوں (پیراگراف کی حدیں) پر تقسیم ہو۔
- `channels.signal.mediaMaxMb`: اِن باؤنڈ/آؤٹ باؤنڈ میڈیا حد (MB)۔

متعلقہ عالمی اختیارات:

- `agents.list[].groupChat.mentionPatterns` (Signal مقامی مینشنز سپورٹ نہیں کرتا)۔
- `messages.groupChat.mentionPatterns` (عالمی فال بیک)۔
- `messages.responsePrefix`۔


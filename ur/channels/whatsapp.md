---
title: "WhatsApp"
---

# WhatsApp (ویب چینل)

اسٹیٹس: صرف Baileys کے ذریعے WhatsApp Web۔ Gateway سیشن(ز) کا مالک ہوتا ہے۔

## فوری سیٹ اپ (مبتدی)

1. اگر ممکن ہو تو **الگ فون نمبر** استعمال کریں (سفارش کردہ)۔
2. `~/.openclaw/openclaw.json` میں WhatsApp کنفیگر کریں۔
3. QR کوڈ (Linked Devices) اسکین کرنے کے لیے `openclaw channels login` چلائیں۔
4. Gateway شروع کریں۔

کم از کم کنفیگ:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

## مقاصد

- ایک Gateway پروسیس میں متعدد WhatsApp اکاؤنٹس (ملٹی اکاؤنٹ)۔
- قطعی روٹنگ: جوابات WhatsApp ہی پر واپس آئیں، ماڈل روٹنگ نہیں۔
- ماڈل کو اقتباس شدہ جوابات سمجھنے کے لیے کافی سیاق فراہم کرنا۔

## کنفیگ لکھائیاں

بطورِ طے شدہ، WhatsApp کو `/config set|unset` سے متحرک ہونے والی کنفیگ اپڈیٹس لکھنے کی اجازت ہوتی ہے (اس کے لیے `commands.config: true` درکار ہے)۔

غیرفعال کرنے کے لیے:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## معماری (کون کس چیز کا مالک ہے)

- **Gateway** Baileys ساکٹ اور اِن باکس لوپ کا مالک ہے۔
- **CLI / macOS ایپ** Gateway سے بات کرتی ہیں؛ Baileys کا براہِ راست استعمال نہیں۔
- آؤٹ باؤنڈ ارسال کے لیے **Active listener** درکار ہے؛ بصورتِ دیگر ارسال فوراً ناکام ہو جاتی ہے۔

## فون نمبر حاصل کرنا (دو طریقے)

WhatsApp requires a real mobile number for verification. VoIP اور ورچوئل نمبرز عام طور پر بلاک ہو جاتے ہیں۔ WhatsApp پر OpenClaw چلانے کے دو سپورٹڈ طریقے ہیں:

### مخصوص نمبر (سفارش کردہ)

OpenClaw کے لیے **الگ فون نمبر** استعمال کریں۔ Best UX, clean routing, no self-chat quirks. مثالی سیٹ اپ: **اسپیئر/پرانا Android فون + eSIM**۔ اسے Wi‑Fi اور پاور پر چھوڑ دیں، اور QR کے ذریعے لنک کریں۔

**WhatsApp Business:** آپ اسی ڈیوائس پر مختلف نمبر کے ساتھ WhatsApp Business استعمال کر سکتے ہیں۔ اپنے ذاتی WhatsApp کو الگ رکھنے کے لیے بہترین — WhatsApp Business انسٹال کریں اور OpenClaw نمبر وہیں رجسٹر کریں۔

**نمونہ کنفیگ (مخصوص نمبر، واحد صارف اجازت فہرست):**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

**Pairing موڈ (اختیاری):**
اگر آپ allowlist کے بجائے pairing چاہتے ہیں تو `channels.whatsapp.dmPolicy` کو `pairing` پر سیٹ کریں۔ نامعلوم بھیجنے والوں کو ایک pairing کوڈ ملتا ہے؛ اس کے ساتھ منظوری دیں:
`openclaw pairing approve whatsapp <code>`

### ذاتی نمبر (متبادل)

فوری متبادل: OpenClaw کو **اپنے ہی نمبر** پر چلائیں۔ ٹیسٹنگ کے لیے خود کو پیغام بھیجیں (WhatsApp “Message yourself”) تاکہ آپ کانٹیکٹس کو اسپام نہ کریں۔ سیٹ اپ اور تجربات کے دوران اپنے مین فون پر ویریفکیشن کوڈز پڑھنے کی توقع رکھیں۔ **Self-chat موڈ لازمی فعال کریں۔**
جب وزرڈ آپ سے آپ کا ذاتی WhatsApp نمبر پوچھے تو وہ فون نمبر درج کریں جس سے آپ پیغام بھیجیں گے (مالک/بھیجنے والا)، نہ کہ اسسٹنٹ نمبر۔

**نمونہ کنفیگ (ذاتی نمبر، self-chat):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Self-chat کے جوابات ڈیفالٹ طور پر `[{identity.name}]` ہوتے ہیں جب یہ سیٹ ہو (ورنہ `[openclaw]`)
اگر `messages.responsePrefix` سیٹ نہ ہو۔ اسے واضح طور پر سیٹ کریں تاکہ
پری فکس کو حسبِ منشا بنائیں یا غیر فعال کریں (ہٹانے کے لیے `""` استعمال کریں)۔

### نمبر حاصل کرنے کے مشورے

- **مقامی eSIM** آپ کے ملک کے موبائل کیریئر سے (سب سے قابلِ اعتماد)
  - آسٹریا: [hot.at](https://www.hot.at)
  - برطانیہ: [giffgaff](https://www.giffgaff.com) — مفت SIM، بغیر معاہدہ
- **پری پیڈ SIM** — سستی، صرف ایک SMS وصول کرنا کافی ہے

**اجتناب کریں:** TextNow، Google Voice، اور اکثر "مفت SMS" سروسز — WhatsApp انہیں سختی سے بلاک کرتا ہے۔

**ٹِپ:** نمبر کو صرف ایک ویریفکیشن SMS وصول کرنے کی ضرورت ہوتی ہے۔ اس کے بعد، WhatsApp Web سیشنز `creds.json` کے ذریعے برقرار رہتے ہیں۔

## Twilio کیوں نہیں؟

- ابتدائی OpenClaw بلڈز میں Twilio کی WhatsApp Business انضمام کی معاونت تھی۔
- WhatsApp Business نمبرز ذاتی اسسٹنٹ کے لیے موزوں نہیں۔
- Meta 24 گھنٹے کی جواب دہی ونڈو نافذ کرتا ہے؛ اگر پچھلے 24 گھنٹوں میں جواب نہ دیا ہو تو بزنس نمبر نئے پیغامات شروع نہیں کر سکتا۔
- زیادہ حجم یا “باتونی” استعمال سخت بلاکنگ کو متحرک کرتا ہے، کیونکہ بزنس اکاؤنٹس ذاتی اسسٹنٹ کے درجنوں پیغامات کے لیے نہیں بنے۔
- نتیجہ: ناقابلِ اعتماد ترسیل اور بار بار بلاکس، اس لیے سپورٹ ہٹا دی گئی۔

## لاگ اِن + اسناد

- لاگ اِن کمانڈ: `openclaw channels login` (Linked Devices کے ذریعے QR)۔
- ملٹی اکاؤنٹ لاگ اِن: `openclaw channels login --account <id>` (`<id>` = `accountId`)۔
- ڈیفالٹ اکاؤنٹ (جب `--account` چھوڑ دیا جائے): اگر موجود ہو تو `default`، ورنہ پہلا کنفیگر شدہ اکاؤنٹ آئی ڈی (مرتب)۔
- اسناد `~/.openclaw/credentials/whatsapp/<accountId>/creds.json` میں محفوظ۔
- بیک اپ کاپی `creds.json.bak` پر (خرابی پر بحال)۔
- لیگیسی مطابقت: پرانے انسٹالز Baileys فائلیں براہِ راست `~/.openclaw/credentials/` میں محفوظ کرتے تھے۔
- لاگ آؤٹ: `openclaw channels logout` (یا `--account <id>`) WhatsApp auth state حذف کرتا ہے (مشترکہ `oauth.json` برقرار رہتا ہے)۔
- لاگ آؤٹڈ ساکٹ ⇒ دوبارہ لنک کی ہدایت دینے والی خرابی۔

## اِن باؤنڈ فلو (DM + گروپ)

- WhatsApp ایونٹس `messages.upsert` (Baileys) سے آتے ہیں۔
- شٹ ڈاؤن پر اِن باکس لسٹنرز الگ کر دیے جاتے ہیں تاکہ ٹیسٹس/ری اسٹارٹس میں ایونٹ ہینڈلرز جمع نہ ہوں۔
- اسٹیٹس/براڈکاسٹ چیٹس نظرانداز۔
- براہِ راست چیٹس E.164 استعمال کرتی ہیں؛ گروپس گروپ JID۔
- **DM پالیسی**: `channels.whatsapp.dmPolicy` براہِ راست چیٹ رسائی کنٹرول کرتا ہے (ڈیفالٹ: `pairing`)۔
  - Pairing: نامعلوم ارسال کنندگان کو pairing کوڈ ملتا ہے (منظوری `openclaw pairing approve whatsapp <code>` کے ذریعے؛ کوڈز 1 گھنٹے بعد ختم)۔
  - Open: `channels.whatsapp.allowFrom` میں `"*"` شامل ہونا لازم۔
  - آپ کا لنک شدہ WhatsApp نمبر بالواسطہ طور پر معتبر ہے، اس لیے خود پیغامات `channels.whatsapp.dmPolicy` اور `channels.whatsapp.allowFrom` چیکس کو چھوڑ دیتے ہیں۔

### ذاتی نمبر موڈ (متبادل)

اگر آپ OpenClaw کو **اپنے ذاتی WhatsApp نمبر** پر چلاتے ہیں تو `channels.whatsapp.selfChatMode` فعال کریں (اوپر نمونہ دیکھیں)۔

رویّہ:

- آؤٹ باؤنڈ DMs کبھی pairing جوابات متحرک نہیں کرتے (رابطوں کو اسپام سے بچانے کے لیے)۔
- اِن باؤنڈ نامعلوم ارسال کنندگان پھر بھی `channels.whatsapp.dmPolicy` کی پیروی کرتے ہیں۔
- Self-chat موڈ (allowFrom میں آپ کا نمبر شامل) خودکار read receipts سے بچتا ہے اور mention JIDs کو نظرانداز کرتا ہے۔
- غیر self-chat DMs کے لیے read receipts بھیجے جاتے ہیں۔

## Read receipts

بطورِ طے شدہ، Gateway قبول ہونے پر اِن باؤنڈ WhatsApp پیغامات کو read (نیلے ٹِکس) نشان زد کرتا ہے۔

عالمی طور پر غیرفعال کریں:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

ہر اکاؤنٹ کے لیے غیرفعال کریں:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

نوٹس:

- Self-chat موڈ میں read receipts ہمیشہ چھوڑ دیے جاتے ہیں۔

## WhatsApp FAQ: پیغامات بھیجنا + pairing

**کیا WhatsApp لنک کرنے پر OpenClaw بے ترتیب کانٹیکٹس کو پیغام بھیجے گا؟**  
نہیں۔ ڈیفالٹ DM پالیسی **pairing** ہے، اس لیے نامعلوم بھیجنے والوں کو صرف ایک pairing کوڈ ملتا ہے اور ان کا پیغام **پروسیس نہیں ہوتا**۔ OpenClaw صرف انہی چیٹس کا جواب دیتا ہے جو اسے موصول ہوں، یا اُن بھیجائیوں کا جو آپ واضح طور پر ٹرگر کریں (agent/CLI)۔

**WhatsApp پر pairing کیسے کام کرتی ہے؟**  
Pairing نامعلوم ارسال کنندگان کے لیے DM گیٹ ہے:

- نئے ارسال کنندہ کا پہلا DM ایک مختصر کوڈ واپس کرتا ہے (پیغام پروسیس نہیں ہوتا)۔
- منظوری دیں: `openclaw pairing approve whatsapp <code>` (فہرست: `openclaw pairing list whatsapp`)۔
- کوڈز 1 گھنٹے بعد ختم؛ زیرِ التوا درخواستیں ہر چینل پر 3 تک محدود۔

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Replies still come from the **same WhatsApp account**, and direct chats collapse to each agent's main session, so use **one agent per person**. DM رسائی کنٹرول (`dmPolicy`/`allowFrom`) فی WhatsApp اکاؤنٹ گلوبل ہوتا ہے۔ [Multi-Agent Routing](/concepts/multi-agent) دیکھیں۔

**وزرڈ میرا فون نمبر کیوں پوچھتا ہے؟**  
وزرڈ اسے آپ کی **allowlist/owner** سیٹ کرنے کے لیے استعمال کرتا ہے تاکہ آپ کی اپنی DMs کی اجازت ہو۔ It’s not used for auto-sending. اگر آپ اپنے ذاتی WhatsApp نمبر پر چلاتے ہیں تو وہی نمبر استعمال کریں اور `channels.whatsapp.selfChatMode` کو فعال کریں۔

## پیغام کی نارملائزیشن (ماڈل کیا دیکھتا ہے)

- `Body` موجودہ پیغام کا متن لفافے سمیت ہوتا ہے۔

- اقتباس شدہ جواب کا سیاق **ہمیشہ شامل** کیا جاتا ہے:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- جواب کی میٹاڈیٹا بھی سیٹ ہوتی ہے:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = اقتباس شدہ متن یا میڈیا پلیس ہولڈر
  - `ReplyToSender` = معلوم ہونے پر E.164

- صرف میڈیا والے اِن باؤنڈ پیغامات پلیس ہولڈرز استعمال کرتے ہیں:
  - `<media:image|video|audio|document|sticker>`

## گروپس

- گروپس `agent:<agentId>:whatsapp:group:<jid>` سیشنز سے میپ ہوتے ہیں۔
- گروپ پالیسی: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (ڈیفالٹ `allowlist`)۔
- ایکٹیویشن موڈز:
  - `mention` (ڈیفالٹ): @mention یا regex میچ درکار۔
  - `always`: ہمیشہ متحرک۔
- `/activation mention|always` صرف مالک کے لیے ہے اور اسے الگ پیغام کے طور پر بھیجنا لازم ہے۔
- مالک = `channels.whatsapp.allowFrom` (یا اگر غیر سیٹ ہو تو self E.164)۔
- **ہسٹری انجیکشن** (صرف زیرِ التوا):
  - حالیہ _غیر پروسیس شدہ_ پیغامات (ڈیفالٹ 50) درج کیے جاتے ہیں:
    `[Chat messages since your last reply - for context]` (جو پیغامات پہلے ہی سیشن میں ہوں وہ دوبارہ شامل نہیں ہوتے)
  - موجودہ پیغام درج ہوتا ہے:
    `[Current message - respond to this]`
  - ارسال کنندہ لاحقہ شامل: `[from: Name (+E164)]`
- گروپ میٹاڈیٹا 5 منٹ کے لیے کیش (موضوع + شرکاء)۔

## جواب کی ترسیل (تھریڈنگ)

- WhatsApp Web معیاری پیغامات بھیجتا ہے (موجودہ Gateway میں اقتباسی تھریڈنگ نہیں)۔
- اس چینل پر reply tags نظرانداز کیے جاتے ہیں۔

## اعترافی ردِ عمل (وصولی پر خودکار ری ایکشن)

WhatsApp آنے والے پیغامات پر فوراً ایموجی ری ایکشن خودکار طور پر بھیج سکتا ہے، بوٹ کے جواب بنانے سے پہلے۔ یہ صارفین کو فوری فیڈبیک فراہم کرتا ہے کہ ان کا پیغام موصول ہو گیا ہے۔

**کنفیگریشن:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**اختیارات:**

- `emoji` (string): تصدیق کے لیے استعمال ہونے والا ایموجی (مثلاً "👀", "✅", "📨")۔ خالی یا شامل نہ کرنے کی صورت میں = فیچر غیر فعال۔
- `direct` (boolean، ڈیفالٹ: `true`): براہِ راست/DM چیٹس میں ری ایکشن بھیجیں۔
- `group` (string، ڈیفالٹ: `"mentions"`): گروپ چیٹ رویّہ:
  - `"always"`: تمام گروپ پیغامات پر ری ایکٹ کریں (@mention کے بغیر بھی)
  - `"mentions"`: صرف تب ری ایکٹ کریں جب بوٹ کو @mention کیا جائے
  - `"never"`: گروپس میں کبھی ری ایکٹ نہ کریں

**ہر اکاؤنٹ کے لیے اوور رائیڈ:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**رویّہ نوٹس:**

- ری ایکشن پیغام موصول ہوتے ہی **فوراً** بھیجے جاتے ہیں، ٹائپنگ اشاروں یا بوٹ کے جوابات سے پہلے۔
- `requireMention: false` (ایکٹیویشن: ہمیشہ) والے گروپس میں `group: "mentions"` تمام پیغامات پر ری ایکٹ کرے گا (صرف @mentions پر نہیں)۔
- Fire-and-forget: ری ایکشن کی ناکامیاں لاگ ہو جاتی ہیں مگر بوٹ کے جواب کو نہیں روکتیں۔
- گروپ ری ایکشنز کے لیے شریک JID خودکار طور پر شامل ہوتا ہے۔
- WhatsApp `messages.ackReaction` کو نظرانداز کرتا ہے؛ اس کے بجائے `channels.whatsapp.ackReaction` استعمال کریں۔

## Agent ٹول (ری ایکشنز)

- ٹول: `whatsapp` بذریعہ `react` ایکشن (`chatJid`, `messageId`, `emoji`, اختیاری `remove`)۔
- اختیاری: `participant` (گروپ ارسال کنندہ)، `fromMe` (اپنے پیغام پر ری ایکٹ)، `accountId` (ملٹی اکاؤنٹ)۔
- ری ایکشن ہٹانے کی semantics: دیکھیں [/tools/reactions](/tools/reactions)۔
- ٹول گیٹنگ: `channels.whatsapp.actions.reactions` (ڈیفالٹ: فعال)۔

## حدود

- آؤٹ باؤنڈ متن `channels.whatsapp.textChunkLimit` تک ٹکڑوں میں بھیجا جاتا ہے (ڈیفالٹ 4000)۔
- اختیاری نئی لائن ٹکڑا بندی: خالی لائنوں (پیراگراف حدود) پر تقسیم کے لیے `channels.whatsapp.chunkMode="newline"` سیٹ کریں، پھر لمبائی کے مطابق ٹکڑا بندی۔
- اِن باؤنڈ میڈیا سیوز `channels.whatsapp.mediaMaxMb` سے محدود (ڈیفالٹ 50 MB)۔
- آؤٹ باؤنڈ میڈیا آئٹمز `agents.defaults.mediaMaxMb` سے محدود (ڈیفالٹ 5 MB)۔

## آؤٹ باؤنڈ ارسال (متن + میڈیا)

- فعال ویب لسٹنر استعمال کرتا ہے؛ Gateway نہ چل رہا ہو تو خرابی۔
- متن ٹکڑا بندی: فی پیغام 4k زیادہ سے زیادہ (قابلِ کنفیگ via `channels.whatsapp.textChunkLimit`, اختیاری `channels.whatsapp.chunkMode`)۔
- میڈیا:
  - تصویر/ویڈیو/آڈیو/دستاویز معاون۔
  - آڈیو PTT کے طور پر بھیجا جاتا ہے؛ `audio/ogg` ⇒ `audio/ogg; codecs=opus`۔
  - کیپشن صرف پہلے میڈیا آئٹم پر۔
  - میڈیا فِچ HTTP(S) اور لوکل راستوں کی معاونت کرتا ہے۔
  - متحرک GIFs: WhatsApp اِن لائن لوپنگ کے لیے `gifPlayback: true` کے ساتھ MP4 چاہتا ہے۔
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: `send` پیرامیٹرز میں `gifPlayback: true` شامل

## وائس نوٹس (PTT آڈیو)

WhatsApp آڈیو کو **وائس نوٹس** (PTT ببل) کے طور پر بھیجتا ہے۔

- بہترین نتائج: OGG/Opus۔ OpenClaw `audio/ogg` کو `audio/ogg; codecs=opus` میں دوبارہ لکھتا ہے۔
- `[[audio_as_voice]]` WhatsApp کے لیے نظرانداز (آڈیو پہلے ہی وائس نوٹ کے طور پر جاتا ہے)۔

## میڈیا حدود + اصلاح

- ڈیفالٹ آؤٹ باؤنڈ حد: 5 MB (فی میڈیا آئٹم)۔
- اوور رائیڈ: `agents.defaults.mediaMaxMb`۔
- تصاویر خودکار طور پر حد کے اندر JPEG میں آپٹمائز ہوتی ہیں (ری سائز + کوالٹی سویپ)۔
- حد سے بڑی میڈیا ⇒ خرابی؛ میڈیا جواب متن وارننگ پر واپس آتا ہے۔

## ہارٹ بیٹس

- **Gateway ہارٹ بیٹ** کنکشن صحت لاگ کرتا ہے (`web.heartbeatSeconds`, ڈیفالٹ 60s)۔
- **Agent ہارٹ بیٹ** ہر agent کے لیے (`agents.list[].heartbeat`) یا عالمی طور پر
  `agents.defaults.heartbeat` کے ذریعے کنفیگر کیا جا سکتا ہے (جب فی-agent اندراجات نہ ہوں تو fallback)۔
  - کنفیگر کردہ heartbeat پرامپٹ استعمال کرتا ہے (ڈیفالٹ: `Read HEARTBEAT.md if it exists (workspace context).` اس کی سختی سے پیروی کریں۔ پچھلی چیٹس سے پرانے کام اخذ نہ کریں اور نہ دہرائیں۔ اگر کسی چیز پر توجہ درکار نہ ہو تو `HEARTBEAT_OK` کے ساتھ جواب دیں۔) + `HEARTBEAT_OK` اسکیپ برتاؤ۔
  - ترسیل بطورِ طے شدہ آخری استعمال شدہ چینل پر (یا کنفیگر شدہ ہدف)۔

## دوبارہ کنیکٹ رویّہ

- بیک آف پالیسی: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`۔
- اگر maxAttempts پہنچ جائے تو ویب مانیٹرنگ رک جاتی ہے (degraded)۔
- لاگ آؤٹ ⇒ روک دیں اور دوبارہ لنک درکار۔

## کنفیگ فوری نقشہ

- `channels.whatsapp.dmPolicy` (DM پالیسی: pairing/allowlist/open/disabled)۔
- `channels.whatsapp.selfChatMode` (اسی فون پر سیٹ اپ؛ بوٹ آپ کا ذاتی WhatsApp نمبر استعمال کرتا ہے)۔
- `channels.whatsapp.allowFrom` (DM allowlist)۔ WhatsApp E.164 فون نمبرز استعمال کرتا ہے (یوزرنیم نہیں)۔
- `channels.whatsapp.mediaMaxMb` (اِن باؤنڈ میڈیا سیو حد)۔
- `channels.whatsapp.ackReaction` (پیغام وصولی پر خودکار ری ایکشن: `{emoji, direct, group}`)۔
- `channels.whatsapp.accounts.<accountId>`.\*`(فی اکاؤنٹ سیٹنگز + اختیاری`authDir\`)۔
- `channels.whatsapp.accounts.<accountId>`.mediaMaxMb\` (فی اکاؤنٹ آنے والے میڈیا کی حد)۔
- `channels.whatsapp.accounts.<accountId>`.ackReaction\` (فی اکاؤنٹ ack reaction اووررائیڈ)۔
- `channels.whatsapp.groupAllowFrom` (گروپ ارسال کنندہ اجازت فہرست)۔
- `channels.whatsapp.groupPolicy` (گروپ پالیسی)۔
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>`.historyLimit`(گروپ ہسٹری کانٹیکسٹ؛`0\` غیر فعال کرتا ہے)۔
- `channels.whatsapp.dmHistoryLimit` (DM ہسٹری کی حد یوزر ٹرنز میں)۔ فی یوزر اووررائیڈز: `channels.whatsapp.dms["<phone>"].historyLimit`۔
- `channels.whatsapp.groups` (گروپ اجازت فہرست + mention گیٹنگ ڈیفالٹس؛ سب کی اجازت کے لیے `"*"` استعمال کریں)
- `channels.whatsapp.actions.reactions` (WhatsApp ٹول ری ایکشنز گیٹ)۔
- `agents.list[].groupChat.mentionPatterns` (یا `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (ان باؤنڈ پری فکس؛ فی اکاؤنٹ: `channels.whatsapp.accounts.<accountId>`.messagePrefix`; منسوخ شدہ: `messages.messagePrefix\`)۔
- `messages.responsePrefix` (آؤٹ باؤنڈ پری فکس)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (اختیاری اوور رائیڈ)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (فی-agent اوور رائیڈز)
- `session.*` (اسکوپ، idle، اسٹور، mainKey)
- `web.enabled` (false ہونے پر چینل اسٹارٹ اپ غیرفعال)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## لاگز + خرابیوں کا ازالہ

- ذیلی نظام: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`۔
- لاگ فائل: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (قابلِ کنفیگ)۔
- خرابیوں کے ازالے کی رہنمائی: [Gateway troubleshooting](/gateway/troubleshooting)۔

## خرابیوں کا ازالہ (فوری)

**لنک نہیں / QR لاگ اِن درکار**

- علامت: `channels status` میں `linked: false` دکھتا ہے یا “Not linked” کی وارننگ۔
- حل: گیٹ وے ہوسٹ پر `openclaw channels login` چلائیں اور QR اسکین کریں (WhatsApp → Settings → Linked Devices)۔

**لنک ہے مگر منقطع / دوبارہ کنیکٹ لوپ**

- علامت: `channels status` میں `running, disconnected` دکھتا ہے یا “Linked but disconnected” کی وارننگ۔
- درستگی: `openclaw doctor` (یا گیٹ وے ری اسٹارٹ کریں)۔ اگر مسئلہ برقرار رہے تو `channels login` کے ذریعے دوبارہ لنک کریں اور `openclaw logs --follow` کا معائنہ کریں۔

**Bun رَن ٹائم**

- Bun کی **سفارش نہیں کی جاتی**۔ WhatsApp (Baileys) اور Telegram Bun پر غیر قابلِ اعتماد ہیں۔
  گیٹ وے کو **Node** کے ساتھ چلائیں۔ (Getting Started کے رن ٹائم نوٹ دیکھیں۔)

---
title: "Telegram"
---

# Telegram (Bot API)

Status: production-ready for bot DMs + groups via grammY. Long-polling by default; webhook optional.

## فوری سیٹ اپ (مبتدی)

1. Create a bot with **@BotFather** ([direct link](https://t.me/BotFather)). Confirm the handle is exactly `@BotFather`, then copy the token.
2. ٹوکن سیٹ کریں:
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - یا کنفیگ: `channels.telegram.botToken: "..."`۔
   - اگر دونوں سیٹ ہوں تو کنفیگ کو ترجیح دی جاتی ہے (env فال بیک صرف ڈیفالٹ اکاؤنٹ کے لیے ہے)۔
3. گیٹ وے شروع کریں۔
4. DM رسائی بطورِ طے شدہ pairing ہے؛ پہلی بار رابطے پر pairing کوڈ منظور کریں۔

کم سے کم کنفیگ:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
    },
  },
}
```

## یہ کیا ہے

- Gateway کی ملکیت والا Telegram Bot API چینل۔
- متعین روٹنگ: جوابات Telegram ہی پر واپس جاتے ہیں؛ ماڈل چینلز کا انتخاب نہیں کرتا۔
- DMs ایجنٹ کے مرکزی سیشن کو شیئر کرتے ہیں؛ گروپس علیحدہ رہتے ہیں (`agent:<agentId>:telegram:group:<chatId>`)۔

## سیٹ اپ (تیز راستہ)

### 1. بوٹ ٹوکن بنائیں (BotFather)

1. Open Telegram and chat with **@BotFather** ([direct link](https://t.me/BotFather)). Confirm the handle is exactly `@BotFather`.
2. `/newbot` چلائیں، پھر ہدایات پر عمل کریں (نام + یوزرنیم جو `bot` پر ختم ہو)۔
3. ٹوکن کاپی کریں اور محفوظ رکھیں۔

اختیاری BotFather سیٹنگز:

- `/setjoingroups` — بوٹ کو گروپس میں شامل کرنے کی اجازت/ممانعت۔
- `/setprivacy` — کنٹرول کریں کہ بوٹ تمام گروپ پیغامات دیکھے یا نہیں۔

### 2. ٹوکن کنفیگر کریں (env یا کنفیگ)

مثال:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

Env option: `TELEGRAM_BOT_TOKEN=...` (works for the default account).
اگر env اور کنفیگ دونوں سیٹ ہوں تو کنفیگ کو ترجیح دی جاتی ہے۔

ملٹی اکاؤنٹ سپورٹ: ہر اکاؤنٹ کے ٹوکنز کے ساتھ `channels.telegram.accounts` استعمال کریں اور اختیاری `name` شامل کریں۔ See [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) for the shared pattern.

3. gateway شروع کریں۔ Telegram starts when a token is resolved (config first, env fallback).
4. DM access defaults to pairing. Approve the code when the bot is first contacted.
5. گروپس کے لیے: بوٹ شامل کریں، پرائیویسی/ایڈمن رویہ طے کریں (نیچے)، پھر `channels.telegram.groups` سیٹ کریں تاکہ مینشن گیٹنگ + اجازت فہرست کنٹرول ہو۔

## ٹوکن + پرائیویسی + اجازتیں (Telegram جانب)

### ٹوکن بنانا (BotFather)

- `/newbot` بوٹ بناتا ہے اور ٹوکن واپس دیتا ہے (اسے خفیہ رکھیں)۔
- اگر ٹوکن لیک ہو جائے تو @BotFather کے ذریعے اسے منسوخ/دوبارہ بنائیں اور کنفیگ اپ ڈیٹ کریں۔

### گروپ پیغام کی مرئیت (Privacy Mode)

Telegram bots default to **Privacy Mode**, which limits which group messages they receive.
If your bot must see _all_ group messages, you have two options:

- `/setprivacy` کے ساتھ پرائیویسی موڈ بند کریں **یا**
- بوٹ کو گروپ **ایڈمن** کے طور پر شامل کریں (ایڈمن بوٹس تمام پیغامات وصول کرتے ہیں)۔

**نوٹ:** پرائیویسی موڈ تبدیل کرنے پر Telegram تبدیلی کے مؤثر ہونے کے لیے بوٹ کو ہر گروپ سے ہٹا کر دوبارہ شامل کرنے کا تقاضا کرتا ہے۔

### گروپ اجازتیں (ایڈمن حقوق)

Admin status is set inside the group (Telegram UI). Admin bots always receive all
group messages, so use admin if you need full visibility.

## یہ کیسے کام کرتا ہے (رویہ)

- آنے والے پیغامات کو مشترکہ چینل لفافے میں نارملائز کیا جاتا ہے جس میں جواب کا سیاق اور میڈیا پلیس ہولڈرز شامل ہوتے ہیں۔
- گروپ جوابات بطورِ طے شدہ مینشن چاہتے ہیں (نیٹو @mention یا `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`)۔
- ملٹی ایجنٹ اووررائیڈ: `agents.list[].groupChat.mentionPatterns` پر فی ایجنٹ پیٹرنز سیٹ کریں۔
- جوابات ہمیشہ اسی Telegram چیٹ پر واپس جاتے ہیں۔
- لانگ پولنگ grammY رنر کے ساتھ فی چیٹ سیکوئنسنگ استعمال کرتی ہے؛ مجموعی کنکرنسی `agents.defaults.maxConcurrent` سے محدود ہے۔
- Telegram Bot API ریڈ رسیدز سپورٹ نہیں کرتا؛ `sendReadReceipts` آپشن موجود نہیں۔

## ڈرافٹ اسٹریمنگ

OpenClaw `sendMessageDraft` کے ذریعے Telegram DMs میں جزوی جوابات اسٹریم کر سکتا ہے۔

ضروریات:

- @BotFather میں بوٹ کے لیے Threaded Mode فعال ہو (forum topic mode)۔
- صرف نجی چیٹ تھریڈز (Telegram آنے والے پیغامات پر `message_thread_id` شامل کرتا ہے)۔
- `channels.telegram.streamMode` کو `"off"` پر سیٹ نہ کیا گیا ہو (ڈیفالٹ: `"partial"`، `"block"` چنکڈ ڈرافٹ اپ ڈیٹس فعال کرتا ہے)۔

ڈرافٹ اسٹریمنگ صرف DMs کے لیے ہے؛ Telegram گروپس یا چینلز میں اسے سپورٹ نہیں کرتا۔

## فارمیٹنگ (Telegram HTML)

- باہر جانے والا Telegram متن `parse_mode: "HTML"` استعمال کرتا ہے (Telegram کی سپورٹڈ ٹیگ سب سیٹ)۔
- Markdown نما ان پٹ کو **Telegram-محفوظ HTML** میں رینڈر کیا جاتا ہے (بولڈ/اِٹالک/اسٹرائیک/کوڈ/لنکس)؛ بلاک عناصر کو نئی لائنوں/بلٹس کے ساتھ سادہ متن میں فلیٹن کیا جاتا ہے۔
- ماڈلز سے آنے والا خام HTML Telegram پارس ایررز سے بچنے کے لیے ایسکیپ کیا جاتا ہے۔
- اگر Telegram HTML پے لوڈ مسترد کر دے تو OpenClaw وہی پیغام سادہ متن کے طور پر دوبارہ بھیجتا ہے۔

## کمانڈز (نیٹو + کسٹم)

OpenClaw registers native commands (like `/status`, `/reset`, `/model`) with Telegram’s bot menu on startup.
You can add custom commands to the menu via config:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

## سیٹ اپ خرابیوں کا ازالہ (کمانڈز)

- لاگز میں `setMyCommands failed` عموماً اس بات کی علامت ہے کہ `api.telegram.org` تک آؤٹ باؤنڈ HTTPS/DNS بلاک ہے۔
- اگر `sendMessage` یا `sendChatAction` ناکامیاں نظر آئیں تو IPv6 روٹنگ اور DNS چیک کریں۔

مزید مدد: [Channel troubleshooting](/channels/troubleshooting)۔

نوٹس:

- کسٹم کمانڈز **صرف مینو اندراجات** ہیں؛ OpenClaw انہیں نافذ نہیں کرتا جب تک آپ کہیں اور ہینڈل نہ کریں۔
- Some commands can be handled by plugins/skills without being registered in Telegram’s command menu. These still work when typed (they just won't show up in `/commands` / the menu).
- کمانڈ نام نارملائز ہوتے ہیں (ابتدائی `/` ہٹا دیا جاتا ہے، لوئر کیس) اور انہیں `a-z`, `0-9`, `_` (1–32 حروف) سے میچ کرنا چاہیے۔
- Custom commands **cannot override native commands**. Conflicts are ignored and logged.
- اگر `commands.native` غیرفعال ہو تو صرف کسٹم کمانڈز رجسٹر ہوتی ہیں (یا اگر کوئی نہ ہوں تو کلیئر ہو جاتی ہیں)۔

### Device pairing commands (`device-pair` plugin)

If the `device-pair` plugin is installed, it adds a Telegram-first flow for pairing a new phone:

1. `/pair` generates a setup code (sent as a separate message for easy copy/paste).
2. Paste the setup code in the iOS app to connect.
3. `/pair approve` approves the latest pending device request.

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## حدود

- باہر جانے والا متن `channels.telegram.textChunkLimit` تک چنک کیا جاتا ہے (ڈیفالٹ 4000)۔
- اختیاری نیو لائن چنکنگ: `channels.telegram.chunkMode="newline"` سیٹ کریں تاکہ لمبائی چنکنگ سے پہلے خالی لائنوں (پیراگراف حدود) پر تقسیم ہو۔
- میڈیا ڈاؤن لوڈز/اپ لوڈز `channels.telegram.mediaMaxMb` سے محدود ہیں (ڈیفالٹ 5)۔
- Telegram Bot API requests time out after `channels.telegram.timeoutSeconds` (default 500 via grammY). Set lower to avoid long hangs.
- Group history context uses `channels.telegram.historyLimit` (or `channels.telegram.accounts.*.historyLimit`), falling back to `messages.groupChat.historyLimit`. Set `0` to disable (default 50).
- DM history can be limited with `channels.telegram.dmHistoryLimit` (user turns). Per-user overrides: `channels.telegram.dms["<user_id>"].historyLimit`.

## گروپ ایکٹیویشن موڈز

By default, the bot only responds to mentions in groups (`@botname` or patterns in `agents.list[].groupChat.mentionPatterns`). To change this behavior:

### کنفیگ کے ذریعے (سفارش کردہ)

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // always respond in this group
      },
    },
  },
}
```

**Important:** Setting `channels.telegram.groups` creates an **allowlist** - only listed groups (or `"*"`) will be accepted.
Forum topics inherit their parent group config (allowFrom, requireMention, skills, prompts) unless you add per-topic overrides under `channels.telegram.groups.<groupId>.topics.<topicId>`.

تمام گروپس کے لیے ہمیشہ جواب دینے کی اجازت دینے کے لیے:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }, // all groups, always respond
      },
    },
  },
}
```

تمام گروپس کے لیے مینشن-اونلی برقرار رکھنے کے لیے (ڈیفالٹ رویہ):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

### کمانڈ کے ذریعے (سیشن لیول)

گروپ میں بھیجیں:

- `/activation always` — تمام پیغامات پر جواب دیں
- `/activation mention` — مینشن درکار (ڈیفالٹ)

**Note:** Commands update session state only. For persistent behavior across restarts, use config.

### گروپ چیٹ ID حاصل کرنا

گروپ کا کوئی بھی پیغام `@userinfobot` یا `@getidsbot` کو Telegram پر فارورڈ کریں تاکہ چیٹ ID دیکھیں (منفی نمبر جیسے `-1001234567890`)۔

**مشورہ:** اپنے یوزر ID کے لیے بوٹ کو DM کریں؛ وہ آپ کے یوزر ID کے ساتھ جواب دے گا (pairing پیغام)، یا کمانڈز فعال ہونے پر `/whoami` استعمال کریں۔

**پرائیویسی نوٹ:** `@userinfobot` ایک تھرڈ پارٹی بوٹ ہے۔ اگر آپ چاہیں تو بوٹ کو گروپ میں شامل کریں، ایک پیغام بھیجیں، اور `openclaw logs --follow` استعمال کر کے `chat.id` پڑھیں، یا Bot API کا `getUpdates` استعمال کریں۔

## کنفیگ لکھائی

بطورِ طے شدہ، Telegram کو چینل ایونٹس یا `/config set|unset` سے متحرک کنفیگ اپ ڈیٹس لکھنے کی اجازت ہے۔

یہ اس وقت ہوتا ہے جب:

- کسی گروپ کو سپرگروپ میں اپ گریڈ کیا جاتا ہے اور ٹیلیگرام `migrate_to_chat_id` جاری کرتا ہے (چیٹ آئی ڈی تبدیل ہو جاتی ہے)۔ OpenClaw خودکار طور پر `channels.telegram.groups` کو مائیگریٹ کر سکتا ہے۔
- آپ Telegram چیٹ میں `/config set` یا `/config unset` چلاتے ہیں (درکار: `commands.config: true`)۔

غیرفعال کرنے کے لیے:

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## موضوعات (فورم سپرگروپس)

ٹیلیگرام فورم ٹاپکس میں ہر پیغام کے لیے ایک `message_thread_id` شامل ہوتا ہے۔ OpenClaw:

- Telegram گروپ سیشن کی کلید کے ساتھ `:topic:<threadId>` شامل کرتا ہے تاکہ ہر ٹاپک علیحدہ رہے۔
- ٹائپنگ اشارے اور جوابات `message_thread_id` کے ساتھ بھیجتا ہے تاکہ جوابات اسی ٹاپک میں رہیں۔
- جنرل ٹاپک (تھریڈ ID `1`) خاص ہے: پیغام بھیجتے وقت `message_thread_id` شامل نہیں کیا جاتا (Telegram اسے مسترد کرتا ہے)، مگر ٹائپنگ اشارے میں شامل رہتا ہے۔
- ٹیمپلیٹ سیاق میں روٹنگ/ٹیمپلیٹنگ کے لیے `MessageThreadId` + `IsForum` ظاہر کرتا ہے۔
- ٹاپک کے مطابق کنفیگریشن `channels.telegram.groups.<chatId>` کے تحت دستیاب ہے.topics.&lt;threadId&gt;\` (skills, allowlists, auto-reply, system prompts, disable).
- ٹاپک کنفیگز گروپ سیٹنگز کو وراثت میں لیتے ہیں (requireMention, allowlists, skills, prompts, enabled) جب تک فی ٹاپک اووررائیڈ نہ ہو۔

نجی چیٹس میں کچھ خاص صورتوں میں `message_thread_id` شامل ہو سکتا ہے۔ OpenClaw ڈی ایم سیشن کی کو بغیر تبدیل کیے رکھتا ہے، لیکن جب موجود ہو تو جوابات/ڈرافٹ اسٹریمنگ کے لیے تھریڈ آئی ڈی استعمال کرتا ہے۔

## اِن لائن بٹنز

Telegram اِن لائن کی بورڈز کو کال بیک بٹنز کے ساتھ سپورٹ کرتا ہے۔

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

فی اکاؤنٹ کنفیگریشن کے لیے:

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

دائرۂ کار:

- `off` — اِن لائن بٹنز غیرفعال
- `dm` — صرف DMs (گروپ اہداف بلاک)
- `group` — صرف گروپس (DM اہداف بلاک)
- `all` — DMs + گروپس
- `allowlist` — DMs + گروپس، مگر صرف `allowFrom`/`groupAllowFrom` کے مطابق اجازت یافتہ بھیجنے والے (کنٹرول کمانڈز جیسے ہی قواعد)

ڈیفالٹ: `allowlist`۔
لیگیسی: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`۔

### بٹنز بھیجنا

پیغام ٹول کو `buttons` پیرامیٹر کے ساتھ استعمال کریں:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

جب صارف بٹن پر کلک کرتا ہے تو کال بیک ڈیٹا ایجنٹ کو اس فارمیٹ میں پیغام کے طور پر واپس بھیجا جاتا ہے:
`callback_data: value`

### کنفیگریشن آپشنز

Telegram کی صلاحیتیں دو سطحوں پر کنفیگر کی جا سکتی ہیں (اوپر آبجیکٹ فارم دکھایا گیا ہے؛ لیگیسی اسٹرنگ ارریز اب بھی سپورٹڈ ہیں):

- `channels.telegram.capabilities`: تمام Telegram اکاؤنٹس پر لاگو عالمی ڈیفالٹ کیپیبیلٹی کنفیگ، جب تک اووررائیڈ نہ ہو۔
- `channels.telegram.accounts.<account>.capabilities`: فی اکاؤنٹ صلاحیتیں جو اسی مخصوص اکاؤنٹ کے لیے گلوبل ڈیفالٹس کو اووررائیڈ کرتی ہیں۔

جب تمام ٹیلیگرام بوٹس/اکاؤنٹس کا برتاؤ ایک جیسا ہونا چاہیے تو گلوبل سیٹنگ استعمال کریں۔ جب مختلف بوٹس کو مختلف رویوں کی ضرورت ہو تو فی اکاؤنٹ کنفیگریشن استعمال کریں (مثال کے طور پر، ایک اکاؤنٹ صرف ڈی ایمز ہینڈل کرتا ہو جبکہ دوسرا گروپس میں اجازت رکھتا ہو)۔

## رسائی کا کنٹرول (DMs + گروپس)

### DM رسائی

- ڈیفالٹ: `channels.telegram.dmPolicy = "pairing"`۔ نامعلوم ارسال کنندگان کو ایک پیئرنگ کوڈ ملتا ہے؛ منظوری تک پیغامات نظرانداز کیے جاتے ہیں (کوڈز 1 گھنٹے بعد ختم ہو جاتے ہیں)۔
- منظوری کے طریقے:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- ٹیلیگرام ڈی ایمز کے لیے پیئرنگ ڈیفالٹ ٹوکن ایکسچینج ہے۔ تفصیلات: [Pairing](/channels/pairing)
- `channels.telegram.allowFrom` عددی یوزر آئی ڈیز (سفارش کردہ) یا `@username` اندراجات قبول کرتا ہے۔ یہ بوٹ کا یوزرنیم **نہیں** ہے؛ انسانی بھیجنے والے کی آئی ڈی استعمال کریں۔ وزارڈ `@username` قبول کرتا ہے اور جب ممکن ہو اسے عددی آئی ڈی میں ریزولو کر دیتا ہے۔

#### اپنا Telegram یوزر ID تلاش کرنا

محفوظ (بغیر تھرڈ پارٹی بوٹ):

1. گیٹ وے شروع کریں اور اپنے بوٹ کو DM کریں۔
2. `openclaw logs --follow` چلائیں اور `from.id` تلاش کریں۔

متبادل (آفیشل Bot API):

1. اپنے بوٹ کو DM کریں۔
2. اپنے بوٹ ٹوکن کے ساتھ اپ ڈیٹس حاصل کریں اور `message.from.id` پڑھیں:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

تھرڈ پارٹی (کم پرائیویٹ):

- `@userinfobot` یا `@getidsbot` کو DM کریں اور واپس آنے والا یوزر ID استعمال کریں۔

### گروپ رسائی

دو آزاد کنٹرولز:

**1. کون سے گروپس کی اجازت ہے** (`channels.telegram.groups` کے ذریعے گروپ الاؤلسٹ):

- `groups` کنفیگ نہیں = تمام گروپس اجازت یافتہ
- `groups` کنفیگ کے ساتھ = صرف فہرست میں موجود گروپس یا `"*"` اجازت یافتہ
- مثال: `"groups": { "-1001234567890": {}, "*": {} }` تمام گروپس کی اجازت دیتا ہے

**2. کون سے بھیجنے والوں کی اجازت ہے** (`channels.telegram.groupPolicy` کے ذریعے بھیجنے والے کی فلٹرنگ):

- `"open"` = اجازت یافتہ گروپس میں تمام بھیجنے والے پیغام بھیج سکتے ہیں
- `"allowlist"` = صرف `channels.telegram.groupAllowFrom` میں موجود بھیجنے والے پیغام بھیج سکتے ہیں
- `"disabled"` = کوئی بھی گروپ پیغام قبول نہیں
  ڈیفالٹ `groupPolicy: "allowlist"` ہے (جب تک آپ `groupAllowFrom` شامل نہ کریں، بلاک)۔

زیادہ تر صارفین چاہتے ہیں: `groupPolicy: "allowlist"` + `groupAllowFrom` + `channels.telegram.groups` میں مخصوص گروپس کی فہرست

کسی مخصوص گروپ میں **کسی بھی رکن** کو بات کرنے کی اجازت دینے کے لیے (جبکہ کنٹرول کمانڈز مجاز بھیجنے والوں تک محدود رہیں)، فی گروپ اووررائیڈ سیٹ کریں:

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

## لانگ پولنگ بمقابلہ ویب ہوک

- ڈیفالٹ: لانگ پولنگ (عوامی URL درکار نہیں)۔
- ویب ہوک موڈ: `channels.telegram.webhookUrl` اور `channels.telegram.webhookSecret` سیٹ کریں (اختیاری `channels.telegram.webhookPath`)۔
  - لوکل لسٹنر `0.0.0.0:8787` پر بائنڈ ہوتا ہے اور بطورِ طے شدہ `POST /telegram-webhook` سروس کرتا ہے۔
  - اگر آپ کا عوامی URL مختلف ہو تو ریورس پراکسی استعمال کریں اور `channels.telegram.webhookUrl` کو عوامی اینڈ پوائنٹ پر پوائنٹ کریں۔

## ریپلائی تھریڈنگ

Telegram ٹیگز کے ذریعے اختیاری تھریڈڈ جوابات سپورٹ کرتا ہے:

- `[[reply_to_current]]` -- محرک پیغام کو جواب دیں۔
- `[[reply_to:<id>]]` -- مخصوص پیغام ID کو جواب دیں۔

`channels.telegram.replyToMode` کے ذریعے کنٹرول:

- `first` (ڈیفالٹ)، `all`, `off`۔

## آڈیو پیغامات (وائس بمقابلہ فائل)

ٹیلیگرام **وائس نوٹس** (گول ببل) اور **آڈیو فائلز** (میٹاڈیٹا کارڈ) میں فرق کرتا ہے۔
بیک ورڈ کمپیٹیبلٹی کے لیے OpenClaw بطور ڈیفالٹ آڈیو فائلز استعمال کرتا ہے۔

ایجنٹ کے جوابات میں وائس نوٹ ببل پر مجبور کرنے کے لیے جواب میں کہیں بھی یہ ٹیگ شامل کریں:

- `[[audio_as_voice]]` — آڈیو کو فائل کے بجائے وائس نوٹ کے طور پر بھیجیں۔

ڈیلیور کیے گئے متن سے یہ ٹیگ ہٹا دیا جاتا ہے۔ دیگر چینلز اس ٹیگ کو نظرانداز کرتے ہیں۔

میسج ٹول کے ذریعے بھیجنے کے لیے، وائس کے مطابق آڈیو `media` URL کے ساتھ `asVoice: true` سیٹ کریں
(جب میڈیا موجود ہو تو `message` اختیاری ہے):

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## Video messages (video vs video note)

Telegram distinguishes **video notes** (round bubble) from **video files** (rectangular).
OpenClaw defaults to video files.

For message tool sends, set `asVideoNote: true` with a video `media` URL:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(Note: Video notes do not support captions. If you provide a message text, it will be sent as a separate message.)

## اسٹیکرز

OpenClaw ذہین کیشنگ کے ساتھ Telegram اسٹیکرز وصول اور بھیجنے کی سپورٹ کرتا ہے۔

### اسٹیکرز وصول کرنا

جب صارف اسٹیکر بھیجتا ہے تو OpenClaw اسے اسٹیکر کی قسم کے مطابق ہینڈل کرتا ہے:

- **اسٹیٹک اسٹیکرز (WEBP):** ڈاؤن لوڈ کیے جاتے ہیں اور وژن کے ذریعے پروسیس ہوتے ہیں۔ پیغام کے مواد میں اسٹیکر `<media:sticker>` پلیس ہولڈر کے طور پر ظاہر ہوتا ہے۔
- **اینیمیٹڈ اسٹیکرز (TGS):** نظرانداز (Lottie فارمیٹ پروسیسنگ کے لیے سپورٹڈ نہیں)۔
- **ویڈیو اسٹیکرز (WEBM):** نظرانداز (ویڈیو فارمیٹ پروسیسنگ کے لیے سپورٹڈ نہیں)۔

اسٹیکرز وصول کرتے وقت دستیاب ٹیمپلیٹ سیاق فیلڈ:

- `Sticker` — آبجیکٹ جس میں:
  - `emoji` — اسٹیکر سے وابستہ ایموجی
  - `setName` — اسٹیکر سیٹ کا نام
  - `fileId` — Telegram فائل ID (وہی اسٹیکر واپس بھیجنے کے لیے)
  - `fileUniqueId` — کیش لوک اپ کے لیے مستحکم ID
  - `cachedDescription` — دستیاب ہونے پر کیش شدہ وژن وضاحت

### اسٹیکر کیش

اسٹیکرز کو وضاحتیں بنانے کے لیے AI کی وژن صلاحیتوں کے ذریعے پروسیس کیا جاتا ہے۔ چونکہ ایک ہی اسٹیکرز اکثر بار بار بھیجے جاتے ہیں، OpenClaw غیر ضروری API کالز سے بچنے کے لیے ان وضاحتوں کو کیش کر لیتا ہے۔

**یہ کیسے کام کرتا ہے:**

1. **پہلی بار:** اسٹیکر کی تصویر وژن اینالیسس کے لیے AI کو بھیجی جاتی ہے۔ AI ایک وضاحت تیار کرتا ہے (مثلاً، "ایک کارٹون بلی جو پرجوش انداز میں ہاتھ ہلا رہی ہے")۔
2. **کیش میں ذخیرہ:** وضاحت اسٹیکر کی فائل ID، ایموجی، اور سیٹ نام کے ساتھ محفوظ کی جاتی ہے۔
3. **بعد کی باریں:** جب وہی اسٹیکر دوبارہ دیکھا جاتا ہے تو کیش شدہ وضاحت براہِ راست استعمال کی جاتی ہے۔ تصویر AI کو نہیں بھیجی جاتی۔

**کیش مقام:** `~/.openclaw/telegram/sticker-cache.json`

**کیش اندراج فارمیٹ:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**فوائد:**

- ایک ہی اسٹیکر کے لیے بار بار وژن کالز سے بچ کر API لاگت کم ہوتی ہے
- کیش شدہ اسٹیکرز کے لیے تیز ردِعمل (وژن پروسیسنگ میں تاخیر نہیں)
- کیش شدہ وضاحتوں کی بنیاد پر اسٹیکر سرچ کی سہولت

اسٹیکرز موصول ہونے کے ساتھ ہی کیش خودکار طور پر بھر جاتا ہے۔ کسی دستی کیش مینجمنٹ کی ضرورت نہیں ہوتی۔

### اسٹیکرز بھیجنا

ایجنٹ `sticker` اور `sticker-search` ایکشنز استعمال کر کے اسٹیکرز بھیج اور تلاش کر سکتا ہے۔ یہ بطور ڈیفالٹ غیر فعال ہوتے ہیں اور کنفیگ میں فعال کرنا ضروری ہے:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

**اسٹیکر بھیجیں:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

پیرامیٹرز:

- `fileId` (لازمی) — اسٹیکر کا ٹیلیگرام فائل آئی ڈی۔ اسے اسٹیکر موصول ہونے پر `Sticker.fileId` سے، یا `sticker-search` کے نتیجے سے حاصل کریں۔
- `replyTo` (اختیاری) — جواب دینے کے لیے پیغام ID۔
- `threadId` (اختیاری) — فورم ٹاپکس کے لیے میسج تھریڈ ID۔

**اسٹیکرز تلاش کریں:**

ایجنٹ وضاحت، ایموجی، یا سیٹ نام کے ذریعے کیش شدہ اسٹیکرز تلاش کر سکتا ہے:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

کیش سے ملتے جلتے اسٹیکرز واپس کرتا ہے:

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

سرچ وضاحت متن، ایموجی حروف، اور سیٹ ناموں پر فزی میچنگ استعمال کرتی ہے۔

**تھریڈنگ کے ساتھ مثال:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## اسٹریمنگ (ڈرافٹس)

ٹیلیگرام ایجنٹ کے جواب تیار کرنے کے دوران **ڈرافٹ ببلز** اسٹریمنگ کر سکتا ہے۔
OpenClaw Bot API کا `sendMessageDraft` استعمال کرتا ہے (حقیقی پیغامات نہیں) اور پھر حتمی جواب ایک عام پیغام کے طور پر بھیجتا ہے۔

ضروریات (Telegram Bot API 9.3+):

- **موضوعات فعال نجی چیٹس** (بوٹ کے لیے forum topic mode)۔
- آنے والے پیغامات میں `message_thread_id` شامل ہونا چاہیے (نجی ٹاپک تھریڈ)۔
- گروپس/سپرگروپس/چینلز کے لیے اسٹریمنگ نظرانداز کی جاتی ہے۔

کنفیگ:

- `channels.telegram.streamMode: "off" | "partial" | "block"` (ڈیفالٹ: `partial`)
  - `partial`: تازہ ترین اسٹریمنگ متن کے ساتھ ڈرافٹ ببل اپ ڈیٹ کریں۔
  - `block`: بڑے بلاکس میں ڈرافٹ ببل اپ ڈیٹ کریں (چنکڈ)۔
  - `off`: ڈرافٹ اسٹریمنگ غیر فعال کریں۔
- اختیاری (صرف `streamMode: "block"` کے لیے):
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference?
    }` نوٹ: ڈرافٹ اسٹریمنگ **بلاک اسٹریمنگ** (چینل پیغامات) سے الگ ہے۔
    - ڈیفالٹس: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (حد `channels.telegram.textChunkLimit` تک)۔

بلاک اسٹریمنگ بطور ڈیفالٹ بند ہوتی ہے اور اگر آپ ڈرافٹ اپڈیٹس کے بجائے ابتدائی ٹیلیگرام پیغامات چاہتے ہیں تو `channels.telegram.blockStreaming: true` درکار ہے۔
اگر `channels.telegram.streamMode` `off` ہو تو ریزننگ اسٹریمنگ غیر فعال ہوتی ہے۔

ریزننگ اسٹریم (صرف Telegram):

- `/reasoning stream` جواب تیار ہونے کے دوران ریزننگ کو ڈرافٹ ببل میں اسٹریم کرتا ہے،
  پھر ریزننگ کے بغیر حتمی جواب بھیجتا ہے۔
- If `channels.telegram.streamMode` is `off`, reasoning stream is disabled.
  مزید سیاق و سباق: [Streaming + chunking](/concepts/streaming).

## ری ٹرائی پالیسی

عارضی نیٹ ورک/429 غلطیوں پر آؤٹ باؤنڈ Telegram API کالز ایکسپونینشل بیک آف اور جِٹر کے ساتھ دوبارہ کوشش کرتی ہیں۔ `channels.telegram.retry` کے ذریعے کنفیگر کریں۔ [Retry policy](/concepts/retry) دیکھیں۔

## ایجنٹ ٹول (پیغامات + ری ایکشنز)

- ٹول: `telegram` مع `sendMessage` ایکشن (`to`, `content`, اختیاری `mediaUrl`, `replyToMessageId`, `messageThreadId`)۔
- ٹول: `telegram` مع `react` ایکشن (`chatId`, `messageId`, `emoji`)۔
- ٹول: `telegram` مع `deleteMessage` ایکشن (`chatId`, `messageId`)۔
- ری ایکشن ہٹانے کی معنویت: دیکھیں [/tools/reactions](/tools/reactions)۔
- ٹول گیٹنگ: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (ڈیفالٹ: فعال)، اور `channels.telegram.actions.sticker` (ڈیفالٹ: غیرفعال)۔

## ری ایکشن نوٹیفکیشنز

**ری ایکشنز کیسے کام کرتے ہیں:**
Telegram ری ایکشنز **الگ `message_reaction` ایونٹس کے طور پر آتے ہیں**، نہ کہ میسج پے لوڈ کی پراپرٹیز کے طور پر۔ جب کوئی صارف ری ایکشن شامل کرتا ہے، OpenClaw:

1. Telegram API سے `message_reaction` اپ ڈیٹ وصول کرتا ہے
2. اسے اس فارمیٹ میں **سسٹم ایونٹ** میں تبدیل کرتا ہے: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. اسی **سیشن کی** کے ساتھ سسٹم ایونٹ کو قطار میں ڈالتا ہے جو عام پیغامات کے لیے ہوتی ہے
4. جب اس گفتگو میں اگلا پیغام آتا ہے تو سسٹم ایونٹس نکال کر ایجنٹ کے سیاق کے آغاز میں شامل کر دیے جاتے ہیں

ایجنٹ ری ایکشنز کو گفتگو کی تاریخ میں **سسٹم نوٹیفکیشنز** کے طور پر دیکھتا ہے، پیغام میٹاڈیٹا کے طور پر نہیں۔

**کنفیگریشن:**

- `channels.telegram.reactionNotifications`: کنٹرول کرتا ہے کہ کون سے ری ایکشنز نوٹیفکیشنز ٹرگر کریں
  - `"off"` — تمام ری ایکشنز نظرانداز کریں
  - `"own"` — جب صارفین بوٹ پیغامات پر ری ایکٹ کریں تو مطلع کریں (بہترین کوشش؛ اِن-میموری) (ڈیفالٹ)
  - `"all"` — تمام ری ایکشنز کے لیے مطلع کریں

- `channels.telegram.reactionLevel`: ایجنٹ کی ری ایکشن صلاحیت کنٹرول کرتا ہے
  - `"off"` — ایجنٹ پیغامات پر ری ایکٹ نہیں کر سکتا
  - `"ack"` — بوٹ توثیقی ری ایکشنز بھیجتا ہے (👀 پروسیسنگ کے دوران) (ڈیفالٹ)
  - `"minimal"` — ایجنٹ محدود طور پر ری ایکٹ کر سکتا ہے (رہنمائی: ہر 5–10 تبادلوں میں 1)
  - `"extensive"` — ایجنٹ مناسب مواقع پر فراخدلی سے ری ایکٹ کر سکتا ہے

**فورم گروپس:** فورم گروپس میں ری ایکشنز میں `message_thread_id` شامل ہوتا ہے اور سیشن کیز جیسے `agent:main:telegram:group:{chatId}:topic:{threadId}` استعمال ہوتی ہیں۔ اس سے یہ یقینی بنتا ہے کہ ایک ہی موضوع میں ری ایکشنز اور پیغامات اکٹھے رہیں۔

**مثالی کنفیگ:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**ضروریات:**

- Telegram بوٹس کو `allowed_updates` میں واضح طور پر `message_reaction` کی درخواست کرنا ہوتی ہے (OpenClaw خودکار طور پر کنفیگر کرتا ہے)
- ویب ہوک موڈ کے لیے، ری ایکشنز ویب ہوک `allowed_updates` میں شامل ہوتے ہیں
- پولنگ موڈ کے لیے، ری ایکشنز `getUpdates` `allowed_updates` میں شامل ہوتے ہیں

## ڈیلیوری اہداف (CLI/cron)

- ہدف کے طور پر چیٹ ID (`123456789`) یا یوزرنیم (`@name`) استعمال کریں۔
- مثال: `openclaw message send --channel telegram --target 123456789 --message "hi"`۔

## خرابیوں کا ازالہ

**بوٹ گروپ میں غیر مینشن پیغامات پر جواب نہیں دیتا:**

- اگر آپ نے `channels.telegram.groups.*.requireMention=false` سیٹ کیا ہے تو Telegram Bot API کا **پرائیویسی موڈ** غیر فعال ہونا چاہیے۔
  - BotFather: `/setprivacy` → **Disable** (پھر بوٹ کو گروپ سے ہٹا کر دوبارہ شامل کریں)
- `openclaw channels status` اس وقت وارننگ دکھاتا ہے جب کنفیگ غیر مینشن گروپ پیغامات کی توقع کرے۔
- `openclaw channels status --probe` صریح عددی گروپ IDs کے لیے رکنیت بھی چیک کر سکتا ہے (وائلڈ کارڈ `"*"` قواعد کا آڈٹ نہیں کر سکتا)۔
- فوری ٹیسٹ: `/activation always` (صرف سیشن؛ مستقل کے لیے کنفیگ استعمال کریں)

**بوٹ گروپ پیغامات بالکل نہیں دیکھ رہا:**

- اگر `channels.telegram.groups` سیٹ ہے تو گروپ کو فہرست میں ہونا چاہیے یا `"*"` استعمال کرے
- @BotFather میں پرائیویسی سیٹنگز چیک کریں → "Group Privacy" **OFF** ہونا چاہیے
- تصدیق کریں کہ بوٹ واقعی رکن ہے (صرف ایڈمن نہیں جس کے پاس ریڈ رسائی نہ ہو)
- گیٹ وے لاگز چیک کریں: `openclaw logs --follow` ("skipping group message" تلاش کریں)

**بوٹ مینشنز پر جواب دیتا ہے مگر `/activation always` پر نہیں:**

- `/activation` کمانڈ سیشن اسٹیٹ اپ ڈیٹ کرتی ہے مگر کنفیگ میں محفوظ نہیں ہوتی
- مستقل رویے کے لیے گروپ کو `channels.telegram.groups` میں `requireMention: false` کے ساتھ شامل کریں

**`/status` جیسی کمانڈز کام نہیں کرتیں:**

- یقینی بنائیں کہ آپ کا Telegram یوزر ID مجاز ہے (pairing یا `channels.telegram.allowFrom` کے ذریعے)
- کمانڈز کو اجازت درکار ہوتی ہے حتیٰ کہ گروپس میں `groupPolicy: "open"` کے ساتھ بھی

**Node 22+ پر لانگ پولنگ فوراً ختم ہو جاتی ہے (اکثر پراکسیز/کسٹم fetch کے ساتھ):**

- Node 22+ `AbortSignal` انسٹینسز کے بارے میں زیادہ سخت ہے؛ غیر ملکی سگنلز فوراً `fetch` کالز کو ختم کر سکتے ہیں۔
- ایسے OpenClaw بلڈ پر اپ گریڈ کریں جو abort سگنلز کو نارملائز کرتا ہو، یا اپ گریڈ تک گیٹ وے Node 20 پر چلائیں۔

**بوٹ شروع ہوتا ہے، پھر خاموشی سے جواب دینا بند کر دیتا ہے (یا `HttpError: Network request ...` لاگ کرتا ہے ناکام ہوا\`):**

- کچھ ہوسٹس پہلے `api.telegram.org` کو IPv6 پر ریزولو کرتے ہیں۔ اگر آپ کے سرور میں IPv6 ایگریس درست طور پر کام نہیں کرتا، تو grammY IPv6-only درخواستوں پر اَٹک سکتا ہے۔
- حل: IPv6 ایگریس فعال کریں **یا** `api.telegram.org` کے لیے IPv4 ریزولوشن مجبور کریں (مثلاً IPv4 A ریکارڈ کے ساتھ `/etc/hosts` اندراج شامل کریں، یا OS DNS اسٹیک میں IPv4 کو ترجیح دیں)، پھر گیٹ وے ری اسٹارٹ کریں۔
- فوری جانچ: `dig +short api.telegram.org A` اور `dig +short api.telegram.org AAAA` دیکھیں کہ DNS کیا واپس دیتا ہے۔

## کنفیگریشن حوالہ (Telegram)

مکمل کنفیگریشن: [Configuration](/gateway/configuration)

فراہم کنندہ آپشنز:

- `channels.telegram.enabled`: چینل اسٹارٹ اپ فعال/غیرفعال کریں۔
- `channels.telegram.botToken`: بوٹ ٹوکن (BotFather)۔
- `channels.telegram.tokenFile`: فائل پاتھ سے ٹوکن پڑھیں۔
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (ڈیفالٹ: pairing)۔
- `channels.telegram.allowFrom`: DM اجازت فہرست (ids/usernames)۔ `open` کے لیے `"*"` درکار ہے۔
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (ڈیفالٹ: allowlist)۔
- `channels.telegram.groupAllowFrom`: گروپ سینڈر اجازت فہرست (IDs/یوزرنیمز)۔
- `channels.telegram.groups`: فی گروپ ڈیفالٹس + اجازت فہرست (عالمی ڈیفالٹس کے لیے `"*"` استعمال کریں)۔
  - `channels.telegram.groups.<id>`channels.telegram.groups.&lt;id&gt;
    .requireMention\`: مینشن گیٹنگ کی ڈیفالٹ۔
  - `channels.telegram.groups.<id>
    .groupPolicy`: گروپ کے لیے groupPolicy اووررائیڈ (`open | allowlist | disabled`)۔`channels.telegram.groups.&lt;id&gt;
    .allowFrom`: فی گروپ بھیجنے والے کی اجازت فہرست کا اووررائیڈ۔
  - `channels.telegram.groups.<id>`channels.telegram.groups.&lt;id&gt;
    .enabled`: جب `false\` ہو تو گروپ کو غیر فعال کریں۔
  - `channels.telegram.groups.<id>`channels.telegram.groups.&lt;id&gt;
    .topics.&lt;threadId&gt;
    .groupPolicy`: groupPolicy کے لیے فی موضوع اووررائیڈ (`open | allowlist | disabled\`)۔
  - `channels.telegram.groups.<id>.systemPrompt`: گروپ کے لیے اضافی سسٹم پرامپٹ۔
  - `channels.telegram.groups.<id>
    .topics.&lt;threadId&gt;
    .requireMention`: فی موضوع مینشن گیٹنگ اووررائیڈ۔Happy Eyeballs ٹائم آؤٹس سے بچنے کے لیے Node 22 پر ڈیفالٹ طور پر غیر فعال ہے۔
  - `channels.telegram.network.autoSelectFamily`: Node کے autoSelectFamily کو اووررائیڈ کریں (true=فعال، false=غیر فعال)۔`channels.telegram.commands.native` کے ساتھ اووررائیڈ کریں۔Tlon ایک غیر مرکزی میسنجر ہے جو Urbit پر بنایا گیا ہے۔
  - `commands.native` (ڈیفالٹ `"auto"` → Telegram/Discord کے لیے آن، Slack کے لیے آف)، `commands.text`, `commands.useAccessGroups` (کمانڈ رویّہ)۔گروپ جوابات کے لیے ڈیفالٹ طور پر @ مینشن درکار ہوتا ہے اور انہیں اجازت فہرستوں کے ذریعے مزید محدود کیا جا سکتا ہے۔اسٹیٹس: پلگ اِن کے ذریعے سپورٹڈ۔
  - DMs، گروپ مینشنز، تھریڈ ریپلائیز، اور صرف متن والا میڈیا فال بیک
    (کیپشن کے ساتھ URL شامل کیا جاتا ہے)۔ری ایکشنز، پولز، اور نیٹو میڈیا اپ لوڈز سپورٹڈ نہیں ہیں۔آٹو ڈسکوری ڈیفالٹ طور پر فعال ہے۔
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (ڈیفالٹ: allowlist)۔
- آپ چینلز کو دستی طور پر بھی پن کر سکتے ہیں:IRC کنکشن کے ذریعے Twitch چیٹ سپورٹ۔
- `channels.telegram.replyToMode`: `off | first | all` (ڈیفالٹ: `first`)۔
- `channels.telegram.textChunkLimit`: آؤٹ باؤنڈ چنک سائز (حروف)۔
- `channels.telegram.chunkMode`: `length` (ڈیفالٹ) یا `newline` تاکہ لمبائی چنکنگ سے پہلے خالی لائنوں (پیراگراف حدود) پر تقسیم ہو۔
- `channels.telegram.linkPreview`: آؤٹ باؤنڈ پیغامات کے لیے لنک پری ویوز ٹوگل کریں (ڈیفالٹ: true)۔
- `channels.telegram.streamMode`: `off | partial | block` (ڈرافٹ اسٹریمنگ)۔
- `channels.telegram.mediaMaxMb`: اِن باؤنڈ/آؤٹ باؤنڈ میڈیا حد (MB)۔
- `channels.telegram.retry`: آؤٹ باؤنڈ Telegram API کالز کے لیے ری ٹرائی پالیسی (کوششیں، minDelayMs، maxDelayMs، jitter)۔
- `channels.telegram.network.autoSelectFamily`: Node کے autoSelectFamily کو اووررائیڈ کریں (true=فعال کریں، false=غیر فعال کریں)۔ Node 22 پر ڈیفالٹ طور پر غیر فعال ہوتا ہے تاکہ Happy Eyeballs ٹائم آؤٹس سے بچا جا سکے۔
- `channels.telegram.proxy`: Bot API کالز کے لیے پراکسی URL (SOCKS/HTTP)۔
- `channels.telegram.webhookUrl`: ویب ہوک موڈ فعال کریں (درکار: `channels.telegram.webhookSecret`)۔
- `channels.telegram.webhookSecret`: ویب ہوک سیکرٹ (جب webhookUrl سیٹ ہو تو درکار)۔
- `channels.telegram.webhookPath`: لوکل ویب ہوک پاتھ (ڈیفالٹ `/telegram-webhook`)۔
- `channels.telegram.actions.reactions`: Telegram ٹول ری ایکشنز گیٹ کریں۔
- `channels.telegram.actions.sendMessage`: Telegram ٹول پیغام بھیجنا گیٹ کریں۔
- `channels.telegram.actions.deleteMessage`: Telegram ٹول پیغام حذف کرنا گیٹ کریں۔
- `channels.telegram.actions.sticker`: Telegram اسٹیکر ایکشنز — بھیجنا اور تلاش کرنا — گیٹ کریں (ڈیفالٹ: false)۔
- `channels.telegram.reactionNotifications`: `off | own | all` — کنٹرول کریں کہ کون سے ری ایکشنز سسٹم ایونٹس ٹرگر کریں (جب سیٹ نہ ہو تو ڈیفالٹ: `own`)۔
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — ایجنٹ کی ری ایکشن صلاحیت کنٹرول کریں (جب سیٹ نہ ہو تو ڈیفالٹ: `minimal`)۔

متعلقہ عالمی آپشنز:

- `agents.list[].groupChat.mentionPatterns` (مینشن گیٹنگ پیٹرنز)۔
- `messages.groupChat.mentionPatterns` (عالمی فال بیک)۔
- `commands.native` (defaults to `"auto"` → on for Telegram/Discord, off for Slack), `commands.text`, `commands.useAccessGroups` (command behavior). Override with `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`۔



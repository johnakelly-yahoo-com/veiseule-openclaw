---
summary: "Telegram بوٹ کی معاونت کی حالت، صلاحیتیں، اور کنفیگریشن"
read_when:
  - Telegram خصوصیات یا ویب ہوکس پر کام کرتے وقت
title: "Telegram"
---

# Telegram (Bot API)

Status: production-ready for bot DMs + groups via grammY. Long polling ڈیفالٹ موڈ ہے؛ webhook موڈ اختیاری ہے۔

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Telegram کے لیے ڈیفالٹ DM پالیسی pairing ہے۔
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">کراس چینل ڈائگناسٹکس اور مرمتی رہنمائی۔
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">مکمل چینل کنفیگ پیٹرنز اور مثالیں۔
</Card>
</CardGroup>

## فوری سیٹ اپ

<Steps>
  <Step title="Create the bot token in BotFather">Telegram کھولیں اور **@BotFather** کے ساتھ چیٹ کریں (یقین کریں کہ ہینڈل بالکل `@BotFather` ہی ہو)۔

    ```
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

  
</Step>

  <Step title="Configure token and DM policy">

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

    ```
    Env fallback: `TELEGRAM_BOT_TOKEN=...` (صرف ڈیفالٹ اکاؤنٹ کے لیے)۔
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Pairing کوڈز 1 گھنٹے کے بعد میعاد ختم ہو جاتے ہیں۔
    ```

  
</Step>

  <Step title="Add the bot to a group">بوٹ کو اپنے گروپ میں شامل کریں، پھر `channels.telegram.groups` اور `groupPolicy` کو اپنے رسائی ماڈل کے مطابق سیٹ کریں۔
</Step>
</Steps>

<Note>
Token resolution کی ترتیب اکاؤنٹ کے لحاظ سے ہوتی ہے۔ عملی طور پر، کنفیگ ویلیوز env fallback پر ترجیح رکھتی ہیں، اور `TELEGRAM_BOT_TOKEN` صرف ڈیفالٹ اکاؤنٹ پر لاگو ہوتا ہے۔
</Note>

## Telegram سائیڈ کی ترتیبات

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">
    Telegram بوٹس میں بطورِ ڈیفالٹ **Privacy Mode** فعال ہوتا ہے، جو انہیں گروپس کے کون سے پیغامات موصول ہوں گے اس کو محدود کرتا ہے۔

    ```
    اگر بوٹ کو گروپ کے تمام پیغامات دیکھنے ہوں تو درج ذیل میں سے ایک کریں:
    
    - `/setprivacy` کے ذریعے privacy mode غیر فعال کریں، یا
    - بوٹ کو گروپ ایڈمن بنا دیں۔
    
    جب آپ privacy mode تبدیل کریں تو ہر گروپ میں بوٹ کو ہٹا کر دوبارہ شامل کریں تاکہ Telegram تبدیلی لاگو کر سکے۔
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    ایڈمن اسٹیٹس کو Telegram گروپ سیٹنگز میں کنٹرول کیا جاتا ہے۔

    ```
    ایڈمن بوٹس کو گروپ کے تمام پیغامات موصول ہوتے ہیں، جو ہمیشہ فعال گروپ رویے کے لیے مفید ہے۔
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - گروپ میں شامل کرنے کی اجازت دینے یا روکنے کے لیے `/setjoingroups`
    - گروپ میں نظر آنے کے رویے کے لیے `/setprivacy`
    ```

  
</Accordion>
</AccordionGroup>

## رسائی کنٹرول اور ایکٹیویشن

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` ڈائریکٹ میسج تک رسائی کو کنٹرول کرتا ہے:

    ```
    - `pairing` (ڈیفالٹ)
    - `allowlist`
    - `open` (اس کے لیے `allowFrom` میں `"*"` شامل ہونا ضروری ہے)
    - `disabled`
    
    `channels.telegram.allowFrom` عددی Telegram یوزر IDs قبول کرتا ہے۔ `telegram:` / `tg:` سابقے قبول کیے جاتے ہیں اور نارملائز کر دیے جاتے ہیں۔
    آن بورڈنگ وزرڈ `@username` ان پٹ قبول کرتا ہے اور اسے عددی IDs میں تبدیل کرتا ہے۔
    اگر آپ نے اپگریڈ کیا ہے اور آپ کی کنفیگ میں `@username` allowlist اندراجات موجود ہیں تو انہیں حل کرنے کے لیے `openclaw doctor --fix` چلائیں (بہترین کوشش؛ Telegram بوٹ ٹوکن درکار ہے)۔
    
    ### اپنا Telegram یوزر ID معلوم کرنا
    
    زیادہ محفوظ طریقہ (بغیر کسی تھرڈ پارٹی بوٹ کے):
    
    1. اپنے بوٹ کو DM بھیجیں۔
    2. `openclaw logs --follow` چلائیں۔
    3. `from.id` پڑھیں۔
    
    آفیشل Bot API طریقہ:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    تھرڈ پارٹی طریقہ (کم نجی): `@userinfobot` یا `@getidsbot`۔
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    دو آزاد کنٹرولز ہیں:

    ```
    1. **کون سے گروپس کی اجازت ہے** (`channels.telegram.groups`)
       - اگر `groups` کنفیگ موجود نہ ہو: تمام گروپس کی اجازت ہے
       - اگر `groups` کنفیگر ہو: یہ allowlist کے طور پر کام کرتا ہے (واضح IDs یا `"*"`)
    
    2. **گروپس میں کون سے بھیجنے والے مجاز ہیں** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (ڈیفالٹ)
       - `disabled`
    
    گروپ بھیجنے والوں کی فلٹرنگ کے لیے `groupAllowFrom` استعمال ہوتا ہے۔ اگر یہ سیٹ نہ ہو تو Telegram، `allowFrom` پر واپس چلا جاتا ہے۔
    `groupAllowFrom` اندراجات عددی Telegram یوزر IDs ہونے چاہئیں۔
    
    مثال: کسی مخصوص گروپ میں کسی بھی رکن کو اجازت دینا:
    ```

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

  
</Tab>

  <Tab title="Mention behavior">
    بطورِ ڈیفالٹ گروپ جوابات کے لیے mention ضروری ہے۔

    ```
    mention درج ذیل میں سے کسی ایک طریقے سے ہو سکتی ہے:
    
    - مقامی `@botusername` mention، یا
    - ان میں موجود mention patterns:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    سیشن لیول کمانڈ ٹوگلز:
    
    - `/activation always`
    - `/activation mention`
    
    یہ صرف سیشن اسٹیٹ کو اپڈیٹ کرتے ہیں۔ مستقل مزاجی کے لیے کنفیگ استعمال کریں۔
    
    مستقل کنفیگ کی مثال:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    ```
    گروپ چیٹ ID حاصل کرنا:
    
    - کسی گروپ میسج کو `@userinfobot` / `@getidsbot` پر فارورڈ کریں
    - یا `openclaw logs --follow` سے `chat.id` پڑھیں
    - یا Bot API کے `getUpdates` کا جائزہ لیں
    ```

  
</Tab>
</Tabs>

## رن ٹائم رویہ

- Telegram کو gateway پروسیس چلاتا ہے۔
- روٹنگ متعین (deterministic) ہے: Telegram سے آنے والے پیغامات کا جواب دوبارہ Telegram پر ہی جاتا ہے (ماڈل چینلز منتخب نہیں کرتا)۔
- آنے والے پیغامات کو مشترکہ چینل envelope میں reply metadata اور میڈیا پلیس ہولڈرز کے ساتھ نارملائز کیا جاتا ہے۔
- گروپ سیشنز کو گروپ ID کے مطابق الگ رکھا جاتا ہے۔ فورم ٹاپکس کو الگ رکھنے کے لیے `:topic:<threadId>` شامل کیا جاتا ہے۔
- DM پیغامات میں `message_thread_id` شامل ہو سکتا ہے؛ OpenClaw انہیں thread-aware سیشن کیز کے ساتھ روٹ کرتا ہے اور جوابات کے لیے thread ID برقرار رکھتا ہے۔
- لانگ پولنگ grammY runner کے ذریعے فی چیٹ/فی تھریڈ ترتیب کے ساتھ استعمال ہوتی ہے۔ مجموعی runner sink concurrency میں `agents.defaults.maxConcurrent` استعمال ہوتا ہے۔
- Telegram Bot API میں read-receipt کی سپورٹ نہیں ہے (`sendReadReceipts` لاگو نہیں ہوتا)۔

## فیچر ریفرنس

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">
    OpenClaw عارضی Telegram میسج بھیج کر اور متن آتے ہی اسے ایڈٹ کر کے جزوی جوابات اسٹریم کر سکتا ہے۔

    ```
    ضرورت:
    
    - `channels.telegram.streamMode` کی قدر `"off"` نہ ہو (ڈیفالٹ: `"partial"`)
    
    موڈز:
    
    - `off`: کوئی لائیو پریویو نہیں
    - `partial`: جزوی متن سے بار بار پریویو اپڈیٹس
    - `block`: `channels.telegram.draftChunk` استعمال کرتے ہوئے ٹکڑوں میں پریویو اپڈیٹس
    
    `streamMode: "block"` کے لیے `draftChunk` کی ڈیفالٹس:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` کو `channels.telegram.textChunkLimit` کے مطابق محدود (clamp) کیا جاتا ہے۔
    
    یہ ڈائریکٹ چیٹس اور گروپس/ٹاپکس دونوں میں کام کرتا ہے۔
    
    صرف متن والے جوابات کے لیے، OpenClaw اسی پریویو میسج کو برقرار رکھتا ہے اور آخر میں اسی جگہ فائنل ایڈٹ کرتا ہے (دوسرا میسج نہیں بھیجتا)۔
    
    پیچیدہ جوابات (مثلاً میڈیا payloads) کی صورت میں، OpenClaw عام فائنل ڈلیوری پر واپس آتا ہے اور پھر پریویو میسج صاف کر دیتا ہے۔
    
    `streamMode` بلاک اسٹریمنگ سے الگ ہے۔ جب Telegram کے لیے بلاک اسٹریمنگ واضح طور پر فعال ہو تو OpenClaw ڈبل اسٹریمنگ سے بچنے کے لیے پریویو اسٹریم کو چھوڑ دیتا ہے۔
    
    Telegram-صرف reasoning اسٹریم:
    
    - `/reasoning stream` جنریشن کے دوران reasoning کو لائیو پریویو میں بھیجتا ہے
    - حتمی جواب reasoning متن کے بغیر بھیجا جاتا ہے
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">
    آؤٹ باؤنڈ متن Telegram `parse_mode: "HTML"` استعمال کرتا ہے۔

    ```
    - Markdown جیسے متن کو Telegram کے لیے محفوظ HTML میں رینڈر کیا جاتا ہے۔
    - خام ماڈل HTML کو Telegram parse کی ناکامی کم کرنے کے لیے escape کیا جاتا ہے۔
    - اگر Telegram پارس شدہ HTML مسترد کر دے تو OpenClaw سادہ متن کے طور پر دوبارہ کوشش کرتا ہے۔
    
    لنک پریویوز بطورِ ڈیفالٹ فعال ہیں اور `channels.telegram.linkPreview: false` سے غیر فعال کیے جا سکتے ہیں۔
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Telegram کمانڈ مینو کی رجسٹریشن اسٹارٹ اپ پر `setMyCommands` کے ذریعے کی جاتی ہے۔

    ```
    مقامی کمانڈ ڈیفالٹس:
    
    - `commands.native: "auto"` Telegram کے لیے مقامی کمانڈز فعال کرتا ہے
    
    کسٹم کمانڈ مینو اندراجات شامل کریں:
    ```

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

    ```
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

  
</Accordion>

  <Accordion title="Inline buttons">
    inline کی بورڈ اسکوپ کنفیگر کریں:

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

    ```
    فی اکاؤنٹ اووررائیڈ:
    ```

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

    ```
    اسکوپس:
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (ڈیفالٹ)
    
    پرانی `capabilities: ["inlineButtons"]` میپ ہو کر `inlineButtons: "all"` بن جاتی ہے۔
    ```

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

    ```
    میسج ایکشن کی مثال:
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">Callback کلکس بطور متن ایجنٹ کو بھیجے جاتے ہیں:
`callback_data: <value>`

    ```
    - `sendMessage` (`to`, `content`, اختیاری `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    چینل میسج ایکشنز آسان عرفی نام فراہم کرتے ہیں (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`)۔
    
    Gating controls:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (ڈیفالٹ: غیر فعال)
    
    Reaction ہٹانے کے اصول: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram تیار کردہ آؤٹ پٹ میں واضح reply threading ٹیگز کی حمایت کرتا ہے:

    ```
    - `[[reply_to_current]]` ٹرگر کرنے والے پیغام کا جواب دیتا ہے
    - `[[reply_to:<id>]]` مخصوص Telegram message ID کا جواب دیتا ہے
    
    `channels.telegram.replyToMode` ہینڈلنگ کو کنٹرول کرتا ہے:
    
    - `off` (ڈیفالٹ)
    - `first`
    - `all`
    
    نوٹ: `off` ضمنی reply threading کو غیر فعال کرتا ہے۔ واضح `[[reply_to_*]]` ٹیگز پھر بھی قابلِ قبول ہوتے ہیں۔
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Forum supergroups:

    ```
    - topic session keys میں `:topic:<threadId>` شامل کیا جاتا ہے
    - جوابات اور typing متعلقہ topic thread کو ہدف بناتے ہیں
    - topic config path:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    General topic (`threadId=1`) خاص صورت:
    
    - پیغام بھیجتے وقت `message_thread_id` شامل نہیں کیا جاتا (Telegram `sendMessage(...thread_id=1)` کو مسترد کرتا ہے)
    - typing ایکشنز میں پھر بھی `message_thread_id` شامل ہوتا ہے
    
    Topic inheritance: topic اندراجات گروپ سیٹنگز کو وراثت میں لیتے ہیں جب تک اووررائیڈ نہ کیا جائے (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`)۔
    
    Template context میں شامل ہیں:
    
    - `MessageThreadId`
    - `IsForum`
    
    DM thread رویہ:
    
    - نجی چیٹس جن میں `message_thread_id` ہو، DM روٹنگ برقرار رکھتے ہیں لیکن thread-aware session keys/جوابی اہداف استعمال کرتے ہیں۔
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">### Audio messages

    ```
    Telegram voice notes اور audio files میں فرق کرتا ہے۔
    
    - ڈیفالٹ: audio file کا رویہ
    - ایجنٹ کے جواب میں `[[audio_as_voice]]` ٹیگ شامل کرنے سے voice-note کے طور پر بھیجنے پر مجبور کیا جاتا ہے
    
    Message action مثال:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### Video messages
    
    Telegram video files اور video notes میں فرق کرتا ہے۔
    
    Message action مثال:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Video notes captions کی حمایت نہیں کرتے؛ فراہم کردہ پیغام کا متن الگ سے بھیجا جاتا ہے۔
    
    ### Stickers
    
    موصولہ sticker ہینڈلنگ:
    
    - static WEBP: ڈاؤن لوڈ اور پراسیس کیا جاتا ہے (placeholder `<media:sticker>`)
    - animated TGS: نظرانداز
    - video WEBM: نظرانداز
    
    Sticker context فیلڈز:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Sticker cache فائل:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Stickers کی وضاحت ایک بار (جب ممکن ہو) کی جاتی ہے اور بار بار vision کالز کم کرنے کے لیے cache کیا جاتا ہے۔
    
    Sticker ایکشنز فعال کریں:
    ```

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

    ```
    Send sticker action:
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Search cached stickers:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Telegram reactions `message_reaction` اپڈیٹس کے طور پر موصول ہوتے ہیں (message payloads سے الگ)۔

    ```
    فعال ہونے پر، OpenClaw درج ذیل جیسے system events کو قطار میں شامل کرتا ہے:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Config:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (ڈیفالٹ: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (ڈیفالٹ: `minimal`)
    
    نوٹس:
    
    - `own` سے مراد صرف bot کے بھیجے گئے پیغامات پر صارف کے reactions ہیں (بہتر کوشش sent-message cache کے ذریعے)۔
    - Telegram reaction اپڈیٹس میں thread IDs فراہم نہیں کرتا۔
      - non-forum گروپس گروپ چیٹ سیشن کی طرف روٹ ہوتے ہیں
      - forum گروپس گروپ کے general-topic سیشن (`:topic:1`) کی طرف روٹ ہوتے ہیں، اصل topic کی طرف نہیں
    
    polling/webhook کے لیے `allowed_updates` میں `message_reaction` خودکار طور پر شامل ہوتا ہے۔
    ```

  
</Accordion>

  <Accordion title="Ack reactions">`ackReaction` اس وقت ایک acknowledgement emoji بھیجتا ہے جب OpenClaw موصولہ پیغام کو پراسیس کر رہا ہو۔

    ```
    Resolution ترتیب:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, بصورت دیگر "👀")
    
    نوٹس:
    
    - Telegram کو unicode emoji درکار ہوتا ہے (مثال کے طور پر "👀")۔
    - کسی چینل یا اکاؤنٹ کے لیے reaction غیر فعال کرنے کو `""` استعمال کریں۔
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">Channel config writes ڈیفالٹ طور پر فعال ہوتے ہیں (`configWrites !== false`)۔

    ```
    Telegram سے متحرک ہونے والی writes میں شامل ہیں:
    
    - گروپ مائیگریشن ایونٹس (`migrate_to_chat_id`) تاکہ `channels.telegram.groups` اپڈیٹ ہو سکے
    - `/config set` اور `/config unset` (کمانڈ فعال ہونا ضروری ہے)
    
    غیر فعال کریں:
    ```

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">ڈیفالٹ: long polling۔

    ```
    Webhook موڈ:
    
    - `channels.telegram.webhookUrl` سیٹ کریں
    - `channels.telegram.webhookSecret` سیٹ کریں (جب webhook URL سیٹ ہو تو لازمی)
    - اختیاری `channels.telegram.webhookPath` (ڈیفالٹ `/telegram-webhook`)
    - اختیاری `channels.telegram.webhookHost` (ڈیفالٹ `127.0.0.1`)
    
    Webhook موڈ کے لیے ڈیفالٹ مقامی listener `127.0.0.1:8787` پر bind ہوتا ہے۔
    
    اگر آپ کا public endpoint مختلف ہے تو سامنے reverse proxy رکھیں اور `webhookUrl` کو public URL پر پوائنٹ کریں۔
    جب آپ جان بوجھ کر بیرونی ingress درکار ہو تو `webhookHost` (مثلاً `0.0.0.0`) سیٹ کریں۔
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - `channels.telegram.textChunkLimit` کا ڈیفالٹ 4000 ہے۔
    - `channels.telegram.chunkMode="newline"` لمبائی کے مطابق تقسیم سے پہلے پیراگراف حدود (خالی لائنیں) کو ترجیح دیتا ہے۔
    - `channels.telegram.mediaMaxMb` (ڈیفالٹ 5) موصولہ Telegram میڈیا کی ڈاؤن لوڈ/پراسیسنگ سائز کی حد مقرر کرتا ہے۔
    - `channels.telegram.timeoutSeconds` Telegram API کلائنٹ timeout کو اووررائیڈ کرتا ہے (اگر سیٹ نہ ہو تو grammY کا ڈیفالٹ لاگو ہوگا)۔
    - گروپ context ہسٹری `channels.telegram.historyLimit` یا `messages.groupChat.historyLimit` استعمال کرتی ہے (ڈیفالٹ 50)؛ `0` اسے غیر فعال کرتا ہے۔
    - DM ہسٹری کنٹرولز:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - outbound Telegram API retries کو `channels.telegram.retry` کے ذریعے کنفیگر کیا جا سکتا ہے۔

    ```
    CLI send target عددی chat ID یا username ہو سکتا ہے:
    ```

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  
</Accordion>
</AccordionGroup>

## خرابیوں کا ازالہ

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - اگر `requireMention=false` ہو تو Telegram privacy mode مکمل مرئیت کی اجازت دے۔
      - BotFather: `/setprivacy` -> Disable
      - پھر bot کو گروپ سے ہٹا کر دوبارہ شامل کریں
    - `openclaw channels status` اس وقت وارننگ دیتا ہے جب config بغیر ذکر کے گروپ پیغامات کی توقع کرے۔
    - `openclaw channels status --probe` واضح عددی گروپ IDs کو چیک کر سکتا ہے؛ وائلڈ کارڈ `"*"` کی رکنیت probe نہیں کی جا سکتی۔
    - فوری سیشن ٹیسٹ: `/activation always`۔
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - جب `channels.telegram.groups` موجود ہو تو گروپ کی فہرست میں شامل ہونا ضروری ہے (یا `"*"` شامل کریں)
    - گروپ میں bot کی رکنیت کی تصدیق کریں
    - skip وجوہات کے لیے لاگز دیکھیں: `openclaw logs --follow`
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - اپنی sender identity کو مجاز بنائیں (pairing اور/یا عددی `allowFrom`)
    - کمانڈ کی اجازتیں اس وقت بھی لاگو رہتی ہیں جب group policy `open` ہو
    - `setMyCommands failed` عموماً `api.telegram.org` تک DNS/HTTPS رسائی کے مسائل کی نشاندہی کرتا ہے
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + کسٹم fetch/proxy میں اگر AbortSignal اقسام میل نہ کھائیں تو فوری abort رویہ پیدا ہو سکتا ہے۔
    - کچھ ہوسٹس `api.telegram.org` کو پہلے IPv6 پر resolve کرتے ہیں؛ خراب IPv6 egress وقفے وقفے سے Telegram API ناکامیوں کا سبب بن سکتا ہے۔
    - DNS جوابات کی توثیق کریں:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Telegram ٹیگز کے ذریعے اختیاری تھریڈڈ جوابات سپورٹ کرتا ہے:

## Telegram config حوالہ پوائنٹرز

`channels.telegram.replyToMode` کے ذریعے کنٹرول:

- `first` (ڈیفالٹ)، `all`, `off`۔

- `channels.telegram.botToken`: بوٹ ٹوکن (BotFather)۔

- `channels.telegram.tokenFile`: فائل پاتھ سے ٹوکن پڑھیں۔

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (ڈیفالٹ: pairing)۔

- `channels.telegram.allowFrom`: DM allowlist (عددی Telegram user IDs)۔ `open` کے لیے `"*"` درکار ہے۔ `openclaw doctor --fix` پرانے `@username` اندراجات کو IDs میں تبدیل کر سکتا ہے۔

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (ڈیفالٹ: allowlist)۔

- `channels.telegram.groupAllowFrom`: گروپ sender allowlist (عددی Telegram user IDs)۔ `openclaw doctor --fix` پرانے `@username` اندراجات کو IDs میں تبدیل کر سکتا ہے۔

- `channels.telegram.groups`: فی گروپ ڈیفالٹس + اجازت فہرست (عالمی ڈیفالٹس کے لیے `"*"` استعمال کریں)۔
  - `channels.telegram.groups.<id>`channels.telegram.groups.<id>
    .requireMention\`: مینشن گیٹنگ کی ڈیفالٹ۔
  - `channels.telegram.groups.<id>
    .groupPolicy`: گروپ کے لیے groupPolicy اووررائیڈ (`open | allowlist | disabled`)۔`channels.telegram.groups.<id>
    .allowFrom`: فی گروپ بھیجنے والے کی اجازت فہرست کا اووررائیڈ۔
  - `channels.telegram.groups.<id>`channels.telegram.groups.<id>
    .enabled`: جب `false\` ہو تو گروپ کو غیر فعال کریں۔
  - `channels.telegram.groups.<id>`channels.telegram.groups.<id>
    .topics.<threadId>
    .groupPolicy`: groupPolicy کے لیے فی موضوع اووررائیڈ (`open | allowlist | disabled\`)۔
  - `channels.telegram.groups.<id>.systemPrompt`: extra system prompt for the group.
  - `channels.telegram.groups.<id>
    .topics.<threadId>
    .requireMention`: فی موضوع مینشن گیٹنگ اووررائیڈ۔Happy Eyeballs ٹائم آؤٹس سے بچنے کے لیے Node 22 پر ڈیفالٹ طور پر غیر فعال ہے۔
  - `channels.telegram.network.autoSelectFamily`: Node کے autoSelectFamily کو اووررائیڈ کریں (true=فعال، false=غیر فعال)۔.topics.<threadId>Tlon ایک غیر مرکزی میسنجر ہے جو Urbit پر بنایا گیا ہے۔
  - `commands.native` (ڈیفالٹ `"auto"` → Telegram/Discord کے لیے آن، Slack کے لیے آف)، `commands.text`, `commands.useAccessGroups` (کمانڈ رویّہ)۔`channels.telegram.commands.native` کے ساتھ اووررائیڈ کریں۔اسٹیٹس: پلگ اِن کے ذریعے سپورٹڈ۔
  - DMs، گروپ مینشنز، تھریڈ ریپلائیز، اور صرف متن والا میڈیا فال بیک
    (کیپشن کے ساتھ URL شامل کیا جاتا ہے)۔گروپ جوابات کے لیے ڈیفالٹ طور پر @ مینشن درکار ہوتا ہے اور انہیں اجازت فہرستوں کے ذریعے مزید محدود کیا جا سکتا ہے۔آٹو ڈسکوری ڈیفالٹ طور پر فعال ہے۔

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (ڈیفالٹ: allowlist)۔

- \`channels.telegram.accounts.<account>IRC کنکشن کے ذریعے Twitch چیٹ سپورٹ۔

- `channels.telegram.replyToMode`: `off | first | all` (ڈیفالٹ: `off`)۔

- `channels.telegram.textChunkLimit`: آؤٹ باؤنڈ چنک سائز (حروف)۔

- `channels.telegram.chunkMode`: `length` (ڈیفالٹ) یا `newline` تاکہ لمبائی چنکنگ سے پہلے خالی لائنوں (پیراگراف حدود) پر تقسیم ہو۔

- `channels.telegram.linkPreview`: آؤٹ باؤنڈ پیغامات کے لیے لنک پری ویوز ٹوگل کریں (ڈیفالٹ: true)۔

- `channels.telegram.streamMode`: `off | partial | block` (لائیو اسٹریم پیش منظر)۔

- `channels.telegram.mediaMaxMb`: اِن باؤنڈ/آؤٹ باؤنڈ میڈیا حد (MB)۔

- `channels.telegram.retry`: آؤٹ باؤنڈ Telegram API کالز کے لیے ری ٹرائی پالیسی (کوششیں، minDelayMs، maxDelayMs، jitter)۔

- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`: Bot API کالز کے لیے پراکسی URL (SOCKS/HTTP)۔

- `channels.telegram.webhookUrl`: ویب ہوک موڈ فعال کریں (درکار: `channels.telegram.webhookSecret`)۔

- `channels.telegram.webhookSecret`: ویب ہوک سیکرٹ (جب webhookUrl سیٹ ہو تو درکار)۔

- `channels.telegram.webhookPath`: لوکل ویب ہوک پاتھ (ڈیفالٹ `/telegram-webhook`)۔

- `channels.telegram.webhookHost`: مقامی webhook بائنڈ ہوسٹ (ڈیفالٹ `127.0.0.1`)۔

- `channels.telegram.actions.reactions`: Telegram ٹول ری ایکشنز گیٹ کریں۔

- `channels.telegram.actions.sendMessage`: Telegram ٹول پیغام بھیجنا گیٹ کریں۔

- `channels.telegram.actions.deleteMessage`: Telegram ٹول پیغام حذف کرنا گیٹ کریں۔

- `channels.telegram.actions.sticker`: Telegram اسٹیکر ایکشنز — بھیجنا اور تلاش کرنا — گیٹ کریں (ڈیفالٹ: false)۔

- `channels.telegram.reactionNotifications`: `off | own | all` — کنٹرول کریں کہ کون سے ری ایکشنز سسٹم ایونٹس ٹرگر کریں (جب سیٹ نہ ہو تو ڈیفالٹ: `own`)۔

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — ایجنٹ کی ری ایکشن صلاحیت کنٹرول کریں (جب سیٹ نہ ہو تو ڈیفالٹ: `minimal`)۔

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

Telegram کے لیے مخصوص ہائی سگنل فیلڈز:

- startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- رسائی کنٹرول: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- کمانڈ/مینو: `commands.native`, `customCommands`
- تھریڈنگ/جوابات: `replyToMode`
- اسٹریمنگ: `streamMode` (پیش منظر)، `draftChunk`, `blockStreaming`
- فارمیٹنگ/ڈیلیوری: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- میڈیا/نیٹ ورک: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- ایکشنز/صلاحیتیں: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- ری ایکشنز: `reactionNotifications`, `reactionLevel`
- رائٹس/ہسٹری: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## متعلقہ

- `[[audio_as_voice]]` — آڈیو کو فائل کے بجائے وائس نوٹ کے طور پر بھیجیں۔
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)


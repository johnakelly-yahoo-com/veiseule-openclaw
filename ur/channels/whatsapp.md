---
summary: "WhatsApp (ویب چینل) انضمام: لاگ اِن، اِن باکس، جوابات، میڈیا، اور آپریشنز"
read_when:
  - WhatsApp/ویب چینل کے رویّے یا اِن باکس روٹنگ پر کام کرتے وقت
title: "WhatsApp"
---

# WhatsApp (ویب چینل)

اسٹیٹس: WhatsApp Web (Baileys) کے ذریعے پروڈکشن کے لیے تیار۔ Gateway منسلک سیشن(ز) کا مالک ہوتا ہے۔

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    نامعلوم بھیجنے والوں کے لیے ڈیفالٹ DM پالیسی پیئرنگ ہے۔
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    کراس چینل ڈائگناسٹکس اور مرمت کے پلے بکس۔
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    مکمل چینل کنفیگ پیٹرنز اور مثالیں۔
  
</Card>
</CardGroup>

## فوری سیٹ اپ

<Steps>
  <Step title="Configure WhatsApp access policy">

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

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    کسی مخصوص اکاؤنٹ کے لیے:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    ```
    پیئرنگ ریکویسٹ 1 گھنٹے کے بعد میعاد ختم ہو جاتی ہیں۔ زیر التواء ریکویسٹ ہر چینل کے لیے زیادہ سے زیادہ 3 تک محدود ہیں۔
    ```

  
</Step>
</Steps>

<Note>
OpenClaw ممکن ہو تو WhatsApp کو ایک علیحدہ نمبر پر چلانے کی تجویز دیتا ہے۔ (چینل میٹاڈیٹا اور آن بورڈنگ فلو اس سیٹ اپ کے لیے بہتر بنائے گئے ہیں، تاہم ذاتی نمبر والے سیٹ اپ بھی سپورٹ کیے جاتے ہیں۔)
</Note>

## Deployment پیٹرنز

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    یہ سب سے صاف ستھرا آپریشنل موڈ ہے:

    ````
    - OpenClaw کے لیے علیحدہ WhatsApp شناخت
    - واضح DM allowlists اور روٹنگ کی حدود
    - خود سے چیٹ کی الجھن کا کم امکان
    
    Minimal پالیسی پیٹرن:
    
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
    ````

  
</Accordion>

  <Accordion title="Personal-number fallback">
    آن بورڈنگ ذاتی نمبر موڈ کو سپورٹ کرتا ہے اور self-chat کے لیے موزوں بیس لائن لکھتا ہے:

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    موجودہ OpenClaw چینل آرکیٹیکچر میں میسجنگ پلیٹ فارم چینل WhatsApp Web-based (`Baileys`) ہے۔

    ```
    بلٹ اِن chat-channel رجسٹری میں کوئی علیحدہ Twilio WhatsApp میسجنگ چینل موجود نہیں ہے۔
    ```

  
</Accordion>
</AccordionGroup>

## Runtime ماڈل

- Gateway WhatsApp ساکٹ اور ری کنیکٹ لوپ کا مالک ہوتا ہے۔
- Outbound بھیجنے کے لیے ہدف اکاؤنٹ پر فعال WhatsApp listener درکار ہوتا ہے۔
- Status اور broadcast چیٹس کو نظرانداز کیا جاتا ہے (`@status`, `@broadcast`)۔
- Direct چیٹس DM سیشن رولز استعمال کرتی ہیں (`session.dmScope`; ڈیفالٹ `main` DMs کو ایجنٹ کے مرکزی سیشن میں ضم کر دیتا ہے)۔
- Group سیشنز الگ تھلگ ہوتے ہیں (`agent:<agentId>:whatsapp:group:<jid>`)۔

## رسائی کنٹرول اور ایکٹیویشن

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` direct چیٹ رسائی کو کنٹرول کرتا ہے:

    ```
    - `pairing` (ڈیفالٹ)
    - `allowlist`
    - `open` (اس کے لیے `allowFrom` میں `"*"` شامل ہونا ضروری ہے)
    - `disabled`
    
    `allowFrom` E.164 طرز کے نمبرز قبول کرتا ہے (اندرونی طور پر نارملائز کیے جاتے ہیں)۔
    
    ملٹی اکاؤنٹ اووررائیڈ: `channels.whatsapp.accounts.<id>.dmPolicy` (اور `allowFrom`) اس اکاؤنٹ کے لیے چینل لیول ڈیفالٹس پر فوقیت رکھتے ہیں۔
    
    Runtime رویے کی تفصیلات:
    
    - pairings چینل allow-store میں محفوظ کی جاتی ہیں اور کنفیگر شدہ `allowFrom` کے ساتھ ضم کی جاتی ہیں
    - اگر کوئی allowlist کنفیگر نہ ہو تو لنک کیا گیا self نمبر بطور ڈیفالٹ اجازت یافتہ ہوتا ہے
    - outbound `fromMe` DMs کبھی بھی خودکار طور پر pair نہیں کیے جاتے
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Group رسائی کی دو تہیں ہیں:

    ```
    1. **Group membership allowlist** (`channels.whatsapp.groups`)
       - اگر `groups` شامل نہ ہو تو تمام گروپس اہل ہوں گے
       - اگر `groups` موجود ہو تو یہ گروپ allowlist کے طور پر کام کرتا ہے (`"*"` کی اجازت ہے)
    
    2. **Group sender policy** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: بھیجنے والے کی allowlist کو نظرانداز کیا جاتا ہے
       - `allowlist`: بھیجنے والا `groupAllowFrom` (یا `*`) سے مطابقت رکھتا ہو
       - `disabled`: تمام گروپ inbound کو بلاک کر دیں
    
    Sender allowlist fallback:
    
    - اگر `groupAllowFrom` سیٹ نہ ہو تو runtime دستیاب ہونے پر `allowFrom` کی طرف رجوع کرتا ہے
    
    نوٹ: اگر بالکل بھی `channels.whatsapp` بلاک موجود نہ ہو تو runtime میں group-policy fallback مؤثر طور پر `open` ہوتا ہے۔
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    ڈیفالٹ کے طور پر گروپ جوابات کے لیے mention درکار ہوتی ہے۔

    ```
    Mention کی شناخت میں شامل ہیں:
    
    - بوٹ شناخت کے واضح WhatsApp mentions
    - کنفیگر شدہ mention regex پیٹرنز (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - ضمنی reply-to-bot شناخت (ریپلائی بھیجنے والا بوٹ شناخت سے مطابقت رکھتا ہو)
    
    سیشن لیول ایکٹیویشن کمانڈ:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` سیشن اسٹیٹ کو اپڈیٹ کرتا ہے (گلوبل کنفیگ نہیں)۔ یہ owner-gated ہے۔
    ```

  
</Tab>
</Tabs>

## ذاتی نمبر اور self-chat رویہ

عالمی طور پر غیرفعال کریں:

- self-chat ٹرنز کے لیے read receipts کو چھوڑ دیں
- mention-JID آٹو ٹرگر رویے کو نظرانداز کریں جو بصورت دیگر آپ کو خود ping کرے گا
- اگر `messages.responsePrefix` سیٹ نہ ہو تو self-chat جوابات بطور ڈیفالٹ `[{identity.name}]` یا `[openclaw]` ہوں گے

## میسج نارملائزیشن اور سیاق و سباق

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    آنے والے WhatsApp پیغامات کو مشترکہ inbound envelope میں لپیٹا جاتا ہے۔

    ````
    اگر quoted reply موجود ہو تو سیاق و سباق اس شکل میں شامل کیا جاتا ہے:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    جب دستیاب ہوں تو reply میٹاڈیٹا فیلڈز بھی پُر کی جاتی ہیں (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164)۔
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    صرف میڈیا پر مشتمل inbound پیغامات کو درج ذیل placeholders کے ساتھ نارملائز کیا جاتا ہے:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Location اور contact payloads کو روٹنگ سے پہلے متنی سیاق و سباق میں نارملائز کیا جاتا ہے۔
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    گروپس کے لیے، غیر پراسیس شدہ پیغامات کو بفر کیا جا سکتا ہے اور جب بوٹ آخرکار ٹرگر ہو تو انہیں سیاق و سباق کے طور پر شامل کیا جا سکتا ہے۔

    ```
    - ڈیفالٹ حد: `50`
    - کنفیگ: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` غیر فعال کرتا ہے
    
    Injection مارکرز:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    قبول شدہ inbound WhatsApp پیغامات کے لیے read receipts بطور ڈیفالٹ فعال ہیں۔

    ````
    گلوبلی غیر فعال کریں:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    فی اکاؤنٹ اووررائیڈ:
    
    ```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```
    
    Self-chat ٹرنز، حتیٰ کہ گلوبلی فعال ہونے پر بھی، read receipts کو چھوڑ دیتے ہیں۔
    ````

  
</Accordion>
</AccordionGroup>

## ڈیلیوری، chunking، اور میڈیا

<AccordionGroup>
  <Accordion title="Text chunking">
    - ڈیفالٹ chunk حد: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - `newline` موڈ پیراگراف کی حدود (خالی سطور) کو ترجیح دیتا ہے، پھر لمبائی کے مطابق محفوظ chunking پر واپس آتا ہے
  
</Accordion>

  <Accordion title="Outbound media behavior">
    - image، video، audio (PTT voice-note)، اور document payloads کی سپورٹ
    - `audio/ogg` کو voice-note مطابقت کے لیے `audio/ogg; codecs=opus` میں دوبارہ لکھا جاتا ہے
    - متحرک GIF پلے بیک ویڈیو بھیجتے وقت `gifPlayback: true` کے ذریعے سپورٹ کیا جاتا ہے
    - ملٹی میڈیا ریپلائی payloads بھیجتے وقت caption پہلے میڈیا آئٹم پر لاگو ہوتا ہے
    - میڈیا سورس HTTP(S)، `file://`، یا لوکل پاتھز ہو سکتا ہے
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - آنے والی میڈیا محفوظ کرنے کی حد: `channels.whatsapp.mediaMaxMb` (ڈیفالٹ `50`)
    - خودکار جوابات کے لیے باہر جانے والی میڈیا حد: `agents.defaults.mediaMaxMb` (ڈیفالٹ `5MB`)
    - تصاویر کو حدود میں فٹ کرنے کے لیے خودکار طور پر بہتر بنایا جاتا ہے (سائز/کوالٹی ایڈجسٹمنٹ)
    - میڈیا بھیجنے میں ناکامی کی صورت میں، پہلی آئٹم کے فال بیک کے طور پر خاموشی سے جواب ختم کرنے کے بجائے متنی وارننگ بھیجی جاتی ہے
  
</Accordion>
</AccordionGroup>

## موصولی تصدیق کے ری ایکشنز

**کنفیگریشن:**

```json5
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
- WhatsApp `channels.whatsapp.ackReaction` استعمال کرتا ہے (`messages.ackReaction` یہاں استعمال نہیں ہوتا)

## متعدد اکاؤنٹس اور اسناد

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - اکاؤنٹ آئی ڈیز `channels.whatsapp.accounts` سے لی جاتی ہیں
    - ڈیفالٹ اکاؤنٹ کا انتخاب: اگر `default` موجود ہو تو وہ، ورنہ ترتیب شدہ فہرست میں پہلا کنفیگر شدہ اکاؤنٹ آئی ڈی
    - تلاش کے لیے اکاؤنٹ آئی ڈیز کو اندرونی طور پر نارملائز کیا جاتا ہے
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - موجودہ auth راستہ: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - بیک اپ فائل: `creds.json.bak`
    - `~/.openclaw/credentials/` میں موجود پرانا ڈیفالٹ auth اب بھی ڈیفالٹ اکاؤنٹ فلو کے لیے تسلیم/منتقل کیا جاتا ہے
  
</Accordion>

  <Accordion title="Logout behavior">`openclaw channels logout --channel whatsapp [--account <id>]` اس اکاؤنٹ کے لیے WhatsApp auth اسٹیٹ کو صاف کرتا ہے۔

    ```
    پرانے auth ڈائریکٹریز میں، `oauth.json` محفوظ رہتا ہے جبکہ Baileys auth فائلیں حذف کر دی جاتی ہیں۔
    ```

  
</Accordion>
</AccordionGroup>

## حدود

- آؤٹ باؤنڈ متن `channels.whatsapp.textChunkLimit` تک ٹکڑوں میں بھیجا جاتا ہے (ڈیفالٹ 4000)۔
- اختیاری نئی لائن ٹکڑا بندی: خالی لائنوں (پیراگراف حدود) پر تقسیم کے لیے `channels.whatsapp.chunkMode="newline"` سیٹ کریں، پھر لمبائی کے مطابق ٹکڑا بندی۔
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- اِن باؤنڈ میڈیا سیوز `channels.whatsapp.mediaMaxMb` سے محدود (ڈیفالٹ 50 MB)۔

## آؤٹ باؤنڈ ارسال (متن + میڈیا)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">
    علامت: چینل اسٹیٹس لنکڈ رپورٹ نہیں ہو رہا۔

    ````
    حل:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    علامت: لنک شدہ اکاؤنٹ میں بار بار ڈس کنیکٹ یا ری کنیکٹ کی کوششیں۔

    ````
    حل:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    ضرورت ہو تو `channels login` کے ساتھ دوبارہ لنک کریں۔
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">
    جب ہدف اکاؤنٹ کے لیے کوئی فعال gateway listener موجود نہ ہو تو آؤٹ باؤنڈ بھیجنا فوری طور پر ناکام ہو جاتا ہے۔

    ```
    یقینی بنائیں کہ gateway چل رہا ہو اور اکاؤنٹ لنک ہو۔
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    اس ترتیب سے چیک کریں:

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` allowlist اندراجات
    - mention gating (`requireMention` + mention patterns)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway رن ٹائم کو Node استعمال کرنا چاہیے۔ مستحکم WhatsApp/Telegram gateway آپریشن کے لیے Bun کو غیر مطابقت پذیر قرار دیا گیا ہے۔
  
</Accordion>
</AccordionGroup>

## کنفیگریشن حوالہ جاتی نکات

بنیادی حوالہ:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

اہم WhatsApp فیلڈز:

- رسائی: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- ترسیل: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- متعدد اکاؤنٹس: `accounts.<id>.enabled`, `accounts.<id>.authDir`, اکاؤنٹ لیول اوور رائیڈز
- آپریشنز: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- سیشن رویہ: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## متعلقہ

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)

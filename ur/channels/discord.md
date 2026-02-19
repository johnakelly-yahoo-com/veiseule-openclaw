---
summary: "Discord بوٹ کی سپورٹ کی حیثیت، صلاحیتیں، اور کنفیگریشن"
read_when:
  - Discord چینل کی خصوصیات پر کام کرتے وقت
title: "Discord"
---

# Discord (Bot API)

حیثیت: سرکاری Discord بوٹ گیٹ وے کے ذریعے DMs اور guild ٹیکسٹ چینلز کے لیے تیار۔

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Discord DMs ڈیفالٹ طور پر pairing موڈ استعمال کرتے ہیں۔
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">نیٹو کمانڈ رویہ اور کمانڈ کیٹلاگ۔
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">کراس-چینل ڈائیگناسٹکس اور مرمت کا فلو۔
</Card>
</CardGroup>

## فوری سیٹ اپ

<Steps>
  <Step title="Create a Discord bot and enable intents">Discord Developer Portal میں ایک ایپلیکیشن بنائیں، ایک bot شامل کریں، پھر یہ فعال کریں:

    ```
    - **Message Content Intent**
    - **Server Members Intent** (رول allowlists اور رول پر مبنی روٹنگ کے لیے درکار؛ نام سے ID allowlist میچنگ کے لیے تجویز کردہ)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    ڈیفالٹ اکاؤنٹ کے لیے Env فال بیک:
    ```

```bash
DISCORD_BOT_TOKEN=...
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    اپنے سرور میں پیغام بھیجنے کی اجازت کے ساتھ بوٹ کو مدعو کریں۔

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    پیئرنگ کوڈز 1 گھنٹے کے بعد میعاد ختم ہو جاتے ہیں۔
    ```

  
</Step>
</Steps>

<Note>
ٹوکن ریزولوشن اکاؤنٹ کے مطابق ہوتا ہے۔ کنفیگ میں موجود ٹوکن ویلیوز Env فال بیک پر فوقیت رکھتی ہیں۔ `DISCORD_BOT_TOKEN` صرف ڈیفالٹ اکاؤنٹ کے لیے استعمال ہوتا ہے۔
</Note>

## رن ٹائم ماڈل

- Gateway Discord کنکشن کا مالک ہوتا ہے۔
- جواب کی روٹنگ قطعی (deterministic) ہے: Discord سے آنے والے پیغامات کا جواب دوبارہ Discord پر جاتا ہے۔
- ڈیفالٹ کے طور پر (`session.dmScope=main`)، براہِ راست چیٹس ایجنٹ کے مرکزی سیشن (`agent:main:main`) کو شیئر کرتی ہیں۔
- Guild چینلز الگ سیشن کیز ہوتے ہیں (`agent:<agentId>:discord:channel:<channelId>`)۔
- گروپ DMs کو ڈیفالٹ کے طور پر نظر انداز کیا جاتا ہے (`channels.discord.dm.groupEnabled=false`)۔
- نیٹو سلیش کمانڈز الگ کمانڈ سیشنز میں چلتی ہیں (`agent:<agentId>:discord:slash:<userId>`)، جبکہ روٹ شدہ گفتگو کے سیشن میں `CommandTargetSessionKey` بھی ساتھ لے کر جاتی ہیں۔

## رسائی کنٹرول اور روٹنگ

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` DM رسائی کو کنٹرول کرتا ہے (legacy: `channels.discord.dm.policy`):

    ```
    - `pairing` (ڈیفالٹ)
    - `allowlist`
    - `open` (اس کے لیے ضروری ہے کہ `channels.discord.allowFrom` میں `"*"` شامل ہو؛ legacy: `channels.discord.dm.allowFrom`)
    - `disabled`
    
    اگر DM پالیسی open نہ ہو تو نامعلوم صارفین کو بلاک کر دیا جاتا ہے (یا `pairing` موڈ میں پیئرنگ کا اشارہ دیا جاتا ہے)۔
    
    ڈیلیوری کے لیے DM ہدف کی فارمیٹ:
    
    - `user:<id>`
    - `<@id>` مینشن
    
    صرف عددی IDs مبہم سمجھی جاتی ہیں اور مسترد کر دی جاتی ہیں، جب تک واضح user/channel ہدف کی قسم فراہم نہ کی جائے۔
    ```

  
</Tab>

  <Tab title="Guild policy">
    Guild ہینڈلنگ کو `channels.discord.groupPolicy` کنٹرول کرتا ہے:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    جب `channels.discord` موجود ہو تو محفوظ بنیادی ترتیب `allowlist` ہوتی ہے۔
    
    `allowlist` کا برتاؤ:
    
    - guild کو `channels.discord.guilds` سے میچ کرنا لازمی ہے (`id` کو ترجیح دی جاتی ہے، slug بھی قبول ہے)
    - اختیاری بھیجنے والے کی allowlists: `users` (IDs یا نام) اور `roles` (صرف role IDs)؛ اگر ان میں سے کوئی ترتیب دی گئی ہو تو بھیجنے والا اس وقت مجاز ہوگا جب وہ `users` یا `roles` میں سے کسی ایک سے میچ کرے
    - اگر کسی guild میں `channels` کنفیگر ہوں تو غیر درج شدہ چینلز مسترد کر دیے جاتے ہیں
    - اگر کسی guild میں `channels` بلاک موجود نہ ہو تو اس allowlisted guild کے تمام چینلز مجاز ہوتے ہیں
    
    مثال:
    ```

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

    ```
    اگر آپ صرف `DISCORD_BOT_TOKEN` سیٹ کریں اور `channels.discord` بلاک نہ بنائیں تو رن ٹائم فال بیک `groupPolicy="open"` ہوگا (لاگز میں وارننگ کے ساتھ)۔
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild پیغامات ڈیفالٹ کے طور پر مینشن پر منحصر (mention-gated) ہوتے ہیں۔

    ```
    مینشن کی شناخت میں شامل ہیں:
    
    - بوٹ کا واضح مینشن
    - کنفیگر شدہ مینشن پیٹرنز (`agents.list[].groupChat.mentionPatterns`، فال بیک `messages.groupChat.mentionPatterns`)
    - معاون صورتوں میں بوٹ کو جواب دینے کا ضمنی رویہ
    
    `requireMention` ہر guild/channel کے لیے کنفیگر کیا جاتا ہے (`channels.discord.guilds...`)۔
    
    Group DMs:
    
    - ڈیفالٹ: نظر انداز (`dm.groupEnabled=false`)
    - اختیاری allowlist بذریعہ `dm.groupChannels` (channel IDs یا slugs)
    ```

  
</Tab>
</Tabs>

### رول پر مبنی ایجنٹ روٹنگ

Discord guild ممبرز کو role ID کے ذریعے مختلف ایجنٹس کی طرف روٹ کرنے کے لیے `bindings[].match.roles` استعمال کریں۔ رول پر مبنی بائنڈنگز صرف role IDs قبول کرتی ہیں اور peer یا parent-peer بائنڈنگز کے بعد اور صرف guild بائنڈنگز سے پہلے جانچی جاتی ہیں۔ اگر کسی بائنڈنگ میں دیگر match فیلڈز بھی سیٹ ہوں (مثال کے طور پر `peer` + `guildId` + `roles`) تو تمام کنفیگر شدہ فیلڈز کا میچ ہونا ضروری ہے۔

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Developer Portal سیٹ اپ

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. بوٹ ٹوکن کاپی کریں
    ```

  
</Accordion>

  <Accordion title="Privileged intents">
    **Bot -> Privileged Gateway Intents** میں، یہ فعال کریں:

    ```
    - Message Content Intent
    - Server Members Intent (تجویز کردہ)
    
    Presence intent اختیاری ہے اور صرف اس صورت میں درکار ہے جب آپ presence اپڈیٹس وصول کرنا چاہتے ہوں۔ بوٹ کی presence سیٹ کرنا (`setPresence`) کے لیے ممبرز کی presence اپڈیٹس فعال کرنا ضروری نہیں۔
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    OAuth URL جنریٹر:

    ```
    - scopes: `bot`, `applications.commands`
    
    عام بنیادی اجازتیں:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (اختیاری)
    
    جب تک واضح ضرورت نہ ہو `Administrator` سے گریز کریں۔
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Discord Developer Mode فعال کریں، پھر کاپی کریں:

    ```
    - server ID
    - channel ID
    - user ID
    
    قابلِ اعتماد آڈٹس اور پروبز کے لیے OpenClaw کنفیگ میں عددی IDs کو ترجیح دیں۔
    ```

  
</Accordion>
</AccordionGroup>

## مقامی کمانڈز اور کمانڈ تصدیق

- `commands.native` کی ڈیفالٹ ویلیو `"auto"` ہے اور یہ Discord کے لیے فعال ہوتا ہے۔
- فی چینل اووررائیڈ: `channels.discord.commands.native`۔
- `commands.native=false` پہلے سے رجسٹر شدہ Discord مقامی کمانڈز کو واضح طور پر صاف کر دیتا ہے۔
- مقامی کمانڈ تصدیق عام پیغام ہینڈلنگ کی طرح وہی Discord allowlists/policies استعمال کرتی ہے۔
- کمانڈز Discord UI میں اُن صارفین کو نظر آ سکتی ہیں جو مجاز نہیں ہیں؛ تاہم عمل درآمد کے وقت OpenClaw کی تصدیق نافذ ہوتی ہے اور "not authorized" واپس کیا جاتا ہے۔

کمانڈ کی فہرست اور رویّے کے لیے [Slash commands](/tools/slash-commands) دیکھیں۔

## Retry policy

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord ایجنٹ آؤٹ پٹ میں reply ٹیگز کو سپورٹ کرتا ہے:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    `channels.discord.replyToMode` کے ذریعے کنٹرول کیا جاتا ہے:
    
    - `off` (ڈیفالٹ)
    - `first`
    - `all`
    
    نوٹ: `off` ضمنی reply تھریڈنگ کو غیر فعال کرتا ہے۔ واضح `[[reply_to_*]]` ٹیگز پھر بھی قابلِ قبول ہوتے ہیں۔
    
    پیغام IDs کو context/history میں شامل کیا جاتا ہے تاکہ ایجنٹس مخصوص پیغامات کو ہدف بنا سکیں۔
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild ہسٹری کانٹیکسٹ:

    ```
    - `channels.discord.historyLimit` ڈیفالٹ `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` غیر فعال کرتا ہے
    
    DM ہسٹری کنٹرولز:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    تھریڈ رویّہ:
    
    - Discord تھریڈز کو چینل سیشنز کے طور پر روٹ کیا جاتا ہے
    - parent تھریڈ میٹاڈیٹا parent-session لنکیج کے لیے استعمال ہو سکتا ہے
    - جب تک تھریڈ کے لیے مخصوص اندراج موجود نہ ہو، تھریڈ کنفیگ parent چینل کنفیگ سے وراثت میں ملتی ہے
    
    چینل ٹاپکس کو **untrusted** کانٹیکسٹ کے طور پر شامل کیا جاتا ہے (system prompt کے طور پر نہیں)۔
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">
    فی-guild ری ایکشن نوٹیفکیشن موڈ:

    ```
    - `off`
    - `own` (ڈیفالٹ)
    - `all`
    - `allowlist` (استعمال کرتا ہے `guilds.<id>.users`)
    
    ری ایکشن ایونٹس کو سسٹم ایونٹس میں تبدیل کر کے روٹ کیے گئے Discord سیشن کے ساتھ منسلک کیا جاتا ہے۔
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` اس وقت ایک تصدیقی ایموجی بھیجتا ہے جب OpenClaw آنے والے پیغام پر پروسیسنگ کر رہا ہو۔

    ```
    حل کی ترتیب:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - ایجنٹ شناخت ایموجی fallback (`agents.list[].identity.emoji`, ورنہ "👀")
    
    نوٹس:
    
    - Discord یونیکوڈ ایموجی یا کسٹم ایموجی نام قبول کرتا ہے۔
    - کسی چینل یا اکاؤنٹ کے لیے ری ایکشن غیر فعال کرنے کو `""` استعمال کریں۔
    ```

  
</Accordion>

  <Accordion title="Config writes">
    چینل سے شروع کی گئی کنفیگ رائٹس ڈیفالٹ کے طور پر فعال ہیں۔

    ```
    یہ `/config set|unset` فلوؤں کو متاثر کرتا ہے (جب کمانڈ فیچرز فعال ہوں)۔
    
    غیر فعال کریں:
    ```

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    `channels.discord.proxy` کے ذریعے Discord gateway WebSocket ٹریفک کو HTTP(S) پراکسی سے گزاریں۔

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

    ```
    فی-اکاؤنٹ اووررائیڈ:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    پراکسی کیے گئے پیغامات کو سسٹم ممبر شناخت سے میپ کرنے کے لیے PluralKit ریزولوشن فعال کریں:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // اختیاری؛ نجی سسٹمز کے لیے درکار
      },
    },
  },
}
```

    ```
    نوٹس:
    
    - allowlists میں `pk:<memberId>` استعمال کیا جا سکتا ہے
    - ممبر ڈسپلے نام نام/slug کے ذریعے میچ کیے جاتے ہیں
    - lookup اصل میسج ID استعمال کرتا ہے اور وقت کی حد کے اندر محدود ہوتا ہے
    - اگر lookup ناکام ہو جائے تو پراکسی کیے گئے پیغامات کو بوٹ پیغامات سمجھ کر ڈراپ کر دیا جاتا ہے جب تک کہ `allowBots=true` نہ ہو
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Presence اپڈیٹس صرف اُس وقت لاگو ہوتی ہیں جب آپ status یا activity فیلڈ سیٹ کریں۔

    ```
    صرف اسٹیٹس کی مثال:
    ```

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    ```
    ایکٹیویٹی کی مثال (custom status ڈیفالٹ ایکٹیویٹی ٹائپ ہے):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    اسٹریمنگ کی مثال:
    ```

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    ```
    ایکٹیویٹی ٹائپ میپ:
    
    - 0: Playing
    - 1: Streaming (کے لیے `activityUrl` درکار ہے)
    - 2: Listening
    - 3: Watching
    - 4: Custom (ایکٹیویٹی ٹیکسٹ کو اسٹیٹس اسٹیٹ کے طور پر استعمال کرتا ہے؛ ایموجی اختیاری ہے)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord DMs میں بٹن پر مبنی exec approvals کو سپورٹ کرتا ہے اور اختیاری طور پر منظوری کے پرامپٹس اصل چینل میں پوسٹ کر سکتا ہے۔

    ```
    کنفیگ پاتھ:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, ڈیفالٹ: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    جب `target` کی ویلیو `channel` یا `both` ہو تو منظوری کا پرامپٹ چینل میں نظر آتا ہے۔ صرف کنفیگر کیے گئے approvers بٹن استعمال کر سکتے ہیں؛ دیگر صارفین کو ephemeral انکار ملتا ہے۔ منظوری کے پرامپٹس میں کمانڈ ٹیکسٹ شامل ہوتا ہے، اس لیے چینل ڈلیوری صرف قابلِ اعتماد چینلز میں فعال کریں۔ اگر سیشن کی سے چینل ID حاصل نہ ہو سکے تو OpenClaw DM ڈلیوری پر واپس آ جاتا ہے۔
    
    اگر منظوری نامعلوم approval IDs کے ساتھ ناکام ہو تو approver فہرست اور فیچر کی فعالی کی تصدیق کریں۔
    
    متعلقہ دستاویزات: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## ٹولز اور ایکشن گیٹس

Discord میسج ایکشنز میں میسجنگ، چینل ایڈمن، موڈریشن، presence، اور میٹاڈیٹا ایکشنز شامل ہیں۔

بنیادی مثالیں:

- پیغام رسانی: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- ری ایکشنز: `react`, `reactions`, `emojiList`
- اعتدال (موڈریشن): `timeout`, `kick`, `ban`
- حاضری: `setPresence`

Action gates `channels.discord.actions.*` کے تحت موجود ہوتے ہیں۔

ڈیفالٹ گیٹ رویہ:

| Action group                                                                                                                                                             | Default  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled  |
| roles                                                                                                                                                                    | disabled |
| moderation                                                                                                                                                               | disabled |
| presence                                                                                                                                                                 | disabled |

## Components v2 UI

OpenClaw منظوریِ اجرا (exec approvals) اور کراس-کانٹیکسٹ مارکرز کے لیے Discord components v2 استعمال کرتا ہے۔ Discord میسج ایکشنز `components` بھی قبول کر سکتے ہیں برائے کسٹم UI (ایڈوانسڈ؛ Carbon component instances درکار)، جبکہ پرانے `embeds` اب بھی دستیاب ہیں لیکن تجویز نہیں کیے جاتے۔

- `channels.discord.ui.components.accentColor` Discord component کنٹینرز میں استعمال ہونے والا accent رنگ (hex) سیٹ کرتا ہے۔
- ہر اکاؤنٹ کے لیے سیٹ کریں `channels.discord.accounts.<id> .ui.components.accentColor`.
- جب components v2 موجود ہوں تو `embeds` کو نظر انداز کر دیا جاتا ہے۔

مثال:

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## وائس پیغامات

Discord وائس پیغامات ویوفارم پیش نظارہ دکھاتے ہیں اور OGG/Opus آڈیو کے ساتھ میٹا ڈیٹا درکار ہوتا ہے۔ OpenClaw خودکار طور پر ویوفارم تیار کرتا ہے، لیکن آڈیو فائلز کی جانچ اور کنورژن کے لیے gateway ہوسٹ پر `ffmpeg` اور `ffprobe` دستیاب ہونا ضروری ہے۔

تقاضے اور پابندیاں:

- **لوکل فائل پاتھ** فراہم کریں (URLs مسترد کر دیے جاتے ہیں).
- متنی مواد شامل نہ کریں (Discord ایک ہی payload میں متن + وائس میسج کی اجازت نہیں دیتا).
- کوئی بھی آڈیو فارمیٹ قبول ہے؛ ضرورت پڑنے پر OpenClaw اسے OGG/Opus میں تبدیل کر دیتا ہے۔

مثال:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - Message Content Intent فعال کریں
    - جب آپ صارف/رکن کی شناخت پر انحصار کرتے ہوں تو Server Members Intent فعال کریں
    - intents تبدیل کرنے کے بعد gateway ری اسٹارٹ کریں
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - `groupPolicy` کی تصدیق کریں
    - `channels.discord.guilds` کے تحت guild allowlist کی تصدیق کریں
    - اگر guild `channels` میپ موجود ہو تو صرف درج شدہ چینلز کی اجازت ہوگی
    - `requireMention` کے رویے اور mention پیٹرنز کی تصدیق کریں
    
    مفید جانچ:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    عام وجوہات:

    ```
    - `groupPolicy="allowlist"` مگر متعلقہ guild/channel allowlist موجود نہیں
    - `requireMention` غلط جگہ پر کنفیگر ہے (یہ `channels.discord.guilds` یا چینل انٹری کے تحت ہونا چاہیے)
    - بھیجنے والا guild/channel `users` allowlist کی وجہ سے بلاک ہے
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` کی پرمیشن جانچ صرف عددی چینل IDs کے لیے کام کرتی ہے۔

    ```
    اگر آپ slug keys استعمال کرتے ہیں تو رن ٹائم میچنگ پھر بھی کام کر سکتی ہے، لیکن probe مکمل طور پر پرمیشنز کی تصدیق نہیں کر سکتا۔
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - DM غیر فعال: `channels.discord.dm.enabled=false`
    - DM پالیسی غیر فعال: `channels.discord.dmPolicy="disabled"` (پرانا طریقہ: `channels.discord.dm.policy`)
    - `pairing` موڈ میں منظوری کے انتظار میں
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    بطور ڈیفالٹ bot کی جانب سے بھیجے گئے پیغامات نظر انداز کیے جاتے ہیں۔

    ```
    اگر آپ `channels.discord.allowBots=true` سیٹ کرتے ہیں تو لوپ رویے سے بچنے کے لیے سخت mention اور allowlist قواعد استعمال کریں۔
    ```

  
</Accordion>
</AccordionGroup>

## کنفیگریشن ریفرنس پوائنٹرز

بنیادی حوالہ:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

اہم Discord فیلڈز:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- command: `commands.native`, `commands.useAccessGroups`, `configWrites`
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## حفاظت اور آپریشنز

- بوٹ ٹوکنز کو راز کے طور پر محفوظ رکھیں (`DISCORD_BOT_TOKEN` نگرانی شدہ ماحول میں ترجیحی ہے).
- کم سے کم درکار Discord اجازتیں فراہم کریں۔
- اگر کمانڈ deploy/state پرانا ہو تو gateway دوبارہ شروع کریں اور `openclaw channels status --probe` سے دوبارہ جانچ کریں۔

## متعلقہ

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- [Slash commands](/tools/slash-commands)


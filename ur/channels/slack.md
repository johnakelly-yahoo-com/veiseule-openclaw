---
summary: "Socket یا HTTP webhook موڈ کے لیے Slack سیٹ اپ"
read_when:
  - Slack سیٹ اپ کرنا یا Slack ساکٹ/HTTP موڈ کی ڈیبگنگ کرنا
title: "Slack"
---

# Slack

اسٹیٹس: Slack ایپ انٹیگریشنز کے ذریعے DMs اور چینلز کے لیے پروڈکشن کے لیے تیار۔ ڈیفالٹ موڈ Socket Mode ہے؛ HTTP Events API موڈ بھی سپورٹ کیا جاتا ہے۔

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DMs بطور ڈیفالٹ pairing موڈ پر سیٹ ہوتے ہیں۔
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    نیٹو کمانڈ رویہ اور کمانڈ کیٹلاگ۔
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    کراس چینل ڈائیگناسٹکس اور مرمتی پلے بکس۔
  
</Card>
</CardGroup>

## فوری سیٹ اپ

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        Slack ایپ کی سیٹنگز میں:


        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Env fallback (صرف ڈیفالٹ اکاؤنٹ کے لیے):
        ```

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

        
</Step>
      
        <Step title="ایپ ایونٹس کو سبسکرائب کریں">
          درج ذیل کے لیے بوٹ ایونٹس سبسکرائب کریں:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          نیز DMs کے لیے App Home میں **Messages Tab** کو فعال کریں۔
        
</Step>
      
        <Step title="گیٹ وے شروع کریں">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

        
</Step>
      
        <Step title="ملٹی اکاؤنٹ HTTP کے لیے منفرد webhook paths استعمال کریں">
          فی اکاؤنٹ HTTP موڈ سپورٹ کیا جاتا ہے۔
      
          ہر اکاؤنٹ کو ایک منفرد `webhookPath` دیں تاکہ رجسٹریشنز آپس میں ٹکرائیں نہیں۔
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## ٹوکن ماڈل

- `botToken` + `appToken` Socket Mode کے لیے درکار ہیں۔
- HTTP موڈ کے لیے `botToken` + `signingSecret` درکار ہیں۔
- کنفیگ ٹوکنز env fallback پر فوقیت رکھتے ہیں۔
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback صرف ڈیفالٹ اکاؤنٹ پر لاگو ہوتا ہے۔
- `userToken` (`xoxp-...`) صرف کنفیگ کے ذریعے دستیاب ہے (کوئی env fallback نہیں) اور بطور ڈیفالٹ صرف-ریڈ رویہ رکھتا ہے (`userTokenReadOnly: true`)۔
- اختیاری: اگر آپ چاہتے ہیں کہ آؤٹ گوئنگ پیغامات فعال ایجنٹ کی شناخت (کسٹم `username` اور آئیکن) استعمال کریں تو `chat:write.customize` شامل کریں۔ `icon_emoji` میں `:emoji_name:` کی سنٹیکس استعمال ہوتی ہے۔

<Tip>
ایکشنز/ڈائریکٹری ریڈز کے لیے، اگر کنفیگر کیا گیا ہو تو user token کو ترجیح دی جا سکتی ہے۔ رائٹس کے لیے، bot token بدستور ترجیحی رہتا ہے؛ user-token رائٹس صرف اسی صورت میں مجاز ہیں جب `userTokenReadOnly: false` ہو اور bot token دستیاب نہ ہو۔
</Tip>

## رسائی کنٹرول اور روٹنگ

<Tabs>
  <Tab title="DM policy">
    `channels.slack.dmPolicy` DM رسائی کو کنٹرول کرتا ہے (لیگیسی: `channels.slack.dm.policy`):

    ```
    - `pairing` (ڈیفالٹ)
    - `allowlist`
    - `open` (ضروری ہے کہ `channels.slack.allowFrom` میں `"*"` شامل ہو؛ لیگیسی: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM فلیگز:
    
    - `dm.enabled` (ڈیفالٹ true)
    - `channels.slack.allowFrom` (ترجیحی)
    - `dm.allowFrom` (لیگیسی)
    - `dm.groupEnabled` (گروپ DMs بطور ڈیفالٹ false)
    - `dm.groupChannels` (اختیاری MPIM allowlist)
    
    DMs میں pairing کے لیے `openclaw pairing approve slack <code>` استعمال کریں۔
    ```

  
</Tab>

  <Tab title="Channel policy">
    `channels.slack.groupPolicy` چینل ہینڈلنگ کو کنٹرول کرتا ہے:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    چینل allowlist `channels.slack.channels` کے تحت موجود ہوتی ہے۔
    
    رن ٹائم نوٹ: اگر `channels.slack` مکمل طور پر موجود نہ ہو (صرف env سیٹ اپ) اور `channels.defaults.groupPolicy` سیٹ نہ ہو تو رن ٹائم خودکار طور پر `groupPolicy="open"` پر واپس چلا جاتا ہے اور وارننگ لاگ کرتا ہے۔
    
    نام/ID ریزولوشن:
    
    - چینل allowlist اندراجات اور DM allowlist اندراجات اسٹارٹ اپ پر ریزولو کیے جاتے ہیں جب ٹوکن رسائی اجازت دے
    - جو اندراجات ریزولو نہ ہوں وہ کنفیگ کے مطابق برقرار رہتے ہیں
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    چینل پیغامات بطور ڈیفالٹ mention-gated ہوتے ہیں۔

    ```
    Mention کے ذرائع:
    
    - واضح ایپ mention (`<@botId>`)
    - mention regex پیٹرنز (`agents.list[].groupChat.mentionPatterns`, بطور fallback `messages.groupChat.mentionPatterns`)
    - بوٹ کو جواب دیے گئے تھریڈ کا ضمنی رویہ
    
    فی چینل کنٹرولز (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (allowlist)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>
</Tabs>

## OpenClaw کنفیگ (کم از کم)

- Slack کے لیے Native command auto-mode **بند** ہے (`commands.native: "auto"` Slack native commands کو فعال نہیں کرتا)۔
- `channels.slack.commands.native: true` (یا گلوبل `commands.native: true`) کے ذریعے Slack کے native command handlers کو فعال کریں۔
- جب native commands فعال ہوں تو Slack میں متعلقہ slash commands رجسٹر کریں (`/<command>` ناموں کے ساتھ)۔
- اگر native commands فعال نہ ہوں تو آپ `channels.slack.slashCommand` کے ذریعے ایک واحد کنفیگر شدہ slash command چلا سکتے ہیں۔

ڈیفالٹ slash command سیٹنگز:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

ایپ جلدی بنانے کے لیے اس Slack ایپ مینی فیسٹ کا استعمال کریں (اگر چاہیں تو نام/کمانڈ ایڈجسٹ کریں)۔ اگر آپ یوزر ٹوکن کنفیگر کرنے کا ارادہ رکھتے ہیں تو یوزر اسکوپس شامل کریں۔

- `agent:<agentId>:slack:slash:<userId>`

اگر آپ نیٹو کمانڈز فعال کرتے ہیں تو جس ہر کمانڈ کو ایکسپوز کرنا چاہتے ہیں اس کے لیے ایک `slash_commands` انٹری شامل کریں (`/help` کی فہرست کے مطابق)۔ `channels.slack.commands.native` کے ذریعے اووررائیڈ کریں۔

## Scopes (موجودہ بمقابلہ اختیاری)

- DMs کو `direct` کے طور پر روٹ کیا جاتا ہے؛ چینلز کو `channel`؛ اور MPIMs کو `group`۔
- ڈیفالٹ `session.dmScope=main` کے ساتھ، Slack DMs ایجنٹ کے مین سیشن میں ضم ہو جاتے ہیں۔
- چینل سیشنز: `agent:<agentId>:slack:channel:<channelId>`۔
- جب لاگو ہو تو تھریڈ ریپلائیز تھریڈ سیشن کے سفکس بنا سکتی ہیں (`:thread:<threadTs>`)۔
- `channels.slack.thread.historyScope` کا ڈیفالٹ `thread` ہے؛ `thread.inheritParent` کا ڈیفالٹ `false` ہے۔
- `channels.slack.thread.initialHistoryLimit` یہ کنٹرول کرتا ہے کہ نیا تھریڈ سیشن شروع ہونے پر موجودہ تھریڈ کے کتنے پیغامات حاصل کیے جائیں (ڈیفالٹ `20`؛ غیر فعال کرنے کے لیے `0` سیٹ کریں)۔

ریپلائی تھریڈنگ کنٹرولز:

- `chat:write` (`chat.postMessage` کے ذریعے پیغامات بھیجنا/اپ ڈیٹ/حذف)
  [https://docs.slack.dev/reference/methods/chat.postMessage](https://docs.slack.dev/reference/methods/chat.postMessage)
- `im:write` (یوزر DMs کے لیے `conversations.open` کے ذریعے DMs کھولنا)
  [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
- `channels:history`, `groups:history`, `im:history`, `mpim:history`
  [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)

دستی ریپلائی ٹیگز سپورٹ کیے جاتے ہیں:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

نوٹ: `replyToMode="off"` ضمنی ریپلائی تھریڈنگ کو غیر فعال کر دیتا ہے۔ واضح `[[reply_to_*]]` ٹیگز اب بھی قابلِ قبول ہوتے ہیں۔

## آج درکار نہیں (لیکن ممکنہ مستقبل)

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack فائل اٹیچمنٹس Slack پر ہوسٹ کی گئی نجی URLs سے ڈاؤن لوڈ کی جاتی ہیں (ٹوکن سے مستند ریکوئسٹ فلو) اور جب فَیچ کامیاب ہو اور سائز کی حد اجازت دے تو میڈیا اسٹور میں محفوظ کی جاتی ہیں۔

    ```
    رن ٹائم ان باؤنڈ سائز کی حد ڈیفالٹ طور پر `20MB` ہے، جب تک کہ `channels.slack.mediaMaxMb` کے ذریعے تبدیل نہ کی جائے۔
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - ٹیکسٹ چنکس `channels.slack.textChunkLimit` استعمال کرتے ہیں (ڈیفالٹ 4000)
    - `channels.slack.chunkMode="newline"` پیراگراف-فرسٹ اسپلٹنگ کو فعال کرتا ہے
    - فائل بھیجنے کے لیے Slack اپ لوڈ APIs استعمال ہوتی ہیں اور اس میں تھریڈ ریپلائیز (`thread_ts`) شامل ہو سکتی ہیں
    - آؤٹ باؤنڈ میڈیا کی حد کنفیگر ہونے پر `channels.slack.mediaMaxMb` کے مطابق ہوتی ہے؛ بصورت دیگر چینل سینڈز میڈیا پائپ لائن کے MIME-kind ڈیفالٹس استعمال کرتے ہیں
  
</Accordion>

  <Accordion title="Delivery targets">ترجیحی واضح ٹارگٹس:

    ```
    - DMs کے لیے `user:<id>`
    - چینلز کے لیے `channel:<id>`
    
    Slack DMs کو یوزر ٹارگٹس پر بھیجتے وقت Slack conversation APIs کے ذریعے کھولا جاتا ہے۔
    ```

  
</Accordion>
</AccordionGroup>

## حدود

Slack ایکشنز کو `channels.slack.actions.*` کے ذریعے کنٹرول کیا جاتا ہے۔

موجودہ Slack ٹولنگ میں دستیاب ایکشن گروپس:

| گروپ       | ڈیفالٹ  |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

## ایونٹس اور آپریشنل رویہ

- میسج ایڈیٹس/ڈیلیٹس/تھریڈ براڈکاسٹس کو سسٹم ایونٹس میں میپ کیا جاتا ہے۔
- ری ایکشن شامل/ہٹانے کے ایونٹس کو سسٹم ایونٹس میں میپ کیا جاتا ہے۔
- ممبر کے شامل/چھوڑنے، چینل کے بننے/ری نیم ہونے، اور پن شامل/ہٹانے کے ایونٹس کو سسٹم ایونٹس میں میپ کیا جاتا ہے۔
- `channel_id_changed` جب `configWrites` فعال ہو تو چینل کنفیگ کیز کو مائیگریٹ کر سکتا ہے۔
- چینل ٹاپک/پرپز میٹا ڈیٹا کو غیر معتبر کانٹیکسٹ سمجھا جاتا ہے اور اسے روٹنگ کانٹیکسٹ میں شامل کیا جا سکتا ہے۔

## فی چیٹ-ٹائپ تھریڈنگ

`channels.slack.replyToModeByChatType` سیٹ کر کے ہر چیٹ ٹائپ کے لیے مختلف تھریڈنگ رویّہ کنفیگر کیا جا سکتا ہے:

ریزولوشن آرڈر:

- `channels.slack.accounts.<accountId>`.ackReaction
- `channels.slack.ackReaction`
- `messages.ackReaction`
- ایجنٹ شناخت ایموجی فال بیک (`agents.list[].identity.emoji`، ورنہ "👀")

نوٹس:

- Slack شارٹ کوڈز کی توقع کرتا ہے (مثال کے طور پر `"eyes"`)۔
- کسی چینل یا اکاؤنٹ کے لیے ری ایکشن غیر فعال کرنے کے لیے `""` استعمال کریں۔

## Manifest اور اسکوپ چیک لسٹ

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">اگر آپ `channels.slack.userToken` کو کنفیگر کرتے ہیں تو عام read اسکوپس یہ ہوتے ہیں:

    ```
    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (اگر آپ Slack سرچ ریڈز پر انحصار کرتے ہیں)
    ```

  
</Accordion>
</AccordionGroup>

## خرابیوں کا ازالہ

<AccordionGroup>
  <Accordion title="No replies in channels">درج ذیل کو ترتیب سے چیک کریں:

    ```
    - `groupPolicy`
    - چینل allowlist (`channels.slack.channels`)
    - `requireMention`
    - فی چینل `users` allowlist
    
    مفید کمانڈز:
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">چیک کریں:

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (یا پرانا `channels.slack.dm.policy`)
    - pairing منظوریات / allowlist اندراجات
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">Slack ایپ سیٹنگز میں bot + app ٹوکنز اور Socket Mode کے فعال ہونے کی تصدیق کریں۔
</Accordion>

  <Accordion title="HTTP mode not receiving events">تصدیق کریں:

    ```
    - signing secret
    - webhook path
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - ہر HTTP اکاؤنٹ کے لیے منفرد `webhookPath`
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">تصدیق کریں کہ آپ کی نیت کیا تھی:

    ```
    - native کمانڈ موڈ (`channels.slack.commands.native: true`) جس میں Slack میں متعلقہ slash کمانڈز رجسٹرڈ ہوں
    - یا سنگل slash کمانڈ موڈ (`channels.slack.slashCommand.enabled: true`)
    
    اس کے علاوہ `commands.useAccessGroups` اور چینل/یوزر allowlists بھی چیک کریں۔
    ```

  
</Accordion>
</AccordionGroup>

## ٹول ایکشنز

Slack ٹول ایکشنز کو `channels.slack.actions.*` کے ذریعے گیٹ کیا جا سکتا ہے:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  اہم Slack فیلڈز:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM رسائی: `dm.enabled`, `dmPolicy`, `allowFrom` (پرانا: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - چینل رسائی: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - تھریڈنگ/ہسٹری: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - ڈلیوری: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - آپریشنز/فیچرز: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## سکیورٹی نوٹس

- لکھائیاں بطورِ طے شدہ بوٹ ٹوکن استعمال کرتی ہیں تاکہ حالت بدلنے والی کارروائیاں
  ایپ کے بوٹ کی اجازتوں اور شناخت تک محدود رہیں۔
- [Channel routing](/channels/channel-routing)
- اگر آپ یوزر-ٹوکن لکھائیاں فعال کرتے ہیں تو یقینی بنائیں کہ یوزر ٹوکن میں مطلوبہ
  لکھنے کے اسکوپس شامل ہوں (`chat:write`, `reactions:write`, `pins:write`,
  `files:write`) ورنہ وہ کارروائیاں ناکام ہوں گی۔
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)


---
summary: "Legacy iMessage support via imsg (JSON-RPC over stdio). New setups should use BlueBubbles."
read_when:
  - iMessage سپورٹ سیٹ اپ کرنا
  - iMessage بھیجنے/وصول کرنے کی ڈیبگنگ
title: "iMessage"
---

# iMessage (لیگیسی: imsg)

<Warning>
نئی iMessage تعیناتیوں کے لیے <a href="/channels/bluebubbles">BlueBubbles</a> استعمال کریں۔

`imsg` انٹیگریشن پرانا ہے اور مستقبل کی ریلیز میں ہٹایا جا سکتا ہے۔ 
</Warning>

Status: legacy external CLI integration. Gateway `imsg rpc` شروع کرتا ہے اور stdio پر JSON-RPC کے ذریعے رابطہ کرتا ہے (کوئی علیحدہ daemon/port نہیں).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    نئی سیٹ اپس کے لیے ترجیحی iMessage راستہ۔
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage DMs بطور ڈیفالٹ pairing موڈ استعمال کرتے ہیں۔
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    مکمل iMessage فیلڈ ریفرنس۔
  
</Card>
</CardGroup>

## فوری سیٹ اپ

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="Configure OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

      {
        channels: { imessage: { configWrites: false } },
      }

```bash
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

        ```
            Pairing کی درخواستیں 1 گھنٹے کے بعد ختم ہو جاتی ہیں۔
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw کو صرف ایک stdio-مطابقت رکھنے والا `cliPath` درکار ہوتا ہے، لہٰذا آپ `cliPath` کو ایسے wrapper اسکرپٹ کی طرف اشارہ کر سکتے ہیں جو SSH کے ذریعے کسی ریموٹ Mac سے جڑ کر `imsg` چلائے۔

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    اٹیچمنٹس فعال ہونے پر تجویز کردہ کنفیگ:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // used for SCP attachment fetches
      includeAttachments: true,
    },
  },
}
```

    ```
    اگر `remoteHost` سیٹ نہ ہو تو OpenClaw SSH wrapper اسکرپٹ کو پارس کر کے اسے خودکار طور پر شناخت کرنے کی کوشش کرتا ہے۔
    ```

  
</Tab>
</Tabs>

## ضروریات اور اجازتیں (macOS)

- میسجز لازماً اسی Mac پر سائن اِن ہوں جہاں `imsg` چل رہا ہو۔
- OpenClaw/`imsg` چلانے والے process context کے لیے Full Disk Access درکار ہے (Messages DB تک رسائی کے لیے)۔
- Messages.app کے ذریعے پیغامات بھیجنے کے لیے Automation اجازت درکار ہے۔

<Tip>
اجازتیں ہر process context کے مطابق دی جاتی ہیں۔ اگر gateway headless (LaunchAgent/SSH) پر چل رہا ہو تو prompts کو ٹرگر کرنے کے لیے اسی context میں ایک مرتبہ interactive کمانڈ چلائیں:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## رسائی کنٹرول اور روٹنگ

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` براہِ راست پیغامات کو کنٹرول کرتا ہے:

    ```
    - `pairing` (ڈیفالٹ)
    - `allowlist`
    - `open` (اس کے لیے `allowFrom` میں `"*"` شامل ہونا ضروری ہے)
    - `disabled`
    
    Allowlist فیلڈ: `channels.imessage.allowFrom`۔
    
    Allowlist اندراجات handles یا chat targets ہو سکتے ہیں (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`)۔
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` گروپس کی ہینڈلنگ کو کنٹرول کرتا ہے:

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DMs کے لیے direct routing استعمال ہوتی ہے؛ گروپس کے لیے group routing استعمال ہوتی ہے۔
    - ڈیفالٹ `session.dmScope=main` کے ساتھ، iMessage DMs ایجنٹ کے مین سیشن میں ضم ہو جاتے ہیں۔
    - گروپ سیشنز الگ تھلگ رہتے ہیں (`agent:<agentId> :imessage:group:<chat_id>`)۔
    - جوابات کو اصل چینل/ٹارگٹ metadata استعمال کرتے ہوئے واپس iMessage پر روٹ کیا جاتا ہے۔

    ```
    گروپ جیسا تھریڈ رویہ:
    
    کچھ کثیر شرکاء والے iMessage تھریڈز `is_group=false` کے ساتھ موصول ہو سکتے ہیں۔
    اگر وہ `chat_id` واضح طور پر `channels.imessage.groups` کے تحت کنفیگر کیا گیا ہو تو OpenClaw اسے گروپ ٹریفک کے طور پر ٹریٹ کرتا ہے (group gating + group session isolation)۔
    ```

  
</Tab>
</Tabs>

## ڈپلائمنٹ پیٹرنز

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    ایک مخصوص Apple ID اور macOS یوزر استعمال کریں تاکہ بوٹ ٹریفک آپ کی ذاتی Messages پروفائل سے الگ رہے۔

    ```
    {
      channels: {
        imessage: {
          cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
          remoteHost: "user@gateway-host", // for SCP file transfer
          includeAttachments: true,
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    عام ٹوپولوجی:

    ```
    - gateway Linux/VM پر چلتا ہے
    - iMessage + `imsg` آپ کے tailnet میں موجود ایک Mac پر چلتا ہے
    - `cliPath` wrapper SSH کے ذریعے `imsg` چلاتا ہے
    - `remoteHost` SCP اٹیچمنٹس حاصل کرنے کی اجازت دیتا ہے
    
    مثال:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    SSH keys استعمال کریں تاکہ SSH اور SCP دونوں non-interactive ہوں۔
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    iMessage میں `channels.imessage.accounts` کے تحت فی اکاؤنٹ کنفیگریشن کی سپورٹ موجود ہے۔

    ```
    ہر اکاؤنٹ `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` اور ہسٹری سیٹنگز جیسے فیلڈز کو اووررائیڈ کر سکتا ہے۔
    ```

  
</Accordion>
</AccordionGroup>

## میڈیا، chunking، اور ڈیلیوری ٹارگٹس

<AccordionGroup>
  <Accordion title="Attachments and media">
    - ان باؤنڈ اٹیچمنٹ انجیestion اختیاری ہے: `channels.imessage.includeAttachments`
    - جب `remoteHost` سیٹ ہو تو ریموٹ اٹیچمنٹ پاتھز کو SCP کے ذریعے حاصل کیا جا سکتا ہے
    - آؤٹ باؤنڈ میڈیا سائز `channels.imessage.mediaMaxMb` استعمال کرتا ہے (ڈیفالٹ 16 MB)
  
</Accordion>

  <Accordion title="Outbound chunking">
    - ٹیکسٹ chunk حد: `channels.imessage.textChunkLimit` (ڈیفالٹ 4000)
    - chunk موڈ: `channels.imessage.chunkMode`
      - `length` (ڈیفالٹ)
      - `newline` (پہلے پیراگراف کی بنیاد پر تقسیم)
  
</Accordion>

  <Accordion title="Addressing formats">
    ترجیحی واضح ٹارگٹس:

    ```
    - `chat_id:123` (مستحکم روٹنگ کے لیے تجویز کردہ)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle ٹارگٹس بھی سپورٹ کیے جاتے ہیں:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## یہ کیسے کام کرتا ہے (رویّہ)

iMessage ڈیفالٹ کے طور پر چینل کی جانب سے شروع کی گئی config writes کی اجازت دیتا ہے (جب `commands.config: true` ہو تو `/config set|unset` کے لیے)۔

غیر فعال کریں:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## ٹربل شوٹنگ

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    binary اور RPC سپورٹ کی توثیق کریں:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    {
      channels: {
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15555550123"],
          groups: {
            "42": { requireMention: false },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">
    چیک کریں:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - pairing منظوریوں (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    چیک کریں:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - `channels.imessage.groups` allowlist رویہ
    - mention pattern کی ترتیب (`agents.list[].groupChat.mentionPatterns`)
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">    Check:

    ```
    - `channels.imessage.remoteHost`
    - gateway host سے SSH/SCP key auth
    - Messages چلانے والے Mac پر remote path کی پڑھنے کی اجازت
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">    Re-run in an interactive GUI terminal in the same user/session context and approve prompts:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    یقینی بنائیں کہ Full Disk Access + Automation اس process context کے لیے دی گئی ہیں جو OpenClaw/`imsg` چلاتا ہے۔
    ```

  
</Accordion>
</AccordionGroup>

## Configuration reference pointers

- `agents.list[].groupChat.mentionPatterns` (یا `messages.groupChat.mentionPatterns`)۔
- `messages.responsePrefix`۔
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)


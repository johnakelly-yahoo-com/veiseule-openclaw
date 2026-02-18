---
title: "کنفیگریشن"
---

# کنفیگریشن

OpenClaw ایک اختیاری <Tooltip tip="JSON5 تبصرے اور آخر میں کاما کی اجازت دیتا ہے">**JSON5**</Tooltip> کنفیگ `~/.openclaw/openclaw.json` سے پڑھتا ہے۔

اگر فائل موجود نہ ہو تو OpenClaw محفوظ ڈیفالٹس استعمال کرتا ہے۔ عام وجوہات جن کی بنا پر آپ کنفیگ شامل کرنا چاہیں:

- چینلز کو کنیکٹ کریں اور یہ کنٹرول کریں کہ کون بوٹ کو میسج کر سکتا ہے  
- ماڈلز، ٹولز، سینڈ باکسنگ یا آٹومیشن (cron، hooks) سیٹ کریں  
- سیشنز، میڈیا، نیٹ ورکنگ یا UI کو ٹیون کریں  

تمام دستیاب فیلڈز کے لیے [مکمل حوالہ](/gateway/configuration-reference) دیکھیں۔

<Tip>
**کنفیگریشن میں نئے ہیں؟** انٹرایکٹو سیٹ اپ کے لیے `openclaw onboard` سے آغاز کریں، یا مکمل کاپی‑پیسٹ کنفیگز کے لیے [Configuration Examples](/gateway/configuration-examples) گائیڈ دیکھیں۔
</Tip>

## کم از کم کنفیگ

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## کنفیگ میں ترمیم

<Tabs>
  <Tab title="انٹرایکٹو وزرڈ">
    ```bash
    openclaw onboard       # مکمل سیٹ اپ وزرڈ
    openclaw configure     # کنفیگ وزرڈ
    ```
  
</Tab>
  <Tab title="CLI (ون لائنرز)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="کنٹرول UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) کھولیں اور **Config** ٹیب استعمال کریں۔  
    کنٹرول UI کنفیگ اسکیما سے ایک فارم رینڈر کرتا ہے، اور متبادل کے طور پر **Raw JSON** ایڈیٹر بھی فراہم کرتا ہے۔
  
</Tab>
  <Tab title="براہِ راست ترمیم">
    `~/.openclaw/openclaw.json` کو براہِ راست ایڈٹ کریں۔ Gateway فائل کو واچ کرتا ہے اور تبدیلیاں خودکار طور پر لاگو کرتا ہے (دیکھیں [hot reload](#config-hot-reload))۔
  
</Tab>
</Tabs>

## سخت توثیق

<Warning>
OpenClaw صرف وہی کنفیگریشن قبول کرتا ہے جو مکمل طور پر اسکیما سے میل کھاتی ہو۔ نامعلوم keys، غلط types یا غیر معتبر values کی صورت میں Gateway **شروع ہونے سے انکار** کر دیتا ہے۔ روٹ لیول پر واحد استثناء `$schema` (string) ہے تاکہ ایڈیٹرز JSON Schema میٹا ڈیٹا منسلک کر سکیں۔
</Warning>

جب توثیق ناکام ہو:

- Gateway بوٹ نہیں ہوتا  
- صرف تشخیصی کمانڈز کام کرتی ہیں (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)  
- درست مسائل دیکھنے کے لیے `openclaw doctor` چلائیں  
- مرمت لاگو کرنے کے لیے `openclaw doctor --fix` (یا `--yes`) چلائیں  

## عام کام

<AccordionGroup>
  <Accordion title="چینل سیٹ اپ کریں (WhatsApp, Telegram, Discord وغیرہ)">
    ہر چینل کی اپنی کنفیگ سیکشن ہوتی ہے `channels.<provider>` کے تحت۔ سیٹ اپ کے مراحل کے لیے متعلقہ چینل صفحہ دیکھیں:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    تمام چینلز ایک جیسا DM پالیسی پیٹرن شیئر کرتے ہیں:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```
  
</Accordion>

  <Accordion title="ماڈلز منتخب اور کنفیگر کریں">
    بنیادی ماڈل اور اختیاری فالبیکس سیٹ کریں:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` ماڈل کیٹلاگ کی تعریف کرتا ہے اور `/model` کے لیے allowlist کا کردار ادا کرتا ہے۔  
    - ماڈل ریفرنس `provider/model` فارمیٹ استعمال کرتے ہیں (مثلاً `anthropic/claude-opus-4-6`)۔  
    - چیٹ میں ماڈل سوئچ کرنے کے لیے [Models CLI](/concepts/models) اور آتھ روٹیشن/فالبیک رویے کے لیے [Model Failover](/concepts/model-failover) دیکھیں۔  
    - کسٹم یا self-hosted پرووائیڈرز کے لیے [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) دیکھیں۔  
  
</Accordion>

  <Accordion title="کنٹرول کریں کہ کون بوٹ کو میسج کر سکتا ہے">
    DM رسائی فی چینل `dmPolicy` کے ذریعے کنٹرول ہوتی ہے:

    - `"pairing"` (ڈیفالٹ): نامعلوم بھیجنے والوں کو ایک مرتبہ استعمال ہونے والا کوڈ ملتا ہے  
    - `"allowlist"`: صرف `allowFrom` میں شامل بھیجنے والے  
    - `"open"`: تمام inbound DMs کی اجازت (لازم ہے `allowFrom: ["*"]`)  
    - `"disabled"`: تمام DMs نظر انداز  

    گروپس کے لیے `groupPolicy` + `groupAllowFrom` یا چینل مخصوص allowlists استعمال کریں۔

    مکمل تفصیل کے لیے [مکمل حوالہ](/gateway/configuration-reference#dm-and-group-access) دیکھیں۔
  
</Accordion>

  <Accordion title="گروپ چیٹ منشن گیٹنگ سیٹ اپ کریں">
    گروپ پیغامات بطورِ ڈیفالٹ **منشن درکار** ہوتے ہیں۔ فی ایجنٹ پیٹرنز کنفیگر کریں:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Metadata mentions**: پلیٹ فارم کے مقامی @-منشنز  
    - **Text patterns**: `mentionPatterns` میں regex پیٹرنز  
    - مزید تفصیل کے لیے [مکمل حوالہ](/gateway/configuration-reference#group-chat-mention-gating) دیکھیں۔  
  
</Accordion>

  <Accordion title="سیشنز اور ری سیٹس کنفیگر کریں">
    سیشنز گفتگو کے تسلسل اور آئسولیشن کو کنٹرول کرتے ہیں:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // multi-user کے لیے تجویز کردہ
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`  
    - مزید تفصیل کے لیے [Session Management](/concepts/session) اور [مکمل حوالہ](/gateway/configuration-reference#session) دیکھیں۔  
  
</Accordion>

  <Accordion title="سینڈ باکسنگ فعال کریں">
    ایجنٹ سیشنز کو آئسولیٹڈ Docker کنٹینرز میں چلائیں:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    پہلے امیج بنائیں: `scripts/sandbox-setup.sh`  

    مکمل رہنمائی کے لیے [Sandboxing](/gateway/sandboxing) دیکھیں۔
  
</Accordion>

  <Accordion title="Heartbeat سیٹ اپ کریں (وقفہ وار چیک ان)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: دورانیہ (`30m`, `2h`)۔ `0m` پر سیٹ کریں تو غیر فعال ہو جائے گا۔  
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`  
    - مکمل گائیڈ کے لیے [Heartbeat](/gateway/heartbeat) دیکھیں۔  
  
</Accordion>

  <Accordion title="Cron جابز کنفیگر کریں">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    مزید معلومات کے لیے [Cron jobs](/automation/cron-jobs) دیکھیں۔
  
</Accordion>

  <Accordion title="Webhooks (hooks) سیٹ اپ کریں">
    Gateway پر HTTP webhook endpoints فعال کریں:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    تمام mapping آپشنز کے لیے [مکمل حوالہ](/gateway/configuration-reference#hooks) دیکھیں۔
  
</Accordion>

  <Accordion title="ملٹی ایجنٹ روٹنگ کنفیگر کریں">
    متعدد آئسولیٹڈ ایجنٹس چلائیں:

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    مزید تفصیل کے لیے [Multi-Agent](/concepts/multi-agent) دیکھیں۔
  
</Accordion>

  <Accordion title="کنفیگ کو متعدد فائلوں میں تقسیم کریں (`$include`)">
    بڑی کنفیگز کو منظم کرنے کے لیے `$include` استعمال کریں:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **Single file**: پورے آبجیکٹ کو replace کرتا ہے  
    - **Array of files**: ترتیب کے مطابق deep-merge  
    - **Sibling keys**: includes کے بعد merge ہوتے ہیں  
    - **Nested includes**: 10 لیول تک سپورٹ  
    - **Relative paths**: شامل کرنے والی فائل کے نسبت سے resolve  
    - **Error handling**: گمشدہ فائل، parse ایرر یا circular include پر واضح ایرر  
  
</Accordion>
</AccordionGroup>

## Config hot reload

Gateway `~/.openclaw/openclaw.json` کو واچ کرتا ہے اور زیادہ تر سیٹنگز کے لیے تبدیلیاں خودکار طور پر لاگو کر دیتا ہے — دستی ری اسٹارٹ کی ضرورت نہیں۔

### Reload موڈز

| Mode                   | Behavior                                                                                |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (default) | محفوظ تبدیلیاں فوری hot-apply۔ اہم تبدیلیوں پر خودکار ری اسٹارٹ۔                     |
| **`hot`**              | صرف محفوظ تبدیلیاں hot-apply؛ ری اسٹارٹ درکار ہو تو وارننگ لاگ۔                     |
| **`restart`**          | ہر کنفیگ تبدیلی پر گیٹ وے ری اسٹارٹ۔                                                  |
| **`off`**              | فائل واچنگ بند؛ تبدیلیاں اگلے دستی ری اسٹارٹ پر لاگو ہوں گی۔                        |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

## مکمل حوالہ

تمام فیلڈز کی تفصیلی وضاحت کے لیے دیکھیں:  
**[Configuration Reference](/gateway/configuration-reference)**

---

_Related: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_


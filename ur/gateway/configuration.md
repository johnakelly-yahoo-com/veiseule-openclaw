---
summary: "‏~/.openclaw/openclaw.json کے لیے تمام کنفیگریشن اختیارات مثالوں کے ساتھ"
read_when:
  - کنفیگ فیلڈز شامل کرتے یا ترمیم کرتے وقت
  - عام کنفیگریشن پیٹرنز تلاش کر رہے ہیں
  - مخصوص کنفیگ سیکشنز تک نیویگیٹ کرنا
title: "کنفیگریشن"
---

# کنفیگریشن 🔧

OpenClaw ایک اختیاری **JSON5** کنفیگ `~/.openclaw/openclaw.json` سے پڑھتا ہے (تبصرے + آخر میں کاما کی اجازت ہے)۔

اگر فائل موجود نہ ہو تو OpenClaw محفوظ ڈیفالٹس استعمال کرتا ہے۔ کنفیگ شامل کرنے کی عام وجوہات:

- اس بات کو محدود کرنا چاہیں کہ بوٹ کو کون ٹرگر کر سکتا ہے (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` وغیرہ)
- گروپ اجازت فہرستیں اور منشن رویہ کنٹرول کریں (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- پیغام کے سابقے حسبِ ضرورت بنائیں (`messages`)

ہر دستیاب فیلڈ کے لیے [full reference](/gateway/configuration-reference) دیکھیں۔

<Tip>
**کنفیگریشن میں نئے ہیں؟** انٹرایکٹو سیٹ اپ کے لیے `openclaw onboard` سے آغاز کریں، یا مکمل کاپی-پیسٹ کنفیگز کے لیے [Configuration Examples](/gateway/configuration-examples) گائیڈ دیکھیں۔
</Tip>

## کم سے کم کنفیگ

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## کنفیگ میں ترمیم

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # مکمل سیٹ اپ وزرڈ
    openclaw configure     # کنفیگ وزرڈ
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) کھولیں اور **Config** ٹیب استعمال کریں۔
    Control UI کنفیگ اسکیما سے ایک فارم رینڈر کرتا ہے، اور بطور متبادل **Raw JSON** ایڈیٹر بھی فراہم کرتا ہے۔
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` کو براہِ راست ایڈٹ کریں۔ Gateway فائل کو مانیٹر کرتا ہے اور تبدیلیاں خودکار طور پر لاگو کرتا ہے (دیکھیں [hot reload](#config-hot-reload))۔
  
</Tab>
</Tabs>

## اسکیما + UI اشارے

<Warning>
2. OpenClaw صرف وہی کنفیگریشنز قبول کرتا ہے جو مکمل طور پر اسکیما سے میل کھاتی ہوں۔ نامعلوم کیز، غلط ٹائپس، یا ناجائز ویلیوز کی صورت میں Gateway **شروع ہونے سے انکار کر دیتا ہے**۔ روٹ لیول پر واحد استثنا `$schema` (string) ہے، تاکہ ایڈیٹرز JSON Schema میٹاڈیٹا منسلک کر سکیں۔
</Warning>

چینل پلگ اِنز اور ایکسٹینشنز اپنی کنفیگ کے لیے اسکیما + UI اشارے رجسٹر کر سکتے ہیں، تاکہ
چینل سیٹنگز مختلف ایپس میں بغیر ہارڈ کوڈڈ فارمز کے اسکیما پر مبنی رہیں۔

- Gateway شروع نہیں ہوتا
- صرف تشخیصی کمانڈز کام کرتی ہیں (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- درست مسائل دیکھنے کے لیے `openclaw doctor` چلائیں
- مرمت لاگو کرنے کے لیے `openclaw doctor --fix` (یا `--yes`) چلائیں

## لاگو کریں + ری اسٹارٹ (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    ہر چینل کی اپنی کنفیگ سیکشن `channels.` کے تحت ہوتی ہے<provider>`. سیٹ اپ مراحل کے لیے متعلقہ چینل صفحہ دیکھیں:

    ````
    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`
    
    تمام چینلز ایک ہی DM پالیسی پیٹرن شیئر کرتے ہیں:
    
    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // صرف allowlist/open کے لیے
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Choose and configure models">
    بنیادی ماڈل اور اختیاری فال بیکس سیٹ کریں:

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">
    DM رسائی ہر چینل کے لیے `dmPolicy` کے ذریعے کنٹرول ہوتی ہے:

    ```
    - `"pairing"` (ڈیفالٹ): نامعلوم بھیجنے والوں کو منظوری کے لیے ایک بار استعمال ہونے والا پیئرنگ کوڈ ملتا ہے
    - `"allowlist"`: صرف وہ بھیجنے والے جو `allowFrom` میں ہوں (یا پیئر شدہ allow اسٹور میں)
    - `"open"`: تمام آنے والے DMs کی اجازت (ضروری ہے `allowFrom: ["*"]`)
    - `"disabled"`: تمام DMs کو نظرانداز کریں
    
    گروپس کے لیے، `groupPolicy` + `groupAllowFrom` یا چینل-مخصوص allowlists استعمال کریں۔
    
    ہر چینل کی تفصیلات کے لیے [full reference](/gateway/configuration-reference#dm-and-group-access) دیکھیں۔
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    گروپ پیغامات کا ڈیفالٹ **require mention** ہے۔ ہر ایجنٹ کے لیے پیٹرنز کنفیگر کریں:

    ````
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
    
    - **میٹاڈیٹا مینشنز**: مقامی @-مینشنز (WhatsApp میں ٹیپ-ٹو-مینشن، Telegram @bot، وغیرہ)
    - **ٹیکسٹ پیٹرنز**: `mentionPatterns` میں regex پیٹرنز
    - فی چینل اووررائیڈز اور self-chat موڈ کے لیے [full reference](/gateway/configuration-reference#group-chat-mention-gating) دیکھیں۔
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    سیشنز گفتگو کے تسلسل اور علیحدگی کو کنٹرول کرتے ہیں:

    ```
    {
      agents: { defaults: { workspace: "~/.openclaw/workspace" } },
      channels: { whatsapp: { allowFrom: ["+15555550123"] } },
    }
    ```

  
</Accordion>

  <Accordion title="Enable sandboxing">
    ایجنٹ سیشنز کو علیحدہ Docker کنٹینرز میں چلائیں:

    ````
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
    
    مکمل گائیڈ کے لیے [Sandboxing](/gateway/sandboxing) اور تمام آپشنز کے لیے [full reference](/gateway/configuration-reference#sandbox) دیکھیں۔
    ````

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
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

    ```
    - `every`: دورانیے کی اسٹرنگ (`30m`, `2h`)۔ غیر فعال کرنے کے لیے `0m` سیٹ کریں۔
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - مکمل رہنمائی کے لیے [Heartbeat](/gateway/heartbeat) دیکھیں۔
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
  },
}
```

    ```
    See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Gateway پر HTTP webhook endpoints فعال کریں:

    ````
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
    
    تمام mapping اختیارات اور Gmail انضمام کے لیے [full reference](/gateway/configuration-reference#hooks) دیکھیں۔
    ````

  
</Accordion>

  <Accordion title="Configure multi-agent routing">الگ الگ workspaces اور sessions کے ساتھ متعدد isolated agents چلائیں:

    ````
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
    
    binding کے قواعد اور ہر agent کے رسائی پروفائلز کے لیے [Multi-Agent](/concepts/multi-agent) اور [full reference](/gateway/configuration-reference#multi-agent-routing) دیکھیں۔
    ````

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">بڑے configs کو منظم کرنے کے لیے `$include` استعمال کریں:

    ````
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
    
    - **Single file**: موجودہ object کو مکمل طور پر تبدیل کر دیتا ہے
    - **Array of files**: ترتیب وار deep-merge ہوتے ہیں (بعد والا غالب رہتا ہے)
    - **Sibling keys**: include کے بعد merge ہوتے ہیں (شامل شدہ اقدار کو override کرتے ہیں)
    - **Nested includes**: زیادہ سے زیادہ 10 سطحوں تک معاونت
    - **Relative paths**: شامل کرنے والی فائل کے نسبت سے resolve ہوتے ہیں
    - **Error handling**: گمشدہ فائلز، parse errors، اور circular includes کے لیے واضح غلطیاں
    ````

  
</Accordion>
</AccordionGroup>

## Config کی hot reload

Gateway `~/.openclaw/openclaw.json` کو مانیٹر کرتا ہے اور تبدیلیاں خودکار طور پر لاگو کرتا ہے — زیادہ تر ترتیبات کے لیے دستی restart کی ضرورت نہیں ہوتی۔

### Reload کے طریقے

| Mode                                             | رویّہ                                                                                                  |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **`hybrid`** (پہلے سے طے شدہ) | محفوظ تبدیلیاں فوراً hot-apply کرتا ہے۔ اہم تبدیلیوں کے لیے خودکار طور پر restart کرتا ہے۔             |
| **`hot`**                                        | صرف محفوظ تبدیلیاں hot-apply کرتا ہے۔ جب restart درکار ہو تو وارننگ لاگ کرتا ہے — آپ خود سنبھالتے ہیں۔ |
| **`restart`**                                    | کسی بھی config تبدیلی پر Gateway کو restart کرتا ہے، چاہے محفوظ ہو یا نہ ہو۔                           |
| **`off`**                                        | فائل مانیٹرنگ غیر فعال کر دیتا ہے۔ تبدیلیاں اگلے دستی restart پر لاگو ہوں گی۔                          |

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

### کیا چیز hot-apply ہوتی ہے اور کس کے لیے restart درکار ہے

زیادہ تر فیلڈز بغیر downtime کے hot-apply ہو جاتی ہیں۔ `hybrid` موڈ میں، جن تبدیلیوں کے لیے restart درکار ہو وہ خودکار طور پر سنبھال لی جاتی ہیں۔

| Category          | Fields                                                                                   | Restart درکار ہے؟ |
| ----------------- | ---------------------------------------------------------------------------------------- | ----------------- |
| Channels          | `channels.*`, `web` (WhatsApp) — تمام built-in اور extension channels | نہیں              |
| Agent اور models  | `agent`, `agents`, `models`, `routing`                                                   | نہیں              |
| Automation        | `hooks`, `cron`, `agent.heartbeat`                                                       | نہیں              |
| سیشنز اور پیغامات | `session`, `messages`                                                                    | نہیں              |
| ٹولز اور میڈیا    | `tools`, `browser`, `skills`, `audio`, `talk`                                            | نہیں              |
| UI اور متفرق      | `ui`, `logging`, `identity`, `bindings`                                                  | نہیں              |
| Gateway سرور      | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                  | **ہاں**           |
| انفراسٹرکچر       | `discovery`, `canvasHost`, `plugins`                                                     | **ہاں**           |

<Note>
`gateway.reload` اور `gateway.remote` مستثنیات ہیں — ان میں تبدیلی کرنے سے **ری اسٹارٹ** شروع نہیں ہوتا۔
</Note>

## Config RPC (پروگراماتی اپڈیٹس)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    مکمل کنفیگ کی توثیق کرتا ہے + اسے لکھتا ہے اور ایک ہی مرحلے میں Gateway کو ری اسٹارٹ کرتا ہے۔


    ````
    <Warning>
    `config.apply` **مکمل کنفیگ** کو تبدیل کر دیتا ہے۔ جزوی اپڈیٹس کے لیے `config.patch` استعمال کریں، یا ایکल keys کے لیے `openclaw config set` استعمال کریں۔
    
</Warning>
    
    Params:
    
    - `raw` (string) — مکمل کنفیگ کے لیے JSON5 payload
    - `baseHash` (optional) — `config.get` سے حاصل کردہ کنفیگ hash (جب کنفیگ موجود ہو تو درکار)
    - `sessionKey` (optional) — ری اسٹارٹ کے بعد wake-up ping کے لیے سیشن key
    - `note` (optional) — ری اسٹارٹ sentinel کے لیے نوٹ
    - `restartDelayMs` (optional) — ری اسٹارٹ سے پہلے تاخیر (ڈیفالٹ 2000)
    
    ```bash
    openclaw gateway call config.get --params '{}'  # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    موجودہ کنفیگ میں جزوی اپڈیٹ ضم کرتا ہے (JSON merge patch semantics):

    ```
    {
      env: {
        OPENROUTER_API_KEY: "sk-or-...",
        vars: {
          GROQ_API_KEY: "gsk-...",
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## ماحولیاتی متغیرات

OpenClaw والدین پراسیس کے env vars کے ساتھ ساتھ درج ذیل سے بھی پڑھتا ہے:

- 17. اس کے علاوہ، یہ لوڈ کرتا ہے:
- `~/.openclaw/.env` (عالمی fallback)

کوئی بھی فائل موجودہ env vars کو اووررائیڈ نہیں کرتی۔ آپ کنفیگ میں inline env vars بھی سیٹ کر سکتے ہیں:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  اگر فعال ہو اور متوقع keys سیٹ نہ ہوں تو OpenClaw آپ کا login shell چلاتا ہے اور صرف گمشدہ keys درآمد کرتا ہے:


```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var کے مساوی: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  کسی بھی کنفیگ اسٹرنگ ویلیو میں env vars کو `${VAR_NAME}` کے ساتھ حوالہ دیں:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

قواعد:

- صرف بڑے حروف والے نام میچ ہوتے ہیں: `[A-Z_][A-Z0-9_]*`
- گمشدہ/خالی متغیرات لوڈ کے وقت خرابی پیدا کرتے ہیں
- لفظی آؤٹ پٹ کے لیے `$${VAR}` استعمال کریں
- `$include` فائلز کے اندر بھی کام کرتا ہے
- Inline substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

OpenClaw stores **per-agent** auth profiles (OAuth + API keys) in:

## مکمل حوالہ

مکمل فیلڈ بہ فیلڈ حوالہ کے لیے، **[Configuration Reference](/gateway/configuration-reference)** دیکھیں۔

---

_Related: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_

---
title: "کنفیگریشن"
---

# کنفیگریشن 🔧

OpenClaw ایک اختیاری **JSON5** کنفیگ `~/.openclaw/openclaw.json` سے پڑھتا ہے (تبصرے + آخر میں کاما کی اجازت ہے)۔

If the file is missing, OpenClaw uses safe-ish defaults (embedded Pi agent + per-sender sessions + workspace `~/.openclaw/workspace`). You usually only need a config to:

- اس بات کو محدود کرنا چاہیں کہ بوٹ کو کون ٹرگر کر سکتا ہے (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` وغیرہ)
- گروپ اجازت فہرستیں اور منشن رویہ کنٹرول کریں (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- پیغام کے سابقے حسبِ ضرورت بنائیں (`messages`)
- ایجنٹ کا ورک اسپیس سیٹ کریں (`agents.defaults.workspace` یا `agents.list[].workspace`)
- ایمبیڈڈ ایجنٹ کے ڈیفالٹس (`agents.defaults`) اور سیشن رویے (`session`) کو ٹیون کریں
- ہر ایجنٹ کی شناخت سیٹ کریں (`agents.list[].identity`)

> **کنفیگریشن میں نئے ہیں؟** مکمل مثالوں اور تفصیلی وضاحتوں کے لیے [Configuration Examples](/gateway/configuration-examples) گائیڈ دیکھیں!

## سخت کنفیگ کی توثیق

2. OpenClaw صرف وہی کنفیگریشنز قبول کرتا ہے جو مکمل طور پر اسکیما سے میل کھاتی ہوں۔
3. نامعلوم keys، خراب اقسام (types)، یا غلط قدریں (values) حفاظتی وجہ سے Gateway کو **شروع ہونے سے انکار** کرنے پر مجبور کر دیتی ہیں۔

جب توثیق ناکام ہو:

- Gateway بوٹ نہیں ہوتا۔
- صرف تشخیصی کمانڈز کی اجازت ہوتی ہے (مثلاً: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`)۔
- درست مسائل دیکھنے کے لیے `openclaw doctor` چلائیں۔
- مائیگریشن/مرمت لاگو کرنے کے لیے `openclaw doctor --fix` (یا `--yes`) چلائیں۔

Doctor کبھی تبدیلیاں نہیں لکھتا جب تک آپ واضح طور پر `--fix`/`--yes` میں شامل نہ ہوں۔

## اسکیما + UI اشارے

4. Gateway UI ایڈیٹرز کے لیے کنفیگ کی JSON Schema نمائندگی `config.schema` کے ذریعے فراہم کرتا ہے۔
5. Control UI اس اسکیما سے ایک فارم رینڈر کرتا ہے، اور بطور متبادل ایک **Raw JSON** ایڈیٹر بھی فراہم کرتا ہے۔

چینل پلگ اِنز اور ایکسٹینشنز اپنی کنفیگ کے لیے اسکیما + UI اشارے رجسٹر کر سکتے ہیں، تاکہ
چینل سیٹنگز مختلف ایپس میں بغیر ہارڈ کوڈڈ فارمز کے اسکیما پر مبنی رہیں۔

اشارے (لیبلز، گروپنگ، حساس فیلڈز) اسکیما کے ساتھ فراہم کیے جاتے ہیں تاکہ کلائنٹس
بغیر کنفیگ علم ہارڈ کوڈ کیے بہتر فارمز رینڈر کر سکیں۔

## لاگو کریں + ری اسٹارٹ (RPC)

6. ایک ہی مرحلے میں مکمل کنفیگ کی توثیق + تحریر اور Gateway کو ری اسٹارٹ کرنے کے لیے `config.apply` استعمال کریں۔
7. Gateway کے دوبارہ آنے کے بعد یہ ایک ری اسٹارٹ سینٹینل لکھتا ہے اور آخری فعال سیشن کو پِنگ کرتا ہے۔

8. انتباہ: `config.apply` **مکمل کنفیگ** کو تبدیل کر دیتا ہے۔ 9. اگر آپ صرف چند keys تبدیل کرنا چاہتے ہیں، use `config.patch` یا `openclaw config set` استعمال کریں۔

Params:

- `raw` (string) — پوری کنفیگ کے لیے JSON5 پے لوڈ
- `baseHash` (اختیاری) — `config.get` سے کنفیگ ہیش (جب کنفیگ پہلے سے موجود ہو تو لازم)
- `sessionKey` (اختیاری) — ویک اپ پِنگ کے لیے آخری فعال سیشن کلید
- `note` (اختیاری) — ری اسٹارٹ سینٹینل میں شامل کرنے کے لیے نوٹ
- `restartDelayMs` (اختیاری) — ری اسٹارٹ سے پہلے تاخیر (ڈیفالٹ 2000)

مثال (`gateway call` کے ذریعے):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## جزوی اپ ڈیٹس (RPC)

10. `~/.openclaw/openclaw.json` کا بیک اپ محفوظ رکھیں۔ 11. موجودہ کنفیگ میں جزوی اپ ڈیٹ کو بغیر غیر متعلقہ keys کو متاثر کیے ضم کرنے کے لیے `config.patch` استعمال کریں۔

- آبجیکٹس ریکرسیولی ضم ہوتے ہیں
- `null` کسی کلید کو حذف کرتا ہے
- arrays مکمل طور پر بدل دیے جاتے ہیں  
  `config.apply` کی طرح، یہ توثیق کرتا ہے، کنفیگ لکھتا ہے، ری اسٹارٹ سینٹینل محفوظ کرتا ہے، اور
  Gateway ری اسٹارٹ شیڈول کرتا ہے (جب `sessionKey` فراہم ہو تو اختیاری ویک کے ساتھ)۔

Params:

- `raw` (string) — صرف تبدیل ہونے والی کلیدوں پر مشتمل JSON5 پے لوڈ
- `baseHash` (لازم) — `config.get` سے کنفیگ ہیش
- `sessionKey` (اختیاری) — ویک اپ پِنگ کے لیے آخری فعال سیشن کلید
- `note` (اختیاری) — ری اسٹارٹ سینٹینل میں شامل کرنے کے لیے نوٹ
- `restartDelayMs` (اختیاری) — ری اسٹارٹ سے پہلے تاخیر (ڈیفالٹ 2000)

مثال:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## کم از کم کنفیگ (سفارش کردہ ابتدائی نقطہ)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

ڈیفالٹ امیج ایک بار اس طرح بنائیں:

```bash
scripts/sandbox-setup.sh
```

## سیلف-چیٹ موڈ (گروپ کنٹرول کے لیے سفارش کردہ)

WhatsApp گروپس میں @-منشنز پر بوٹ کے جواب کو روکنے کے لیے (صرف مخصوص متنی ٹرگرز پر جواب):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
      },
    ],
  },
  channels: {
    whatsapp: {
      // Allowlist is DMs only; including your own number enables self-chat mode.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## کنفیگ Includes (`$include`)

12. یہ JSON merge patch semantics لاگو کرتا ہے: This is useful for:

- بڑی کنفیگز کو منظم کرنے کے لیے (مثلاً ہر کلائنٹ کے لیے ایجنٹ تعریفیں)
- مختلف ماحولوں میں مشترکہ سیٹنگز شیئر کرنے کے لیے
- حساس کنفیگز کو الگ رکھنے کے لیے

### بنیادی استعمال

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },

  // Include a single file (replaces the key's value)
  agents: { $include: "./agents.json5" },

  // Include multiple files (deep-merged in order)
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### مرج رویہ

- **ایک فائل**: `$include` رکھنے والے آبجیکٹ کو بدل دیتی ہے
- **فائلوں کی array**: ترتیب کے مطابق ڈیپ مرج (بعد والی فائلیں پہلے والیوں کو اوور رائیڈ کرتی ہیں)
- **ہمسایہ کلیدوں کے ساتھ**: includes کے بعد ہمسایہ کلیدیں مرج ہوتی ہیں (شامل شدہ اقدار کو اوور رائیڈ کرتی ہیں)
- **ہمسایہ کلیدیں + arrays/primitives**: سپورٹڈ نہیں (شامل شدہ مواد لازماً آبجیکٹ ہونا چاہیے)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### نیسٹڈ includes

شامل شدہ فائلیں خود بھی `$include` ڈائریکٹوز رکھ سکتی ہیں (زیادہ سے زیادہ 10 سطحیں):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### راستے کی ریزولوشن

- **نسبتی راستے**: شامل کرنے والی فائل کے نسبت سے حل ہوتے ہیں
- **مطلق راستے**: جوں کے توں استعمال ہوتے ہیں
- **پیرنٹ ڈائریکٹریز**: `../` حوالہ جات متوقع طور پر کام کرتے ہیں

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### خرابیوں کا ازالہ

- **گمشدہ فائل**: حل شدہ راستے کے ساتھ واضح خرابی
- **پارْس خرابی**: بتاتا ہے کون سی شامل شدہ فائل ناکام ہوئی
- **سرکولر includes**: include چین کے ساتھ شناخت اور رپورٹ

### مثال: ملٹی کلائنٹ قانونی سیٹ اپ

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

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## Common options

### Env vars + `.env`

OpenClaw reads env vars from the parent process (shell, launchd/systemd, CI, etc.).

Additionally, it loads:

- 17. اس کے علاوہ، یہ لوڈ کرتا ہے:
- a global fallback `.env` from `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`)

Neither `.env` file overrides existing env vars.

You can also provide inline env vars in config. 21. آپ کنفیگ میں inline env vars بھی فراہم کر سکتے ہیں۔

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

See [/environment](/help/environment) for full precedence and sources.

### 23. مکمل ترجیح اور ذرائع کے لیے [/environment](/help/environment) دیکھیں۔

Opt-in convenience: if enabled and none of the expected keys are set yet, OpenClaw runs your login shell and imports only the missing expected keys (never overrides).
25. Opt-in سہولت: اگر فعال ہو اور متوقع keys میں سے کوئی بھی ابھی سیٹ نہ ہو، تو OpenClaw آپ کا لاگ اِن شیل چلاتا ہے اور صرف وہی متوقع keys امپورٹ کرتا ہے جو غائب ہوں (کبھی اووررائیڈ نہیں کرتا)۔

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Env var equivalent:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### کنفیگ میں Env var متبادل

27. Env var مساوی: Variables are substituted at config load time, before validation.

```json5
29. ویری ایبلز کی جگہ کنفیگ لوڈ کے وقت، ویلیڈیشن سے پہلے کی جاتی ہے۔
```

**Rules:**

- 31. **قواعد:**
- 32. صرف بڑے حروف والے env var نام میچ ہوتے ہیں: `[A-Z_][A-Z0-9_]*`
- 33. غائب یا خالی env vars کنفیگ لوڈ کے وقت ایک ایرر پیدا کرتے ہیں۔
- 34. لفظی `${VAR}` آؤٹ پٹ کرنے کے لیے `$${VAR}` کے ساتھ escape کریں۔

35. `$include` کے ساتھ کام کرتا ہے (شامل کی گئی فائلوں میں بھی سبسٹی ٹیوشن ہوتی ہے)۔

```json5
36. **Inline substitution:**
```

### 37. {&#xA;models: {&#xA;providers: {&#xA;custom: {&#xA;baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"&#xA;},&#xA;},&#xA;},&#xA;}

OpenClaw stores **per-agent** auth profiles (OAuth + API keys) in:

- 39. OpenClaw **ہر ایجنٹ کے لیے الگ** auth پروفائلز (OAuth + API keys) یہاں محفوظ کرتا ہے:

40. `<agentDir>/auth-profiles.json` (ڈیفالٹ: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

41. یہ بھی دیکھیں: [/concepts/oauth](/concepts/oauth)

- 42. Legacy OAuth امپورٹس:

43. `~/.openclaw/credentials/oauth.json` (یا `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

- 44. ایمبیڈڈ Pi ایجنٹ رَن ٹائم کیش یہاں برقرار رکھتا ہے:

45. `<agentDir>/auth.json` (خودکار طور پر منیج ہوتا ہے؛ دستی طور پر ترمیم نہ کریں)

- 46. Legacy ایجنٹ ڈائریکٹری (ملٹی ایجنٹ سے پہلے):

47. `~/.openclaw/agent/*` (`openclaw doctor` کے ذریعے `~/.openclaw/agents/<defaultAgentId>/agent/*` میں مائیگریٹ کیا جاتا ہے)

- OAuth dir (legacy import only): `OPENCLAW_OAUTH_DIR`
- 49. OAuth ڈائریکٹری (صرف legacy امپورٹ): `OPENCLAW_OAUTH_DIR`

On first use, OpenClaw imports `oauth.json` entries into `auth-profiles.json`.

### `تصدیق`

Optional metadata for auth profiles. This does **not** store secrets; it maps
profile IDs to a provider + mode (and optional email) and defines the provider
rotation order used for failover.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

### `agents.list[].identity`

Optional per-agent identity used for defaults and UX. This is written by the macOS onboarding assistant.

If set, OpenClaw derives defaults (only when you haven’t set them explicitly):

- `messages.ackReaction` from the **active agent**’s `identity.emoji` (falls back to 👀)
- `agents.list[].groupChat.mentionPatterns` from the agent’s `identity.name`/`identity.emoji` (so “@Samantha” works in groups across Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
- `identity.avatar` accepts a workspace-relative image path or a remote URL/data URL. Local files must live inside the agent workspace.

`identity.avatar` accepts:

- Workspace-relative path (must stay within the agent workspace)
- `http(s)` URL
- `data:` URI

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

### `wizard`

Metadata written by CLI wizards (`onboard`, `configure`, `doctor`).

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

### `لاگنگ`

- Default log file: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- If you want a stable path, set `logging.file` to `/tmp/openclaw/openclaw.log`.
- Console output can be tuned separately via:
  - `logging.consoleLevel` (defaults to `info`, bumps to `debug` when `--verbose`)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- Tool summaries can be redacted to avoid leaking secrets:
  - `logging.redactSensitive` (`off` | `tools`, default: `tools`)
  - `logging.redactPatterns` (array of regex strings; overrides defaults)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

Controls how WhatsApp direct chats (DMs) are handled:

- `"pairing"` (default): unknown senders get a pairing code; owner must approve
- `"allowlist"`: only allow senders in `channels.whatsapp.allowFrom` (or paired allow store)
- `"open"`: allow all inbound DMs (**requires** `channels.whatsapp.allowFrom` to include `"*"`)
- `"disabled"`: ignore all inbound DMs

Pairing codes expire after 1 hour; the bot only sends a pairing code when a new request is created. Pending DM pairing requests are capped at **3 per channel** by default.

Pairing approvals:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

Allowlist of E.164 phone numbers that may trigger WhatsApp auto-replies (**DMs only**).
If empty and `channels.whatsapp.dmPolicy="pairing"`, unknown senders will receive a pairing code.
For groups, use `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50, // optional inbound media cap (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

Controls whether inbound WhatsApp messages are marked as read (blue ticks). Default: `true`.

Self-chat mode always skips read receipts, even when enabled.

Per-account override: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (multi-account)

Run multiple WhatsApp accounts in one gateway:

```json5
1. {
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

نوٹس:

- 2. آؤٹ باؤنڈ کمانڈز بطورِ ڈیفالٹ اکاؤنٹ `default` استعمال کرتی ہیں اگر موجود ہو؛ بصورت دیگر پہلا کنفیگر کیا گیا اکاؤنٹ آئی ڈی (ترتیب وار) استعمال کیا جاتا ہے۔
- The legacy single-account Baileys auth dir is migrated by `openclaw doctor` into `whatsapp/default`.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

5. ہر چینل پر متعدد اکاؤنٹس چلائیں (ہر اکاؤنٹ کا اپنا `accountId` اور اختیاری `name` ہوتا ہے):

```json5
6. {
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

نوٹس:

- 7. جب `accountId` فراہم نہ کیا جائے تو `default` استعمال ہوتا ہے (CLI + routing)۔
- Env tokens only apply to the **default** account.
- 9. بنیادی چینل سیٹنگز (گروپ پالیسی، منشن گیٹنگ، وغیرہ) 10. تمام اکاؤنٹس پر لاگو ہوتی ہیں جب تک کہ فی اکاؤنٹ اووررائیڈ نہ کیا جائے۔
- 11. ہر اکاؤنٹ کو مختلف agents.defaults کی طرف روٹ کرنے کے لیے `bindings[].match.accountId` استعمال کریں۔

### Group chat mention gating (`agents.list[].groupChat` + `messages.groupChat`)

13. گروپ پیغامات بطورِ ڈیفالٹ **منشن درکار** ہوتے ہیں (یا تو میٹاڈیٹا منشن یا regex پیٹرنز)۔ 14. یہ WhatsApp، Telegram، Discord، Google Chat، اور iMessage گروپ چیٹس پر لاگو ہوتا ہے۔

15. **منشن کی اقسام:**

- 16. **میٹاڈیٹا منشنز**: پلیٹ فارم کے مقامی @-منشنز (مثلاً WhatsApp میں tap-to-mention)۔ 17. WhatsApp self-chat موڈ میں نظرانداز کیے جاتے ہیں (دیکھیں `channels.whatsapp.allowFrom`)۔
- **Text patterns**: Regex patterns defined in `agents.list[].groupChat.mentionPatterns`. Always checked regardless of self-chat mode.
- 20. منشن گیٹنگ صرف اسی وقت نافذ ہوتی ہے جب منشن کی شناخت ممکن ہو (مقامی منشنز یا کم از کم ایک `mentionPattern`)۔

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` sets the global default for group history context. 23. چینلز `channels.<channel>` کے ذریعے اووررائیڈ کر سکتے ہیں24. `.historyLimit` (یا `channels.<channel>`25. `.accounts.*.historyLimit` ملٹی اکاؤنٹ کے لیے)۔ Set `0` to disable history wrapping.

#### 27. DM ہسٹری کی حدود

DM conversations use session-based history managed by the agent. You can limit the number of user turns retained per DM session:

```json5
30. {
  channels: {
    telegram: {
      dmHistoryLimit: 30, // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

Resolution order:

1. 31. فی-DM اووررائیڈ: `channels.<provider>`.dms[userId].historyLimit\`
2. Provider default: `channels.<provider>34. `.dmHistoryLimit\`
3. 35. کوئی حد نہیں (تمام ہسٹری محفوظ رکھی جاتی ہے)

36) سپورٹ شدہ فراہم کنندگان: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`۔

37. فی-ایجنٹ اووررائیڈ (سیٹ ہونے پر ترجیح لیتا ہے، حتیٰ کہ `[]` بھی):

```json5
38. {
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

39. منشن گیٹنگ کے ڈیفالٹس ہر چینل میں موجود ہوتے ہیں (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`)۔ 40. جب `*.groups` سیٹ کیا جائے تو یہ گروپ allowlist کے طور پر بھی کام کرتا ہے؛ تمام گروپس کی اجازت کے لیے `"*"` شامل کریں۔

41. صرف مخصوص متنی ٹرگرز پر **جواب دینے** کے لیے (مقامی @-منشنز کو نظرانداز کرتے ہوئے):

```json5
42. {
  channels: {
    whatsapp: {
      // Include your own number to enable self-chat mode (ignore native @-mentions).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### 43. گروپ پالیسی (فی چینل)

44. گروپ/روم پیغامات کو بالکل قبول کیا جائے یا نہیں، اس کو کنٹرول کرنے کے لیے `channels.*.groupPolicy` استعمال کریں:

```json5
45. {
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"],
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: {
          channels: { help: { allow: true } },
        },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
  },
}
```

نوٹس:

- 46. `"open"`: گروپس allowlists کو بائی پاس کرتے ہیں؛ منشن گیٹنگ اب بھی لاگو رہتی ہے۔
- 47. `"disabled"`: تمام گروپ/روم پیغامات بلاک کر دیتا ہے۔
- 48. `"allowlist"`: صرف وہی گروپس/رومز اجازت یافتہ ہوتے ہیں جو کنفیگر کی گئی allowlist سے میچ کریں۔
- 49. `channels.defaults.groupPolicy` اس وقت ڈیفالٹ سیٹ کرتا ہے جب کسی فراہم کنندہ کا `groupPolicy` غیر متعین ہو۔
- 50. WhatsApp/Telegram/Signal/iMessage/Microsoft Teams میں `groupAllowFrom` استعمال ہوتا ہے (fallback: واضح `allowFrom`)۔
- Discord/Slack use channel allowlists (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
- Group DMs (Discord/Slack) are still controlled by `dm.groupEnabled` + `dm.groupChannels`.
- Default is `groupPolicy: "allowlist"` (unless overridden by `channels.defaults.groupPolicy`); if no allowlist is configured, group messages are blocked.

### Multi-agent routing (`agents.list` + `bindings`)

Run multiple isolated agents (separate workspace, `agentDir`, sessions) inside one Gateway.
Inbound messages are routed to an agent via bindings.

- `agents.list[]`: per-agent overrides.
  - `id`: stable agent id (required).
  - `default`: optional; when multiple are set, the first wins and a warning is logged.
    If none are set, the **first entry** in the list is the default agent.
  - `name`: display name for the agent.
  - `workspace`: default `~/.openclaw/workspace-<agentId>` (for `main`, falls back to `agents.defaults.workspace`).
  - `agentDir`: default `~/.openclaw/agents/<agentId>/agent`.
  - `model`: per-agent default model, overrides `agents.defaults.model` for that agent.
    - string form: `"provider/model"`, overrides only `agents.defaults.model.primary`
    - object form: `{ primary, fallbacks }` (fallbacks override `agents.defaults.model.fallbacks`; `[]` disables global fallbacks for that agent)
  - `identity`: per-agent name/theme/emoji (used for mention patterns + ack reactions).
  - `groupChat`: per-agent mention-gating (`mentionPatterns`).
  - `sandbox`: per-agent sandbox config (overrides `agents.defaults.sandbox`).
    - `mode`: `"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    - `scope`: `"session"` | `"agent"` | `"shared"`
    - `workspaceRoot`: custom sandbox workspace root
    - `docker`: per-agent docker overrides (e.g. `image`, `network`, `env`, `setupCommand`, limits; ignored when `scope: "shared"`)
    - `browser`: per-agent sandboxed browser overrides (ignored when `scope: "shared"`)
    - `prune`: per-agent sandbox pruning overrides (ignored when `scope: "shared"`)
  - `subagents`: ہر ایجنٹ کے لیے ذیلی ایجنٹ کی ڈیفالٹ ترتیبات۔
    - `allowAgents`: allowlist of agent ids for `sessions_spawn` from this agent (`["*"]` = allow any; default: only same agent)
  - `tools`: per-agent tool restrictions (applied before sandbox tool policy).
    - `profile`: base tool profile (applied before allow/deny)
    - `allow`: اجازت یافتہ ٹول ناموں کی فہرست
    - `deny`: ممنوعہ ٹول ناموں کی فہرست (deny کو فوقیت حاصل ہے)
- `agents.defaults`: shared agent defaults (model, workspace, sandbox, etc.).
- `bindings[]`: routes inbound messages to an `agentId`.
  - `match.channel` (required)
  - `match.accountId` (optional; `*` = any account; omitted = default account)
  - `match.peer` (اختیاری؛ `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (optional; channel-specific)

Deterministic match order:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exact, no peer/guild/team)
5. `match.accountId: "*"` (channel-wide, no peer/guild/team)
6. default agent (`agents.list[].default`, else first list entry, else `"main"`)

Within each match tier, the first matching entry in `bindings` wins.

#### فی ایجنٹ رسائی پروفائلز (ملٹی ایجنٹ)

Each agent can carry its own sandbox + tool policy. Use this to mix access
levels in one gateway:

- **Full access** (personal agent)
- **Read-only** tools + workspace
- **No filesystem access** (messaging/session tools only)

See [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) for precedence and
additional examples.

Full access (no sandbox):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

Read-only tools + read-only workspace:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

فائل سسٹم تک رسائی نہیں (میسجنگ/سیشن ٹولز فعال ہیں):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

Example: two WhatsApp accounts → two agents:

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
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

### `tools.agentToAgent` (optional)

Agent-to-agent messaging is opt-in:

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `messages.queue`

Controls how inbound messages behave when an agent run is already active.

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

Debounce rapid inbound messages from the **same sender** so multiple back-to-back
messages become a single agent turn. Debouncing is scoped per channel + conversation
and uses the most recent message for reply threading/IDs.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

نوٹس:

- Debounce batches **text-only** messages; media/attachments flush immediately.
- Control commands (e.g. `/queue`, `/new`) bypass debouncing so they stay standalone.

### `commands` (chat command handling)

کنٹرول کرتا ہے کہ مختلف کنیکٹرز میں چیٹ کمانڈز کیسے فعال ہوں۔

```json5
{
  commands: {
    native: "auto", // register native commands when supported (auto)
    text: true, // parse slash commands in chat messages
    bash: false, // allow ! (alias: /bash) (host-only; requires tools.elevated allowlists)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false, // allow /config (writes to disk)
    debug: false, // allow /debug (runtime-only overrides)
    restart: false, // allow /restart + gateway restart tool
    useAccessGroups: true, // enforce access-group allowlists/policies for commands
  },
}
```

نوٹس:

- Text commands must be sent as a **standalone** message and use the leading `/` (no plain-text aliases).
- `commands.text: false` disables parsing chat messages for commands.
- `commands.native: "auto"` (default) turns on native commands for Discord/Telegram and leaves Slack off; unsupported channels stay text-only.
- Set `commands.native: true|false` to force all, or override per channel with `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool or `"auto"`). `false` clears previously registered commands on Discord/Telegram at startup; Slack commands are managed in the Slack app.
- `channels.telegram.customCommands` adds extra Telegram bot menu entries. Names are normalized; conflicts with native commands are ignored.
- `commands.bash: true` enables `! <cmd>` to run host shell commands (`/bash <cmd>` also works as an alias). Requires `tools.elevated.enabled` and allowlisting the sender in `tools.elevated.allowFrom.<channel>`.
- `commands.bashForegroundMs` controls how long bash waits before backgrounding. While a bash job is running, new `! <cmd>` requests are rejected (one at a time).
- `commands.config: true` enables `/config` (reads/writes `openclaw.json`).
- `channels.<provider>`.configWrites`اس چینل کی جانب سے شروع کی گئی کنفیگ تبدیلیوں کو محدود کرتا ہے (ڈیفالٹ: true)۔ This applies to`/config set|unset\` plus provider-specific auto-migrations (Telegram supergroup ID changes, Slack channel ID changes).
- `commands.debug: true` enables `/debug` (runtime-only overrides).
- `commands.restart: true` enables `/restart` and the gateway tool restart action.
- `commands.useAccessGroups: false` allows commands to bypass access-group allowlists/policies.
- Slash commands and directives are only honored for **authorized senders**. Authorization is derived from
  channel allowlists/pairing plus `commands.useAccessGroups`.

### `web` (WhatsApp web channel runtime)

WhatsApp runs through the gateway’s web channel (Baileys Web). It starts automatically when a linked session exists.
`web.enabled: false` کو سیٹ کریں تاکہ یہ ڈیفالٹ طور پر بند رہے۔

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

### `channels.telegram` (بوٹ ٹرانسپورٹ)

OpenClaw ٹیلیگرام کو صرف اسی وقت شروع کرتا ہے جب `channels.telegram` کی کنفیگ سیکشن موجود ہو۔ بوٹ ٹوکن `channels.telegram.botToken` (یا `channels.telegram.tokenFile`) سے حاصل کیا جاتا ہے، جبکہ ڈیفالٹ اکاؤنٹ کے لیے `TELEGRAM_BOT_TOKEN` بطور فال بیک استعمال ہوتا ہے۔
خودکار آغاز کو غیر فعال کرنے کے لیے `channels.telegram.enabled: false` سیٹ کریں۔
ملٹی اکاؤنٹ سپورٹ `channels.telegram.accounts` کے تحت موجود ہے (اوپر ملٹی اکاؤنٹ سیکشن دیکھیں)۔ ماحولیاتی (Env) ٹوکنز صرف ڈیفالٹ اکاؤنٹ پر لاگو ہوتے ہیں۔
`channels.telegram.configWrites: false` سیٹ کریں تاکہ ٹیلیگرام کی جانب سے کنفیگ رائٹس روکی جا سکیں (بشمول سپرگروپ آئی ڈی مائیگریشن اور `/config set|unset`)۔

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50, // include last N group messages as context (0 disables)
      replyToMode: "first", // off | first | all
      linkPreview: true, // toggle outbound link previews
      streamMode: "partial", // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {
        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        // transport overrides
        autoSelectFamily: false,
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook", // requires webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

ڈرافٹ اسٹریمنگ کے نوٹس:

- ٹیلیگرام `sendMessageDraft` استعمال کرتا ہے (ڈرافٹ ببل، حقیقی پیغام نہیں)۔
- **پرائیویٹ چیٹ ٹاپکس** درکار ہیں (DMs میں message_thread_id؛ بوٹ میں ٹاپکس فعال ہوں)۔
- `/reasoning stream` ڈرافٹ میں استدلال اسٹریمنگ کرتا ہے، پھر حتمی جواب بھیجتا ہے۔
  ریٹرائی پالیسی کی ڈیفالٹس اور رویہ [Retry policy](/concepts/retry) میں دستاویزی ہیں۔

### `channels.discord` (بوٹ ٹرانسپورٹ)

بوٹ ٹوکن اور اختیاری گیٹنگ سیٹ کر کے ڈسکارڈ بوٹ کنفیگر کریں:
ملٹی اکاؤنٹ سپورٹ `channels.discord.accounts` کے تحت موجود ہے (اوپر ملٹی اکاؤنٹ سیکشن دیکھیں)۔ ماحولیاتی (Env) ٹوکنز صرف ڈیفالٹ اکاؤنٹ پر لاگو ہوتے ہیں۔

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8, // clamp inbound media size
      allowBots: false, // allow bot-authored messages
      actions: {
        // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dm: {
        enabled: true, // disable all DMs when false
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false, // enable group DMs
        groupChannels: ["openclaw-dm"], // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {
          // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false, // per-guild default
          reactionNotifications: "own", // off | own | all | allowlist
          users: ["987654321098765432"], // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20, // include last N guild messages as context
      textChunkLimit: 2000, // optional outbound text chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      maxLinesPerMessage: 17, // soft max lines per message (Discord UI clipping)
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

OpenClaw ڈسکارڈ کو صرف اسی وقت شروع کرتا ہے جب `channels.discord` کی کنفیگ سیکشن موجود ہو۔ ٹوکن `channels.discord.token` سے حاصل کیا جاتا ہے، جبکہ ڈیفالٹ اکاؤنٹ کے لیے `DISCORD_BOT_TOKEN` فال بیک کے طور پر استعمال ہوتا ہے (جب تک `channels.discord.enabled` `false` نہ ہو)۔ کرون/CLI کمانڈز کے لیے ڈیلیوری ٹارگٹس بتاتے وقت `user:<id>` (DM) یا `channel:<id>` (گلڈ چینل) استعمال کریں؛ صرف عددی IDs مبہم سمجھی جاتی ہیں اور مسترد کر دی جاتی ہیں۔
گلڈ سلگز لوئرکیس ہوتے ہیں اور اسپیسز کو `-` سے بدلا جاتا ہے؛ چینل کیز سلگ شدہ چینل نام استعمال کرتی ہیں (ابتدائی `#` کے بغیر)۔ نام تبدیل ہونے کی مبہم صورتحال سے بچنے کے لیے گلڈ IDs کو بطور کی استعمال کرنا بہتر ہے۔
بوٹ کے لکھے ہوئے پیغامات ڈیفالٹ طور پر نظرانداز کر دیے جاتے ہیں۔ `channels.discord.allowBots` کے ذریعے فعال کریں (اپنے پیغامات اب بھی سیلف ریپلائی لوپس سے بچنے کے لیے فلٹر رہتے ہیں)۔
ری ایکشن نوٹیفکیشن موڈز:

- `off`: کوئی reaction events نہیں۔
- `own`: بوٹ کے اپنے پیغامات پر reactions (default)۔
- `all`: تمام پیغامات پر تمام reactions۔
- `allowlist`: `guilds.<id>` سے آنے والی ری ایکشنز.users تمام پیغامات پر (خالی فہرست غیر فعال کرتی ہے)۔
  آؤٹ باؤنڈ متن کو `channels.discord.textChunkLimit` (ڈیفالٹ 2000) کے مطابق حصوں میں تقسیم کیا جاتا ہے۔ `channels.discord.chunkMode="newline"` سیٹ کریں تاکہ لمبائی کے مطابق تقسیم سے پہلے خالی لائنوں (پیراگراف حدود) پر تقسیم ہو۔ ڈسکارڈ کلائنٹس بہت لمبے پیغامات کو کاٹ سکتے ہیں، اس لیے `channels.discord.maxLinesPerMessage` (ڈیفالٹ 17) طویل کثیر سطری جوابات کو تقسیم کرتا ہے چاہے وہ 2000 حروف سے کم ہی کیوں نہ ہوں۔
  ریٹرائی پالیسی کی ڈیفالٹس اور رویہ [Retry policy](/concepts/retry) میں دستاویزی ہیں۔

### `channels.googlechat` (چیٹ API ویب ہوک)

گوگل چیٹ HTTP ویب ہوکس کے ذریعے ایپ لیول آتھنٹیکیشن (سروس اکاؤنٹ) کے ساتھ چلتا ہے۔
ملٹی اکاؤنٹ سپورٹ `channels.googlechat.accounts` کے تحت موجود ہے (اوپر ملٹی اکاؤنٹ سیکشن دیکھیں)۔ ماحولیاتی متغیرات صرف ڈیفالٹ اکاؤنٹ پر لاگو ہوتے ہیں۔

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; improves mention detection
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"], // optional; "open" requires ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

نوٹس:

- سروس اکاؤنٹ JSON کو اِن لائن (`serviceAccount`) یا فائل کی صورت میں (`serviceAccountFile`) فراہم کیا جا سکتا ہے۔
- ڈیفالٹ اکاؤنٹ کے لیے Env فال بیکس: `GOOGLE_CHAT_SERVICE_ACCOUNT` یا `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`۔
- `audienceType` + `audience` کو چیٹ ایپ کی ویب ہوک آتھنٹیکیشن کنفیگ سے مطابقت رکھنی چاہیے۔
- ڈیلیوری ٹارگٹس سیٹ کرتے وقت `spaces/<spaceId>` یا `users/<userId|email>` استعمال کریں۔

### `channels.slack` (ساکٹ موڈ)

Slack ساکٹ موڈ میں چلتا ہے اور بوٹ ٹوکن اور ایپ ٹوکن دونوں درکار ہوتے ہیں:

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"],
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50, // include last N channel/group messages as context (0 disables)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

ملٹی اکاؤنٹ سپورٹ `channels.slack.accounts` کے تحت موجود ہے (اوپر ملٹی اکاؤنٹ سیکشن دیکھیں)۔ ماحولیاتی (Env) ٹوکنز صرف ڈیفالٹ اکاؤنٹ پر لاگو ہوتے ہیں۔

OpenClaw اس وقت Slack شروع کرتا ہے جب پرووائیڈر فعال ہو اور دونوں ٹوکن سیٹ ہوں (کنفیگ کے ذریعے یا `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`)۔ کرون/CLI کمانڈز کے لیے ڈیلیوری ٹارگٹس بتاتے وقت `user:<id>` (DM) یا `channel:<id>` استعمال کریں۔
`channels.slack.configWrites: false` سیٹ کریں تاکہ Slack کی جانب سے کنفیگ رائٹس روکی جا سکیں (بشمول چینل ID مائیگریشن اور `/config set|unset`)۔

1. بطور ڈیفالٹ بوٹ کی جانب سے تحریر کردہ پیغامات نظر انداز کر دیے جاتے ہیں۔ 2. `channels.slack.allowBots` یا `channels.slack.channels.<id>` کے ذریعے فعال کریں3. .allowBots\`.

4. ردِعمل کی اطلاع کے موڈز:

- `off`: کوئی reaction events نہیں۔
- `own`: بوٹ کے اپنے پیغامات پر reactions (default)۔
- `all`: تمام پیغامات پر تمام reactions۔
- 5. `allowlist`: تمام پیغامات پر `channels.slack.reactionAllowlist` سے آنے والے ردِعمل (خالی فہرست غیر فعال کر دیتی ہے)۔

6. تھریڈ سیشن کی علیحدگی:

- 7. `channels.slack.thread.historyScope` یہ کنٹرول کرتا ہے کہ تھریڈ کی ہسٹری فی تھریڈ ہو (`thread`، ڈیفالٹ) یا پورے چینل میں مشترک ہو (`channel`)۔
- 8. `channels.slack.thread.inheritParent` یہ کنٹرول کرتا ہے کہ نئے تھریڈ سیشن والد چینل کا ٹرانسکرپٹ وراثت میں لیں یا نہیں (ڈیفالٹ: false)۔

9. Slack ایکشن گروپس (`slack` ٹول ایکشنز کے لیے گیٹ):

| ایکشن گروپ | ڈیفالٹ  | Notes                        |
| ---------- | ------- | ---------------------------- |
| reactions  | enabled | ری ایکٹ + ری ایکشنز کی فہرست |
| messages   | enabled | پڑھنا/بھیجنا/ترمیم/حذف       |
| pins       | enabled | پن/ان پن/فہرست               |
| memberInfo | enabled | ممبر معلومات                 |
| emojiList  | enabled | کسٹم ایموجی فہرست            |

### `channels.mattermost` (بوٹ ٹوکن)

Mattermost بطور پلگ اِن فراہم کیا جاتا ہے اور کور انسٹال کے ساتھ شامل نہیں ہوتا۔
11. پہلے اسے انسٹال کریں: `openclaw plugins install @openclaw/mattermost` (یا گِٹ چیک آؤٹ سے `./extensions/mattermost`)۔

12. Mattermost کے لیے بوٹ ٹوکن کے ساتھ آپ کے سرور کا بیس URL درکار ہوتا ہے:

```json5
13. {
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

14. جب اکاؤنٹ کنفیگر (بوٹ ٹوکن + بیس URL) ہو اور فعال ہو تو OpenClaw Mattermost شروع کرتا ہے۔ 15. ڈیفالٹ اکاؤنٹ کے لیے ٹوکن + بیس URL `channels.mattermost.botToken` + `channels.mattermost.baseUrl` یا `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` سے حل کیے جاتے ہیں (جب تک `channels.mattermost.enabled` `false` نہ ہو)۔

چیٹ موڈز:

- `oncall` (ڈیفالٹ): صرف اس وقت چینل پیغامات کا جواب دیں جب @mention کیا جائے۔
- `onmessage`: ہر چینل پیغام پر جواب دیں۔
- 18. `onchar`: جب پیغام کسی ٹرگر پری فکس سے شروع ہو تو جواب دیں (`channels.mattermost.oncharPrefixes`، ڈیفالٹ `[">", "!"]`)۔

19. رسائی کنٹرول:

- 20. ڈیفالٹ DMs: `channels.mattermost.dmPolicy="pairing"` (نامعلوم بھیجنے والوں کو پیئرنگ کوڈ ملتا ہے)۔
- عوامی DMs: `channels.mattermost.dmPolicy="open"` کے ساتھ `channels.mattermost.allowFrom=["*"]`۔
- گروپس: `channels.mattermost.groupPolicy="allowlist"` بطور ڈیفالٹ (mention‑gated)۔ 22. بھیجنے والوں کو محدود کرنے کے لیے `channels.mattermost.groupAllowFrom` استعمال کریں۔

23. ملٹی اکاؤنٹ سپورٹ `channels.mattermost.accounts` کے تحت موجود ہے (اوپر ملٹی اکاؤنٹ سیکشن دیکھیں)۔ ماحولیاتی متغیرات صرف ڈیفالٹ اکاؤنٹ پر لاگو ہوتے ہیں۔
24. ڈیلیوری اہداف بتاتے وقت `channel:<id>` یا `user:<id>` (یا `@username`) استعمال کریں؛ بغیر سابقہ کے آئی ڈیز کو چینل آئی ڈیز سمجھا جاتا ہے۔

### 26. `channels.signal` (signal-cli)

27. Signal کے ردِعمل سسٹم ایونٹس خارج کر سکتے ہیں (مشترکہ ردِعمل ٹولنگ):

```json5
28. {
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // include last N group messages as context (0 disables)
    },
  },
}
```

29. ردِعمل کی اطلاع کے موڈز:

- `off`: کوئی reaction events نہیں۔
- `own`: بوٹ کے اپنے پیغامات پر reactions (default)۔
- `all`: تمام پیغامات پر تمام reactions۔
- 30. `allowlist`: تمام پیغامات پر `channels.signal.reactionAllowlist` سے آنے والے ردِعمل (خالی فہرست غیر فعال کر دیتی ہے)۔

### 31. `channels.imessage` (imsg CLI)

32. OpenClaw `imsg rpc` شروع کرتا ہے (stdio پر JSON-RPC)۔ 33. کسی ڈیمَن یا پورٹ کی ضرورت نہیں۔

```json5
34. {
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SCP for remote attachments when using SSH wrapper
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50, // include last N group messages as context (0 disables)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

35. ملٹی اکاؤنٹ سپورٹ `channels.imessage.accounts` کے تحت موجود ہے (اوپر ملٹی اکاؤنٹ سیکشن دیکھیں)۔

نوٹس:

- 36. Messages DB تک مکمل ڈسک رسائی درکار ہے۔
- 37. پہلی بار بھیجنے پر Messages آٹومیشن کی اجازت کا اشارہ آئے گا۔
- 38. `chat_id:<id>` اہداف کو ترجیح دیں۔ 39. چیٹس کی فہرست کے لیے `imsg chats --limit 20` استعمال کریں۔
- 40. `channels.imessage.cliPath` کسی ریپر اسکرپٹ کی طرف اشارہ کر سکتا ہے (مثلاً کسی دوسرے Mac پر `imsg rpc` چلانے کے لیے `ssh`)؛ پاس ورڈ پرامپٹس سے بچنے کے لیے SSH کیز استعمال کریں۔
- 41. ریموٹ SSH ریپرز کے لیے، جب `includeAttachments` فعال ہو تو SCP کے ذریعے اٹیچمنٹس حاصل کرنے کے لیے `channels.imessage.remoteHost` سیٹ کریں۔

مثالی ریپر:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

42. ایجنٹ کے ذریعے فائل آپریشنز کے لیے استعمال ہونے والی **واحد عالمی ورک اسپیس ڈائریکٹری** سیٹ کرتا ہے۔

بطورِ طے شدہ: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

43. اگر `agents.defaults.sandbox` فعال ہو تو غیر مین سیشنز اسے `agents.defaults.sandbox.workspaceRoot` کے تحت اپنی فی اسکوپ ورک اسپیس کے ساتھ اوور رائیڈ کر سکتے ہیں۔

### 44. `agents.defaults.repoRoot`

45. سسٹم پرامپٹ کی Runtime لائن میں دکھانے کے لیے اختیاری ریپوزٹری روٹ۔ 46. اگر سیٹ نہ ہو تو OpenClaw ورک اسپیس (اور موجودہ ورکنگ ڈائریکٹری) سے اوپر کی طرف چلتے ہوئے `.git` ڈائریکٹری تلاش کرنے کی کوشش کرتا ہے۔ 47. استعمال کے لیے راستے کا موجود ہونا ضروری ہے۔

```json5
48. {
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### 49. `agents.defaults.skipBootstrap`

50. ورک اسپیس کے بوٹ اسٹرَیپ فائلز (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, اور `BOOTSTRAP.md`) کی خودکار تخلیق کو غیر فعال کرتا ہے۔

Use this for pre-seeded deployments where your workspace files come from a repo.

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Max characters of each workspace bootstrap file injected into the system prompt
before truncation. بطورِ طے شدہ: `20000`.

When a file exceeds this limit, OpenClaw logs a warning and injects a truncated
head/tail with a marker.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

Sets the user’s timezone for **system prompt context** (not for timestamps in
message envelopes). If unset, OpenClaw uses the host timezone at runtime.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

سسٹم پرامپٹ کے Current Date & Time سیکشن میں دکھائے جانے والے **وقت کے فارمیٹ** کو کنٹرول کرتا ہے۔
Default: `auto` (OS preference).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `پیغامات`

Controls inbound/outbound prefixes and optional ack reactions.
See [Messages](/concepts/messages) for queueing, sessions, and streaming context.

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

`responsePrefix` is applied to **all outbound replies** (tool summaries, block
streaming, final replies) across channels unless already present.

Overrides can be configured per channel and per account:

- `channels.<channel>.responsePrefix`
- `channels.<channel>.accounts.<id>.responsePrefix`

حل کی ترتیب (سب سے مخصوص کو ترجیح):

1. `channels.<channel>`.accounts.<id>.responsePrefix\`
2. `channels.<channel>`.responsePrefix\`
3. `messages.responsePrefix`

معانی:

- `undefined` falls through to the next level.
- `""` explicitly disables the prefix and stops the cascade.
- `"auto"` derives `[{identity.name}]` for the routed agent.

Overrides apply to all channels, including extensions, and to every outbound reply kind.

If `messages.responsePrefix` is unset, no prefix is applied by default. WhatsApp self-chat
replies are the exception: they default to `[{identity.name}]` when set, otherwise
`[openclaw]`, so same-phone conversations stay legible.
Set it to `"auto"` to derive `[{identity.name}]` for the routed agent (when set).

#### Template variables

The `responsePrefix` string can include template variables that resolve dynamically:

| متغیر             | Description            | Example                                      |
| ----------------- | ---------------------- | -------------------------------------------- |
| `{model}`         | Short model name       | `claude-opus-4-6`, `gpt-4o`                  |
| `{modelFull}`     | Full model identifier  | `anthropic/claude-opus-4-6`                  |
| `{provider}`      | Provider name          | `anthropic`, `openai`                        |
| `{thinkingLevel}` | Current thinking level | `high`, `low`, `off`                         |
| `{identity.name}` | ایجنٹ کی شناخت کا نام  | (بالکل `"auto"` موڈ جیسا) |

ویری ایبلز حروفِ بڑے چھوٹے سے غیر حساس ہیں (`{MODEL}` = `{model}`)۔ `{think}`، `{thinkingLevel}` کا متبادل نام ہے۔
غیر حل شدہ متغیرات لفظی متن کی صورت میں برقرار رہتے ہیں۔

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]",
  },
}
```

مثالی آؤٹ پٹ: `[claude-opus-4-6 | think:high] Here's my response...`

WhatsApp ان باؤنڈ پری فکس `channels.whatsapp.messagePrefix` کے ذریعے کنفیگر کیا جاتا ہے (متروک:
`messages.messagePrefix`)۔ ڈیفالٹ **غیر تبدیل** رہتا ہے: `"[openclaw]"` جب
`channels.whatsapp.allowFrom` خالی ہو، بصورتِ دیگر `""` (کوئی پری فکس نہیں)۔ جب
`"[openclaw]"` استعمال کیا جائے، تو OpenClaw اس کے بجائے `[{identity.name}]` استعمال کرے گا جب روٹ کیا گیا ایجنٹ `identity.name` سیٹ رکھتا ہو۔

`ackReaction` ان چینلز پر آنے والے پیغامات کی توثیق کے لیے بہترین کوشش کے تحت ایک ایموجی ری ایکشن بھیجتا ہے جو ری ایکشنز کو سپورٹ کرتے ہیں (Slack/Discord/Telegram/Google Chat)۔ ڈیفالٹ کے طور پر فعال ایجنٹ کے `identity.emoji` پر سیٹ ہوتا ہے اگر موجود ہو، ورنہ `"👀"`۔ غیر فعال کرنے کے لیے اسے `""` پر سیٹ کریں۔

`ackReactionScope` یہ کنٹرول کرتا ہے کہ ری ایکشنز کب فائر ہوں:

- `group-mentions` (ڈیفالٹ): صرف اس وقت جب کسی گروپ/روم میں منشن درکار ہوں **اور** بوٹ کو منشن کیا گیا ہو
- `group-all`: تمام گروپ/روم پیغامات
- `direct`: صرف ڈائریکٹ پیغامات
- `all`: تمام پیغامات

`removeAckAfterReply` جواب بھیجنے کے بعد بوٹ کا ack ری ایکشن ہٹا دیتا ہے
(صرف Slack/Discord/Telegram/Google Chat)۔ ڈیفالٹ: `false`۔

#### `messages.tts`

آؤٹ باؤنڈ جوابات کے لیے ٹیکسٹ ٹو اسپیچ فعال کریں۔ آن ہونے پر، OpenClaw ElevenLabs یا OpenAI استعمال کر کے آڈیو بناتا ہے
اور اسے جوابات کے ساتھ منسلک کرتا ہے۔ Telegram Opus وائس نوٹس استعمال کرتا ہے؛ دیگر چینلز MP3 آڈیو بھیجتے ہیں۔

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

نوٹس:

- `messages.tts.auto` خودکار TTS کو کنٹرول کرتا ہے (`off`, `always`, `inbound`, `tagged`)۔
- `/tts off|always|inbound|tagged` فی سیشن آٹو موڈ سیٹ کرتا ہے (کنفیگ کو اووررائیڈ کرتا ہے)۔
- `messages.tts.enabled` لیگیسی ہے؛ doctor اسے `messages.tts.auto` میں منتقل کرتا ہے۔
- `prefsPath` لوکل اووررائیڈز (provider/limit/summarize) محفوظ کرتا ہے۔
- `maxTextLength` TTS ان پٹ کے لیے سخت حد ہے؛ خلاصے اس میں فِٹ ہونے کے لیے کاٹ دیے جاتے ہیں۔
- `summaryModel` آٹو سمری کے لیے `agents.defaults.model.primary` کو اووررائیڈ کرتا ہے۔
  - `provider/model` یا `agents.defaults.models` سے کوئی عرف قبول کرتا ہے۔
- `modelOverrides` ماڈل سے چلنے والے اووررائیڈز جیسے `[[tts:...]]` ٹیگز کو فعال کرتا ہے (ڈیفالٹ طور پر آن)۔
- `/tts limit` اور `/tts summary` فی یوزر خلاصہ سازی کی سیٹنگز کنٹرول کرتے ہیں۔
- `apiKey` ویلیوز `ELEVENLABS_API_KEY`/`XI_API_KEY` اور `OPENAI_API_KEY` پر فال بیک کرتی ہیں۔
- `elevenlabs.baseUrl` ElevenLabs API کے بیس URL کو اووررائیڈ کرتا ہے۔
- `elevenlabs.voiceSettings` میں `stability`/`similarityBoost`/`style` (0..1)،
  `useSpeakerBoost`، اور `speed` (0.5..2.0) سپورٹ ہوتے ہیں۔

### `talk`

ٹاک موڈ کے لیے ڈیفالٹس (macOS/iOS/Android)۔ جب سیٹ نہ ہوں تو وائس IDs `ELEVENLABS_VOICE_ID` یا `SAG_VOICE_ID` پر فال بیک کرتی ہیں۔
جب سیٹ نہ ہو تو `apiKey`، `ELEVENLABS_API_KEY` (یا گیٹ وے کے شیل پروفائل) پر فال بیک کرتا ہے۔
`voiceAliases` ٹاک ڈائریکٹوز کو دوستانہ نام استعمال کرنے دیتا ہے (مثلاً `"voice":"Clawd"`)۔

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

### `agents.defaults`

ایمبیڈڈ ایجنٹ رن ٹائم کو کنٹرول کرتا ہے (ماڈل/سوچ/وربوز/ٹائم آؤٹس)۔
`agents.defaults.models` کنفیگر شدہ ماڈل کیٹلاگ کی تعریف کرتا ہے (اور `/model` کے لیے الاو لسٹ کا کردار ادا کرتا ہے)۔
`agents.defaults.model.primary` ڈیفالٹ ماڈل سیٹ کرتا ہے؛ `agents.defaults.model.fallbacks` عالمی فال اوورز ہیں۔
`agents.defaults.imageModel` اختیاری ہے اور **صرف اس صورت میں استعمال ہوتا ہے جب پرائمری ماڈل میں امیج ان پٹ نہ ہو**۔
ہر `agents.defaults.models` انٹری میں شامل ہو سکتا ہے:

- `alias` (اختیاری ماڈل شارٹ کٹ، مثلاً `/opus`).
- `params` (اختیاری پرووائیڈر مخصوص API پیرامیٹرز جو ماڈل ریکویسٹ تک پاس کیے جاتے ہیں).

`params` اسٹریمنگ رنز پر بھی لاگو ہوتا ہے (ایمبیڈڈ ایجنٹ + کمپیکشن). آج سپورٹ کی جانے والی کلیدیں: `temperature`, `maxTokens`۔ یہ کال ٹائم آپشنز کے ساتھ ضم ہو جاتے ہیں؛ کالر کی فراہم کردہ قدریں غالب رہتی ہیں۔ `temperature` ایک ایڈوانسڈ کنٹرول ہے—جب تک آپ ماڈل کے ڈیفالٹس جانتے نہ ہوں اور تبدیلی کی ضرورت نہ ہو، اسے غیر متعین چھوڑیں۔

Example:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

Z.AI GLM-4.x ماڈلز خودکار طور پر تھنکنگ موڈ فعال کرتے ہیں جب تک کہ آپ:

- `--thinking off` سیٹ نہ کریں، یا
- خود `agents.defaults.models["zai/<model>"].params.thinking` کی تعریف نہ کریں۔

OpenClaw چند بلٹ اِن عرفی شارٹ ہینڈز بھی فراہم کرتا ہے۔ ڈیفالٹس صرف اس وقت لاگو ہوتے ہیں جب ماڈل پہلے سے
`agents.defaults.models` میں موجود ہو:

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

اگر آپ خود اسی عرفی نام (کیس اِن سینسِٹو) کو کنفیگر کریں، تو آپ کی قدر غالب ہوگی (ڈیفالٹس کبھی اووررائیڈ نہیں کرتے)۔

مثال: Opus 4.6 پرائمری کے ساتھ MiniMax M2.1 فالبیک (ہوسٹڈ MiniMax):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
    },
  },
}
```

MiniMax توثیق: `MINIMAX_API_KEY` (env) سیٹ کریں یا `models.providers.minimax` کنفیگر کریں۔

#### `agents.defaults.cliBackends` (CLI فالبیک)

ٹیکسٹ-اونلی فالبیک رنز کے لیے اختیاری CLI بیک اینڈز (کوئی ٹول کالز نہیں)۔ یہ اس وقت بیک اپ راستے کے طور پر مفید ہوتے ہیں جب API پرووائیڈرز ناکام ہو جائیں۔ جب آپ ایسا `imageArg` کنفیگر کریں جو فائل پاتھس قبول کرتا ہو تو امیج پاس تھرو سپورٹ ہوتی ہے۔

نوٹس:

- CLI بیک اینڈز **ٹیکسٹ‑فرسٹ** ہیں؛ ٹولز ہمیشہ غیر فعال ہوتے ہیں۔
- `sessionArg` سیٹ ہونے پر سیشنز سپورٹ ہوتے ہیں؛ سیشن آئی ڈیز فی بیک اینڈ محفوظ رہتی ہیں۔
- `claude-cli` کے لیے ڈیفالٹس وائرڈ اِن ہوتے ہیں۔ اگر PATH محدود ہو تو کمانڈ پاتھ اووررائیڈ کریں
  (launchd/systemd).

Example:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false,
            },
          },
        },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free",
        ],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last",
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000,
      },
      contextTokens: 200000,
    },
  },
}
```

#### `agents.defaults.contextPruning` (ٹول رزلٹ پرُوننگ)

`agents.defaults.contextPruning` LLM کو ریکویسٹ بھیجنے سے عین پہلے اِن-میموری کانٹیکسٹ سے **پرانے ٹول رزلٹس** کو کم کرتا ہے۔
یہ ڈسک پر موجود سیشن ہسٹری میں کوئی ترمیم نہیں کرتا (`*.jsonl` مکمل رہتا ہے)۔

اس کا مقصد وقت کے ساتھ بڑے ٹول آؤٹ پٹس جمع کرنے والے باتونی ایجنٹس کے لیے ٹوکن استعمال کم کرنا ہے۔

اعلیٰ سطح:

- یوزر/اسسٹنٹ پیغامات کو کبھی نہیں چھیڑتا۔
- آخری `keepLastAssistants` اسسٹنٹ پیغامات کو محفوظ رکھتا ہے (اس نقطے کے بعد کوئی ٹول رزلٹس پرُون نہیں ہوتے)۔
- بوٹ اسٹرپ پری فکس کو محفوظ رکھتا ہے (پہلے یوزر پیغام سے پہلے کچھ بھی پرُون نہیں ہوتا)۔
- موڈز:
  - `adaptive`: جب اندازہ شدہ کانٹیکسٹ ریشو `softTrimRatio` سے تجاوز کرے تو حد سے بڑے ٹول رزلٹس کو نرم انداز میں ٹرم کرتا ہے (ہیڈ/ٹیل رکھتا ہے)۔
    پھر جب اندازہ شدہ کانٹیکسٹ ریشو `hardClearRatio` سے تجاوز کرے **اور** پرُون کیے جا سکنے والے ٹول رزلٹس کی مقدار کافی ہو (`minPrunableToolChars`) تو سب سے پرانے اہل ٹول رزلٹس کو ہارڈ کلیئر کرتا ہے۔
  - `aggressive`: کٹ آف سے پہلے اہل ٹول رزلٹس کو ہمیشہ `hardClear.placeholder` سے بدل دیتا ہے (کوئی ریشو چیکس نہیں)۔

سافٹ بمقابلہ ہارڈ پرُوننگ (LLM کو بھیجے گئے کانٹیکسٹ میں کیا بدلتا ہے):

- **سافٹ-ٹرم**: صرف _حد سے بڑے_ ٹول رزلٹس کے لیے۔ آغاز + اختتام رکھتا ہے اور درمیان میں `...` داخل کرتا ہے۔
  - پہلے: `toolResult("…very long output…")`
  - بعد میں: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
- **ہارڈ-کلیئر**: پورے ٹول رزلٹ کو پلیس ہولڈر سے بدل دیتا ہے۔
  - پہلے: `toolResult("…very long output…")`
  - بعد میں: `toolResult("[Old tool result content cleared]")`

نوٹس / موجودہ حدود:

- ایسے ٹول نتائج جن میں **تصویری بلاکس شامل ہوں فی الحال نظرانداز کر دیے جاتے ہیں** (انہیں کبھی trim/clear نہیں کیا جاتا)۔
- اندازاً “context ratio” **کریکٹرز** پر مبنی ہے (تقریبی)، نہ کہ عین ٹوکنز پر۔
- اگر سیشن میں ابھی تک کم از کم `keepLastAssistants` اسسٹنٹ پیغامات موجود نہیں ہیں تو pruning کو چھوڑ دیا جاتا ہے۔
- `aggressive` موڈ میں، `hardClear.enabled` کو نظر انداز کیا جاتا ہے (اہل ٹول نتائج ہمیشہ `hardClear.placeholder` سے بدل دیے جاتے ہیں)۔

ڈیفالٹ (adaptive):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

غیر فعال کرنے کے لیے:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

ڈیفالٹس (جب `mode`، `"adaptive"` یا `"aggressive"` ہو):

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (صرف adaptive)
- `hardClearRatio`: `0.5` (صرف adaptive)
- `minPrunableToolChars`: `50000` (صرف adaptive)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (صرف adaptive)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

مثال (aggressive، کم سے کم):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

مثال (adaptive ٹیونڈ):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Optional: restrict pruning to specific tools (deny wins; supports "*" wildcards)
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

رویّے کی تفصیلات کے لیے [/concepts/session-pruning](/concepts/session-pruning) دیکھیں۔

#### `agents.defaults.compaction` (ہیڈ روم محفوظ کرنا + میموری فلش)

`agents.defaults.compaction.mode` کمپیکشن سمریائزیشن حکمتِ عملی منتخب کرتا ہے۔ ڈیفالٹ `default` ہے؛ بہت طویل ہسٹری کے لیے چنکڈ سمریائزیشن فعال کرنے کو `safeguard` سیٹ کریں۔ [/concepts/compaction](/concepts/compaction) دیکھیں۔

`agents.defaults.compaction.reserveTokensFloor` Pi کمپیکشن کے لیے کم از کم `reserveTokens` کی قدر نافذ کرتا ہے (ڈیفالٹ: `20000`)۔ اسے غیر فعال کرنے کے لیے `0` پر سیٹ کریں۔

`agents.defaults.compaction.memoryFlush` خودکار کمپیکشن سے پہلے ایک **خاموش** agentic ٹرن چلاتا ہے، جس میں ماڈل کو دیرپا یادداشتیں ڈسک پر محفوظ کرنے کی ہدایت دی جاتی ہے (مثلاً `memory/YYYY-MM-DD.md`)۔ یہ اس وقت متحرک ہوتا ہے جب سیشن ٹوکن کا اندازہ کمپیکشن حد سے نیچے کسی نرم حد کو عبور کر جائے۔

لیگیسی ڈیفالٹس:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: `NO_REPLY` کے ساتھ بلٹ اِن ڈیفالٹس
- نوٹ: جب سیشن ورک اسپیس صرف پڑھنے کے قابل ہو تو میموری فلش کو چھوڑ دیا جاتا ہے
  (`agents.defaults.sandbox.workspaceAccess: "ro"` یا `"none"`)۔

مثال (حسبِ ضرورت ترتیب دیا گیا):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

بلاک اسٹریمنگ:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (بطورِ طے شدہ بند)۔

- چینل اووررائیڈز: `*.blockStreaming` (اور فی اکاؤنٹ ویریئنٹس) بلاک اسٹریمنگ کو آن/آف کرنے کے لیے۔
  نان-ٹیلیگرام چینلز میں بلاک جوابات فعال کرنے کے لیے واضح طور پر `*.blockStreaming: true` درکار ہوتا ہے۔

- `agents.defaults.blockStreamingBreak`: `"text_end"` یا `"message_end"` (ڈیفالٹ: text_end)۔

- `agents.defaults.blockStreamingChunk`: اسٹریمنگ بلاکس کے لیے سافٹ چنکنگ۔ ڈیفالٹس:
  800–1200 حروف، ترجیحاً پیراگراف بریکس (`\n\n`)، پھر نئی سطور، پھر جملے۔
  Example:

  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: بھیجنے سے پہلے اسٹریمنگ بلاکس کو ضم کرنا۔
  ڈیفالٹس `{ idleMs: 1000 }` ہیں اور `blockStreamingChunk` سے `minChars` وراثت میں لیتا ہے
  جبکہ `maxChars` چینل کی متن حد تک محدود ہوتا ہے۔ Signal/Slack/Discord/Google Chat میں ڈیفالٹ طور پر
  `minChars: 1500` ہوتا ہے جب تک اووررائیڈ نہ کیا جائے۔
  چینل اووررائیڈز: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (اور فی اکاؤنٹ ویریئنٹس)۔

- `agents.defaults.humanDelay`: پہلے کے بعد **بلاک جوابات** کے درمیان بے ترتیب وقفہ۔
  موڈز: `off` (ڈیفالٹ)، `natural` (800–2500ms)، `custom` (`minMs`/`maxMs` استعمال کریں)۔
  فی ایجنٹ اووررائیڈ: `agents.list[].humanDelay`۔
  Example:

  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } },
  }
  ```

  رویے اور چنکنگ کی تفصیلات کے لیے [/concepts/streaming](/concepts/streaming) دیکھیں۔

ٹائپنگ اشارے:

- `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`۔ براہِ راست چیٹس / منشنز کے لیے بطورِ طے شدہ `instant` اور بغیر منشن والے گروپ چیٹس کے لیے `message` ہوتا ہے۔
- `session.typingMode`: موڈ کے لیے فی سیشن اوور رائیڈ۔
- `agents.defaults.typingIntervalSeconds`: ٹائپنگ سگنل کتنی بار ریفریش ہوتا ہے (طے شدہ: 6s)۔
- `session.typingIntervalSeconds`: ریفریش انٹرول کے لیے فی سیشن اوور رائیڈ۔
  رویے کی تفصیلات کے لیے [/concepts/typing-indicators](/concepts/typing-indicators) دیکھیں۔

`agents.defaults.model.primary` کو `provider/model` کے طور پر سیٹ کیا جانا چاہیے (مثلاً `anthropic/claude-opus-4-6`)۔
عرف (Aliases) `agents.defaults.models.*.alias` سے آتے ہیں (مثلاً `Opus`)۔
اگر آپ پرووائیڈر چھوڑ دیں تو OpenClaw فی الحال عارضی ڈیپریکیشن فال بیک کے طور پر `anthropic` فرض کرتا ہے۔
Z.AI ماڈلز `zai/<model>` کے طور پر دستیاب ہیں (مثلاً `zai/glm-4.7`) اور ماحول میں `ZAI_API_KEY` (یا پرانا `Z_AI_API_KEY`) درکار ہوتا ہے۔

`agents.defaults.heartbeat` وقفہ وار ہارٹ بیٹ رنز کو کنفیگر کرتا ہے:

- `every`: دورانیے کی اسٹرنگ (`ms`, `s`, `m`, `h`); طے شدہ اکائی منٹ ہے۔ طے شدہ:
  `30m`۔ غیر فعال کرنے کے لیے `0m` سیٹ کریں۔
- `model`: ہارٹ بیٹ رنز کے لیے اختیاری اوور رائیڈ ماڈل (`provider/model`)۔
- `includeReasoning`: جب `true` ہو تو ہارٹ بیٹس دستیاب ہونے پر الگ `Reasoning:` پیغام بھی فراہم کریں گے (وہی ساخت جیسی `/reasoning on`)۔ طے شدہ: `false`۔
- `session`: اختیاری سیشن کلید تاکہ کنٹرول کیا جا سکے کہ ہارٹ بیٹ کس سیشن میں چلیں۔ ڈیفالٹ: `main`۔
- `to`: اختیاری وصول کنندہ اوور رائیڈ (چینل مخصوص آئی ڈی، مثلاً WhatsApp کے لیے E.164، Telegram کے لیے چیٹ آئی ڈی)۔
- `target`: اختیاری ترسیلی چینل (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`)۔ طے شدہ: `last`۔
- `prompt`: ہارٹ بیٹ باڈی کے لیے اختیاری اوور رائیڈ (طے شدہ: \`Read HEARTBEAT.md if it exists (workspace context).Follow it strictly.Do not infer or repeat old tasks from prior chats.If nothing needs attention, reply HEARTBEAT_OK.`)۔ اوور رائیڈز لفظ بہ لفظ بھیجے جاتے ہیں؛ اگر آپ اب بھی فائل پڑھوانا چاہتے ہیں تو `Read HEARTBEAT.md`کی ایک لائن شامل کریں۔`ackMaxChars`: ترسیل سے پہلے `HEARTBEAT_OK`کے بعد اجازت یافتہ زیادہ سے زیادہ حروف (طے شدہ: 300)۔ کسی مخصوص ایجنٹ کے لیے ہارٹ بیٹ سیٹنگز فعال یا اوور رائیڈ کرنے کے لیے`agents.list[].heartbeat`سیٹ کریں۔ اگر کسی بھی ایجنٹ انٹری میں`heartbeat\` متعین ہو تو **صرف وہی ایجنٹس** ہارٹ بیٹس چلائیں گے؛ ڈیفالٹس ان ایجنٹس کے لیے مشترکہ بنیاد بن جاتے ہیں۔
- ہارٹ بیٹس مکمل ایجنٹ ٹرنز چلاتے ہیں۔

ہر ایجنٹ کے لیے ہارٹ بیٹس:

- کم وقفے زیادہ ٹوکن استعمال کرتے ہیں؛ `every` کے بارے میں محتاط رہیں، `HEARTBEAT.md` کو مختصر رکھیں، اور/یا سستا `model` منتخب کریں۔
- `tools.exec` بیک گراؤنڈ exec کے ڈیفالٹس کو کنفیگر کرتا ہے:

`backgroundMs`: خودکار بیک گراؤنڈ ہونے سے پہلے وقت (ms، طے شدہ 10000) `timeoutSec`: اس رن ٹائم کے بعد خودکار طور پر ختم (سیکنڈز، طے شدہ 1800)

`cleanupMs`: مکمل شدہ سیشنز کو میموری میں کتنی دیر رکھنا ہے (ms، طے شدہ 1800000)

- `notifyOnExit`: بیک گراؤنڈ کیے گئے exec کے ختم ہونے پر سسٹم ایونٹ قطار میں ڈالیں + ہارٹ بیٹ کی درخواست کریں (طے شدہ true)
- `applyPatch.enabled`: تجرباتی `apply_patch` فعال کریں (صرف OpenAI/OpenAI Codex؛ طے شدہ false)
- `cleanupMs`: مکمل شدہ سیشنز کو میموری میں رکھنے کی مدت (ملی سیکنڈز، ڈیفالٹ 1800000)
- `tools.web` ویب سرچ + فِیچ ٹولز کو کنفیگر کرتا ہے:
- `applyPatch.enabled`: تجرباتی `apply_patch` کو فعال کریں (صرف OpenAI/OpenAI Codex؛ ڈیفالٹ false)
- `tools.web.search.apiKey` (تجویز کردہ: `openclaw configure --section web` کے ذریعے سیٹ کریں، یا `BRAVE_API_KEY` ماحولاتی متغیر استعمال کریں)

`tools.web.search.maxResults` (1–10، طے شدہ 5)

- `tools.web.fetch.enabled` (طے شدہ true)
- `tools.web.fetch.maxCharsCap` (طے شدہ 50000؛ کنفیگ/ٹول کالز سے maxChars کو محدود کرتا ہے)
- `tools.web.search.maxResults` (1–10، ڈیفالٹ 5)
- `tools.web.search.timeoutSeconds` (بطورِ طے شدہ 30)
- `tools.web.search.cacheTtlMinutes` (بطورِ طے شدہ 15)
- `tools.web.fetch.firecrawl.enabled` (API کلید سیٹ ہونے پر طے شدہ true)
- `tools.web.fetch.maxChars` (بطورِ طے شدہ 50000)
- `tools.web.fetch.maxCharsCap` (default 50000; clamps maxChars from config/tool calls)
- `tools.web.fetch.timeoutSeconds` (بطورِ طے شدہ 30)
- `tools.web.fetch.cacheTtlMinutes` (بطورِ طے شدہ 15)
- `tools.web.fetch.userAgent` (اختیاری اوور رائیڈ)
- `tools.web.fetch.readability` (default true; disable to use basic HTML cleanup only)
- `tools.web.fetch.firecrawl.enabled` (default true when an API key is set)
- `tools.web.fetch.firecrawl.apiKey` (اختیاری؛ بطورِ طے شدہ `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (بطورِ طے شدہ [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (بطورِ طے شدہ true)
- `tools.web.fetch.firecrawl.maxAgeMs` (اختیاری)
- `tools.web.fetch.firecrawl.timeoutSeconds` (اختیاری)

`tools.media` آنے والے میڈیا کی سمجھ بوجھ (تصویر/آڈیو/ویڈیو) کو ترتیب دیتا ہے:

- `tools.media.models`: مشترکہ ماڈلز کی فہرست (صلاحیت کے ٹیگ کے ساتھ؛ فی-کیپ فہرستوں کے بعد استعمال ہوتی ہے)
- `tools.media.concurrency`: بیک وقت چلنے والی زیادہ سے زیادہ صلاحیتیں (بطورِ طے شدہ 2)
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `enabled`: آپٹ آؤٹ سوئچ (جب ماڈلز کنفیگر ہوں تو بطورِ طے شدہ true)
  - `prompt`: اختیاری پرامپٹ اووررائیڈ (تصویر/ویڈیو خودکار طور پر `maxChars` اشارہ شامل کرتے ہیں)
  - `maxChars`: زیادہ سے زیادہ آؤٹ پٹ کریکٹرز (تصویر/ویڈیو کے لیے بطورِ طے شدہ 500؛ آڈیو کے لیے غیر متعین)
  - `maxBytes`: بھیجنے کے لیے زیادہ سے زیادہ میڈیا سائز (ڈیفالٹس: تصویر 10MB، آڈیو 20MB، ویڈیو 50MB)۔
  - `timeoutSeconds`: درخواست کا ٹائم آؤٹ (بطورِ طے شدہ: تصویر 60s، آڈیو 60s، ویڈیو 120s)
  - `language`: اختیاری آڈیو اشارہ
  - `attachments`: اٹیچمنٹ پالیسی (`mode`, `maxAttachments`, `prefer`)
  - `scope`: اختیاری گیٹنگ (پہلا میچ جیتتا ہے) جس میں `match.channel`, `match.chatType`, یا `match.keyPrefix` شامل ہیں
  - `models`: ماڈل انٹریز کی ترتیب وار فہرست؛ ناکامی یا حد سے بڑے میڈیا پر اگلی انٹری پر فال بیک ہوتا ہے
- ہر `models[]` انٹری:
  - پرووائیڈر انٹری (`type: "provider"` یا غیر موجود):
    - `provider`: API پرووائیڈر آئی ڈی (`openai`, `anthropic`, `google`/`gemini`, `groq` وغیرہ)
    - `model`: ماڈل آئی ڈی اووررائیڈ (تصویر کے لیے لازمی؛ آڈیو پرووائیڈرز کے لیے ڈیفالٹس `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo`، اور ویڈیو کے لیے `gemini-3-flash-preview`)۔
    - `profile` / `preferredProfile`: توثیقی پروفائل کا انتخاب
  - CLI انٹری (`type: "cli"`):
    - `command`: چلانے کے لیے ایگزیکیوبل
    - `args`: ٹیمپلیٹڈ دلائل ( `{{MediaPath}}`، `{{Prompt}}`، `{{MaxChars}}` وغیرہ کی سپورٹ)
  - `capabilities`: اختیاری فہرست (`image`، `audio`، `video`) تاکہ مشترکہ انٹری کو گیٹ کیا جا سکے۔ نظرانداز ہونے پر ڈیفالٹس: `openai`/`anthropic`/`minimax` → تصویر، `google` → تصویر+آڈیو+ویڈیو، `groq` → آڈیو۔
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` کو فی انٹری اووررائیڈ کیا جا سکتا ہے

اگر کوئی ماڈلز کنفیگر نہیں ہیں (یا `enabled: false`)، تو سمجھ بوجھ کو اسکیپ کر دیا جاتا ہے؛ ماڈل کو پھر بھی اصل اٹیچمنٹس ملتی ہیں

پرووائیڈر توثیق معیاری ماڈل توثیقی ترتیب کی پیروی کرتی ہے (auth profiles، ماحولاتی متغیرات جیسے `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`، یا `models.providers.*.apiKey`)

مثال:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

`agents.defaults.subagents` ذیلی ایجنٹس کے ڈیفالٹس کو کنفیگر کرتا ہے:

- `model`: پیدا کیے گئے ذیلی ایجنٹس کے لیے ڈیفالٹ ماڈل (اسٹرنگ یا `{ primary, fallbacks }`) اگر غیر متعین ہو تو، ذیلی ایجنٹس کالر کے ماڈل کو وراثت میں لیتے ہیں جب تک کہ فی ایجنٹ یا فی کال اووررائیڈ نہ کیا جائے
- `maxConcurrent`: بیک وقت چلنے والے ذیلی ایجنٹس کی زیادہ سے زیادہ تعداد (بطورِ طے شدہ 1)
- `archiveAfterMinutes`: N منٹ بعد سب-ایجنٹ سیشنز کو خودکار طور پر آرکائیو کریں (ڈیفالٹ 60؛ غیر فعال کرنے کے لیے `0` سیٹ کریں)
- فی ذیلی ایجنٹ ٹول پالیسی: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (deny کو فوقیت)

`tools.profile`، `tools.allow`/`tools.deny` سے پہلے **بنیادی ٹول الاؤ لسٹ** سیٹ کرتا ہے:

- `minimal`: صرف `session_status`
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: کوئی پابندی نہیں (unset کے برابر)

فی ایجنٹ اووررائیڈ: `agents.list[].tools.profile`

مثال (بطورِ طے شدہ صرف میسجنگ، اور Slack + Discord ٹولز کی اجازت بھی):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

مثال (کوڈنگ پروفائل، مگر ہر جگہ exec/process کی ممانعت):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` مخصوص پرووائیڈرز (یا کسی ایک `provider/model`) کے لیے ٹولز کو **مزید محدود** کرنے دیتا ہے
فی ایجنٹ اووررائیڈ: `agents.list[].tools.byProvider`

ترتیب: بنیادی پروفائل → پرووائیڈر پروفائل → allow/deny پالیسیاں
پرووائیڈر کیز `provider` (مثلاً `google-antigravity`) یا `provider/model` دونوں قبول کرتی ہیں
(مثلاً `openai/gpt-5.2`)

مثال (عالمی کوڈنگ پروفائل برقرار رکھیں، مگر Google Antigravity کے لیے کم سے کم ٹولز):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

مثال (پرووائیڈر/ماڈل مخصوص الاؤ لسٹ):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

`tools.allow` / `tools.deny` عالمی ٹول الاؤ/ڈینی پالیسی کو کنفیگر کرتے ہیں (deny کو فوقیت)
میچنگ کیس اِن سنسیٹو ہے اور `*` وائلڈ کارڈز کی سپورٹ کرتی ہے (`"*"` کا مطلب تمام ٹولز)
یہ Docker سینڈ باکس **بند** ہونے پر بھی لاگو ہوتی ہے

مثال (ہر جگہ براؤزر/کینوس غیر فعال کریں):

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

ٹول گروپس (شارٹ ہینڈز) **عالمی** اور **فی ایجنٹ** ٹول پالیسیوں میں کام کرتے ہیں:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: تمام بلٹ اِن OpenClaw اوزار (فراہم کنندہ پلگ انز شامل نہیں)

`tools.elevated` ایلیویٹڈ (ہوسٹ) exec رسائی کو کنٹرول کرتا ہے:

- `enabled`: ایلیویٹڈ موڈ کی اجازت دیں (ڈیفالٹ true)
- `allowFrom`: فی چینل اجازت کی فہرستیں (خالی = غیر فعال)
  - `whatsapp`: E.164 نمبرز
  - `telegram`: چیٹ آئی ڈیز یا یوزر نیمز
  - `discord`: یوزر آئی ڈیز یا یوزر نیمز (اگر شامل نہ ہو تو `channels.discord.dm.allowFrom` پر واپس جاتا ہے)
  - `signal`: E.164 نمبرز
  - `imessage`: ہینڈلز/چیٹ آئی ڈیز
  - `webchat`: سیشن آئی ڈیز یا یوزر نیمز

Example:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

فی ایجنٹ اووررائیڈ (مزید پابندی کے لیے):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false },
        },
      },
    ],
  },
}
```

نوٹس:

- `tools.elevated` عالمی بنیاد ہے۔ `agents.list[].tools.elevated` صرف مزید پابندی لگا سکتا ہے (دونوں کی اجازت ضروری ہے)۔
- `/elevated on|off|ask|full` فی سیشن کلید کے مطابق حالت محفوظ کرتا ہے؛ اِن لائن ہدایات ایک ہی پیغام پر لاگو ہوتی ہیں۔
- ایلیویٹڈ `exec` ہوسٹ پر چلتا ہے اور سینڈباکسنگ کو بائی پاس کرتا ہے۔
- ٹول پالیسی بدستور لاگو رہتی ہے؛ اگر `exec` مسترد ہو تو ایلیویٹڈ استعمال نہیں کیا جا سکتا۔

`agents.defaults.maxConcurrent` ایمبیڈڈ ایجنٹ رنز کی زیادہ سے زیادہ تعداد سیٹ کرتا ہے جو سیشنز کے درمیان متوازی چل سکتی ہیں۔ ہر سیشن اب بھی سیریلائز ہوتا ہے (ایک وقت میں فی سیشن کلید ایک رن)۔ ڈیفالٹ: 1۔

### `agents.defaults.sandbox`

ایمبیڈڈ ایجنٹ کے لیے اختیاری **Docker sandboxing**۔ غیر مین سیشنز کے لیے بنایا گیا ہے تاکہ وہ آپ کے ہوسٹ سسٹم تک رسائی حاصل نہ کر سکیں۔

تفصیلات: [Sandboxing](/gateway/sandboxing)

ڈیفالٹس (اگر فعال ہو):

- scope: `"agent"` (ہر ایجنٹ کے لیے ایک کنٹینر + ورک اسپیس)
- Debian bookworm-slim پر مبنی امیج
- ایجنٹ ورک اسپیس رسائی: `workspaceAccess: "none"` (ڈیفالٹ)
  - `"none"`: `~/.openclaw/sandboxes` کے تحت فی اسکوپ سینڈباکس ورک اسپیس استعمال کریں
- `"ro"`: سینڈباکس ورک اسپیس کو `/workspace` پر رکھیں، اور ایجنٹ ورک اسپیس کو صرف پڑھنے کے لیے `/agent` پر ماؤنٹ کریں (`write`/`edit`/`apply_patch` غیر فعال ہو جاتے ہیں)
  - `"rw"`: ایجنٹ ورک اسپیس کو پڑھنے/لکھنے کے ساتھ `/workspace` پر ماؤنٹ کریں
- آٹو پرُون: غیر فعال > 24 گھنٹے یا عمر > 7 دن
- ٹول پالیسی: صرف `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` کی اجازت (انکار غالب ہے)
  - `tools.sandbox.tools` کے ذریعے کنفیگر کریں، اور فی ایجنٹ اووررائیڈ `agents.list[].tools.sandbox.tools` کے ذریعے
  - سینڈباکس پالیسی میں ٹول گروپ شارٹ ہینڈز کی سپورٹ: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (دیکھیں [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- اختیاری سینڈباکسڈ براؤزر (Chromium + CDP، noVNC آبزرور)
- ہارڈننگ کے اختیارات: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

انتباہ: `scope: "shared"` کا مطلب مشترکہ کنٹینر اور مشترکہ ورک اسپیس ہے۔ سیشنز کے درمیان کوئی آئسولیشن نہیں۔ فی سیشن آئسولیشن کے لیے `scope: "session"` استعمال کریں۔

لیگیسی: `perSession` اب بھی سپورٹڈ ہے (`true` → `scope: "session"`, `false` → `scope: "shared"`)۔

`setupCommand` کنٹینر بننے کے بعد **ایک بار** چلتا ہے (کنٹینر کے اندر `sh -lc` کے ذریعے)۔
پیکج انسٹالیشن کے لیے نیٹ ورک ایگریس، قابلِ تحریر روٹ FS، اور روٹ یوزر کو یقینی بنائیں۔

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Per-agent override (multi-agent): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7, // 0 disables max-age pruning
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

ڈیفالٹ سینڈباکس امیج ایک بار اس کمانڈ سے بنائیں:

```bash
scripts/sandbox-setup.sh
```

نوٹ: سینڈباکس کنٹینرز ڈیفالٹ طور پر `network: "none"` ہوتے ہیں؛ اگر ایجنٹ کو آؤٹ باؤنڈ رسائی درکار ہو تو `agents.defaults.sandbox.docker.network` کو `"bridge"` (یا اپنے کسٹم نیٹ ورک) پر سیٹ کریں۔

نوٹ: اِن باؤنڈ اٹیچمنٹس کو فعال ورک اسپیس میں `media/inbound/*` پر اسٹیج کیا جاتا ہے۔ `workspaceAccess: "rw"` کے ساتھ، اس کا مطلب ہے کہ فائلیں ایجنٹ ورک اسپیس میں لکھی جاتی ہیں۔

نوٹ: `docker.binds` اضافی ہوسٹ ڈائریکٹریز ماؤنٹ کرتا ہے؛ عالمی اور فی ایجنٹ بائنڈز کو ضم کیا جاتا ہے۔

اختیاری براؤزر امیج اس کمانڈ سے بنائیں:

```bash
scripts/sandbox-browser-setup.sh
```

جب `agents.defaults.sandbox.browser.enabled=true` ہو تو براؤزر ٹول سینڈباکسڈ Chromium انسٹینس (CDP) استعمال کرتا ہے۔ 1. اگر noVNC فعال ہو (headless=false ہونے پر ڈیفالٹ)،
noVNC URL کو سسٹم پرامپٹ میں شامل کر دیا جاتا ہے تاکہ ایجنٹ اس کا حوالہ دے سکے۔
2. اس کے لیے مرکزی کنفگ میں `browser.enabled` کی ضرورت نہیں؛ سینڈ باکس کنٹرول
URL ہر سیشن کے لیے شامل کیا جاتا ہے۔

3. `agents.defaults.sandbox.browser.allowHostControl` (ڈیفالٹ: false) سینڈ باکس شدہ سیشنز کو اجازت دیتا ہے کہ وہ براؤزر ٹول کے ذریعے (`target: "host"`) **ہوسٹ** براؤزر کنٹرول سرور کو واضح طور پر ہدف بنائیں۔ اگر آپ سخت سینڈ باکس آئسولیشن چاہتے ہیں تو اسے بند رکھیں۔

5. ریموٹ کنٹرول کے لیے الاؤ لسٹس:

- 6. `allowedControlUrls`: وہ عین کنٹرول URLs جو `target: "custom"` کے لیے اجازت یافتہ ہوں۔
- 7. `allowedControlHosts`: اجازت یافتہ ہوسٹ نیمز (صرف ہوسٹ نیم، پورٹ نہیں)۔
- 8. `allowedControlPorts`: اجازت یافتہ پورٹس (ڈیفالٹس: http=80, https=443)۔
  9. ڈیفالٹس: تمام الاؤ لسٹس غیر متعین ہیں (کوئی پابندی نہیں)۔ 10. `allowHostControl` کا ڈیفالٹ false ہے۔

### 11. `models` (کسٹم پرووائیڈرز + بیس URLs)

12. OpenClaw **pi-coding-agent** ماڈل کیٹلاگ استعمال کرتا ہے۔ 13. آپ کسٹم پرووائیڈرز شامل کر سکتے ہیں
    (LiteLLM، لوکل OpenAI-مطابقت رکھنے والے سرورز، Anthropic پراکسیز، وغیرہ) 14. اس کے لیے
    `~/.openclaw/agents/<agentId>/agent/models.json` لکھ کر یا وہی اسکیما اپنی OpenClaw کنفگ میں `models.providers` کے تحت متعین کر کے۔
13. ہر پرووائیڈر کا جائزہ + مثالیں: [/concepts/model-providers](/concepts/model-providers)۔

16. جب `models.providers` موجود ہو، OpenClaw اسٹارٹ اپ پر ایک `models.json` لکھتا/مرج کرتا ہے
    `~/.openclaw/agents/<agentId>/agent/` میں:

- ڈیفالٹ رویہ: **merge** (موجودہ پرووائیڈرز کو برقرار رکھتا ہے، نام کی بنیاد پر اووررائیڈ کرتا ہے)
- 18. فائل کے مواد کو اووررائیٹ کرنے کے لیے `models.mode: "replace"` سیٹ کریں

19. ماڈل منتخب کریں بذریعہ `agents.defaults.model.primary` (provider/model)۔

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {},
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

### 21. OpenCode Zen (ملٹی-ماڈل پراکسی)

22. OpenCode Zen ایک ملٹی-ماڈل گیٹ وے ہے جس میں ہر ماڈل کے لیے الگ اینڈ پوائنٹس ہوتے ہیں۔ 23. OpenClaw
    pi-ai کے بلٹ اِن `opencode` پرووائیڈر کو استعمال کرتا ہے؛ `OPENCODE_API_KEY` (یا
    `OPENCODE_ZEN_API_KEY`) کو [https://opencode.ai/auth](https://opencode.ai/auth) سے سیٹ کریں۔

نوٹس:

- 24. ماڈل ریفس `opencode/<modelId>` استعمال کرتے ہیں (مثال: `opencode/claude-opus-4-6`)۔
- 25. اگر آپ `agents.defaults.models` کے ذریعے الاؤ لسٹ فعال کرتے ہیں، تو ہر وہ ماڈل شامل کریں جسے آپ استعمال کرنے کا ارادہ رکھتے ہیں۔
- 26. شارٹ کٹ: `openclaw onboard --auth-choice opencode-zen`۔

```json5
27. {
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

### 28. Z.AI (GLM-4.7) — پرووائیڈر عرفی نام کی سپورٹ

29. Z.AI ماڈلز بلٹ اِن `zai` پرووائیڈر کے ذریعے دستیاب ہیں۔ 30. اپنے ماحول میں `ZAI_API_KEY`
    سیٹ کریں اور ماڈل کو provider/model کے ذریعے حوالہ دیں۔

31. شارٹ کٹ: `openclaw onboard --auth-choice zai-api-key`۔

```json5
32. {
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

نوٹس:

- 33. `z.ai/*` اور `z-ai/*` قابلِ قبول عرفی نام ہیں اور `zai/*` میں نارملائز ہو جاتے ہیں۔
- 34. اگر `ZAI_API_KEY` موجود نہ ہو تو `zai/*` پر کی گئی درخواستیں رن ٹائم پر آتھنٹیکیشن ایرر کے ساتھ ناکام ہو جائیں گی۔
- 35. مثال کے طور پر ایرر: `No API key found for provider "zai".`
- 36. Z.AI کا عمومی API اینڈ پوائنٹ `https://api.z.ai/api/paas/v4` ہے۔ 37. GLM کوڈنگ
      درخواستیں مخصوص کوڈنگ اینڈ پوائنٹ `https://api.z.ai/api/coding/paas/v4` استعمال کرتی ہیں۔
  37. بلٹ اِن `zai` پرووائیڈر کوڈنگ اینڈ پوائنٹ استعمال کرتا ہے۔ 39. اگر آپ کو عمومی
      اینڈ پوائنٹ درکار ہو تو `models.providers` میں بیس URL اووررائیڈ کے ساتھ ایک کسٹم پرووائیڈر متعین کریں (اوپر دیے گئے کسٹم پرووائیڈرز سیکشن کو دیکھیں)۔
- 40. ڈاکس/کنفگز میں جعلی پلیس ہولڈر استعمال کریں؛ کبھی بھی اصل API کیز کمٹ نہ کریں۔

### Moonshot AI (Kimi)

41. Moonshot کے OpenAI-مطابقت رکھنے والے اینڈ پوائنٹ کا استعمال کریں:

```json5
42. {
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

نوٹس:

- 43. ماحول میں `MOONSHOT_API_KEY` سیٹ کریں یا `openclaw onboard --auth-choice moonshot-api-key` استعمال کریں۔
- 44. ماڈل ریف: `moonshot/kimi-k2.5`۔
- 45. چینا اینڈ پوائنٹ کے لیے، ان میں سے ایک کریں:
  - 46. `openclaw onboard --auth-choice moonshot-api-key-cn` چلائیں (وزارڈ `https://api.moonshot.cn/v1` سیٹ کر دے گا)، یا
  - 47. `models.providers.moonshot` میں دستی طور پر `baseUrl: "https://api.moonshot.cn/v1"` سیٹ کریں۔

### Kimi Coding

48. Moonshot AI کا Kimi Coding اینڈ پوائنٹ استعمال کریں (Anthropic-مطابقت رکھنے والا، بلٹ اِن پرووائیڈر):

```json5
49. {
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

نوٹس:

- 50. ماحول میں `KIMI_API_KEY` سیٹ کریں یا `openclaw onboard --auth-choice kimi-code-api-key` استعمال کریں۔
- ماڈل حوالہ: `kimi-coding/k2p5`.

### سنتھیٹک (Anthropic کے ساتھ مطابقت پذیر)

سنتھیٹک کا Anthropic-مطابقت پذیر اینڈپوائنٹ استعمال کریں:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

نوٹس:

- `SYNTHETIC_API_KEY` سیٹ کریں یا `openclaw onboard --auth-choice synthetic-api-key` استعمال کریں۔
- ماڈل حوالہ: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`۔
- بیس URL میں `/v1` شامل نہ کریں کیونکہ Anthropic کلائنٹ اسے خود شامل کرتا ہے۔

### لوکل ماڈلز (LM Studio) — تجویز کردہ سیٹ اپ

موجودہ لوکل رہنمائی کے لیے [/gateway/local-models](/gateway/local-models) دیکھیں۔ TL;DR: سنجیدہ ہارڈویئر پر LM Studio Responses API کے ذریعے MiniMax M2.1 چلائیں؛ بیک اپ کے لیے ہوسٹڈ ماڈلز کو مرج رکھیں۔

### MiniMax M2.1

LM Studio کے بغیر MiniMax M2.1 کو براہِ راست استعمال کریں:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Pricing: update in models.json if you need exact cost tracking.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

نوٹس:

- `MINIMAX_API_KEY` انوائرنمنٹ ویری ایبل سیٹ کریں یا `openclaw onboard --auth-choice minimax-api` استعمال کریں۔
- دستیاب ماڈل: `MiniMax-M2.1` (ڈیفالٹ)۔
- اگر آپ کو درست لاگت کی ٹریکنگ درکار ہو تو `models.json` میں قیمتیں اپ ڈیٹ کریں۔

### Cerebras (GLM 4.6 / 4.7)

Cerebras کو ان کے OpenAI-مطابقت پذیر اینڈپوائنٹ کے ذریعے استعمال کریں:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

نوٹس:

- Cerebras کے لیے `cerebras/zai-glm-4.7` استعمال کریں؛ Z.AI براہِ راست کے لیے `zai/glm-4.7` استعمال کریں۔
- انوائرنمنٹ یا کنفیگ میں `CEREBRAS_API_KEY` سیٹ کریں۔

نوٹس:

- سپورٹڈ APIs: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
- کسٹم آتھنٹیکیشن کی ضروریات کے لیے `authHeader: true` + `headers` استعمال کریں۔
- `OPENCLAW_AGENT_DIR` (یا `PI_CODING_AGENT_DIR`) کے ذریعے ایجنٹ کنفیگ روٹ اووررائیڈ کریں
  اگر آپ چاہتے ہیں کہ `models.json` کہیں اور محفوظ ہو (ڈیفالٹ: `~/.openclaw/agents/main/agent`)۔

### `session`

سیشن اسکوپنگ، ری سیٹ پالیسی، ری سیٹ ٹرگرز، اور یہ کہ سیشن اسٹور کہاں لکھا جائے، کو کنٹرول کرتا ہے۔

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    // Default is already per-agent under ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // You can override with {agentId} templating:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

فیلڈز:

- `mainKey`: ڈائریکٹ چیٹ کا بکٹ کی (ڈیفالٹ: `"main"`)۔ اس وقت مفید ہے جب آپ `agentId` تبدیل کیے بغیر بنیادی DM تھریڈ کو “نام بدلنا” چاہتے ہوں۔
  - سینڈباکس نوٹ: `agents.defaults.sandbox.mode: "non-main"` اس کی کو مرکزی سیشن کی شناخت کے لیے استعمال کرتا ہے۔ کوئی بھی سیشن کی جو `mainKey` سے میل نہ کھائے (گروپس/چینلز) سینڈباکس میں ہوتا ہے۔
- `dmScope`: DM سیشنز کو کس طرح گروپ کیا جاتا ہے (ڈیفالٹ: `"main"`)۔
  - `main`: تسلسل کے لیے تمام DMs مرکزی سیشن شیئر کرتے ہیں۔
  - `per-peer`: چینلز کے پار بھیجنے والے کی آئی ڈی کے لحاظ سے DMs کو الگ رکھتا ہے۔
  - `per-channel-peer`: فی چینل + بھیجنے والا کے حساب سے DMs کو الگ رکھتا ہے (ملٹی یوزر ان باکسز کے لیے تجویز کردہ)۔
  - `per-account-channel-peer`: اکاؤنٹ + چینل + بھیجنے والے کے لحاظ سے DMs کو الگ کریں (ملٹی اکاؤنٹ اِن باکسز کے لیے تجویز کردہ)۔
  - محفوظ DM موڈ (تجویز کردہ): جب متعدد لوگ بوٹ کو DM کر سکتے ہوں (شیئرڈ ان باکسز، ملٹی پرسن الاولسٹس، یا `dmPolicy: "open"`) تو `session.dmScope: "per-channel-peer"` سیٹ کریں۔
- `identityLinks`: کینونیکل آئی ڈیز کو پرووائیڈر-پری فکسڈ پیئرز سے میپ کرتا ہے تاکہ `per-peer`, `per-channel-peer`, یا `per-account-channel-peer` استعمال کرتے وقت ایک ہی شخص مختلف چینلز پر ایک ہی DM سیشن شیئر کرے۔
  - مثال: `alice: ["telegram:123456789", "discord:987654321012345678"]`۔
- `reset`: بنیادی ری سیٹ پالیسی۔ ڈیفالٹ طور پر گیٹ وے ہوسٹ پر مقامی وقت کے مطابق صبح 4:00 بجے روزانہ ری سیٹ ہوتا ہے۔
  - `mode`: `daily` یا `idle` (ڈیفالٹ: جب `reset` موجود ہو تو `daily`)۔
  - `atHour`: روزانہ ری سیٹ کی حد کے لیے مقامی گھنٹہ (0-23)۔
  - `idleMinutes`: منٹوں میں سلائیڈنگ آئڈل ونڈو۔ جب daily اور idle دونوں کنفیگر ہوں، تو جو پہلے ایکسپائر ہو وہ لاگو ہوتا ہے۔
- `resetByType`: per-session overrides for `direct`, `group`, and `thread`. Legacy `dm` key is accepted as an alias for `direct`.
  - اگر آپ صرف لیگیسی `session.idleMinutes` سیٹ کریں اور کوئی `reset`/`resetByType` نہ ہو تو بیک ورڈ کمپیٹیبلٹی کے لیے OpenClaw صرف idle موڈ میں رہتا ہے۔
- `heartbeatIdleMinutes`: ہارٹ بیٹ چیکس کے لیے اختیاری idle اووررائیڈ (جب فعال ہو تو daily ری سیٹ لاگو رہتا ہے)۔
- `agentToAgent.maxPingPongTurns`: ریکوئسٹر/ٹارگٹ کے درمیان زیادہ سے زیادہ جوابی تبادلے (0–5، ڈیفالٹ 5)۔
- `sendPolicy.default`: جب کوئی رول میچ نہ ہو تو `allow` یا `deny` فال بیک۔
- `sendPolicy.rules[]`: `channel`, `chatType` (`direct|group|room`)، یا `keyPrefix` (مثلاً `cron:`) کے ذریعے میچ کریں۔ پہلا deny غالب آتا ہے؛ بصورت دیگر allow۔

### `skills` (اسکلز کنفیگ)

Controls bundled allowlist, install preferences, extra skill folders, and per-skill
overrides. Applies to **bundled** skills and `~/.openclaw/skills` (workspace skills
still win on name conflicts).

فیلڈز:

- `allowBundled`: optional allowlist for **bundled** skills only. If set, only those
  bundled skills are eligible (managed/workspace skills unaffected).
- `load.extraDirs`: اسکین کرنے کے لیے اضافی Skill ڈائریکٹریاں (کم ترین ترجیح)۔
- `install.preferBrew`: دستیاب ہونے پر brew انسٹالرز کو ترجیح دیں (بطورِ طے شدہ: true)۔
- `install.nodeManager`: node installer preference (`npm` | `pnpm` | `yarn`, default: npm).
- `entries.<skillKey>`: per-skill config overrides.

فی-Skill فیلڈز:

- `enabled`: `false` سیٹ کریں تاکہ Skill کو غیر فعال کیا جا سکے چاہے وہ بنڈلڈ/انسٹالڈ ہو۔
- `env`: ایجنٹ رن کے لیے انجیکٹ کیے گئے ماحولیاتی متغیرات (صرف اس صورت میں جب پہلے سے سیٹ نہ ہوں)۔
- `apiKey`: optional convenience for skills that declare a primary env var (e.g. `nano-banana-pro` → `GEMINI_API_KEY`).

Example:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### `plugins` (extensions)

Controls plugin discovery, allow/deny, and per-plugin config. پلگ انز لوڈ ہوتے ہیں
`~/.openclaw/extensions`، `<workspace>/.openclaw/extensions`، نیز کسی بھی
`plugins.load.paths` انٹریز سے۔ **Config changes require a gateway restart.**
See [/plugin](/tools/plugin) for full usage.

فیلڈز:

- `enabled`: پلگ ان لوڈنگ کے لیے ماسٹر ٹوگل (ڈیفالٹ: true)۔
- `allow`: پلگ ان آئی ڈیز کی اختیاری اجازت فہرست؛ جب سیٹ ہو تو صرف درج شدہ پلگ انز لوڈ ہوتے ہیں۔
- `deny`: optional denylist of plugin ids (deny wins).
- `load.paths`: لوڈ کرنے کے لیے اضافی پلگ ان فائلیں یا ڈائریکٹریز (مکمل راستہ یا `~`)۔
- `entries.<pluginId>`:\` فی-پلگ ان اووررائیڈز۔
  - `enabled`: set `false` to disable.
  - `config`: plugin-specific config object (validated by the plugin if provided).

Example:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio",
        },
      },
    },
  },
}
```

### `browser` (openclaw-managed browser)

OpenClaw can start a **dedicated, isolated** Chrome/Brave/Edge/Chromium instance for openclaw and expose a small loopback control service.
Profiles can point at a **remote** Chromium-based browser via `profiles.<name>.cdpUrl`. ریموٹ
پروفائلز صرف اٹیچ کے لیے ہوتے ہیں (start/stop/reset غیر فعال ہیں)۔

`browser.cdpUrl` لیگیسی سنگل-پروفائل کنفیگز کے لیے برقرار رہتا ہے اور اُن پروفائلز کے لیے بیس اسکیم/ہوسٹ کے طور پر بھی جو صرف `cdpPort` سیٹ کرتے ہیں۔

Defaults:

- enabled: `true`
- evaluateEnabled: `true` (set `false` to disable `act:evaluate` and `wait --fn`)
- control service: loopback only (port derived from `gateway.port`, default `18791`)
- CDP URL: `http://127.0.0.1:18792` (control service + 1, legacy single-profile)
- profile color: `#FF4500` (lobster-orange)
- نوٹ: کنٹرول سرور چلتے ہوئے گیٹ وے کے ذریعے شروع کیا جاتا ہے (OpenClaw.app مینو بار، یا `openclaw gateway`)۔
- Auto-detect order: default browser if Chromium-based; otherwise Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // set true when tunneling a remote CDP to localhost
  },
}
```

### `ui` (Appearance)

Optional accent color used by the native apps for UI chrome (e.g. Talk Mode bubble tint).

If unset, clients fall back to a muted light-blue.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Optional: Control UI assistant identity override.
    // If unset, the Control UI uses the active agent identity (config or IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, or image URL/data URI
    },
  },
}
```

### `gateway` (Gateway server mode + bind)

Use `gateway.mode` to explicitly declare whether this machine should run the Gateway.

Defaults:

- mode: **unset** (treated as “do not auto-start”)
- bind: `loopback`
- port: `18789` (single port for WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // token gates WS + Control UI access
    // tailscale: { mode: "off" | "serve" | "funnel" }
  },
}
```

Control UI base path:

- `gateway.controlUi.basePath` sets the URL prefix where the Control UI is served.
- Examples: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
- Default: root (`/`) (unchanged).
- `gateway.controlUi.root` کنٹرول UI اثاثوں کے لیے فائل سسٹم روٹ سیٹ کرتا ہے (ڈیفالٹ: `dist/control-ui`)۔
- `gateway.controlUi.allowInsecureAuth` allows token-only auth for the Control UI when
  device identity is omitted (typically over HTTP). Default: `false`. Prefer HTTPS
  (Tailscale Serve) or `127.0.0.1`.
- `gateway.controlUi.dangerouslyDisableDeviceAuth` disables device identity checks for the
  Control UI (token/password only). Default: `false`. Break-glass only.

متعلقہ دستاویزات:

- [کنٹرول UI](/web/control-ui)
- [Web overview](/web)
- [Tailscale](/gateway/tailscale)
- [ریموٹ رسائی](/gateway/remote)

قابلِ اعتماد پراکسیز:

- `gateway.trustedProxies`: list of reverse proxy IPs that terminate TLS in front of the Gateway.
- When a connection comes from one of these IPs, OpenClaw uses `x-forwarded-for` (or `x-real-ip`) to determine the client IP for local pairing checks and HTTP auth/local checks.
- Only list proxies you fully control, and ensure they **overwrite** incoming `x-forwarded-for`.

نوٹس:

- `openclaw gateway` اس وقت تک شروع ہونے سے انکار کرتا ہے جب تک `gateway.mode` کو `local` پر سیٹ نہ کیا جائے (یا آپ اووررائیڈ فلیگ پاس کریں)۔
- `gateway.port` controls the single multiplexed port used for WebSocket + HTTP (control UI, hooks, A2UI).
- OpenAI Chat Completions endpoint: **disabled by default**; enable with `gateway.http.endpoints.chatCompletions.enabled: true`.
- ترجیح: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > ڈیفالٹ `18789`۔
- Gateway auth is required by default (token/password or Tailscale Serve identity). Non-loopback binds require a shared token/password.
- آن بورڈنگ وزرڈ ڈیفالٹ طور پر ایک گیٹ وے ٹوکن بناتا ہے (لوپ بیک پر بھی)۔
- `gateway.remote.token` is **only** for remote CLI calls; it does not enable local gateway auth. `gateway.token` is ignored.

Auth and Tailscale:

- `gateway.auth.mode` ہینڈ شیک کی ضروریات سیٹ کرتا ہے (`token` یا `password`)۔ When unset, token auth is assumed.
- `gateway.auth.token` ٹوکن آتھ کے لیے مشترکہ ٹوکن محفوظ کرتا ہے (اسی مشین پر CLI کے ذریعے استعمال ہوتا ہے)۔
- When `gateway.auth.mode` is set, only that method is accepted (plus optional Tailscale headers).
- `gateway.auth.password` can be set here, or via `OPENCLAW_GATEWAY_PASSWORD` (recommended).
- `gateway.auth.allowTailscale` Tailscale Serve شناخت ہیڈرز کو اجازت دیتا ہے
  (`tailscale-user-login`) تاکہ جب درخواست لوپ بیک پر آئے اور
  `x-forwarded-for`، `x-forwarded-proto`، اور `x-forwarded-host` موجود ہوں تو آتھ پوری ہو سکے۔ OpenClaw
  شناخت کی توثیق `x-forwarded-for` ایڈریس کو
  `tailscale whois` کے ذریعے حل کر کے قبول کرنے سے پہلے کرتا ہے۔ When `true`, Serve requests do not need
  a token/password; set `false` to require explicit credentials. Defaults to
  `true` when `tailscale.mode = "serve"` and auth mode is not `password`.
- `gateway.tailscale.mode: "serve"` uses Tailscale Serve (tailnet only, loopback bind).
- `gateway.tailscale.mode: "funnel"` exposes the dashboard publicly; requires auth.
- `gateway.tailscale.resetOnExit` resets Serve/Funnel config on shutdown.

Remote client defaults (CLI):

- `gateway.remote.url` sets the default Gateway WebSocket URL for CLI calls when `gateway.mode = "remote"`.
- `gateway.remote.transport` selects the macOS remote transport (`ssh` default, `direct` for ws/wss). When `direct`, `gateway.remote.url` must be `ws://` or `wss://`. `ws://host` کا ڈیفالٹ پورٹ `18789` ہے۔
- `gateway.remote.token` supplies the token for remote calls (leave unset for no auth).
- `gateway.remote.password` supplies the password for remote calls (leave unset for no auth).

macOS app behavior:

- OpenClaw.app watches `~/.openclaw/openclaw.json` and switches modes live when `gateway.mode` or `gateway.remote.url` changes.
- If `gateway.mode` is unset but `gateway.remote.url` is set, the macOS app treats it as remote mode.
- When you change connection mode in the macOS app, it writes `gateway.mode` (and `gateway.remote.url` + `gateway.remote.transport` in remote mode) back to the config file.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Direct transport example (macOS app):

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### `gateway.reload` (Config hot reload)

The Gateway watches `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`) and applies changes automatically.

موڈز:

- `hybrid` (default): hot-apply safe changes; restart the Gateway for critical changes.
- `hot`: صرف hot-safe تبدیلیاں لاگو کریں؛ جب ری اسٹارٹ درکار ہو تو لاگ کریں۔
- `restart`: کسی بھی کنفیگ تبدیلی پر گیٹ وے کو ری اسٹارٹ کریں۔
- `off`: hot reload کو غیر فعال کریں۔

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

#### Hot reload میٹرکس (فائلیں + اثر)

مانیٹر کی گئی فائلیں:

- `~/.openclaw/openclaw.json` (یا `OPENCLAW_CONFIG_PATH`)

Hot-applied (بغیر مکمل گیٹ وے ری اسٹارٹ کے):

- `hooks` (ویب ہُک auth/path/mappings) + `hooks.gmail` (Gmail watcher ری اسٹارٹ)
- `browser` (براؤزر کنٹرول سرور ری اسٹارٹ)
- `cron` (کرون سروس ری اسٹارٹ + ہم وقتی اپڈیٹ)
- `agents.defaults.heartbeat` (heartbeat رنر ری اسٹارٹ)
- `web` (WhatsApp ویب چینل ری اسٹارٹ)
- `telegram`، `discord`، `signal`، `imessage` (چینل ری اسٹارٹس)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (ڈائنامک ریڈز)

مکمل گیٹ وے ری اسٹارٹ درکار:

- `gateway` (port/bind/auth/control UI/tailscale)
- `bridge` (legacy)
- `ڈسکوری`
- `canvasHost`
- `پلگ اِنز`
- کوئی بھی نامعلوم/غیر معاون کنفیگ پاتھ (حفاظت کے لیے ڈیفالٹ ری اسٹارٹ)

### ملٹی-انسٹینس آئسولیشن

ایک ہی ہوسٹ پر متعدد گیٹ ویز چلانے کے لیے (ریڈنڈنسی یا ریسکیو بوٹ کے لیے)، فی-انسٹینس اسٹیٹ + کنفیگ کو الگ کریں اور منفرد پورٹس استعمال کریں:

- `OPENCLAW_CONFIG_PATH` (فی انسٹینس کنفیگ)
- `OPENCLAW_STATE_DIR` (سیشنز/کریڈینشلز)
- `agents.defaults.workspace` (میموریز)
- `gateway.port` (ہر انسٹینس کے لیے منفرد)

سہولت فلیگز (CLI):

- `openclaw --dev …` → `~/.openclaw-dev` استعمال کرتا ہے + بیس `19001` سے پورٹس شفٹ کرتا ہے
- `openclaw --profile <name> …` → `~/.openclaw-<name>` استعمال کرتا ہے (پورٹ کنفیگ/این وی/فلیگز کے ذریعے)

اخذ شدہ پورٹ میپنگ (gateway/browser/canvas) کے لیے [Gateway runbook](/gateway) دیکھیں۔
براؤزر/CDP پورٹ آئسولیشن کی تفصیلات کے لیے [Multiple gateways](/gateway/multiple-gateways) دیکھیں۔

مثال:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

### `hooks` (گیٹ وے ویب ہُکس)

گیٹ وے HTTP سرور پر ایک سادہ HTTP ویب ہُک اینڈ پوائنٹ فعال کریں۔

بطورِ طے شدہ:

- enabled: `false`
- path: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

درخواستوں میں ہُک ٹوکن شامل ہونا چاہیے:

- `Authorization: Bearer <token>` **یا**
- `x-openclaw-token: <token>`

اینڈپوائنٹس:

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` `}`
- `POST /hooks/<name>` → `hooks.mappings` کے ذریعے resolve ہوتا ہے

`/hooks/agent` ہمیشہ مین سیشن میں ایک خلاصہ پوسٹ کرتا ہے (اور اختیاری طور پر `wakeMode: "now"` کے ذریعے فوری heartbeat ٹرگر کر سکتا ہے)۔

Mapping نوٹس:

- `match.path` `/hooks` کے بعد کے سب-پاتھ سے میچ کرتا ہے (مثلاً `/hooks/gmail` → `gmail`)۔
- `match.source` پے لوڈ فیلڈ سے میچ کرتا ہے (مثلاً `{ source: "gmail" }`) تاکہ آپ ایک عمومی `/hooks/ingest` پاتھ استعمال کر سکیں۔
- `{{messages[0].subject}}` جیسے ٹیمپلیٹس پے لوڈ سے ڈیٹا پڑھتے ہیں۔
- 1. `transform` کسی JS/TS ماڈیول کی طرف اشارہ کر سکتا ہے جو ایک ہُک ایکشن واپس کرے۔
- 2. `deliver: true` آخری جواب کو کسی چینل پر بھیجتا ہے؛ `channel` بطورِ ڈیفالٹ `last` ہوتا ہے (اور WhatsApp پر واپس آ جاتا ہے)۔
- 3. اگر پہلے سے کوئی ڈیلیوری روٹ موجود نہ ہو تو `channel` + `to` واضح طور پر سیٹ کریں (Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams کے لیے لازمی)۔
- 4. `model` اس ہُک رن کے لیے LLM کو اووررائیڈ کرتا ہے (`provider/model` یا عرفی نام؛ اگر `agents.defaults.models` سیٹ ہو تو اس کی اجازت ہونی چاہیے)۔

5. Gmail ہیلپر کنفیگ (استعمال شدہ بذریعہ `openclaw webhooks gmail setup` / `run`):

```json5
6. {
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // اختیاری: Gmail ہُک پروسیسنگ کے لیے سستا ماڈل استعمال کریں
      // auth/rate-limit/timeout کی صورت میں agents.defaults.model.fallbacks، پھر primary پر واپس جاتا ہے
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // اختیاری: Gmail ہُکس کے لیے ڈیفالٹ تھنکنگ لیول
      thinking: "off",
    },
  },
}
```

7. Gmail ہُکس کے لیے ماڈل اووررائیڈ:

- 8. `hooks.gmail.model` Gmail ہُک پروسیسنگ کے لیے استعمال ہونے والا ماڈل متعین کرتا ہے (بطورِ ڈیفالٹ سیشن primary)۔
- 9. `provider/model` ریفرنسز یا `agents.defaults.models` سے عرفی نام قبول کرتا ہے۔
- 10. auth/rate-limit/timeouts کی صورت میں `agents.defaults.model.fallbacks`، پھر `agents.defaults.model.primary` پر واپس جاتا ہے۔
- 11. اگر `agents.defaults.models` سیٹ ہو تو ہُکس ماڈل کو allowlist میں شامل کریں۔
- 12. اسٹارٹ اپ پر وارننگ دیتا ہے اگر کنفیگر کیا گیا ماڈل ماڈل کیٹلاگ یا allowlist میں موجود نہ ہو۔
- 13. `hooks.gmail.thinking` Gmail ہُکس کے لیے ڈیفالٹ تھنکنگ لیول سیٹ کرتا ہے اور فی ہُک `thinking` کے ذریعے اووررائیڈ ہو جاتا ہے۔

گیٹ وے خودکار آغاز:

- 15. اگر `hooks.enabled=true` اور `hooks.gmail.account` سیٹ ہو تو گیٹ وے بوٹ پر `gog gmail watch serve` شروع کرتا ہے اور واچ کو خودکار طور پر تجدید کرتا ہے۔
- 16. آٹو-اسٹارٹ کو غیر فعال کرنے کے لیے `OPENCLAW_SKIP_GMAIL_WATCHER=1` سیٹ کریں (دستی رنز کے لیے)۔
- 17. گیٹ وے کے ساتھ علیحدہ `gog gmail watch serve` چلانے سے گریز کریں؛ یہ `listen tcp 127.0.0.1:8788: bind: address already in use` کے ساتھ فیل ہو جائے گا۔

18. نوٹ: جب `tailscale.mode` آن ہو تو OpenClaw بطورِ ڈیفالٹ `serve.path` کو `/` پر سیٹ کرتا ہے تاکہ Tailscale `/gmail-pubsub` کو درست طور پر پراکسی کر سکے (یہ سیٹ-پاتھ پری فکس کو ہٹا دیتا ہے)۔
19. اگر آپ کو بیک اینڈ کو پری فکسڈ پاتھ وصول کرنا ہو تو `hooks.gmail.tailscale.target` کو مکمل URL پر سیٹ کریں (اور `serve.path` کو ہم آہنگ کریں)۔

### 20. `canvasHost` (LAN/tailnet کینوس فائل سرور + لائیو ری لوڈ)

21. گیٹ وے HTML/CSS/JS کی ایک ڈائریکٹری کو HTTP پر سرو کرتا ہے تاکہ iOS/Android نوڈز سادہ طور پر اس پر `canvas.navigate` کر سکیں۔

22. ڈیفالٹ روٹ: `~/.openclaw/workspace/canvas`  
    ڈیفالٹ پورٹ: `18793` (openclaw براؤزر CDP پورٹ `18792` سے بچنے کے لیے منتخب)  
    سرور **gateway bind host** (LAN یا Tailnet) پر سنتا ہے تاکہ نوڈز اس تک پہنچ سکیں۔

23. سرور:

- 24. `canvasHost.root` کے تحت فائلیں سرو کرتا ہے
- 25. سرو کی گئی HTML میں ایک چھوٹا لائیو-ری لوڈ کلائنٹ انجیکٹ کرتا ہے
- 26. ڈائریکٹری کو واچ کرتا ہے اور `/__openclaw__/ws` پر WebSocket اینڈپوائنٹ کے ذریعے ری لوڈز براڈکاسٹ کرتا ہے
- 27. جب ڈائریکٹری خالی ہو تو ایک اسٹارٹر `index.html` خودکار طور پر بناتا ہے (تاکہ فوراً کچھ نظر آئے)
- 28. `/__openclaw__/a2ui/` پر A2UI بھی سرو کرتا ہے اور نوڈز کو `canvasHostUrl` کے طور پر مشتہر کیا جاتا ہے  
      (ہمیشہ نوڈز کے ذریعے Canvas/A2UI کے لیے استعمال ہوتا ہے)

29. اگر ڈائریکٹری بڑی ہو یا `EMFILE` آئے تو لائیو ری لوڈ (اور فائل واچنگ) غیر فعال کریں:

- 30. کنفیگ: `canvasHost: { liveReload: false }`

```json5
31. {
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

32. `canvasHost.*` میں تبدیلیوں کے لیے گیٹ وے ری اسٹارٹ درکار ہے (کنفیگ ری لوڈ ری اسٹارٹ کرے گا)۔

اسے بند کرنے کے لیے:

- 33. کنفیگ: `canvasHost: { enabled: false }`
- 34. env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### 35. `bridge` (لیگیسی TCP برج، ہٹا دیا گیا)

36. موجودہ بلڈز میں TCP برج لِسنر شامل نہیں؛ `bridge.*` کنفیگ کیز نظر انداز کی جاتی ہیں۔
37. نوڈز گیٹ وے WebSocket کے ذریعے کنیکٹ ہوتے ہیں۔ 38. یہ سیکشن تاریخی حوالہ کے لیے رکھا گیا ہے۔

39. لیگیسی رویہ:

- 40. گیٹ وے نوڈز (iOS/Android) کے لیے ایک سادہ TCP برج ایکسپوز کر سکتا تھا، عموماً پورٹ `18790` پر۔

Defaults:

- 41. enabled: `true`
- 42. port: `18790`
- 43. bind: `lan` (`0.0.0.0` پر بائنڈ کرتا ہے)

44. بائنڈ موڈز:

- 45. `lan`: `0.0.0.0` (کسی بھی انٹرفیس پر قابلِ رسائی، بشمول LAN/Wi‑Fi اور Tailscale)
- 46. `tailnet`: صرف مشین کے Tailscale IP پر بائنڈ (Vienna ⇄ London کے لیے تجویز کردہ)
- 47. `loopback`: `127.0.0.1` (صرف لوکل)
- 48. `auto`: اگر دستیاب ہو تو tailnet IP کو ترجیح، ورنہ `lan`

49. TLS:

- 50. `bridge.tls.enabled`: برج کنکشنز کے لیے TLS فعال کریں (فعال ہونے پر صرف TLS)
- `bridge.tls.autoGenerate`: generate a self-signed cert when no cert/key are present (default: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: PEM paths for the bridge certificate + private key.
- `bridge.tls.caPath`: optional PEM CA bundle (custom roots or future mTLS).

When TLS is enabled, the Gateway advertises `bridgeTls=1` and `bridgeTlsSha256` in discovery TXT
records so nodes can pin the certificate. Manual connections use trust-on-first-use if no
fingerprint is stored yet.
Auto-generated certs require `openssl` on PATH; if generation fails, the bridge will not start.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Uses ~/.openclaw/bridge/tls/bridge-{cert,key}.pem when omitted.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS broadcast mode)

Controls LAN mDNS discovery broadcasts (`_openclaw-gw._tcp`).

- `minimal` (default): omit `cliPath` + `sshPort` from TXT records
- `full`: include `cliPath` + `sshPort` in TXT records
- `off`: disable mDNS broadcasts entirely
- Hostname: defaults to `openclaw` (advertises `openclaw.local`). Override with `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { mode: "minimal" } },
}
```

### `discovery.wideArea` (Wide-Area Bonjour / unicast DNS‑SD)

When enabled, the Gateway writes a unicast DNS-SD zone for `_openclaw-gw._tcp` under `~/.openclaw/dns/` using the configured discovery domain (example: `openclaw.internal.`).

To make iOS/Android discover across networks (Vienna ⇄ London), pair this with:

- a DNS server on the gateway host serving your chosen domain (CoreDNS is recommended)
- Tailscale **split DNS** so clients resolve that domain via the gateway DNS server

One-time setup helper (gateway host):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## Media model template variables

Template placeholders are expanded in `tools.media.*.models[].args` and `tools.media.models[].args` (and any future templated argument fields).

\| Variable           | Description                                                                     |
\| ------------------ | ------------------------------------------------------------------------------- | -------- | ------- | ---------- | ----- | ------ | -------- | ------- | ------- | --- |
\| `{{Body}}`         | Full inbound message body                                                       |
\| `{{RawBody}}`      | Raw inbound message body (no history/sender wrappers; best for command parsing) |
\| `{{BodyStripped}}` | Body with group mentions stripped (best default for agents)                     |
\| `{{From}}`         | Sender identifier (E.164 for WhatsApp; may differ per channel)                  |
\| `{{To}}`           | Destination identifier                                                          |
\| `{{MessageSid}}`   | Channel message id (when available)                                             |
\| `{{SessionId}}`    | Current session UUID                                                            |
\| `{{IsNewSession}}` | `"true"` when a new session was created                                         |
\| `{{MediaUrl}}`     | Inbound media pseudo-URL (if present)                                           |
\| `{{MediaPath}}`    | Local media path (if downloaded)                                                |
\| `{{MediaType}}`    | Media type (image/audio/document/…)                                             |
\| `{{Transcript}}`   | Audio transcript (when enabled)                                                 |
\| `{{Prompt}}`       | Resolved media prompt for CLI entries                                           |
\| `{{MaxChars}}`     | Resolved max output chars for CLI entries                                       |
\| `{{ChatType}}`     | `"direct"` or `"group"`                                                         |
\| `{{GroupSubject}}` | Group subject (best effort)                                                     |
\| `{{GroupMembers}}` | Group members preview (best effort)                                             |
\| `{{SenderName}}`   | Sender display name (best effort)                                               |
\| `{{SenderE164}}`   | Sender phone number (best effort)                                               |
\| `{{Provider}}`     | Provider hint (whatsapp                                                         | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## Cron (Gateway scheduler)

Cron is a Gateway-owned scheduler for wakeups and scheduled jobs. See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_Next: [Agent Runtime](/concepts/agent)_ 🦞

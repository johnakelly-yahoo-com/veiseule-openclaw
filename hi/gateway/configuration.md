---
title: "विन्यास"
---

# विन्यास 🔧

OpenClaw `~/.openclaw/openclaw.json` से एक वैकल्पिक **JSON5** विन्यास पढ़ता है (टिप्पणियाँ + ट्रेलिंग कॉमा अनुमत)।

8. यदि फ़ाइल मौजूद नहीं है, तो OpenClaw सुरक्षित-से डिफ़ॉल्ट्स का उपयोग करता है (एम्बेडेड Pi एजेंट + प्रति-प्रेषक सेशंस + वर्कस्पेस `~/.openclaw/workspace`)। 9. आमतौर पर आपको कॉन्फ़िग की ज़रूरत केवल इन मामलों में होती है:

- यह सीमित करना कि बॉट को कौन ट्रिगर कर सकता है (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, आदि)
- समूह allowlist + उल्लेख (mention) व्यवहार को नियंत्रित करना (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- संदेश उपसर्गों (prefixes) को अनुकूलित करना (`messages`)
- एजेंट का कार्यक्षेत्र सेट करना (`agents.defaults.workspace` या `agents.list[].workspace`)
- एंबेडेड एजेंट डिफ़ॉल्ट्स (`agents.defaults`) और सत्र व्यवहार (`session`) को ट्यून करना
- प्रति‑एजेंट पहचान सेट करना (`agents.list[].identity`)

> **विन्यास में नए हैं?** विस्तृत व्याख्याओं सहित पूर्ण उदाहरणों के लिए [Configuration Examples](/gateway/configuration-examples) मार्गदर्शिका देखें!

## सख्त विन्यास सत्यापन

10. OpenClaw केवल वही कॉन्फ़िगरेशन स्वीकार करता है जो स्कीमा से पूरी तरह मेल खाते हों।
11. अज्ञात keys, गलत प्रकार, या अमान्य मान सुरक्षा के लिए Gateway को **स्टार्ट होने से मना** कर देते हैं।

जब सत्यापन विफल होता है:

- Gateway बूट नहीं होता।
- केवल डायग्नोस्टिक कमांड्स अनुमत होते हैं (उदाहरण: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`)।
- सटीक समस्याएँ देखने के लिए `openclaw doctor` चलाएँ।
- माइग्रेशन/मरम्मत लागू करने के लिए `openclaw doctor --fix` (या `--yes`) चलाएँ।

Doctor तब तक परिवर्तन नहीं लिखता जब तक आप स्पष्ट रूप से `--fix`/`--yes` में ऑप्ट‑इन न करें।

## स्कीमा + UI संकेत

12. Gateway UI एडिटर्स के लिए `config.schema` के माध्यम से कॉन्फ़िग का JSON Schema प्रतिनिधित्व उपलब्ध कराता है।
13. Control UI इस स्कीमा से एक फ़ॉर्म रेंडर करता है, और आपात स्थिति के लिए **Raw JSON** एडिटर भी देता है।

चैनल प्लगइन्स और एक्सटेंशन्स अपने विन्यास के लिए स्कीमा + UI संकेत पंजीकृत कर सकते हैं, ताकि
चैनल सेटिंग्स ऐप्स के बीच हार्ड‑कोडेड फ़ॉर्म के बिना स्कीमा‑आधारित बनी रहें।

संकेत (लेबल, समूहकरण, संवेदनशील फ़ील्ड) स्कीमा के साथ ही आते हैं, ताकि क्लाइंट
विन्यास ज्ञान को हार्ड‑कोड किए बिना बेहतर फ़ॉर्म रेंडर कर सकें।

## लागू करें + पुनःआरंभ (RPC)

14. `config.apply` का उपयोग एक ही चरण में पूरे कॉन्फ़िग को वैलिडेट + लिखने और Gateway को रीस्टार्ट करने के लिए करें।
15. Gateway के वापस आने के बाद यह एक रीस्टार्ट सेंटिनल लिखता है और अंतिम सक्रिय सेशन को पिंग करता है।

Warning: `config.apply` replaces the **entire config**. 17. यदि आप केवल कुछ keys बदलना चाहते हैं,
`config.patch` या `openclaw config set` का उपयोग करें। 18. `~/.openclaw/openclaw.json` का बैकअप रखें।

पैरामीटर:

- `raw` (string) — पूरे विन्यास के लिए JSON5 पेलोड
- `baseHash` (वैकल्पिक) — `config.get` से विन्यास हैश (जब कोई विन्यास पहले से मौजूद हो तब आवश्यक)
- `sessionKey` (वैकल्पिक) — वेक‑अप पिंग के लिए अंतिम सक्रिय सत्र कुंजी
- `note` (वैकल्पिक) — restart sentinel में शामिल करने के लिए नोट
- `restartDelayMs` (वैकल्पिक) — पुनःआरंभ से पहले विलंब (डिफ़ॉल्ट 2000)

उदाहरण (`gateway call` के माध्यम से):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## आंशिक अपडेट (RPC)

19. मौजूदा कॉन्फ़िग में आंशिक अपडेट मर्ज करने के लिए `config.patch` का उपयोग करें, ताकि असंबंधित keys प्रभावित न हों। 20. यह JSON merge patch semantics लागू करता है:

- ऑब्जेक्ट्स पुनरावृत्त रूप से मर्ज होते हैं
- `null` किसी कुंजी को हटाता है
- एरेज़ प्रतिस्थापित होते हैं
  `config.apply` की तरह, यह सत्यापित करता है, विन्यास लिखता है, restart sentinel सहेजता है, और
  Gateway पुनःआरंभ को शेड्यूल करता है (जब `sessionKey` दिया गया हो तो वैकल्पिक वेक के साथ)।

पैरामीटर:

- `raw` (string) — केवल बदलने वाली कुंजियों वाला JSON5 पेलोड
- `baseHash` (आवश्यक) — `config.get` से विन्यास हैश
- `sessionKey` (वैकल्पिक) — वेक‑अप पिंग के लिए अंतिम सक्रिय सत्र कुंजी
- `note` (वैकल्पिक) — restart sentinel में शामिल करने के लिए नोट
- `restartDelayMs` (वैकल्पिक) — पुनःआरंभ से पहले विलंब (डिफ़ॉल्ट 2000)

उदाहरण:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## न्यूनतम विन्यास (अनुशंसित प्रारंभ बिंदु)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

डिफ़ॉल्ट इमेज एक बार बनाएँ:

```bash
scripts/sandbox-setup.sh
```

## Self‑chat मोड (समूह नियंत्रण के लिए अनुशंसित)

समूहों में WhatsApp @‑mentions पर बॉट के उत्तर देने से रोकने के लिए (केवल विशिष्ट टेक्स्ट ट्रिगर्स पर उत्तर दें):

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

## Config Includes (`$include`)

21. `$include` निर्देश का उपयोग करके अपने कॉन्फ़िग को कई फ़ाइलों में विभाजित करें। 22. यह इन मामलों में उपयोगी है:

- बड़े विन्यासों को व्यवस्थित करने के लिए (जैसे, प्रति‑क्लाइंट एजेंट परिभाषाएँ)
- परिवेशों के बीच सामान्य सेटिंग्स साझा करने के लिए
- संवेदनशील विन्यासों को अलग रखने के लिए

### मूल उपयोग

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

### मर्ज व्यवहार

- **एकल फ़ाइल**: `$include` वाले ऑब्जेक्ट को प्रतिस्थापित करती है
- **फ़ाइलों की एरे**: क्रम में फ़ाइलों को डीप‑मर्ज करती है (बाद की फ़ाइलें पहले वाली को ओवरराइड करती हैं)
- **सिब्लिंग कुंजियों के साथ**: includes के बाद सिब्लिंग कुंजियाँ मर्ज होती हैं (शामिल मानों को ओवरराइड करती हैं)
- **सिब्लिंग कुंजियाँ + एरे/प्रिमिटिव्स**: समर्थित नहीं (शामिल सामग्री एक ऑब्जेक्ट होनी चाहिए)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### नेस्टेड includes

शामिल की गई फ़ाइलें स्वयं `$include` निर्देश रख सकती हैं (अधिकतम 10 स्तर गहराई तक):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### पाथ समाधान

- **रिलेटिव पाथ्स**: शामिल करने वाली फ़ाइल के सापेक्ष हल किए जाते हैं
- **एब्सोल्यूट पाथ्स**: जैसे‑के‑तैसे उपयोग होते हैं
- **पैरेंट डायरेक्टरीज़**: `../` संदर्भ अपेक्षित रूप से काम करते हैं

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### त्रुटि प्रबंधन

- **गायब फ़ाइल**: हल किए गए पाथ के साथ स्पष्ट त्रुटि
- **पार्स त्रुटि**: बताता है कि कौन‑सी शामिल फ़ाइल विफल हुई
- **परिपत्र includes**: include चेन के साथ पता लगाकर रिपोर्ट किया जाता है

### उदाहरण: मल्टी‑क्लाइंट लीगल सेटअप

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

## सामान्य विकल्प

### Env vars + `.env`

OpenClaw पैरेंट प्रोसेस (shell, launchd/systemd, CI, आदि) से env vars पढ़ता है।

इसके अतिरिक्त, यह लोड करता है:

- वर्तमान कार्य निर्देशिका से `.env` (यदि मौजूद हो)
- `~/.openclaw/.env` से एक वैश्विक फ़ॉलबैक `.env` (उर्फ `$OPENCLAW_STATE_DIR/.env`)

इनमें से कोई भी `.env` फ़ाइल मौजूदा env vars को ओवरराइड नहीं करती।

23. आप कॉन्फ़िग में इनलाइन env vars भी प्रदान कर सकते हैं। 24. ये केवल तभी लागू होते हैं जब
    process env में वह key मौजूद न हो (वही non-overriding नियम):

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

पूर्ण प्राथमिकता और स्रोतों के लिए [/environment](/help/environment) देखें।

### `env.shellEnv` (वैकल्पिक)

25. ऑप्ट-इन सुविधा: यदि सक्षम हो और अपेक्षित कोई भी key अभी सेट न हो, तो OpenClaw आपका लॉगिन शेल चलाता है और केवल गायब अपेक्षित keys इम्पोर्ट करता है (कभी भी ओवरराइड नहीं करता)।
26. यह प्रभावी रूप से आपके शेल प्रोफ़ाइल को सोर्स करता है।

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

Env var समकक्ष:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### विन्यास में Env var प्रतिस्थापन

27. आप `${VAR_NAME}` सिंटैक्स का उपयोग करके किसी भी कॉन्फ़िग स्ट्रिंग मान में सीधे environment variables को संदर्भित कर सकते हैं। 28. वैरिएबल्स कॉन्फ़िग लोड समय पर, वैलिडेशन से पहले, सब्स्टिट्यूट किए जाते हैं।

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**नियम:**

- केवल अपरकेस env var नाम मिलान होते हैं: `[A-Z_][A-Z0-9_]*`
- गायब या खाली env vars विन्यास लोड पर त्रुटि फेंकते हैं
- शाब्दिक `${VAR}` आउटपुट करने के लिए `$${VAR}` से एस्केप करें
- `$include` के साथ काम करता है (शामिल फ़ाइलों में भी प्रतिस्थापन होता है)

**Inline प्रतिस्थापन:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### Auth स्टोरेज (OAuth + API keys)

OpenClaw **प्रति‑एजेंट** auth प्रोफ़ाइल्स (OAuth + API keys) को यहाँ संग्रहीत करता है:

- `<agentDir>/auth-profiles.json` (डिफ़ॉल्ट: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

यह भी देखें: [/concepts/oauth](/concepts/oauth)

Legacy OAuth आयात:

- `~/.openclaw/credentials/oauth.json` (या `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

एंबेडेड Pi एजेंट एक रनटाइम कैश बनाए रखता है:

- `<agentDir>/auth.json` (स्वचालित रूप से प्रबंधित; मैन्युअली संपादित न करें)

Legacy एजेंट डायरेक्टरी (pre multi‑agent):

- `~/.openclaw/agent/*` (`openclaw doctor` द्वारा `~/.openclaw/agents/<defaultAgentId>/agent/*` में माइग्रेट)

Overrides:

- OAuth dir (केवल legacy आयात): `OPENCLAW_OAUTH_DIR`
- Agent dir (डिफ़ॉल्ट एजेंट रूट ओवरराइड): `OPENCLAW_AGENT_DIR` (पसंदीदा), `PI_CODING_AGENT_DIR` (legacy)

पहले उपयोग पर, OpenClaw `oauth.json` प्रविष्टियों को `auth-profiles.json` में आयात करता है।

### `auth`

29. auth प्रोफ़ाइल्स के लिए वैकल्पिक मेटाडेटा। 30. यह **सीक्रेट्स स्टोर नहीं करता**; यह प्रोफ़ाइल IDs को एक प्रदाता + मोड (और वैकल्पिक ईमेल) से मैप करता है और फ़ेलओवर के लिए उपयोग किए जाने वाले प्रदाता रोटेशन क्रम को परिभाषित करता है।

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

Optional per-agent identity used for defaults and UX. 32. यह macOS ऑनबोर्डिंग असिस्टेंट द्वारा लिखा जाता है।

यदि सेट हो, तो OpenClaw डिफ़ॉल्ट्स व्युत्पन्न करता है (केवल तब जब आपने उन्हें स्पष्ट रूप से सेट न किया हो):

- `messages.ackReaction` **सक्रिय एजेंट** के `identity.emoji` से (👀 पर फ़ॉलबैक)
- `agents.list[].groupChat.mentionPatterns` एजेंट के `identity.name`/`identity.emoji` से
  (ताकि “@Samantha” Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp समूहों में काम करे)
- 33. `identity.avatar` वर्कस्पेस-रिलेटिव इमेज पाथ या रिमोट URL/data URL स्वीकार करता है। 34. लोकल फ़ाइलें एजेंट वर्कस्पेस के अंदर ही होनी चाहिए।

`identity.avatar` स्वीकार करता है:

- कार्यक्षेत्र‑सापेक्ष पाथ (एजेंट कार्यक्षेत्र के भीतर रहना चाहिए)
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

CLI विज़ार्ड्स द्वारा लिखा गया मेटाडेटा (`onboard`, `configure`, `doctor`)।

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

### `logging`

- डिफ़ॉल्ट लॉग फ़ाइल: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- यदि आप स्थिर पाथ चाहते हैं, तो `logging.file` को `/tmp/openclaw/openclaw.log` पर सेट करें।
- कंसोल आउटपुट को अलग से ट्यून किया जा सकता है:
  - `logging.consoleLevel` (डिफ़ॉल्ट `info`, `--verbose` होने पर `debug` तक बढ़ता है)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- सीक्रेट्स लीक होने से बचाने के लिए टूल सारांशों को रिडैक्ट किया जा सकता है:
  - `logging.redactSensitive` (`off` | `tools`, डिफ़ॉल्ट: `tools`)
  - `logging.redactPatterns` (regex स्ट्रिंग्स की एरे; डिफ़ॉल्ट्स को ओवरराइड करती है)

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

36. यह नियंत्रित करता है कि WhatsApp डायरेक्ट चैट्स (DMs) को कैसे संभाला जाता है:

- 37. `"pairing"` (डिफ़ॉल्ट): अज्ञात प्रेषकों को एक पेयरिंग कोड मिलता है; मालिक को अनुमोदन करना होता है
- 38. `"allowlist"`: केवल `channels.whatsapp.allowFrom` (या पेयर्ड allow स्टोर) में मौजूद प्रेषकों को अनुमति देता है
- 39. `"open"`: सभी इनबाउंड DMs की अनुमति देता है (**आवश्यक** है कि `channels.whatsapp.allowFrom` में `"*"` शामिल हो)
- 40. `"disabled"`: सभी इनबाउंड DMs को अनदेखा करता है

Pairing codes expire after 1 hour; the bot only sends a pairing code when a new request is created. 42. लंबित DM पेयरिंग अनुरोध डिफ़ॉल्ट रूप से **प्रति चैनल 3** तक सीमित हैं।

43. पेयरिंग अनुमोदन:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

45. E.164 फ़ोन नंबरों की allowlist जो WhatsApp ऑटो-रिप्लाई ट्रिगर कर सकती है (**केवल DMs**)।
46. यदि खाली है और `channels.whatsapp.dmPolicy="pairing"` है, तो अज्ञात प्रेषकों को पेयरिंग कोड मिलेगा।
47. समूहों के लिए, `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom` का उपयोग करें।

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

50. यह नियंत्रित करता है कि इनबाउंड WhatsApp संदेशों को पढ़ा हुआ (नीले टिक) के रूप में मार्क किया जाए या नहीं। Default: `true`.

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
{
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

टिप्पणियाँ:

- Outbound commands default to account `default` if present; otherwise the first configured account id (sorted).
- The legacy single-account Baileys auth dir is migrated by `openclaw doctor` into `whatsapp/default`.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

Run multiple accounts per channel (each account has its own `accountId` and optional `name`):

```json5
{
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

Notes:

- `default` is used when `accountId` is omitted (CLI + routing).
- Env tokens only apply to the **default** account.
- Base channel settings (group policy, mention gating, etc.) apply to all accounts unless overridden per account.
- Use `bindings[].match.accountId` to route each account to a different agents.defaults.

### Group chat mention gating (`agents.list[].groupChat` + `messages.groupChat`)

Group messages default to **require mention** (either metadata mention or regex patterns). Applies to WhatsApp, Telegram, Discord, Google Chat, and iMessage group chats.

**Mention types:**

- **Metadata mentions**: Native platform @-mentions (e.g., WhatsApp tap-to-mention). Ignored in WhatsApp self-chat mode (see `channels.whatsapp.allowFrom`).
- **Text patterns**: Regex patterns defined in `agents.list[].groupChat.mentionPatterns`. Always checked regardless of self-chat mode.
- Mention gating is enforced only when mention detection is possible (native mentions or at least one `mentionPattern`).

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

`messages.groupChat.historyLimit` sets the global default for group history context. Channels can override with `channels.<channel>.historyLimit` (or `channels.<channel>.accounts.*.historyLimit` for multi-account). Set `0` to disable history wrapping.

#### DM history limits

DM conversations use session-based history managed by the agent. You can limit the number of user turns retained per DM session:

```json5
{
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

रिज़ॉल्यूशन क्रम:

1. Per-DM override: `channels.<provider>.dms[userId].historyLimit`
2. Provider default: `channels.<provider>.dmHistoryLimit`
3. No limit (all history retained)

Supported providers: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Per-agent override (takes precedence when set, even `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

Mention gating defaults live per channel (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). When `*.groups` is set, it also acts as a group allowlist; include `"*"` to allow all groups.

To respond **only** to specific text triggers (ignoring native @-mentions):

```json5
{
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

### Group policy (per channel)

Use `channels.*.groupPolicy` to control whether group/room messages are accepted at all:

```json5
{
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

टिप्पणियाँ:

- `"open"`: groups bypass allowlists; mention-gating still applies.
- `"disabled"`: block all group/room messages.
- `"allowlist"`: only allow groups/rooms that match the configured allowlist.
- `channels.defaults.groupPolicy` sets the default when a provider’s `groupPolicy` is unset.
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams use `groupAllowFrom` (fallback: explicit `allowFrom`).
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
  - `subagents`: per-agent sub-agent defaults.
    - `allowAgents`: allowlist of agent ids for `sessions_spawn` from this agent (`["*"]` = allow any; default: only same agent)
  - `tools`: per-agent tool restrictions (applied before sandbox tool policy).
    - `profile`: base tool profile (applied before allow/deny)
    - `allow`: array of allowed tool names
    - `deny`: array of denied tool names (deny wins)
- `agents.defaults`: shared agent defaults (model, workspace, sandbox, etc.).
- `bindings[]`: routes inbound messages to an `agentId`.
  - `match.channel` (required)
  - `match.accountId` (optional; `*` = any account; omitted = default account)
  - `match.peer` (optional; `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (optional; channel-specific)

Deterministic match order:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exact, no peer/guild/team)
5. `match.accountId: "*"` (चैनल-व्यापी, कोई peer/guild/team नहीं)
6. डिफ़ॉल्ट एजेंट (`agents.list[].default`, अन्यथा सूची की पहली प्रविष्टि, अन्यथा `"main"`)

प्रत्येक मैच टियर के भीतर, `bindings` में पहली मेल खाने वाली प्रविष्टि मान्य होती है।

#### प्रति‑एजेंट एक्सेस प्रोफ़ाइल्स (multi‑agent)

प्रत्येक एजेंट अपना स्वयं का सैंडबॉक्स + टूल नीति रख सकता है। एक ही गेटवे में एक्सेस स्तरों को मिलाने के लिए इसका उपयोग करें:

- **पूर्ण एक्सेस** (पर्सनल एजेंट)
- **रीड-ओनली** टूल्स + वर्कस्पेस
- **कोई फ़ाइलसिस्टम एक्सेस नहीं** (केवल मैसेजिंग/सेशन टूल्स)

प्राथमिकता और अतिरिक्त उदाहरणों के लिए [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) देखें।

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

रीड-ओनली टूल्स + रीड-ओनली वर्कस्पेस:

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

कोई फ़ाइलसिस्टम एक्सेस नहीं (मैसेजिंग/सेशन टूल्स सक्षम):

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

उदाहरण: दो WhatsApp अकाउंट → दो एजेंट:

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

### `tools.agentToAgent` (वैकल्पिक)

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

जब किसी एजेंट रन पहले से सक्रिय हो, तब इनबाउंड संदेशों का व्यवहार नियंत्रित करता है।

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

**उसी प्रेषक** से आने वाले तेज़ इनबाउंड संदेशों को डिबाउंस करता है ताकि लगातार आने वाले कई संदेश एक ही एजेंट टर्न बन जाएँ। डिबाउंसिंग प्रति चैनल + वार्तालाप के अनुसार सीमित होती है और उत्तर थ्रेडिंग/ID के लिए सबसे हाल के संदेश का उपयोग करती है।

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

टिप्पणियाँ:

- **केवल टेक्स्ट** संदेशों के बैच को डिबाउंस करता है; मीडिया/अटैचमेंट तुरंत फ़्लश होते हैं।
- कंट्रोल कमांड (जैसे `/queue`, `/new`) डिबाउंसिंग को बायपास करते हैं ताकि वे अलग-अलग बने रहें।

### `commands` (चैट कमांड हैंडलिंग)

Controls how chat commands are enabled across connectors.

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

टिप्पणियाँ:

- टेक्स्ट कमांड को **स्टैंडअलोन** संदेश के रूप में और अग्रणी `/` के साथ भेजना आवश्यक है (कोई प्लेन-टेक्स्ट उपनाम नहीं)।
- `commands.text: false` चैट संदेशों में कमांड पार्सिंग को अक्षम करता है।
- `commands.native: "auto"` (डिफ़ॉल्ट) Discord/Telegram के लिए नेटिव कमांड चालू करता है और Slack को बंद छोड़ता है; असमर्थित चैनल केवल टेक्स्ट-आधारित रहते हैं।
- `commands.native: true|false` सेट करके सभी के लिए बाध्य करें, या प्रति चैनल `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool या `"auto"`) से ओवरराइड करें। `false` स्टार्टअप पर Discord/Telegram में पहले से पंजीकृत कमांड साफ़ कर देता है; Slack कमांड Slack ऐप में प्रबंधित होते हैं।
- `channels.telegram.customCommands` अतिरिक्त Telegram बॉट मेनू प्रविष्टियाँ जोड़ता है। नाम सामान्यीकृत किए जाते हैं; नेटिव कमांड के साथ टकराव को नज़रअंदाज़ किया जाता है।
- `commands.bash: true` `!` को सक्षम करता है `<cmd>` से होस्ट शेल कमांड चलाने के लिए (`/bash <cmd>` उपनाम के रूप में भी काम करता है)। `tools.elevated.enabled` की आवश्यकता होती है और `tools.elevated.allowFrom.<channel>` में प्रेषक को allowlist करना होता है`.`
- `commands.bashForegroundMs` यह नियंत्रित करता है कि बैकग्राउंड में भेजने से पहले bash कितनी देर प्रतीक्षा करे। जब कोई bash जॉब चल रहा हो, नए `!` `<cmd>` अनुरोध अस्वीकार कर दिए जाते हैं (एक समय में एक)।
- `commands.config: true` `/config` को सक्षम करता है (`openclaw.json` पढ़ता/लिखता है)।
- `channels.<provider>``.configWrites` उस चैनल द्वारा शुरू किए गए कॉन्फ़िग म्यूटेशन को नियंत्रित करता है (डिफ़ॉल्ट: true)। यह `/config set|unset` के साथ-साथ प्रदाता-विशिष्ट ऑटो-माइग्रेशन (Telegram सुपरग्रुप ID परिवर्तन, Slack चैनल ID परिवर्तन) पर भी लागू होता है।
- `commands.debug: true` `/debug` को सक्षम करता है (केवल रनटाइम ओवरराइड)।
- 2. `commands.useAccessGroups: false` कमांड्स को एक्सेस-ग्रुप अलाउलिस्ट/पॉलिसी को बायपास करने की अनुमति देता है।
- 3. स्लैश कमांड्स और निर्देश केवल **अधिकृत प्रेषकों** के लिए मान्य होते हैं।
- 4. प्राधिकरण चैनल अलाउलिस्ट/पेयरिंग और `commands.useAccessGroups` से प्राप्त होता है। 5. `web` (WhatsApp वेब चैनल रनटाइम)

### 6. WhatsApp गेटवे के वेब चैनल (Baileys Web) के माध्यम से चलता है।

7. जब कोई लिंक किया हुआ सेशन मौजूद होता है तो यह अपने आप शुरू हो जाता है। 8. डिफ़ॉल्ट रूप से बंद रखने के लिए `web.enabled: false` सेट करें।
8. {
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

```json5
10. `channels.telegram` (बॉट ट्रांसपोर्ट)
```

### 11. OpenClaw केवल तभी Telegram शुरू करता है जब `channels.telegram` का कॉन्फ़िग सेक्शन मौजूद हो।

12. बॉट टोकन `channels.telegram.botToken` (या `channels.telegram.tokenFile`) से प्राप्त होता है, और डिफ़ॉल्ट अकाउंट के लिए `TELEGRAM_BOT_TOKEN` एक फ़ॉलबैक है। The bot token is resolved from `channels.telegram.botToken` (or `channels.telegram.tokenFile`), with `TELEGRAM_BOT_TOKEN` as a fallback for the default account.
13. मल्टी-अकाउंट सपोर्ट `channels.telegram.accounts` के अंतर्गत होता है (ऊपर दिए गए मल्टी-अकाउंट सेक्शन को देखें)।
    Multi-account support lives under `channels.telegram.accounts` (see the multi-account section above). 16. Telegram द्वारा शुरू की गई कॉन्फ़िग राइट्स (जिसमें सुपरग्रुप ID माइग्रेशन और `/config set|unset` शामिल हैं) को रोकने के लिए `channels.telegram.configWrites: false` सेट करें।
14. {
    channels: {
    telegram: {
    enabled: true,
    botToken: "your-bot-token",
    dmPolicy: "pairing", // pairing | allowlist | open | disabled
    allowFrom: ["tg:123456789"], // optional; "open" requires ["_"]
    groups: {
    "_": { requireMention: true },
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

```json5
18. ड्राफ्ट स्ट्रीमिंग नोट्स:
```

Draft streaming notes:

- Uses Telegram `sendMessageDraft` (draft bubble, not a real message).
- Requires **private chat topics** (message_thread_id in DMs; bot has topics enabled).
- 22. रिट्राई पॉलिसी के डिफ़ॉल्ट्स और व्यवहार [Retry policy](/concepts/retry) में प्रलेखित हैं।
  23. `channels.discord` (बॉट ट्रांसपोर्ट)

### `channels.discord` (bot transport)

25. एनवायरनमेंट टोकन केवल डिफ़ॉल्ट अकाउंट पर लागू होते हैं। 26. {
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
    allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["\*"])
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

```json5
27. OpenClaw केवल तभी Discord शुरू करता है जब `channels.discord` का कॉन्फ़िग सेक्शन मौजूद हो।
```

28. टोकन `channels.discord.token` से प्राप्त होता है, और डिफ़ॉल्ट अकाउंट के लिए `DISCORD_BOT_TOKEN` एक फ़ॉलबैक है (जब तक `channels.discord.enabled` `false` न हो)। The token is resolved from `channels.discord.token`, with `DISCORD_BOT_TOKEN` as a fallback for the default account (unless `channels.discord.enabled` is `false`). 30. गिल्ड स्लग लोअरकेस होते हैं और स्पेस को `-` से बदला जाता है; चैनल कीज़ स्लग किए गए चैनल नाम का उपयोग करती हैं (कोई अग्रणी `#` नहीं)।
29. नाम बदलने की अस्पष्टता से बचने के लिए की के रूप में गिल्ड IDs को प्राथमिकता दें। 32. बॉट द्वारा लिखे गए संदेश डिफ़ॉल्ट रूप से अनदेखा किए जाते हैं।
30. `channels.discord.allowBots` के साथ सक्षम करें (स्वयं के संदेश अभी भी सेल्फ-रिप्लाई लूप रोकने के लिए फ़िल्टर किए जाते हैं)। 34. रिएक्शन नोटिफ़िकेशन मोड्स:
31. `allowlist`: `guilds.<id>` से रिएक्शन

- `off`: कोई reaction events नहीं।
- `own`: बॉट के अपने संदेशों पर reactions (डिफ़ॉल्ट)।
- `all`: सभी संदेशों पर सभी reactions।
- 36. `.users` सभी संदेशों पर (खाली सूची अक्षम करती है)।37. आउटबाउंड टेक्स्ट `channels.discord.textChunkLimit` (डिफ़ॉल्ट 2000) के अनुसार चंक किया जाता है।
  37. लंबाई के अनुसार चंकिंग से पहले खाली लाइनों (पैराग्राफ सीमाओं) पर विभाजित करने के लिए `channels.discord.chunkMode="newline"` सेट करें। 39. Discord क्लाइंट बहुत ऊँचे संदेशों को क्लिप कर सकते हैं, इसलिए `channels.discord.maxLinesPerMessage` (डिफ़ॉल्ट 17) लंबे मल्टी-लाइन उत्तरों को 2000 कैरेक्टर से कम होने पर भी विभाजित करता है। 40. रिट्राई पॉलिसी के डिफ़ॉल्ट्स और व्यवहार [Retry policy](/concepts/retry) में प्रलेखित हैं।
  38. `channels.googlechat` (Chat API वेबहुक)

### 42. Google Chat HTTP वेबहुक्स के माध्यम से ऐप-स्तरीय ऑथ (सर्विस अकाउंट) के साथ चलता है।

43. मल्टी-अकाउंट सपोर्ट `channels.googlechat.accounts` के अंतर्गत होता है (ऊपर दिए गए मल्टी-अकाउंट सेक्शन को देखें)।
44. एनवायरनमेंट वेरिएबल्स केवल डिफ़ॉल्ट अकाउंट पर लागू होते हैं। 45. {
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
    allowFrom: ["users/1234567890"], // optional; "open" requires ["\*"]
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

```json5
46. सर्विस अकाउंट JSON इनलाइन (`serviceAccount`) या फ़ाइल-आधारित (`serviceAccountFile`) हो सकता है।
```

टिप्पणियाँ:

- 47. डिफ़ॉल्ट अकाउंट के लिए एनवायरनमेंट फ़ॉलबैक: `GOOGLE_CHAT_SERVICE_ACCOUNT` या `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`।
- 48. `audienceType` + `audience` को Chat ऐप के वेबहुक ऑथ कॉन्फ़िग से मेल खाना चाहिए।
- 49. डिलीवरी टार्गेट सेट करते समय `spaces/<spaceId>` या `users/<userId|email>` का उपयोग करें।
- 50. `channels.slack` (सॉकेट मोड)

### `channels.slack` (socket mode)

Slack Socket Mode में चलता है और इसके लिए bot token और app token दोनों आवश्यक हैं:

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

मल्टी-अकाउंट सपोर्ट `channels.slack.accounts` के अंतर्गत होता है (ऊपर दिए गए मल्टी-अकाउंट सेक्शन को देखें)। Env tokens केवल डिफ़ॉल्ट अकाउंट पर लागू होते हैं।

जब provider सक्षम हो और दोनों tokens सेट हों (config या `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` के माध्यम से), तो OpenClaw Slack शुरू करता है। cron/CLI कमांड्स के लिए डिलीवरी टार्गेट निर्दिष्ट करते समय `user:<id>` (DM) या `channel:<id>` का उपयोग करें।
Set `channels.slack.configWrites: false` to block Slack-initiated config writes (including channel ID migrations and `/config set|unset`).

डिफ़ॉल्ट रूप से bot द्वारा लिखे गए संदेशों को अनदेखा किया जाता है। `channels.slack.allowBots` या `channels.slack.channels.<id>` के साथ सक्षम करें.allowBots\`.

Reaction notification modes:

- `off`: कोई reaction events नहीं।
- `own`: बॉट के अपने संदेशों पर reactions (डिफ़ॉल्ट)।
- `all`: सभी संदेशों पर सभी reactions।
- `allowlist`: सभी संदेशों पर `channels.slack.reactionAllowlist` से आने वाली reactions (खाली सूची होने पर अक्षम)।

Thread session isolation:

- `channels.slack.thread.historyScope` यह नियंत्रित करता है कि thread history प्रति-thread (`thread`, डिफ़ॉल्ट) हो या पूरे channel में साझा (`channel`) हो।
- `channels.slack.thread.inheritParent` यह नियंत्रित करता है कि नए thread sessions parent channel transcript को विरासत में लें या नहीं (डिफ़ॉल्ट: false)।

Slack action groups (`slack` tool actions को gate करते हैं):

| Action group | डिफ़ॉल्ट | नोट्स                     |
| ------------ | -------- | ------------------------- |
| reactions    | enabled  | React + reactions सूची    |
| messages     | enabled  | पढ़ना/भेजना/संपादित/हटाना |
| pins         | enabled  | Pin/unpin/सूची            |
| memberInfo   | enabled  | सदस्य जानकारी             |
| emojiList    | enabled  | कस्टम emoji सूची          |

### `channels.mattermost` (bot token)

Mattermost एक प्लगइन के रूप में प्रदान किया जाता है और कोर इंस्टॉल के साथ बंडल नहीं होता।
पहले इसे इंस्टॉल करें: `openclaw plugins install @openclaw/mattermost` (या git checkout से `./extensions/mattermost`)।

Mattermost को bot token के साथ आपके server का base URL भी चाहिए:

```json5
{
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

जब अकाउंट कॉन्फ़िगर (bot token + base URL) और सक्षम हो, तो OpenClaw Mattermost शुरू करता है। डिफ़ॉल्ट अकाउंट के लिए token + base URL को `channels.mattermost.botToken` + `channels.mattermost.baseUrl` या `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` से resolve किया जाता है (जब तक `channels.mattermost.enabled` `false` न हो)।

Chat modes:

- `oncall` (डिफ़ॉल्ट): केवल तब channel संदेशों का उत्तर देता है जब @mention किया जाए।
- `onmessage`: हर चैनल संदेश का उत्तर दें।
- `onchar`: जब कोई संदेश trigger prefix से शुरू होता है, तब उत्तर देता है (`channels.mattermost.oncharPrefixes`, डिफ़ॉल्ट `[">", "!"]`)।

Access control:

- डिफ़ॉल्ट DMs: `channels.mattermost.dmPolicy="pairing"` (अज्ञात भेजने वालों को pairing code मिलता है)।
- सार्वजनिक DMs: `channels.mattermost.dmPolicy="open"` के साथ `channels.mattermost.allowFrom=["*"]`।
- Groups: `channels.mattermost.groupPolicy="allowlist"` डिफ़ॉल्ट रूप से (mention-gated)। भेजने वालों को सीमित करने के लिए `channels.mattermost.groupAllowFrom` का उपयोग करें।

मल्टी-अकाउंट सपोर्ट `channels.mattermost.accounts` के अंतर्गत होता है (ऊपर दिए गए मल्टी-अकाउंट सेक्शन को देखें)। Env vars केवल डिफ़ॉल्ट अकाउंट पर लागू होते हैं।
डिलीवरी टार्गेट निर्दिष्ट करते समय `channel:<id>` या `user:<id>` (या `@username`) का उपयोग करें; बिना prefix वाले ids को channel ids माना जाता है।

### `channels.signal` (signal-cli)

Signal reactions system events उत्पन्न कर सकती हैं (shared reaction tooling):

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // include last N group messages as context (0 disables)
    },
  },
}
```

Reaction notification modes:

- `off`: कोई reaction events नहीं।
- `own`: बॉट के अपने संदेशों पर reactions (डिफ़ॉल्ट)।
- `all`: सभी संदेशों पर सभी reactions।
- `allowlist`: सभी संदेशों पर `channels.signal.reactionAllowlist` से आने वाली reactions (खाली सूची होने पर अक्षम)।

### `channels.imessage` (imsg CLI)

OpenClaw `imsg rpc` (stdio पर JSON-RPC) शुरू करता है। कोई daemon या port आवश्यक नहीं है।

```json5
{
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

मल्टी-अकाउंट सपोर्ट `channels.imessage.accounts` के अंतर्गत होता है (ऊपर दिए गए मल्टी-अकाउंट सेक्शन को देखें)।

टिप्पणियाँ:

- Messages DB के लिए Full Disk Access आवश्यक है।
- पहली बार भेजने पर Messages automation permission के लिए prompt आएगा।
- `chat_id:<id>` targets को प्राथमिकता दें। चैट्स की सूची देखने के लिए `imsg chats --limit 20` का उपयोग करें।
- `channels.imessage.cliPath` किसी wrapper script की ओर इशारा कर सकता है (उदाहरण: दूसरे Mac पर `imsg rpc` चलाने के लिए `ssh`); पासवर्ड prompts से बचने के लिए SSH keys का उपयोग करें।
- remote SSH wrappers के लिए, जब `includeAttachments` सक्षम हो तो attachments को SCP के माध्यम से प्राप्त करने हेतु `channels.imessage.remoteHost` सेट करें।

उदाहरण रैपर:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

एजेंट द्वारा file operations के लिए उपयोग की जाने वाली **एकल global workspace directory** सेट करता है।

डिफ़ॉल्ट: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

यदि `agents.defaults.sandbox` सक्षम है, तो non-main sessions इसे `agents.defaults.sandbox.workspaceRoot` के अंतर्गत अपने प्रति-scope workspaces के साथ override कर सकते हैं।

### `agents.defaults.repoRoot`

सिस्टम प्रॉम्प्ट की Runtime लाइन में दिखाने के लिए वैकल्पिक रिपॉज़िटरी रूट। यदि सेट नहीं है, तो OpenClaw वर्कस्पेस (और वर्तमान कार्यशील डायरेक्टरी) से ऊपर की ओर चलते हुए `.git` डायरेक्टरी का पता लगाने की कोशिश करता है। उपयोग के लिए पाथ का मौजूद होना आवश्यक है।

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

वर्कस्पेस बूटस्ट्रैप फ़ाइलों (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, और `BOOTSTRAP.md`) के स्वचालित निर्माण को अक्षम करता है।

इसे उन प्री-सीडेड डिप्लॉयमेंट्स के लिए उपयोग करें जहाँ आपकी वर्कस्पेस फ़ाइलें किसी repo से आती हैं।

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

ट्रंकेशन से पहले सिस्टम प्रॉम्प्ट में इंजेक्ट की जाने वाली प्रत्येक वर्कस्पेस बूटस्ट्रैप फ़ाइल के अधिकतम अक्षर। डिफ़ॉल्ट: `20000`.

जब कोई फ़ाइल इस सीमा से अधिक हो जाती है, तो OpenClaw एक चेतावनी लॉग करता है और मार्कर के साथ ट्रंकेट किया हुआ head/tail इंजेक्ट करता है।

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

उपयोगकर्ता का टाइमज़ोन **सिस्टम प्रॉम्प्ट संदर्भ** के लिए सेट करता है (मैसेज एनवेलप्स में टाइमस्टैम्प्स के लिए नहीं)। यदि सेट नहीं है, तो OpenClaw रनटाइम पर होस्ट टाइमज़ोन का उपयोग करता है।

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

सिस्टम प्रॉम्प्ट के Current Date & Time सेक्शन में दिखाए जाने वाले **समय प्रारूप** को नियंत्रित करता है।
डिफ़ॉल्ट: `auto` (OS वरीयता)।

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `संदेश`

इनबाउंड/आउटबाउंड प्रीफ़िक्स और वैकल्पिक ack प्रतिक्रियाओं को नियंत्रित करता है।
क्यूइंग, सेशंस और स्ट्रीमिंग संदर्भ के लिए [Messages](/concepts/messages) देखें।

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

`responsePrefix` सभी **आउटबाउंड उत्तरों** (टूल समरी, ब्लॉक स्ट्रीमिंग, अंतिम उत्तर) पर, सभी चैनलों में लागू होता है, जब तक कि वह पहले से मौजूद न हो।

ओवरराइड्स प्रति चैनल और प्रति अकाउंट कॉन्फ़िगर किए जा सकते हैं:

- `channels.<channel>``.responsePrefix`
- `channels.<channel>``.accounts.<id>``.responsePrefix`

समाधान क्रम (सबसे विशिष्ट को प्राथमिकता):

1. `channels.<channel>``.accounts.<id>``.responsePrefix`
2. `channels.<channel>``.responsePrefix`
3. `messages.responsePrefix`

अर्थ-विज्ञान (Semantics):

- `undefined` अगले स्तर पर फ़ॉल-थ्रू करता है।
- `""` प्रीफ़िक्स को स्पष्ट रूप से अक्षम करता है और कैस्केड को रोक देता है।
- `"auto"` रूट किए गए एजेंट के लिए `[{identity.name}]` निकालता है।

ओवरराइड्स सभी चैनलों (एक्सटेंशन्स सहित) और हर प्रकार के आउटबाउंड उत्तर पर लागू होते हैं।

यदि `messages.responsePrefix` सेट नहीं है, तो डिफ़ॉल्ट रूप से कोई प्रीफ़िक्स लागू नहीं होता। WhatsApp सेल्फ-चैट उत्तर अपवाद हैं: सेट होने पर वे डिफ़ॉल्ट रूप से `[{identity.name}]` का उपयोग करते हैं, अन्यथा `[openclaw]`, ताकि एक ही फ़ोन की बातचीत स्पष्ट बनी रहे।
रूट किए गए एजेंट के लिए `[{identity.name}]` निकालने हेतु इसे `"auto"` पर सेट करें (जब सेट हो)।

#### टेम्पलेट वेरिएबल्स

`responsePrefix` स्ट्रिंग में ऐसे टेम्पलेट वेरिएबल्स शामिल हो सकते हैं जो डायनामिक रूप से रेज़ॉल्व होते हैं:

| Variable                        | विवरण                 | Example                                 |
| ------------------------------- | --------------------- | --------------------------------------- |
| `{model}`                       | संक्षिप्त मॉडल नाम    | `claude-opus-4-6`, `gpt-4o`             |
| `{modelFull}`                   | पूर्ण मॉडल पहचानकर्ता | `anthropic/claude-opus-4-6`             |
| {provider}                      | प्रोवाइडर का नाम      | `anthropic`, `openai`                   |
| {thinkingLevel}                 | वर्तमान सोच स्तर      | `high`, `low`, `off`                    |
| {identity.name} | एजेंट पहचान का नाम    | ("auto" मोड के समान) |

Variables are case-insensitive (`{MODEL}` = `{model}`). `{think}` `{thinkingLevel}` का एक उपनाम है।
अनरिज़ॉल्व्ड वेरिएबल्स लिटरल टेक्स्ट के रूप में ही रहते हैं।

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]",
  },
}
```

उदाहरण आउटपुट: `[claude-opus-4-6 | think:high] Here's my response...`

WhatsApp इनबाउंड प्रीफ़िक्स `channels.whatsapp.messagePrefix` के माध्यम से कॉन्फ़िगर किया जाता है (डिप्रिकेटेड:
`messages.messagePrefix`)। डिफ़ॉल्ट **अपरिवर्तित** रहता है: `"[openclaw]"` जब
`channels.whatsapp.allowFrom` खाली हो, अन्यथा `""` (कोई प्रीफ़िक्स नहीं)। `"[openclaw]"` का उपयोग करते समय, OpenClaw इसके बजाय `[{identity.name}]` का उपयोग करेगा जब रूट किए गए
एजेंट में `identity.name` सेट हो।

`ackReaction` उन चैनलों पर इनबाउंड संदेशों को स्वीकार करने के लिए सर्वश्रेष्ठ‑प्रयास इमोजी रिएक्शन भेजता है
जो रिएक्शन का समर्थन करते हैं (Slack/Discord/Telegram/Google Chat)। डिफ़ॉल्ट रूप से सक्रिय एजेंट के `identity.emoji` का उपयोग करता है, यदि सेट हो; अन्यथा `"👀"`। इसे अक्षम करने के लिए `""` पर सेट करें।

`ackReactionScope` नियंत्रित करता है कि रिएक्शन कब ट्रिगर हों:

- `group-mentions` (डिफ़ॉल्ट): केवल तब जब किसी ग्रुप/रूम में मेंशन आवश्यक हों **और** बॉट को मेंशन किया गया हो
- `group-all`: सभी ग्रुप/रूम संदेश
- `direct`: केवल डायरेक्ट संदेश
- `all`: सभी संदेश

`removeAckAfterReply` उत्तर भेजे जाने के बाद बॉट का ack रिएक्शन हटा देता है
(Slack/Discord/Telegram/Google Chat केवल)। डिफ़ॉल्ट: `false`।

#### `messages.tts`

आउटबाउंड उत्तरों के लिए टेक्स्ट‑टू‑स्पीच सक्षम करें। चालू होने पर, OpenClaw ElevenLabs या OpenAI का उपयोग करके ऑडियो जेनरेट करता है
और उसे प्रतिक्रियाओं के साथ संलग्न करता है। Telegram Opus वॉइस नोट्स का उपयोग करता है; अन्य चैनल MP3 ऑडियो भेजते हैं।

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

टिप्पणियाँ:

- `messages.tts.auto` ऑटो‑TTS को नियंत्रित करता है (`off`, `always`, `inbound`, `tagged`)।
- `/tts off|always|inbound|tagged` प्रति‑सेशन ऑटो मोड सेट करता है (कॉन्फ़िग को ओवरराइड करता है)।
- `messages.tts.enabled` लेगेसी है; doctor इसे `messages.tts.auto` में माइग्रेट करता है।
- `prefsPath` लोकल ओवरराइड्स (provider/limit/summarize) को स्टोर करता है।
- `maxTextLength` TTS इनपुट के लिए एक हार्ड कैप है; सारांशों को फिट होने के लिए ट्रंकेट किया जाता है।
- `summaryModel` ऑटो‑सारांश के लिए `agents.defaults.model.primary` को ओवरराइड करता है।
  - `provider/model` या `agents.defaults.models` से किसी एलियस को स्वीकार करता है।
- `modelOverrides` मॉडल‑ड्रिवन ओवरराइड्स जैसे `[[tts:...]]` टैग्स को सक्षम करता है (डिफ़ॉल्ट रूप से चालू)।
- `/tts limit` और `/tts summary` प्रति‑यूज़र सारांश सेटिंग्स को नियंत्रित करते हैं।
- `apiKey` वैल्यूज़ `ELEVENLABS_API_KEY`/`XI_API_KEY` और `OPENAI_API_KEY` पर फ़ॉलबैक करती हैं।
- `elevenlabs.baseUrl` ElevenLabs API के बेस URL को ओवरराइड करता है।
- `elevenlabs.voiceSettings` `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost`, और `speed` (0.5..2.0) को सपोर्ट करता है।

### `talk`

Talk मोड के लिए डिफ़ॉल्ट्स (macOS/iOS/Android)। Voice IDs अनसेट होने पर `ELEVENLABS_VOICE_ID` या `SAG_VOICE_ID` पर फ़ॉलबैक करते हैं।
`apiKey` अनसेट होने पर `ELEVENLABS_API_KEY` (या गेटवे के शेल प्रोफ़ाइल) पर फ़ॉलबैक करता है।
`voiceAliases` Talk डायरेक्टिव्स को फ्रेंडली नामों का उपयोग करने देता है (उदा. `"voice":"Clawd"`)।

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

एम्बेडेड एजेंट रनटाइम (मॉडल/थिंकिंग/वर्बोज़/टाइमआउट) को नियंत्रित करता है।
`agents.defaults.models` कॉन्फ़िगर किए गए मॉडल कैटलॉग को परिभाषित करता है (और `/model` के लिए allowlist के रूप में कार्य करता है)।
`agents.defaults.model.primary` डिफ़ॉल्ट मॉडल सेट करता है; `agents.defaults.model.fallbacks` वैश्विक फ़ेलओवर हैं।
`agents.defaults.imageModel` वैकल्पिक है और **केवल तभी उपयोग होता है जब प्राइमरी मॉडल में इमेज इनपुट का समर्थन न हो**।
प्रत्येक `agents.defaults.models` एंट्री में शामिल हो सकता है:

- `alias` (वैकल्पिक मॉडल शॉर्टकट, जैसे `/opus`)।
- `params` (वैकल्पिक प्रोवाइडर-विशिष्ट API पैरामीटर जो मॉडल अनुरोध में पास किए जाते हैं)।

`params` स्ट्रीमिंग रन (एम्बेडेड एजेंट + कम्पैक्शन) पर भी लागू होता है। आज समर्थित कुंजियाँ: `temperature`, `maxTokens`। ये कॉल-टाइम विकल्पों के साथ मर्ज होते हैं; कॉलर द्वारा दिए गए मान प्राथमिकता लेते हैं। `temperature` एक उन्नत नियंत्रण है—जब तक आपको मॉडल के डिफ़ॉल्ट पता न हों और बदलाव की ज़रूरत न हो, इसे सेट न करें।

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

Z.AI GLM-4.x मॉडल स्वचालित रूप से थिंकिंग मोड सक्षम करते हैं, जब तक कि आप:

- `--thinking off` सेट न करें, या
- `agents.defaults.models["zai/<model>"].params.thinking` को स्वयं परिभाषित न करें।

OpenClaw कुछ बिल्ट-इन एलियस शॉर्टहैंड्स भी प्रदान करता है। डिफ़ॉल्ट्स केवल तब लागू होते हैं जब मॉडल पहले से `agents.defaults.models` में मौजूद हो:

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

यदि आप वही एलियस नाम (केस-इन्सेंसिटिव) स्वयं कॉन्फ़िगर करते हैं, तो आपका मान लागू होगा (डिफ़ॉल्ट कभी ओवरराइड नहीं करते)।

उदाहरण: Opus 4.6 प्राइमरी के साथ MiniMax M2.1 फ़ॉलबैक (होस्टेड MiniMax):

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

MiniMax प्रमाणीकरण: `MINIMAX_API_KEY` (env) सेट करें या `models.providers.minimax` कॉन्फ़िगर करें।

#### `agents.defaults.cliBackends` (CLI फ़ॉलबैक)

केवल टेक्स्ट वाले फ़ॉलबैक रन (कोई टूल कॉल नहीं) के लिए वैकल्पिक CLI बैकएंड। API प्रोवाइडर विफल होने पर ये बैकअप पथ के रूप में उपयोगी होते हैं। जब आप फ़ाइल पाथ स्वीकार करने वाला `imageArg` कॉन्फ़िगर करते हैं, तो इमेज पास-थ्रू समर्थित होता है।

टिप्पणियाँ:

- CLI बैकएंड **टेक्स्ट-फ़र्स्ट** होते हैं; टूल्स हमेशा अक्षम रहते हैं।
- जब `sessionArg` सेट होता है, तब सेशन्स समर्थित होते हैं; सेशन आईडी प्रति बैकएंड स्थायी रहती हैं।
- `claude-cli` के लिए, डिफ़ॉल्ट्स पहले से वायर्ड होते हैं। यदि PATH न्यूनतम हो (launchd/systemd), तो कमांड पाथ ओवरराइड करें।

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

#### `agents.defaults.contextPruning` (टूल-रिज़ल्ट प्रूनिंग)

`agents.defaults.contextPruning` LLM को अनुरोध भेजने से ठीक पहले इन-मेमोरी कॉन्टेक्स्ट से **पुराने टूल परिणामों** को हटाता है।
यह डिस्क पर सेशन इतिहास को **संशोधित नहीं करता** (`*.jsonl` पूर्ण रहता है)।

यह उन चैटी एजेंट्स के लिए टोकन उपयोग कम करने के लिए है जो समय के साथ बड़े टूल आउटपुट जमा करते हैं।

उच्च-स्तरीय:

- यूज़र/असिस्टेंट संदेशों को कभी नहीं छूता।
- आख़िरी `keepLastAssistants` असिस्टेंट संदेशों की रक्षा करता है (उस बिंदु के बाद कोई टूल परिणाम प्रून नहीं होते)।
- बूटस्ट्रैप प्रीफ़िक्स की रक्षा करता है (पहले यूज़र संदेश से पहले कुछ भी प्रून नहीं होता)।
- मोड्स:
  - `adaptive`: जब अनुमानित कॉन्टेक्स्ट अनुपात `softTrimRatio` को पार करता है, तो बड़े टूल परिणामों को सॉफ्ट-ट्रिम करता है (हेड/टेल बनाए रखता है)।
    फिर जब अनुमानित कॉन्टेक्स्ट अनुपात `hardClearRatio` को पार करता है **और** पर्याप्त प्रूनेबल टूल-रिज़ल्ट बल्क (`minPrunableToolChars`) मौजूद हो, तो सबसे पुराने योग्य टूल परिणामों को हार्ड-क्लियर करता है।
  - `aggressive`: कटऑफ से पहले योग्य टूल परिणामों को हमेशा `hardClear.placeholder` से बदल देता है (कोई अनुपात जाँच नहीं)।

Soft बनाम hard pruning (LLM को भेजे गए संदर्भ में क्या बदलता है):

- **Soft-trim**: केवल _oversized_ टूल परिणामों के लिए। शुरुआत + अंत को रखता है और बीच में `...` डालता है।
  - पहले: `toolResult("…very long output…")`
  - बाद में: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
- **Hard-clear**: पूरे टूल परिणाम को प्लेसहोल्डर से बदल देता है।
  - पहले: `toolResult("…very long output…")`
  - बाद में: `toolResult("[Old tool result content cleared]")`

नोट्स / वर्तमान सीमाएँ:

- **image blocks वाले टूल परिणाम अभी छोड़े जाते हैं** (कभी trim/clear नहीं होते)।
- अनुमानित “context ratio” **characters** पर आधारित है (लगभग), सटीक tokens पर नहीं।
- यदि सत्र में अभी तक कम से कम `keepLastAssistants` सहायक संदेश नहीं हैं, तो pruning छोड़ी जाती है।
- `aggressive` मोड में, `hardClear.enabled` को अनदेखा किया जाता है (योग्य टूल परिणाम हमेशा `hardClear.placeholder` से बदले जाते हैं)।

डिफ़ॉल्ट (adaptive):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

अक्षम करने के लिए:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

डिफ़ॉल्ट्स (जब `mode` "adaptive" या "aggressive" हो):

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (केवल adaptive)
- `hardClearRatio`: `0.5` (केवल adaptive)
- `minPrunableToolChars`: `50000` (केवल adaptive)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (केवल adaptive)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

उदाहरण (aggressive, न्यूनतम):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

उदाहरण (adaptive ट्यून किया हुआ):

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
        // वैकल्पिक: pruning को विशिष्ट टूल्स तक सीमित करें (deny की प्राथमिकता; "*" वाइल्डकार्ड समर्थित)
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

व्यवहार विवरण के लिए [/concepts/session-pruning](/concepts/session-pruning) देखें।

#### `agents.defaults.compaction` (headroom आरक्षित + memory flush)

`agents.defaults.compaction.mode` compaction summarization रणनीति चुनता है। डिफ़ॉल्ट `default` है; बहुत लंबी histories के लिए chunked summarization सक्षम करने हेतु `safeguard` सेट करें। [/concepts/compaction](/concepts/compaction) देखें।

`agents.defaults.compaction.reserveTokensFloor` Pi compaction के लिए न्यूनतम `reserveTokens`
मान लागू करता है (डिफ़ॉल्ट: `20000`)। floor को अक्षम करने के लिए इसे `0` पर सेट करें।

`agents.defaults.compaction.memoryFlush` auto-compaction से पहले एक **silent** agentic turn चलाता है,
मॉडल को डिस्क पर durable memories स्टोर करने का निर्देश देता है (उदा.
`memory/YYYY-MM-DD.md`)। यह तब ट्रिगर होता है जब सत्र का token अनुमान compaction सीमा से नीचे
एक soft threshold को पार करता है।

Legacy डिफ़ॉल्ट्स:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: `NO_REPLY` के साथ built-in डिफ़ॉल्ट्स
- नोट: जब सत्र workspace read-only हो तो memory flush छोड़ी जाती है
  (`agents.defaults.sandbox.workspaceAccess: "ro"` या `"none"`)।

उदाहरण (ट्यून किया हुआ):

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

Block streaming:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (डिफ़ॉल्ट बंद)।

- Channel overrides: `*.blockStreaming` (और प्रति-खाता वेरिएंट) block streaming को force on/off करने के लिए।
  Non-Telegram चैनलों को block replies सक्षम करने के लिए स्पष्ट `*.blockStreaming: true` की आवश्यकता होती है।

- `agents.defaults.blockStreamingBreak`: `"text_end"` या `"message_end"` (डिफ़ॉल्ट: text_end)।

- `agents.defaults.blockStreamingChunk`: streamed blocks के लिए soft chunking। डिफ़ॉल्ट्स
  800–1200 chars, पैराग्राफ ब्रेक (`\n\n`) को प्राथमिकता देता है, फिर newlines, फिर वाक्य।
  Example:

  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: भेजने से पहले स्ट्रीम किए गए ब्लॉक्स को मर्ज करें।
  डिफ़ॉल्ट `{ idleMs: 1000 }` होता है और `blockStreamingChunk` से `minChars` विरासत में लेता है, जिसमें `maxChars` को चैनल टेक्स्ट सीमा तक सीमित किया जाता है। Signal/Slack/Discord/Google Chat डिफ़ॉल्ट रूप से `minChars: 1500` पर सेट होते हैं, जब तक कि ओवरराइड न किया जाए।
  चैनल ओवरराइड्स: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (और प्रति-खाता वैरिएंट्स)।

- `agents.defaults.humanDelay`: पहले के बाद **ब्लॉक उत्तरों** के बीच यादृच्छिक विराम।
  मोड्स: `off` (डिफ़ॉल्ट), `natural` (800–2500ms), `custom` (`minMs`/`maxMs` का उपयोग करें)।
  प्रति-एजेंट ओवरराइड: `agents.list[].humanDelay`।
  Example:

  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } },
  }
  ```

  व्यवहार + चंकिंग विवरण के लिए [/concepts/streaming](/concepts/streaming) देखें।

टाइपिंग इंडिकेटर्स:

- `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`। डिफ़ॉल्ट रूप से सीधे चैट / मेंशन्स के लिए `instant` और बिना मेंशन वाले ग्रुप चैट्स के लिए `message`।
- `session.typingMode`: मोड के लिए प्रति-सेशन ओवरराइड।
- `agents.defaults.typingIntervalSeconds`: टाइपिंग सिग्नल कितनी बार रिफ़्रेश होता है (डिफ़ॉल्ट: 6s)।
- `session.typingIntervalSeconds`: रिफ़्रेश अंतराल के लिए प्रति-सेशन ओवरराइड।
  व्यवहार विवरण के लिए [/concepts/typing-indicators](/concepts/typing-indicators) देखें।

`agents.defaults.model.primary` को `provider/model` के रूप में सेट किया जाना चाहिए (उदा. `anthropic/claude-opus-4-6`)।
एलियास `agents.defaults.models.*.alias` से आते हैं (उदा. `Opus`)।
यदि आप प्रोवाइडर छोड़ देते हैं, तो OpenClaw वर्तमान में अस्थायी डिप्रिकेशन फ़ॉलबैक के रूप में `anthropic` मान लेता है।
Z.AI मॉडल `zai/<model>` (उदा. `zai/glm-4.7`) के रूप में उपलब्ध हैं और पर्यावरण में `ZAI_API_KEY` (या लेगेसी `Z_AI_API_KEY`) की आवश्यकता होती है।

`agents.defaults.heartbeat` आवधिक हार्टबीट रन कॉन्फ़िगर करता है:

- `every`: अवधि स्ट्रिंग (`ms`, `s`, `m`, `h`); डिफ़ॉल्ट इकाई मिनट। डिफ़ॉल्ट:
  `30m`। अक्षम करने के लिए `0m` सेट करें।
- `model`: हार्टबीट रन के लिए वैकल्पिक ओवरराइड मॉडल (`provider/model`)।
- `includeReasoning`: जब `true` हो, तो उपलब्ध होने पर हार्टबीट्स अलग `Reasoning:` संदेश भी भेजेंगे (आकार `/reasoning on` जैसा ही)। डिफ़ॉल्ट: `false`।
- `session`: वैकल्पिक सेशन कुंजी यह नियंत्रित करने के लिए कि हार्टबीट किस सेशन में चले। डिफ़ॉल्ट: `main`।
- `to`: वैकल्पिक प्राप्तकर्ता ओवरराइड (चैनल-विशिष्ट id, उदा. WhatsApp के लिए E.164, Telegram के लिए chat id)।
- `target`: वैकल्पिक डिलीवरी चैनल (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`)। डिफ़ॉल्ट: `last`।
- `prompt`: हार्टबीट बॉडी के लिए वैकल्पिक ओवरराइड (डिफ़ॉल्ट: `Read HEARTBEAT.md if it exists (workspace context). 34. Follow it strictly. 35. Do not infer or repeat old tasks from prior chats. 36. If nothing needs attention, reply HEARTBEAT_OK.`)। ओवरराइड्स शब्दशः भेजे जाते हैं; यदि आप फ़ाइल पढ़ना जारी रखना चाहते हैं तो `Read HEARTBEAT.md` पंक्ति शामिल करें। `ackMaxChars`: डिलीवरी से पहले `HEARTBEAT_OK` के बाद अनुमत अधिकतम अक्षर (डिफ़ॉल्ट: 300)। किसी विशिष्ट एजेंट के लिए हार्टबीट सेटिंग्स सक्षम या ओवरराइड करने के लिए `agents.list[].heartbeat` सेट करें। यदि किसी भी एजेंट एंट्री में `heartbeat` परिभाषित है, तो **केवल वही एजेंट** हार्टबीट चलाते हैं; डिफ़ॉल्ट्स उन एजेंट्स के लिए साझा बेसलाइन बन जाते हैं।
- हार्टबीट्स पूरे एजेंट टर्न्स चलाते हैं।

प्रति-एजेंट Heartbeat:

- छोटे अंतराल अधिक टोकन खर्च करते हैं; `every` के प्रति सचेत रहें, `HEARTBEAT.md` को छोटा रखें, और/या सस्ता `model` चुनें।
- `tools.exec` बैकग्राउंड exec डिफ़ॉल्ट्स कॉन्फ़िगर करता है:

`backgroundMs`: ऑटो-बैकग्राउंड से पहले का समय (ms, डिफ़ॉल्ट 10000) `timeoutSec`: इस रनटाइम के बाद ऑटो-किल (सेकंड, डिफ़ॉल्ट 1800)

`cleanupMs`: समाप्त सत्रों को मेमोरी में कितनी देर रखना है (ms, डिफ़ॉल्ट 1800000)

- `notifyOnExit`: बैकग्राउंड किए गए exec के बाहर निकलने पर सिस्टम इवेंट क्यू में डालें + हार्टबीट का अनुरोध करें (डिफ़ॉल्ट true)
- `applyPatch.enabled`: प्रयोगात्मक `apply_patch` सक्षम करें (केवल OpenAI/OpenAI Codex; डिफ़ॉल्ट false)
- `applyPatch.allowModels`: मॉडल ids की वैकल्पिक अलाउलिस्ट (उदा. `gpt-5.2` या `openai/gpt-5.2`)
  नोट: `applyPatch` केवल `tools.exec` के अंतर्गत है।
- `tools.web` वेब सर्च + फ़ेच टूल्स कॉन्फ़िगर करता है:
- `applyPatch.enabled`: enable experimental `apply_patch` (OpenAI/OpenAI Codex only; default false)
- `applyPatch.allowModels`: optional allowlist of model ids (e.g. `gpt-5.2` or `openai/gpt-5.2`)
  Note: `applyPatch` is only under `tools.exec`.

`tools.web` configures web search + fetch tools:

- `tools.web.search.enabled` (default: true when key is present)
- `tools.web.search.apiKey` (recommended: set via `openclaw configure --section web`, or use `BRAVE_API_KEY` env var)
- `tools.web.search.maxResults` (1–10, default 5)
- `tools.web.search.timeoutSeconds` (डिफ़ॉल्ट 30)
- `tools.web.search.cacheTtlMinutes` (डिफ़ॉल्ट 15)
- `tools.web.fetch.enabled` (default true)
- `tools.web.fetch.maxChars` (डिफ़ॉल्ट 50000)
- `tools.web.fetch.maxCharsCap` (default 50000; clamps maxChars from config/tool calls)
- `tools.web.fetch.timeoutSeconds` (डिफ़ॉल्ट 30)
- `tools.web.fetch.cacheTtlMinutes` (डिफ़ॉल्ट 15)
- `tools.web.fetch.userAgent` (वैकल्पिक ओवरराइड)
- `tools.web.fetch.readability` (default true; disable to use basic HTML cleanup only)
- `tools.web.fetch.firecrawl.enabled` (default true when an API key is set)
- `tools.web.fetch.firecrawl.apiKey` (optional; defaults to `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (default [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (default true)
- `tools.web.fetch.firecrawl.maxAgeMs` (वैकल्पिक)
- `tools.web.fetch.firecrawl.timeoutSeconds` (वैकल्पिक)

`tools.media` configures inbound media understanding (image/audio/video):

- `tools.media.models`: shared model list (capability-tagged; used after per-cap lists).
- `tools.media.concurrency`: max concurrent capability runs (default 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `enabled`: opt-out switch (default true when models are configured).
  - `prompt`: optional prompt override (image/video append a `maxChars` hint automatically).
  - `maxChars`: max output characters (default 500 for image/video; unset for audio).
  - `maxBytes`: max media size to send (defaults: image 10MB, audio 20MB, video 50MB).
  - `timeoutSeconds`: request timeout (defaults: image 60s, audio 60s, video 120s).
  - `language`: optional audio hint.
  - `attachments`: attachment policy (`mode`, `maxAttachments`, `prefer`).
  - `scope`: optional gating (first match wins) with `match.channel`, `match.chatType`, or `match.keyPrefix`.
  - `models`: ordered list of model entries; failures or oversize media fall back to the next entry.
- Each `models[]` entry:
  - Provider entry (`type: "provider"` or omitted):
    - `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc).
    - `model`: model id override (required for image; defaults to `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` for audio providers, and `gemini-3-flash-preview` for video).
    - `profile` / `preferredProfile`: auth profile selection.
  - CLI entry (`type: "cli"`):
    - `command`: executable to run.
    - `args`: templated args (supports `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc).
  - `capabilities`: optional list (`image`, `audio`, `video`) to gate a shared entry. Defaults when omitted: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` can be overridden per entry.

If no models are configured (or `enabled: false`), understanding is skipped; the model still receives the original attachments.

Provider auth follows the standard model auth order (auth profiles, env vars like `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, or `models.providers.*.apiKey`).

उदाहरण:

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

`agents.defaults.subagents` configures sub-agent defaults:

- `model`: default model for spawned sub-agents (string or `{ primary, fallbacks }`). If omitted, sub-agents inherit the caller’s model unless overridden per agent or per call.
- `maxConcurrent`: max concurrent sub-agent runs (default 1)
- `archiveAfterMinutes`: auto-archive sub-agent sessions after N minutes (default 60; set `0` to disable)
- Per-subagent tool policy: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (deny wins)

`tools.profile` sets a **base tool allowlist** before `tools.allow`/`tools.deny`:

- `minimal`: केवल `session_status`
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: कोई प्रतिबंध नहीं (unset के समान)

Per-agent override: `agents.list[].tools.profile`.

उदाहरण (डिफ़ॉल्ट रूप से केवल messaging, साथ में Slack + Discord टूल्स की अनुमति):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

उदाहरण (coding प्रोफ़ाइल, लेकिन exec/process को हर जगह deny):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` lets you **further restrict** tools for specific providers (or a single `provider/model`).
Per-agent override: `agents.list[].tools.byProvider`.

Order: base profile → provider profile → allow/deny policies.
Provider keys accept either `provider` (e.g. `google-antigravity`) or `provider/model`
(e.g. `openai/gpt-5.2`).

उदाहरण (वैश्विक coding प्रोफ़ाइल रखें, लेकिन Google Antigravity के लिए न्यूनतम टूल्स):

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

Example (provider/model-specific allowlist):

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

`tools.allow` / `tools.deny` configure a global tool allow/deny policy (deny wins).
Matching is case-insensitive and supports `*` wildcards (`"*"` means all tools).
This is applied even when the Docker sandbox is **off**.

Example (disable browser/canvas everywhere):

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

Tool groups (shorthands) work in **global** and **per-agent** tool policies:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: सभी अंतर्निहित OpenClaw टूल्स (प्रदाता प्लगइन्स शामिल नहीं)

`tools.elevated` controls elevated (host) exec access:

- `enabled`: allow elevated mode (default true)
- `allowFrom`: per-channel allowlists (empty = disabled)
  - `whatsapp`: E.164 numbers
  - `telegram`: chat ids or usernames
  - `discord`: user ids or usernames (falls back to `channels.discord.dm.allowFrom` if omitted)
  - `signal`: E.164 numbers
  - `imessage`: handles/chat ids
  - `webchat`: session ids or usernames

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

Per-agent override (further restrict):

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

टिप्पणियाँ:

- `tools.elevated` is the global baseline. `agents.list[].tools.elevated` can only further restrict (both must allow).
- `/elevated on|off|ask|full` stores state per session key; inline directives apply to a single message.
- Elevated `exec` runs on the host and bypasses sandboxing.
- Tool policy still applies; if `exec` is denied, elevated cannot be used.

`agents.defaults.maxConcurrent` sets the maximum number of embedded agent runs that can
execute in parallel across sessions. Each session is still serialized (one run
per session key at a time). Default: 1.

### `agents.defaults.sandbox`

Optional **Docker sandboxing** for the embedded agent. Intended for non-main
sessions so they cannot access your host system.

Details: [Sandboxing](/gateway/sandboxing)

Defaults (if enabled):

- scope: `"agent"` (one container + workspace per agent)
- Debian bookworm-slim based image
- agent workspace access: `workspaceAccess: "none"` (default)
  - `"none"`: use a per-scope sandbox workspace under `~/.openclaw/sandboxes`
- `"ro"`: keep the sandbox workspace at `/workspace`, and mount the agent workspace read-only at `/agent` (disables `write`/`edit`/`apply_patch`)
  - `"rw"`: mount the agent workspace read/write at `/workspace`
- ऑटो-प्रून: निष्क्रिय > 24 घंटे या आयु > 7 दिन
- tool policy: allow only `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (deny wins)
  - configure via `tools.sandbox.tools`, override per-agent via `agents.list[].tools.sandbox.tools`
  - tool group shorthands supported in sandbox policy: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (see [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- optional sandboxed browser (Chromium + CDP, noVNC observer)
- hardening knobs: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Warning: `scope: "shared"` means a shared container and shared workspace. No
cross-session isolation. Use `scope: "session"` for per-session isolation.

Legacy: `perSession` is still supported (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` runs **once** after the container is created (inside the container via `sh -lc`).
For package installs, ensure network egress, a writable root FS, and a root user.

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

Build the default sandbox image once with:

```bash
scripts/sandbox-setup.sh
```

Note: sandbox containers default to `network: "none"`; set `agents.defaults.sandbox.docker.network`
to `"bridge"` (or your custom network) if the agent needs outbound access.

1. नोट: इनबाउंड अटैचमेंट्स को सक्रिय वर्कस्पेस में `media/inbound/*` पर स्टेज किया जाता है। 2. `workspaceAccess: "rw"` के साथ, इसका मतलब है कि फ़ाइलें एजेंट वर्कस्पेस में लिखी जाती हैं।

3. नोट: `docker.binds` अतिरिक्त होस्ट डायरेक्टरीज़ को माउंट करता है; ग्लोबल और प्रति-एजेंट बाइंड्स को मर्ज किया जाता है।

4. वैकल्पिक ब्राउज़र इमेज को इस प्रकार बिल्ड करें:

```bash
scripts/sandbox-browser-setup.sh
```

5. जब `agents.defaults.sandbox.browser.enabled=true` होता है, तो ब्राउज़र टूल एक सैंडबॉक्स्ड Chromium इंस्टेंस (CDP) का उपयोग करता है। 6. यदि noVNC सक्षम है (headless=false होने पर डिफ़ॉल्ट), तो noVNC URL सिस्टम प्रॉम्प्ट में इंजेक्ट किया जाता है ताकि एजेंट उसका संदर्भ ले सके।
6. इसके लिए मुख्य कॉन्फ़िग में `browser.enabled` की आवश्यकता नहीं है; सैंडबॉक्स नियंत्रण URL प्रति सत्र इंजेक्ट किया जाता है।

8. `agents.defaults.sandbox.browser.allowHostControl` (डिफ़ॉल्ट: false) सैंडबॉक्स्ड सत्रों को ब्राउज़र टूल (`target: "host"`) के माध्यम से **होस्ट** ब्राउज़र कंट्रोल सर्वर को स्पष्ट रूप से लक्षित करने की अनुमति देता है। 9. यदि आप सख़्त सैंडबॉक्स आइसोलेशन चाहते हैं तो इसे बंद रखें।

10. रिमोट कंट्रोल के लिए अलाउलिस्ट्स:

- 11. `allowedControlUrls`: `target: "custom"` के लिए अनुमत सटीक कंट्रोल URLs।
- 12. `allowedControlHosts`: अनुमत होस्टनेम (केवल होस्टनेम, पोर्ट नहीं)।
- 13. `allowedControlPorts`: अनुमत पोर्ट्स (डिफ़ॉल्ट: http=80, https=443)।
  14. डिफ़ॉल्ट्स: सभी अलाउलिस्ट्स अनसेट होती हैं (कोई प्रतिबंध नहीं)। 15. `allowHostControl` का डिफ़ॉल्ट मान false है।

### 16. `models` (कस्टम प्रोवाइडर्स + बेस URLs)

17. OpenClaw **pi-coding-agent** मॉडल कैटलॉग का उपयोग करता है। 18. आप कस्टम प्रोवाइडर्स जोड़ सकते हैं (LiteLLM, लोकल OpenAI-संगत सर्वर, Anthropic प्रॉक्सी, आदि)। 19. इसके लिए `~/.openclaw/agents/<agentId>/agent/models.json` लिखकर या OpenClaw कॉन्फ़िग में `models.providers` के अंतर्गत वही स्कीमा परिभाषित करके।
18. प्रोवाइडर-वार अवलोकन + उदाहरण: [/concepts/model-providers](/concepts/model-providers)।

21. जब `models.providers` मौजूद होता है, तो OpenClaw स्टार्टअप पर `~/.openclaw/agents/<agentId>/agent/` में एक `models.json` लिखता/मर्ज करता है:

- 22. डिफ़ॉल्ट व्यवहार: **merge** (मौजूदा प्रोवाइडर्स को रखता है, नाम पर ओवरराइड करता है)।
- 23. फ़ाइल सामग्री को ओवरराइट करने के लिए `models.mode: "replace"` सेट करें।

24. मॉडल को `agents.defaults.model.primary` (provider/model) के माध्यम से चुनें।

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

### 26. OpenCode Zen (मल्टी-मॉडल प्रॉक्सी)

27. OpenCode Zen प्रति-मॉडल एंडपॉइंट्स के साथ एक मल्टी-मॉडल गेटवे है। 28. OpenClaw pi-ai से बिल्ट-इन `opencode` प्रोवाइडर का उपयोग करता है; [https://opencode.ai/auth](https://opencode.ai/auth) से `OPENCODE_API_KEY` (या `OPENCODE_ZEN_API_KEY`) सेट करें।

टिप्पणियाँ:

- 29. मॉडल रेफ़रेंसेज़ `opencode/<modelId>` का उपयोग करते हैं (उदाहरण: `opencode/claude-opus-4-6`)।
- 30. यदि आप `agents.defaults.models` के माध्यम से अलाउलिस्ट सक्षम करते हैं, तो जिन भी मॉडलों का आप उपयोग करने की योजना बनाते हैं उन्हें जोड़ें।
- 31. शॉर्टकट: `openclaw onboard --auth-choice opencode-zen`।

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

### 33. Z.AI (GLM-4.7) — प्रोवाइडर एलियास समर्थन

34. Z.AI मॉडल बिल्ट-इन `zai` प्रोवाइडर के माध्यम से उपलब्ध हैं। 35. अपने वातावरण में `ZAI_API_KEY` सेट करें और मॉडल को provider/model द्वारा संदर्भित करें।

36. शॉर्टकट: `openclaw onboard --auth-choice zai-api-key`।

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

टिप्पणियाँ:

- 38. `z.ai/*` और `z-ai/*` स्वीकृत एलियास हैं और `zai/*` में नॉर्मलाइज़ होते हैं।
- 39. यदि `ZAI_API_KEY` अनुपस्थित है, तो `zai/*` के लिए अनुरोध रनटाइम पर ऑथ एरर के साथ विफल हो जाएंगे।
- उदाहरण त्रुटि: `No API key found for provider "zai".`
- 41. Z.AI का सामान्य API एंडपॉइंट `https://api.z.ai/api/paas/v4` है। 42. GLM कोडिंग अनुरोध समर्पित Coding एंडपॉइंट `https://api.z.ai/api/coding/paas/v4` का उपयोग करते हैं।
  42. बिल्ट-इन `zai` प्रोवाइडर Coding एंडपॉइंट का उपयोग करता है। 44. यदि आपको सामान्य एंडपॉइंट की आवश्यकता है, तो बेस URL ओवरराइड के साथ `models.providers` में एक कस्टम प्रोवाइडर परिभाषित करें (ऊपर कस्टम प्रोवाइडर्स अनुभाग देखें)।
- 45. डॉक्स/कॉन्फ़िग्स में एक नकली प्लेसहोल्डर का उपयोग करें; कभी भी वास्तविक API keys कमिट न करें।

### Moonshot AI (Kimi)

46. Moonshot का OpenAI-संगत एंडपॉइंट उपयोग करें:

```json5
{
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

टिप्पणियाँ:

- 48. वातावरण में `MOONSHOT_API_KEY` सेट करें या `openclaw onboard --auth-choice moonshot-api-key` का उपयोग करें।
- 49. मॉडल रेफ़: `moonshot/kimi-k2.5`।
- 50. चीन एंडपॉइंट के लिए, इनमें से कोई एक:
  - 1. `openclaw onboard --auth-choice moonshot-api-key-cn` चलाएँ (विज़ार्ड `https://api.moonshot.cn/v1` सेट करेगा), या
  - 2. `models.providers.moonshot` में मैन्युअल रूप से `baseUrl: "https://api.moonshot.cn/v1"` सेट करें।

### Kimi Coding

3. Moonshot AI के Kimi Coding एंडपॉइंट का उपयोग करें (Anthropic-संगत, बिल्ट-इन प्रोवाइडर):

```json5
4. {
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

टिप्पणियाँ:

- 5. एनवायरनमेंट में `KIMI_API_KEY` सेट करें या `openclaw onboard --auth-choice kimi-code-api-key` का उपयोग करें।
- 6. मॉडल रेफ: `kimi-coding/k2p5`।

### 7. Synthetic (Anthropic-संगत)

8. Synthetic के Anthropic-संगत एंडपॉइंट का उपयोग करें:

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

टिप्पणियाँ:

- 9. `SYNTHETIC_API_KEY` सेट करें या `openclaw onboard --auth-choice synthetic-api-key` का उपयोग करें।
- 10. मॉडल रेफ: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`।
- 11. Base URL में `/v1` शामिल नहीं होना चाहिए क्योंकि Anthropic क्लाइंट इसे अपने आप जोड़ता है।

### 12. लोकल मॉडल (LM Studio) — अनुशंसित सेटअप

13. वर्तमान लोकल गाइडेंस के लिए [/gateway/local-models](/gateway/local-models) देखें। 14. TL;DR: शक्तिशाली हार्डवेयर पर LM Studio Responses API के माध्यम से MiniMax M2.1 चलाएँ; फ़ॉलबैक के लिए होस्टेड मॉडल्स को मर्ज करके रखें।

### MiniMax M2.1

15. LM Studio के बिना सीधे MiniMax M2.1 का उपयोग करें:

```json5
16. {
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

टिप्पणियाँ:

- 17. `MINIMAX_API_KEY` एनवायरनमेंट वेरिएबल सेट करें या `openclaw onboard --auth-choice minimax-api` का उपयोग करें।
- 18. उपलब्ध मॉडल: `MiniMax-M2.1` (डिफ़ॉल्ट)।
- 19. यदि आपको सटीक लागत ट्रैकिंग चाहिए तो `models.json` में प्राइसिंग अपडेट करें।

### 20. Cerebras (GLM 4.6 / 4.7)

21. Cerebras को उनके OpenAI-संगत एंडपॉइंट के माध्यम से उपयोग करें:

```json5
22. {
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

टिप्पणियाँ:

- 23. Cerebras के लिए `cerebras/zai-glm-4.7` उपयोग करें; Z.AI डायरेक्ट के लिए `zai/glm-4.7` उपयोग करें।
- 24. एनवायरनमेंट या कॉन्फ़िग में `CEREBRAS_API_KEY` सेट करें।

टिप्पणियाँ:

- 25. समर्थित APIs: `openai-completions`, `openai-responses`, `anthropic-messages`,
      `google-generative-ai`
- 26. कस्टम ऑथ आवश्यकताओं के लिए `authHeader: true` + `headers` का उपयोग करें।
- 27. यदि आप `models.json` को कहीं और स्टोर करना चाहते हैं (डिफ़ॉल्ट: `~/.openclaw/agents/main/agent`), तो `OPENCLAW_AGENT_DIR` (या `PI_CODING_AGENT_DIR`) के साथ एजेंट कॉन्फ़िग रूट को ओवरराइड करें।

### `session`

28. सेशन स्कोपिंग, रीसेट पॉलिसी, रीसेट ट्रिगर्स, और सेशन स्टोर कहाँ लिखा जाता है, को नियंत्रित करता है।

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

फ़ील्ड्स:

- 30. `mainKey`: डायरेक्ट-चैट बकेट कुंजी (डिफ़ॉल्ट: `"main"`)। 31. तब उपयोगी जब आप `agentId` बदले बिना प्राथमिक DM थ्रेड का “नाम बदलना” चाहते हों।
  - 32. सैंडबॉक्स नोट: `agents.defaults.sandbox.mode: "non-main"` मुख्य सेशन का पता लगाने के लिए इस कुंजी का उपयोग करता है। 33. कोई भी सेशन कुंजी जो `mainKey` से मेल नहीं खाती (ग्रुप/चैनल) सैंडबॉक्स की जाती है।
- 34. `dmScope`: DM सेशन्स को कैसे समूहित किया जाता है (डिफ़ॉल्ट: `"main"`)।
  - 35. `main`: निरंतरता के लिए सभी DMs मुख्य सेशन साझा करते हैं।
  - 36. `per-peer`: चैनलों के पार प्रेषक आईडी के अनुसार DMs को अलग करें।
  - 37. `per-channel-peer`: प्रति चैनल + प्रेषक DMs को अलग करें (मल्टी-यूज़र इनबॉक्स के लिए अनुशंसित)।
  - 38. `per-account-channel-peer`: प्रति अकाउंट + चैनल + प्रेषक DMs को अलग करें (मल्टी-अकाउंट इनबॉक्स के लिए अनुशंसित)।
  - 39. सुरक्षित DM मोड (अनुशंसित): जब कई लोग बॉट को DM कर सकते हों (शेयर्ड इनबॉक्स, मल्टी-पर्सन अलाउलिस्ट्स, या `dmPolicy: "open"`), तो `session.dmScope: "per-channel-peer"` सेट करें।
- 40. `identityLinks`: कैनॉनिकल आईडीज़ को प्रोवाइडर-प्रिफ़िक्स्ड पीयर्स से मैप करें ताकि `per-peer`, `per-channel-peer`, या `per-account-channel-peer` का उपयोग करते समय वही व्यक्ति चैनलों के पार एक ही DM सेशन साझा करे।
  - 41. उदाहरण: `alice: ["telegram:123456789", "discord:987654321012345678"]`।
- 42. `reset`: प्राथमिक रीसेट पॉलिसी। 43. डिफ़ॉल्ट रूप से गेटवे होस्ट के लोकल समयानुसार सुबह 4:00 बजे दैनिक रीसेट होता है।
  - 44. `mode`: `daily` या `idle` (डिफ़ॉल्ट: `reset` मौजूद होने पर `daily`)।
  - 45. `atHour`: दैनिक रीसेट सीमा के लिए लोकल घंटा (0-23)।
  - 46. `idleMinutes`: मिनटों में स्लाइडिंग आइडल विंडो। 47. जब दैनिक + आइडल दोनों कॉन्फ़िगर हों, तो जो पहले समाप्त होता है वही लागू होता है।
- `resetByType`: per-session overrides for `direct`, `group`, and `thread`. Legacy `dm` key is accepted as an alias for `direct`.
  - 49. यदि आप केवल लेगेसी `session.idleMinutes` सेट करते हैं और कोई `reset`/`resetByType` नहीं है, तो बैकवर्ड कम्पैटिबिलिटी के लिए OpenClaw आइडल-ओनली मोड में रहता है।
- 50. `heartbeatIdleMinutes`: हार्टबीट चेक्स के लिए वैकल्पिक आइडल ओवरराइड (सक्रिय होने पर दैनिक रीसेट फिर भी लागू होता है)।
- `agentToAgent.maxPingPongTurns`: max reply-back turns between requester/target (0–5, default 5).
- `sendPolicy.default`: `allow` or `deny` fallback when no rule matches.
- `sendPolicy.rules[]`: match by `channel`, `chatType` (`direct|group|room`), or `keyPrefix` (e.g. `cron:`). First deny wins; otherwise allow.

### `skills` (skills config)

Controls bundled allowlist, install preferences, extra skill folders, and per-skill
overrides. Applies to **bundled** skills and `~/.openclaw/skills` (workspace skills
still win on name conflicts).

फ़ील्ड्स:

- `allowBundled`: optional allowlist for **bundled** skills only. If set, only those
  bundled skills are eligible (managed/workspace skills unaffected).
- `load.extraDirs`: स्कैन करने के लिए अतिरिक्त skill निर्देशिकाएँ (सबसे कम प्राथमिकता)।
- `install.preferBrew`: उपलब्ध होने पर brew installers को प्राथमिकता दें (डिफ़ॉल्ट: true)।
- `install.nodeManager`: node installer preference (`npm` | `pnpm` | `yarn`, default: npm).
- `entries.<skillKey>`: per-skill config overrides.

प्रति-skill फ़ील्ड्स:

- `enabled`: किसी skill को अक्षम करने के लिए `false` सेट करें, भले ही वह bundled/installed हो।
- `env`: एजेंट रन के लिए इंजेक्ट किए गए environment variables (केवल तब, जब पहले से सेट न हों)।
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

Controls plugin discovery, allow/deny, and per-plugin config. Plugins are loaded
from `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, plus any
`plugins.load.paths` entries. **Config changes require a gateway restart.**
See [/plugin](/tools/plugin) for full usage.

फ़ील्ड्स:

- `enabled`: master toggle for plugin loading (default: true).
- `allow`: optional allowlist of plugin ids; when set, only listed plugins load.
- `deny`: optional denylist of plugin ids (deny wins).
- `load.paths`: extra plugin files or directories to load (absolute or `~`).
- `entries.<pluginId>`: per-plugin overrides.
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
Profiles can point at a **remote** Chromium-based browser via `profiles.<name>.cdpUrl`. Remote
profiles are attach-only (start/stop/reset are disabled).

`browser.cdpUrl` remains for legacy single-profile configs and as the base
scheme/host for profiles that only set `cdpPort`.

डिफ़ॉल्ट्स:

- enabled: `true`
- evaluateEnabled: `true` (set `false` to disable `act:evaluate` and `wait --fn`)
- control service: loopback only (port derived from `gateway.port`, default `18791`)
- CDP URL: `http://127.0.0.1:18792` (control service + 1, legacy single-profile)
- profile color: `#FF4500` (lobster-orange)
- Note: the control server is started by the running gateway (OpenClaw.app menubar, or `openclaw gateway`).
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

डिफ़ॉल्ट्स:

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
- `gateway.controlUi.root` sets the filesystem root for Control UI assets (default: `dist/control-ui`).
- `gateway.controlUi.allowInsecureAuth` allows token-only auth for the Control UI when
  device identity is omitted (typically over HTTP). Default: `false`. Prefer HTTPS
  (Tailscale Serve) or `127.0.0.1`.
- `gateway.controlUi.dangerouslyDisableDeviceAuth` disables device identity checks for the
  Control UI (token/password only). Default: `false`. Break-glass only.

संबंधित दस्तावेज़:

- [Control UI](/web/control-ui)
- [Web overview](/web)
- [Tailscale](/gateway/tailscale)
- [Remote access](/gateway/remote)

Trusted proxies:

- `gateway.trustedProxies`: list of reverse proxy IPs that terminate TLS in front of the Gateway.
- When a connection comes from one of these IPs, OpenClaw uses `x-forwarded-for` (or `x-real-ip`) to determine the client IP for local pairing checks and HTTP auth/local checks.
- Only list proxies you fully control, and ensure they **overwrite** incoming `x-forwarded-for`.

Notes:

- `openclaw gateway` refuses to start unless `gateway.mode` is set to `local` (or you pass the override flag).
- `gateway.port` controls the single multiplexed port used for WebSocket + HTTP (control UI, hooks, A2UI).
- OpenAI Chat Completions endpoint: **disabled by default**; enable with `gateway.http.endpoints.chatCompletions.enabled: true`.
- Precedence: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > default `18789`.
- Gateway auth is required by default (token/password or Tailscale Serve identity). Non-loopback binds require a shared token/password.
- The onboarding wizard generates a gateway token by default (even on loopback).
- `gateway.remote.token` is **only** for remote CLI calls; it does not enable local gateway auth. `gateway.token` is ignored.

Auth and Tailscale:

- `gateway.auth.mode` sets the handshake requirements (`token` or `password`). When unset, token auth is assumed.
- `gateway.auth.token` stores the shared token for token auth (used by the CLI on the same machine).
- When `gateway.auth.mode` is set, only that method is accepted (plus optional Tailscale headers).
- `gateway.auth.password` can be set here, or via `OPENCLAW_GATEWAY_PASSWORD` (recommended).
- `gateway.auth.allowTailscale` allows Tailscale Serve identity headers
  (`tailscale-user-login`) to satisfy auth when the request arrives on loopback
  with `x-forwarded-for`, `x-forwarded-proto`, and `x-forwarded-host`. OpenClaw
  verifies the identity by resolving the `x-forwarded-for` address via
  `tailscale whois` before accepting it. When `true`, Serve requests do not need
  a token/password; set `false` to require explicit credentials. Defaults to
  `true` when `tailscale.mode = "serve"` and auth mode is not `password`.
- `gateway.tailscale.mode: "serve"` uses Tailscale Serve (tailnet only, loopback bind).
- `gateway.tailscale.mode: "funnel"` exposes the dashboard publicly; requires auth.
- `gateway.tailscale.resetOnExit` resets Serve/Funnel config on shutdown.

Remote client defaults (CLI):

- `gateway.remote.url` sets the default Gateway WebSocket URL for CLI calls when `gateway.mode = "remote"`.
- `gateway.remote.transport` selects the macOS remote transport (`ssh` default, `direct` for ws/wss). When `direct`, `gateway.remote.url` must be `ws://` or `wss://`. `ws://host` defaults to port `18789`.
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

Modes:

- `hybrid` (default): hot-apply safe changes; restart the Gateway for critical changes.
- `hot`: only apply hot-safe changes; log when a restart is required.
- `restart`: restart the Gateway on any config change.
- `off`: disable hot reload.

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

#### Hot reload matrix (files + impact)

Files watched:

- `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`)

Hot-applied (no full gateway restart):

- `hooks` (webhook auth/path/mappings) + `hooks.gmail` (Gmail watcher restarted)
- `browser` (browser control server restart)
- `cron` (cron service restart + concurrency update)
- `agents.defaults.heartbeat` (heartbeat runner restart)
- `web` (WhatsApp web channel restart)
- `telegram`, `discord`, `signal`, `imessage` (channel restarts)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (dynamic reads)

Requires full Gateway restart:

- `gateway` (port/bind/auth/control UI/tailscale)
- `bridge` (legacy)
- `discovery`
- `canvasHost`
- `प्लगइन्स`
- Any unknown/unsupported config path (defaults to restart for safety)

### Multi-instance isolation

To run multiple gateways on one host (for redundancy or a rescue bot), isolate per-instance state + config and use unique ports:

- `OPENCLAW_CONFIG_PATH` (per-instance config)
- `OPENCLAW_STATE_DIR` (sessions/creds)
- `agents.defaults.workspace` (memories)
- `gateway.port` (unique per instance)

Convenience flags (CLI):

- `openclaw --dev …` → uses `~/.openclaw-dev` + shifts ports from base `19001`
- `openclaw --profile <name> …` → uses `~/.openclaw-<name>` (port via config/env/flags)

See [Gateway runbook](/gateway) for the derived port mapping (gateway/browser/canvas).
See [Multiple gateways](/gateway/multiple-gateways) for browser/CDP port isolation details.

उदाहरण:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

### `hooks` (Gateway webhooks)

Enable a simple HTTP webhook endpoint on the Gateway HTTP server.

डिफ़ॉल्ट्स:

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

Requests must include the hook token:

- `Authorization: Bearer <token>` **or**
- `x-openclaw-token: <token>`

Endpoints:

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
- `POST /hooks/<name>` → resolved via `hooks.mappings`

`/hooks/agent` always posts a summary into the main session (and can optionally trigger an immediate heartbeat via `wakeMode: "now"`).

Mapping notes:

- `match.path` matches the sub-path after `/hooks` (e.g. `/hooks/gmail` → `gmail`).
- `match.source` matches a payload field (e.g. `{ source: "gmail" }`) so you can use a generic `/hooks/ingest` path.
- Templates like `{{messages[0].subject}}` read from the payload.
- `transform` can point to a JS/TS module that returns a hook action.
- `deliver: true` sends the final reply to a channel; `channel` defaults to `last` (falls back to WhatsApp).
- If there is no prior delivery route, set `channel` + `to` explicitly (required for Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams).
- `model` overrides the LLM for this hook run (`provider/model` or alias; must be allowed if `agents.defaults.models` is set).

Gmail helper config (used by `openclaw webhooks gmail setup` / `run`):

```json5
{
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

      // Optional: use a cheaper model for Gmail hook processing
      // Falls back to agents.defaults.model.fallbacks, then primary, on auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    },
  },
}
```

Model override for Gmail hooks:

- `hooks.gmail.model` specifies a model to use for Gmail hook processing (defaults to session primary).
- Accepts `provider/model` refs or aliases from `agents.defaults.models`.
- Falls back to `agents.defaults.model.fallbacks`, then `agents.defaults.model.primary`, on auth/rate-limit/timeouts.
- If `agents.defaults.models` is set, include the hooks model in the allowlist.
- At startup, warns if the configured model is not in the model catalog or allowlist.
- `hooks.gmail.thinking` sets the default thinking level for Gmail hooks and is overridden by per-hook `thinking`.

Gateway auto-start:

- If `hooks.enabled=true` and `hooks.gmail.account` is set, the Gateway starts
  `gog gmail watch serve` on boot and auto-renews the watch.
- Set `OPENCLAW_SKIP_GMAIL_WATCHER=1` to disable the auto-start (for manual runs).
- Avoid running a separate `gog gmail watch serve` alongside the Gateway; it will
  fail with `listen tcp 127.0.0.1:8788: bind: address already in use`.

Note: when `tailscale.mode` is on, OpenClaw defaults `serve.path` to `/` so
Tailscale can proxy `/gmail-pubsub` correctly (it strips the set-path prefix).
If you need the backend to receive the prefixed path, set
`hooks.gmail.tailscale.target` to a full URL (and align `serve.path`).

### `canvasHost` (LAN/tailnet Canvas file server + live reload)

The Gateway serves a directory of HTML/CSS/JS over HTTP so iOS/Android nodes can simply `canvas.navigate` to it.

Default root: `~/.openclaw/workspace/canvas`  
Default port: `18793` (chosen to avoid the openclaw browser CDP port `18792`)  
The server listens on the **gateway bind host** (LAN or Tailnet) so nodes can reach it.

The server:

- serves files under `canvasHost.root`
- injects a tiny live-reload client into served HTML
- watches the directory and broadcasts reloads over a WebSocket endpoint at `/__openclaw__/ws`
- auto-creates a starter `index.html` when the directory is empty (so you see something immediately)
- also serves A2UI at `/__openclaw__/a2ui/` and is advertised to nodes as `canvasHostUrl`
  (always used by nodes for Canvas/A2UI)

Disable live reload (and file watching) if the directory is large or you hit `EMFILE`:

- config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

Changes to `canvasHost.*` require a gateway restart (config reload will restart).

इसे अक्षम करने के लिए:

- config: `canvasHost: { enabled: false }`
- env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `bridge` (legacy TCP bridge, removed)

Current builds no longer include the TCP bridge listener; `bridge.*` config keys are ignored.
Nodes connect over the Gateway WebSocket. This section is kept for historical reference.

Legacy behavior:

- The Gateway could expose a simple TCP bridge for nodes (iOS/Android), typically on port `18790`.

डिफ़ॉल्ट्स:

- enabled: `true`
- port: `18790`
- bind: `lan` (binds to `0.0.0.0`)

Bind modes:

- `lan`: `0.0.0.0` (reachable on any interface, including LAN/Wi‑Fi and Tailscale)
- `tailnet`: केवल मशीन के Tailscale IP से बाइंड करें (Vienna ⇄ London के लिए अनुशंसित)
- `loopback`: `127.0.0.1` (केवल लोकल)
- `auto`: यदि उपलब्ध हो तो tailnet IP को प्राथमिकता दें, अन्यथा `lan`

TLS:

- `bridge.tls.enabled`: ब्रिज कनेक्शनों के लिए TLS सक्षम करें (सक्षम होने पर केवल TLS)।
- `bridge.tls.autoGenerate`: जब कोई cert/key मौजूद न हो तो self-signed cert जनरेट करें (डिफ़ॉल्ट: true)।
- `bridge.tls.certPath` / `bridge.tls.keyPath`: ब्रिज सर्टिफ़िकेट + प्राइवेट की के लिए PEM पाथ।
- `bridge.tls.caPath`: वैकल्पिक PEM CA बंडल (कस्टम रूट्स या भविष्य के mTLS के लिए)।

जब TLS सक्षम होता है, Gateway डिस्कवरी TXT रिकॉर्ड्स में `bridgeTls=1` और `bridgeTlsSha256` का विज्ञापन करता है ताकि नोड्स सर्टिफ़िकेट को पिन कर सकें। मैनुअल कनेक्शनों में trust-on-first-use का उपयोग होता है यदि अभी तक कोई फ़िंगरप्रिंट स्टोर नहीं है।
ऑटो-जनरेटेड certs के लिए PATH में `openssl` आवश्यक है; यदि जनरेशन विफल होती है, तो ब्रिज शुरू नहीं होगा।

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

### `discovery.mdns` (Bonjour / mDNS ब्रॉडकास्ट मोड)

LAN mDNS डिस्कवरी ब्रॉडकास्ट्स (`_openclaw-gw._tcp`) को नियंत्रित करता है।

- `minimal` (डिफ़ॉल्ट): TXT रिकॉर्ड्स से `cliPath` + `sshPort` को छोड़ देता है
- `full`: TXT रिकॉर्ड्स में `cliPath` + `sshPort` शामिल करता है
- `off`: mDNS ब्रॉडकास्ट्स को पूरी तरह अक्षम करें
- Hostname: डिफ़ॉल्ट रूप से `openclaw` ( `openclaw.local` का विज्ञापन करता है)। `OPENCLAW_MDNS_HOSTNAME` से ओवरराइड करें।

```json5
{
  discovery: { mdns: { mode: "minimal" } },
}
```

### `discovery.wideArea` (Wide-Area Bonjour / unicast DNS‑SD)

सक्षम होने पर, Gateway कॉन्फ़िगर किए गए डिस्कवरी डोमेन (उदाहरण: `openclaw.internal.`) का उपयोग करते हुए `_openclaw-gw._tcp` के लिए `~/.openclaw/dns/` के अंतर्गत एक unicast DNS-SD ज़ोन लिखता है।

iOS/Android को नेटवर्क्स के पार (Vienna ⇄ London) डिस्कवर कराने के लिए, इसे इसके साथ पेयर करें:

- गेटवे होस्ट पर एक DNS सर्वर जो आपके चुने हुए डोमेन को सर्व करे (CoreDNS अनुशंसित है)
- Tailscale **split DNS** ताकि क्लाइंट्स उस डोमेन को गेटवे DNS सर्वर के माध्यम से रेज़ॉल्व करें

वन-टाइम सेटअप हेल्पर (गेटवे होस्ट):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## मीडिया मॉडल टेम्पलेट वेरिएबल्स

टेम्पलेट प्लेसहोल्डर्स `tools.media.*.models[].args` और `tools.media.models[].args` (और भविष्य के किसी भी टेम्पलेटेड आर्ग्युमेंट फ़ील्ड) में एक्सपैंड होते हैं।

\| Variable           | Description                                                                     |
\| ------------------ | ------------------------------------------------------------------------------- | -------- | ------- | ---------- | ----- | ------ | -------- | ------- | ------- | --- |
\| `{{Body}}`         | पूर्ण इनबाउंड संदेश बॉडी                                                       |
\| `{{RawBody}}`      | रॉ इनबाउंड संदेश बॉडी (कोई हिस्ट्री/सेंडर रैपर नहीं; कमांड पार्सिंग के लिए सर्वोत्तम) |
\| `{{BodyStripped}}` | बॉडी जिसमें ग्रुप मेंशन हटाए गए हों (एजेंट्स के लिए सर्वोत्तम डिफ़ॉल्ट)                     |
\| `{{From}}`         | सेंडर पहचानकर्ता (WhatsApp के लिए E.164; चैनल के अनुसार भिन्न हो सकता है)                  |
\| `{{To}}`           | डेस्टिनेशन पहचानकर्ता                                                          |
\| `{{MessageSid}}`   | चैनल संदेश आईडी (जब उपलब्ध हो)                                             |
\| `{{SessionId}}`    | वर्तमान सेशन UUID                                                            |
\| `{{IsNewSession}}` | नया सेशन बनाए जाने पर `"true"`                                         |
\| `{{MediaUrl}}`     | इनबाउंड मीडिया pseudo-URL (यदि मौजूद हो)                                           |
\| `{{MediaPath}}`    | लोकल मीडिया पाथ (यदि डाउनलोड किया गया हो)                                                |
\| `{{MediaType}}`    | मीडिया प्रकार (image/audio/document/…)                                             |
\| `{{Transcript}}`   | ऑडियो ट्रांसक्रिप्ट (जब सक्षम हो)                                                 |
\| `{{Prompt}}`       | CLI एंट्रीज़ के लिए रेज़ॉल्व किया गया मीडिया प्रॉम्प्ट                                           |
\| `{{MaxChars}}`     | CLI एंट्रीज़ के लिए रेज़ॉल्व किए गए अधिकतम आउटपुट कैरेक्टर्स                                       |
\| `{{ChatType}}`     | `"direct"` या `"group"`                                                         |
\| `{{GroupSubject}}` | ग्रुप विषय (best effort)                                                     |
\| `{{GroupMembers}}` | ग्रुप मेंबर्स प्रीव्यू (best effort)                                             |
\| `{{SenderName}}`   | सेंडर डिस्प्ले नाम (best effort)                                               |
\| `{{SenderE164}}`   | सेंडर फ़ोन नंबर (best effort)                                               |
\| `{{Provider}}`     | प्रोवाइडर संकेत (whatsapp                                                         | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## Cron (Gateway शेड्यूलर)

Cron गेटवे-स्वामित्व वाला शेड्यूलर है जो वेकअप्स और शेड्यूल्ड जॉब्स के लिए उपयोग होता है। फ़ीचर ओवरव्यू और CLI उदाहरणों के लिए [Cron jobs](/automation/cron-jobs) देखें।

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

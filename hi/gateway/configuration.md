---
summary: "उदाहरणों सहित ~/.openclaw/openclaw.json के लिए सभी विन्यास विकल्प"
read_when:
  - विन्यास फ़ील्ड जोड़ते या संशोधित करते समय
  - सामान्य कॉन्फ़िगरेशन पैटर्न ढूंढ रहे हैं
  - विशिष्ट कॉन्फ़िग सेक्शन पर नेविगेट करना
title: "विन्यास"
---

# विन्यास 🔧

OpenClaw `~/.openclaw/openclaw.json` से एक वैकल्पिक **JSON5** विन्यास पढ़ता है (टिप्पणियाँ + ट्रेलिंग कॉमा अनुमत)।

यदि फ़ाइल अनुपस्थित है, तो OpenClaw सुरक्षित डिफ़ॉल्ट का उपयोग करता है। कॉन्फ़िग जोड़ने के सामान्य कारण:

- यह सीमित करना कि बॉट को कौन ट्रिगर कर सकता है (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, आदि)
- समूह allowlist + उल्लेख (mention) व्यवहार को नियंत्रित करना (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- संदेश उपसर्गों (prefixes) को अनुकूलित करना (`messages`)

उपलब्ध हर फ़ील्ड के लिए [full reference](/gateway/configuration-reference) देखें।

<Tip>
**कॉन्फ़िगरेशन में नए हैं?** इंटरैक्टिव सेटअप के लिए `openclaw onboard` से शुरू करें, या पूरी कॉपी-पेस्ट कॉन्फ़िग के लिए [Configuration Examples](/gateway/configuration-examples) गाइड देखें।
</Tip>

## न्यूनतम कॉन्फ़िग

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## कॉन्फ़िग संपादित करना

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # full setup wizard
    openclaw configure     # config wizard
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
    [http://127.0.0.1:18789](http://127.0.0.1:18789) खोलें और **Config** टैब का उपयोग करें।
    Control UI कॉन्फ़िग स्कीमा से एक फ़ॉर्म रेंडर करता है, और आवश्यकता पड़ने पर **Raw JSON** एडिटर उपलब्ध है।
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` को सीधे संपादित करें। Gateway फ़ाइल को मॉनिटर करता है और बदलावों को स्वतः लागू करता है (देखें [hot reload](#config-hot-reload))।
  
</Tab>
</Tabs>

## स्कीमा + UI संकेत

<Warning>
10. OpenClaw केवल वही कॉन्फ़िगरेशन स्वीकार करता है जो स्कीमा से पूरी तरह मेल खाते हों। अज्ञात कुंजियाँ, गलत प्रकार, या अमान्य मान होने पर Gateway **स्टार्ट होने से इंकार कर देगा**। रूट-लेवल पर एकमात्र अपवाद `$schema` (string) है, ताकि एडिटर JSON Schema मेटाडेटा जोड़ सकें।
</Warning>

चैनल प्लगइन्स और एक्सटेंशन्स अपने विन्यास के लिए स्कीमा + UI संकेत पंजीकृत कर सकते हैं, ताकि
चैनल सेटिंग्स ऐप्स के बीच हार्ड‑कोडेड फ़ॉर्म के बिना स्कीमा‑आधारित बनी रहें।

- Gateway बूट नहीं होता
- केवल डायग्नोस्टिक कमांड काम करते हैं (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- सटीक समस्याएँ देखने के लिए `openclaw doctor` चलाएँ
- मरम्मत लागू करने के लिए `openclaw doctor --fix` (या `--yes`) चलाएँ

## लागू करें + पुनःआरंभ (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    प्रत्येक चैनल का अपना कॉन्फ़िग सेक्शन `channels.` के अंतर्गत होता है।<provider>`.` सेटअप चरणों के लिए समर्पित चैनल पेज देखें:

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
    
    सभी चैनल समान DM नीति पैटर्न साझा करते हैं:
    
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
    ````

  
</Accordion>

  <Accordion title="Choose and configure models">
    प्राथमिक मॉडल और वैकल्पिक फॉलबैक सेट करें:

    ````
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
    
    - `agents.defaults.models` मॉडल कैटलॉग को परिभाषित करता है और `/model` के लिए allowlist का कार्य करता है।
    - मॉडल रेफ़रेंस `provider/model` प्रारूप का उपयोग करते हैं (उदा. `anthropic/claude-opus-4-6`)।
    - चैट में मॉडल बदलने के लिए [Models CLI](/concepts/models) और ऑथ रोटेशन व फॉलबैक व्यवहार के लिए [Model Failover](/concepts/model-failover) देखें।
    - कस्टम/स्व-होस्टेड प्रदाताओं के लिए, रेफ़रेंस में [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) देखें।
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    DM एक्सेस प्रति चैनल `dmPolicy` के माध्यम से नियंत्रित होता है:

    ```
    - `"pairing"` (डिफ़ॉल्ट): अज्ञात प्रेषकों को अनुमोदन के लिए एक बार का pairing कोड मिलता है
    - `"allowlist"`: केवल `allowFrom` (या paired allow store) में शामिल प्रेषक
    - `"open"`: सभी इनबाउंड DM की अनुमति (इसके लिए `allowFrom: ["*"]` आवश्यक है)
    - `"disabled"`: सभी DM को अनदेखा करें
    
    ग्रुप के लिए, `groupPolicy` + `groupAllowFrom` या चैनल-विशिष्ट allowlists का उपयोग करें।
    
    प्रति-चैनल विवरण के लिए [full reference](/gateway/configuration-reference#dm-and-group-access) देखें।
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    ग्रुप संदेशों के लिए डिफ़ॉल्ट रूप से **मेंशन आवश्यक** होता है। प्रति एजेंट पैटर्न कॉन्फ़िगर करें:

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
    
    - **Metadata mentions**: मूल @-mentions (WhatsApp tap-to-mention, Telegram @bot, आदि)
    - **Text patterns**: `mentionPatterns` में regex पैटर्न
    - प्रति-चैनल ओवरराइड और self-chat मोड के लिए [full reference](/gateway/configuration-reference#group-chat-mention-gating) देखें।
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">    सेशंस बातचीत की निरंतरता और पृथक्करण को नियंत्रित करते हैं:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // बहु-उपयोगकर्ता के लिए अनुशंसित
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (साझा) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - स्कोपिंग, पहचान लिंक और भेजने की नीति के लिए [Session Management](/concepts/session) देखें।
    - सभी फ़ील्ड्स के लिए [full reference](/gateway/configuration-reference#session) देखें।
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">    एजेंट सेशंस को अलग-थलग Docker कंटेनरों में चलाएँ:

    ```
    scripts/sandbox-setup.sh
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">    ```json5
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

  
</Accordion>

  <Accordion title="Configure cron jobs">    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    ```
    फ़ीचर ओवरव्यू और CLI उदाहरणों के लिए [Cron jobs](/automation/cron-jobs) देखें।
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">    Gateway पर HTTP webhook एंडपॉइंट्स सक्षम करें:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">    अलग-अलग वर्कस्पेस और सेशंस के साथ कई पृथक एजेंट चलाएँ:

    ```
    // Sibling keys override included values
    {
      $include: "./base.json5", // { a: 1, b: 2 }
      b: 99, // Result: { a: 1, b: 99 }
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">    बड़े कॉन्फ़िग को व्यवस्थित करने के लिए `$include` का उपयोग करें:

    ```
    // clients/mueller.json5
    {
      agents: { $include: "./mueller/agents.json5" },
      broadcast: { $include: "./mueller/broadcast.json5" },
    }
    ```

  
</Accordion>
</AccordionGroup>

## कॉन्फ़िग हॉट रीलोड

Gateway `~/.openclaw/openclaw.json` को मॉनिटर करता है और बदलावों को स्वचालित रूप से लागू करता है — अधिकांश सेटिंग्स के लिए मैनुअल रीस्टार्ट की आवश्यकता नहीं होती।

### त्रुटि प्रबंधन

| मोड                                        | व्यवहार                                                                                                                          |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (डिफ़ॉल्ट) | सुरक्षित बदलावों को तुरंत हॉट-अप्लाई करता है। महत्वपूर्ण बदलावों के लिए स्वचालित रूप से रीस्टार्ट करता है।                       |
| **`hot`**                                  | केवल सुरक्षित बदलावों को हॉट-अप्लाई करता है। जब रीस्टार्ट की आवश्यकता होती है तो चेतावनी लॉग करता है — इसे आपको संभालना होता है। |
| **`restart`**                              | किसी भी कॉन्फ़िग बदलाव पर Gateway को रीस्टार्ट करता है, चाहे वह सुरक्षित हो या नहीं।                                             |
| **`off`**                                  | फ़ाइल मॉनिटरिंग को अक्षम करता है। बदलाव अगली मैनुअल रीस्टार्ट पर प्रभावी होते हैं।                                               |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### क्या हॉट-अप्लाई होता है और किसके लिए रीस्टार्ट आवश्यक है

अधिकांश फ़ील्ड्स बिना डाउनटाइम के हॉट-अप्लाई हो जाते हैं। `hybrid` मोड में, जिन बदलावों के लिए रीस्टार्ट आवश्यक है उन्हें स्वचालित रूप से संभाला जाता है।

| श्रेणी           | फ़ील्ड्स                                                                             | रीस्टार्ट आवश्यक? |
| ---------------- | ------------------------------------------------------------------------------------ | ----------------- |
| चैनल्स           | `channels.*`, `web` (WhatsApp) — सभी बिल्ट-इन और एक्सटेंशन चैनल्स | नहीं              |
| एजेंट और मॉडल्स  | `agent`, `agents`, `models`, `routing`                                               | नहीं              |
| ऑटोमेशन          | `hooks`, `cron`, `agent.heartbeat`                                                   | नहीं              |
| सत्र और संदेश    | `session`, `messages`                                                                | नहीं              |
| टूल्स और मीडिया  | `tools`, `browser`, `skills`, `audio`, `talk`                                        | नहीं              |
| UI और अन्य       | `ui`, `logging`, `identity`, `bindings`                                              | नहीं              |
| Gateway सर्वर    | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)              | **हाँ**           |
| इन्फ्रास्ट्रक्चर | `discovery`, `canvasHost`, `plugins`                                                 | **हाँ**           |

<Note>
`gateway.reload` और `gateway.remote` अपवाद हैं — इन्हें बदलने से **restart** ट्रिगर नहीं होता।
</Note>

## Env vars + `.env`

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    पूर्ण config को validate करता है + लिखता है और एक ही चरण में Gateway को restart करता है।

    ````
    <Warning>
    `config.apply` पूरे **entire config** को replace करता है। आंशिक अपडेट के लिए `config.patch` का उपयोग करें, या एकल keys के लिए `openclaw config set` का उपयोग करें।
    
</Warning>
    
    Params:
    
    - `raw` (string) — पूरे config के लिए JSON5 payload
    - `baseHash` (optional) — `config.get` से config hash (जब config मौजूद हो तो आवश्यक)
    - `sessionKey` (optional) — restart के बाद wake-up ping के लिए session key
    - `note` (optional) — restart sentinel के लिए नोट
    - `restartDelayMs` (optional) — restart से पहले विलंब (default 2000)
    
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
    मौजूदा config में आंशिक अपडेट को merge करता है (JSON merge patch semantics):

    ````
    - Objects को recursive तरीके से merge किया जाता है
    - `null` किसी key को हटाता है
    - Arrays replace हो जाती हैं
    
    Params:
    
    - `raw` (string) — केवल बदली जाने वाली keys के साथ JSON5
    - `baseHash` (required) — `config.get` से config hash
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` के समान
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Environment variables

OpenClaw parent process से env vars पढ़ता है, साथ ही:

- ऑप्ट-इन सुविधा: यदि सक्षम हो और अपेक्षित कोई भी key अभी सेट न हो, तो OpenClaw आपका लॉगिन शेल चलाता है और केवल गायब अपेक्षित keys इम्पोर्ट करता है (कभी भी ओवरराइड नहीं करता)।
- यह प्रभावी रूप से आपके शेल प्रोफ़ाइल को सोर्स करता है।

कोई भी फ़ाइल मौजूदा env vars को override नहीं करती। आप config में inline env vars भी सेट कर सकते हैं:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  यदि enabled है और अपेक्षित keys सेट नहीं हैं, तो OpenClaw आपकी login shell चलाता है और केवल missing keys को import करता है:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var समकक्ष: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  किसी भी config string value में `${VAR_NAME}` के साथ env vars का संदर्भ दें:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

नियम:

- केवल uppercase नाम मैच किए जाते हैं: `[A-Z_][A-Z0-9_]*`
- Missing/empty vars load समय पर error उत्पन्न करते हैं
- Literal output के लिए `$${VAR}` का उपयोग करके escape करें
- `$include` फ़ाइलों के अंदर भी काम करता है
- Inline substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

पूर्ण precedence और sources के लिए [Environment](/help/environment) देखें।

## पूर्ण संदर्भ

संपूर्ण field-by-field संदर्भ के लिए, **[Configuration Reference](/gateway/configuration-reference)** देखें।

---

Legacy OAuth आयात:

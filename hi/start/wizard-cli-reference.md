---
title: "CLI ऑनबोर्डिंग संदर्भ"
sidebarTitle: "CLI संदर्भ"
---

# CLI ऑनबोर्डिंग संदर्भ

यह पेज `openclaw onboard` के लिए पूरा रेफ़रेंस है।
संक्षिप्त गाइड के लिए देखें [Onboarding Wizard (CLI)](/start/wizard)।

## विज़ार्ड क्या करता है

लोकल मोड (डिफ़ॉल्ट) आपको निम्न चरणों से गुज़ारता है:

- मॉडल और प्रमाणीकरण सेटअप (OpenAI Code subscription OAuth, Anthropic API key या setup token, साथ ही MiniMax, GLM, Moonshot और AI Gateway विकल्प)
- वर्कस्पेस स्थान और बूटस्ट्रैप फ़ाइलें
- Gateway सेटिंग्स (पोर्ट, bind, auth, tailscale)
- चैनल और प्रदाता (Telegram, WhatsApp, Discord, Google Chat, Mattermost plugin, Signal)
- डेमन इंस्टॉल (LaunchAgent या systemd user unit)
- हेल्थ चेक
- Skills सेटअप

Remote मोड इस मशीन को किसी अन्य स्थान पर मौजूद Gateway से कनेक्ट करने के लिए कॉन्फ़िगर करता है।
यह रिमोट होस्ट पर कुछ भी इंस्टॉल या मॉडिफ़ाई नहीं करता।

## लोकल फ्लो विवरण

<Steps>
  <Step title="Existing config detection">
    - यदि `~/.openclaw/openclaw.json` मौजूद है, तो Keep, Modify, या Reset चुनें।
    - विज़ार्ड को दोबारा चलाने से कुछ भी नहीं मिटता जब तक आप स्पष्ट रूप से Reset न चुनें (या `--reset` पास न करें)।
    - यदि कॉन्फ़िग अमान्य है या उसमें legacy keys हैं, तो विज़ार्ड रुक जाता है और आगे बढ़ने से पहले आपको `openclaw doctor` चलाने के लिए कहता है।
    - Reset में `trash` का उपयोग होता है और scopes प्रदान किए जाते हैं:
      - केवल Config
      - कॉन्फ़िगरेशन + क्रेडेंशियल्स + सत्र
      - Full reset (workspace भी हटाता है)
  
</Step>
  <Step title="Model and auth">
    - पूर्ण विकल्प मैट्रिक्स [Auth and model options](#auth-and-model-options) में है।
  
</Step>
  <Step title="Workspace">
    - डिफ़ॉल्ट `~/.openclaw/workspace` (कॉन्फ़िगर करने योग्य)।
    - पहली बार चलाने के bootstrap ritual के लिए आवश्यक workspace फ़ाइल्स seed करता है।
    - वर्कस्पेस लेआउट: [Agent workspace](/concepts/agent-workspace).
  
</Step>
  <Step title="Gateway">
    - पोर्ट, bind, auth मोड, और tailscale exposure के लिए प्रॉम्प्ट करता है।
    - अनुशंसित: loopback के लिए भी token auth सक्षम रखें ताकि लोकल WS क्लाइंट्स को authenticate करना पड़े।
    - auth केवल तभी disable करें जब आप हर लोकल प्रोसेस पर पूरी तरह भरोसा करते हों।
    - Non-loopback बाइंड के लिए अभी भी प्रमाणीकरण आवश्यक है।
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): वैकल्पिक QR लॉगिन
    - [Telegram](/channels/telegram): बॉट टोकन
    - [Discord](/channels/discord): बॉट टोकन
    - [Google Chat](/channels/googlechat): सर्विस अकाउंट JSON + वेबहुक ऑडियंस
    - [Mattermost](/channels/mattermost) plugin: बॉट टोकन + बेस URL
    - [Signal](/channels/signal): वैकल्पिक `signal-cli` इंस्टॉल + अकाउंट कॉन्फ़िगरेशन
    - [BlueBubbles](/channels/bluebubbles): iMessage के लिए अनुशंसित; सर्वर URL + पासवर्ड + वेबहुक
    - [iMessage](/channels/imessage): लेगेसी `imsg` CLI पाथ + DB एक्सेस
    - DM security: डिफ़ॉल्ट pairing है। पहला DM एक कोड भेजता है; इसे
      `openclaw pairing approve <channel> <code>` के माध्यम से स्वीकृत करें या allowlists का उपयोग करें।
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - लॉग-इन यूज़र सेशन आवश्यक है; headless के लिए custom LaunchDaemon (शामिल नहीं) का उपयोग करें।
    - Linux और Windows via WSL2: systemd user unit
      - Wizard `loginctl enable-linger <user>` चलाने का प्रयास करता है ताकि logout के बाद भी gateway चालू रहे।
      - sudo के लिए प्रॉम्प्ट कर सकता है ( `/var/lib/systemd/linger` लिखता है ); पहले बिना sudo के प्रयास करता है।
    - Runtime चयन: Node (अनुशंसित; WhatsApp और Telegram के लिए आवश्यक)। Bun अनुशंसित नहीं है।
  
</Step>
  <Step title="Health check">
    - Gateway (यदि आवश्यक हो) शुरू करता है और `openclaw health` चलाता है।
    - `openclaw status --deep` स्टेटस आउटपुट में gateway health probes जोड़ता है।
  
</Step>
  <Step title="Skills">
    - उपलब्ध skills पढ़ता है और आवश्यकताओं की जाँच करता है।
    - Node manager चुनने देता है: npm या pnpm (bun अनुशंसित नहीं)।
    - वैकल्पिक dependencies इंस्टॉल करता है (कुछ macOS पर Homebrew का उपयोग करते हैं)।
  
</Step>
  <Step title="Finish">
    - सारांश और अगले चरण, जिनमें iOS, Android, और macOS ऐप विकल्प शामिल हैं।
  
</Step>
</Steps>

<Note>
यदि कोई GUI नहीं मिलता है, तो विज़ार्ड ब्राउज़र खोलने के बजाय Control UI के लिए SSH port-forward निर्देश प्रिंट करता है।
यदि Control UI एसेट्स गायब हैं, तो विज़ार्ड उन्हें build करने का प्रयास करता है; fallback है `pnpm ui:build` (UI deps को auto-install करता है)।
</Note>

## Remote मोड विवरण

Remote मोड इस मशीन को किसी अन्य स्थान पर मौजूद Gateway से कनेक्ट करने के लिए कॉन्फ़िगर करता है।

<Info>
Remote मोड रिमोट होस्ट पर कुछ भी इंस्टॉल या संशोधित नहीं करता।
</Info>

आप जो सेट करते हैं:

- Remote Gateway URL (`ws://...`)
- यदि Remote Gateway auth आवश्यक है तो टोकन (अनुशंसित)

<Note>
- यदि gateway केवल loopback है, तो SSH tunneling या tailnet का उपयोग करें।
- Discovery संकेत:
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)
</Note>

## Auth और मॉडल विकल्प

<AccordionGroup>
  <Accordion title="Anthropic API key (recommended)">
    यदि `ANTHROPIC_API_KEY` मौजूद है तो उसका उपयोग करता है, अन्यथा कुंजी के लिए प्रॉम्प्ट करता है, फिर डेमन उपयोग के लिए सहेजता है।
  
</Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: Keychain आइटम "Claude Code-credentials" की जाँच करता है
    - Linux और Windows: यदि मौजूद हो तो `~/.claude/.credentials.json` का पुनः उपयोग करता है

    macOS पर, "Always Allow" चुनें ताकि launchd स्टार्ट्स अवरुद्ध न हों।
  
</Accordion>
  <Accordion title="Anthropic token (setup-token paste)">
    किसी भी मशीन पर `claude setup-token` चलाएँ, फिर टोकन पेस्ट करें।
    आप इसे नाम दे सकते हैं; खाली छोड़ने पर डिफ़ॉल्ट उपयोग होगा।
  
</Accordion>
  <Accordion title="OpenAI Code subscription (Codex CLI reuse)">
    यदि `~/.codex/auth.json` मौजूद है, तो विज़ार्ड इसका पुनः उपयोग कर सकता है।
  
</Accordion>
  <Accordion title="OpenAI Code subscription (OAuth)">
    ब्राउज़र फ्लो; `code#state` पेस्ट करें।

    मॉडल अनसेट होने या `openai/*` होने पर `agents.defaults.model` को `openai-codex/gpt-5.3-codex` पर सेट करता है।
  
</Accordion>
  <Accordion title="OpenAI API key">
    यदि `OPENAI_API_KEY` मौजूद है तो उसका उपयोग करता है, अन्यथा कुंजी के लिए प्रॉम्प्ट करता है, फिर इसे
    `~/.openclaw/.env` में सहेजता है ताकि launchd इसे पढ़ सके।

    मॉडल अनसेट, `openai/*` या `openai-codex/*` होने पर `agents.defaults.model` को `openai/gpt-5.1-codex` पर सेट करता है।
  
</Accordion>
  <Accordion title="xAI (Grok) API key">
    `XAI_API_KEY` के लिए प्रॉम्प्ट करता है और xAI को मॉडल प्रदाता के रूप में कॉन्फ़िगर करता है।
  
</Accordion>
  <Accordion title="OpenCode Zen">
    `OPENCODE_API_KEY` (या `OPENCODE_ZEN_API_KEY`) के लिए प्रॉम्प्ट करता है।
    Setup URL: [opencode.ai/auth](https://opencode.ai/auth).
  
</Accordion>
  <Accordion title="API key (generic)">
    कुंजी आपके लिए सहेजता है।
  
</Accordion>
  <Accordion title="Vercel AI Gateway">
    `AI_GATEWAY_API_KEY` के लिए प्रॉम्प्ट करता है।
    अधिक विवरण: [Vercel AI Gateway](/providers/vercel-ai-gateway).
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway">
    account ID, gateway ID, और `CLOUDFLARE_AI_GATEWAY_API_KEY` के लिए प्रॉम्प्ट करता है।
    अधिक विवरण: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  
</Accordion>
  <Accordion title="MiniMax M2.1">
    कॉन्फ़िग स्वतः लिखा जाता है।
    अधिक विवरण: [MiniMax](/providers/minimax).
  
</Accordion>
  <Accordion title="Synthetic (Anthropic-compatible)">
    `SYNTHETIC_API_KEY` के लिए प्रॉम्प्ट करता है।
    अधिक विवरण: [Synthetic](/providers/synthetic).
  
</Accordion>
  <Accordion title="Moonshot and Kimi Coding">
    Moonshot (Kimi K2) और Kimi Coding कॉन्फ़िग स्वतः लिखे जाते हैं।
    अधिक विवरण: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot).
  
</Accordion>
  <Accordion title="Custom provider">
    OpenAI-compatible और Anthropic-compatible endpoints के साथ काम करता है।

    Non-interactive flags:
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key` (optional; falls back to `CUSTOM_API_KEY`)
    - `--custom-provider-id` (optional)
    - `--custom-compatibility <openai|anthropic>` (optional; default `openai`)
  
</Accordion>
  <Accordion title="Skip">
    प्रमाणीकरण को अनकॉन्फ़िगर छोड़ देता है।
  
</Accordion>
</AccordionGroup>

मॉडल व्यवहार:

- पता चले विकल्पों से डिफ़ॉल्ट मॉडल चुनें, या प्रदाता और मॉडल मैन्युअल रूप से दर्ज करें।
- विज़ार्ड मॉडल चेक चलाता है और यदि कॉन्फ़िगर किया गया मॉडल अज्ञात है या प्रमाणीकरण अनुपलब्ध है तो चेतावनी देता है।

क्रेडेंशियल और प्रोफ़ाइल पथ:

- OAuth क्रेडेंशियल्स: `~/.openclaw/credentials/oauth.json`
- Auth प्रोफ़ाइल्स (API keys + OAuth): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
Headless और सर्वर टिप: ब्राउज़र वाली मशीन पर OAuth पूरा करें, फिर
`~/.openclaw/credentials/oauth.json` (या `$OPENCLAW_STATE_DIR/credentials/oauth.json`)
को Gateway होस्ट पर कॉपी करें।
</Note>

## आउटपुट्स और आंतरिक संरचना

`~/.openclaw/openclaw.json` में सामान्य फ़ील्ड्स:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (यदि Minimax चुना गया हो)
- `gateway.*` (mode, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- चैनल allowlists (Slack, Discord, Matrix, Microsoft Teams) जब आप प्रॉम्प्ट्स के दौरान ऑप्ट-इन करते हैं (जहाँ संभव हो नाम IDs में रेज़ॉल्व होते हैं)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` `agents.list[]` और वैकल्पिक `bindings` लिखता है।

WhatsApp क्रेडेंशियल्स `~/.openclaw/credentials/whatsapp/<accountId>/` के अंतर्गत जाते हैं।
Sessions `~/.openclaw/agents/<agentId>/sessions/` के अंतर्गत संग्रहीत होते हैं।

<Note>
कुछ चैनल plugins के रूप में उपलब्ध हैं। ऑनबोर्डिंग के दौरान चयन करने पर, विज़ार्ड
चैनल कॉन्फ़िगरेशन से पहले plugin (npm या local path) इंस्टॉल करने के लिए प्रॉम्प्ट करता है।
</Note>

Gateway विज़ार्ड RPC:

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

क्लाइंट्स (macOS ऐप और Control UI) ऑनबोर्डिंग लॉजिक को पुनः लागू किए बिना चरणों को रेंडर कर सकते हैं।

Signal सेटअप व्यवहार:

- उपयुक्त रिलीज़ एसेट डाउनलोड करता है
- इसे `~/.openclaw/tools/signal-cli/<version>/` के अंतर्गत संग्रहीत करता है
- कॉन्फ़िग में `channels.signal.cliPath` लिखता है
- JVM बिल्ड्स के लिए Java 21 आवश्यक है
- जहाँ उपलब्ध हो, native बिल्ड्स का उपयोग किया जाता है
- Windows WSL2 का उपयोग करता है और WSL के भीतर Linux signal-cli फ्लो का पालन करता है

## संबंधित दस्तावेज़

- ऑनबोर्डिंग हब: [Onboarding Wizard (CLI)](/start/wizard)
- ऑटोमेशन और स्क्रिप्ट्स: [CLI Automation](/start/wizard-cli-automation)
- कमांड संदर्भ: [`openclaw onboard`](/cli/onboard)


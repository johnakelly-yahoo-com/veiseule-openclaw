---
title: "ऑनबोर्डिंग विज़ार्ड संदर्भ"
sidebarTitle: "विज़ार्ड संदर्भ"
---

# ऑनबोर्डिंग विज़ार्ड संदर्भ

यह `openclaw onboard` CLI विज़ार्ड के लिए पूर्ण संदर्भ है।  
उच्च‑स्तरीय अवलोकन के लिए देखें: [Onboarding Wizard](/start/wizard).

## फ़्लो विवरण (स्थानीय मोड)

<Steps>
  <Step title="Existing config detection">
    - यदि `~/.openclaw/openclaw.json` मौजूद है, तो **Keep / Modify / Reset** चुनें।
    - विज़ार्ड को दोबारा चलाने से कुछ भी **मिटता नहीं** है जब तक आप स्पष्ट रूप से **Reset** नहीं चुनते
      (या `--reset` पास करें)।
    - यदि कॉन्फ़िग अमान्य है या उसमें लेगेसी कुंजियाँ हैं, तो विज़ार्ड रुक जाता है और आपसे
      आगे बढ़ने से पहले `openclaw doctor` चलाने के लिए कहता है।
    - Reset में `trash` (कभी भी `rm` नहीं) का उपयोग होता है और यह निम्न स्कोप प्रदान करता है:
      - केवल कॉन्फ़िग
      - कॉन्फ़िग + क्रेडेंशियल्स + सेशन्स
      - पूर्ण रीसेट (वर्कस्पेस भी हटाता है)
  </Step>
  <Step title="Model/Auth">
    - **Anthropic API key (recommended)**: यदि `ANTHROPIC_API_KEY` मौजूद है तो उसका उपयोग करता है या कुंजी के लिए पूछता है, फिर डेमन उपयोग के लिए सहेजता है।
    - **Anthropic OAuth (Claude Code CLI)**: macOS पर विज़ार्ड Keychain आइटम "Claude Code-credentials" की जाँच करता है (launchd स्टार्ट ब्लॉक न हों इसके लिए "Always Allow" चुनें); Linux/Windows पर यदि मौजूद हो तो `~/.claude/.credentials.json` का पुनः उपयोग करता है।
    - **Anthropic token (paste setup-token)**: किसी भी मशीन पर `claude setup-token` चलाएँ, फिर टोकन पेस्ट करें (आप इसे नाम दे सकते हैं; खाली = डिफ़ॉल्ट)।
    - **OpenAI Code (Codex) subscription (Codex CLI)**: यदि `~/.codex/auth.json` मौजूद है, तो विज़ार्ड इसे पुनः उपयोग कर सकता है।
    - **OpenAI Code (Codex) subscription (OAuth)**: ब्राउज़र फ़्लो; `code#state` पेस्ट करें।
      - जब मॉडल unset हो या `openai/*` हो, तो `agents.defaults.model` को `openai-codex/gpt-5.2` पर सेट करता है।
    - **OpenAI API key**: यदि मौजूद हो तो `OPENAI_API_KEY` का उपयोग करता है या कुंजी के लिए पूछता है, फिर इसे `~/.openclaw/.env` में सहेजता है ताकि launchd इसे पढ़ सके।
    - **xAI (Grok) API key**: `XAI_API_KEY` के लिए पूछता है और xAI को मॉडल प्रदाता के रूप में कॉन्फ़िगर करता है।
    - **OpenCode Zen (multi-model proxy)**: `OPENCODE_API_KEY` (या `OPENCODE_ZEN_API_KEY`, इसे https://opencode.ai/auth से प्राप्त करें) के लिए पूछता है।
    - **API key**: आपके लिए कुंजी सहेजता है।
    - **Vercel AI Gateway (multi-model proxy)**: `AI_GATEWAY_API_KEY` के लिए पूछता है।  
    - अधिक विवरण: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: Account ID, Gateway ID, और `CLOUDFLARE_AI_GATEWAY_API_KEY` के लिए पूछता है।  
    - अधिक विवरण: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: कॉन्फ़िग स्वचालित रूप से लिखी जाती है।  
    - अधिक विवरण: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-compatible)**: `SYNTHETIC_API_KEY` के लिए पूछता है।  
    - अधिक विवरण: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: कॉन्फ़िग स्वचालित रूप से लिखी जाती है।
    - **Kimi Coding**: कॉन्फ़िग स्वचालित रूप से लिखी जाती है।  
    - अधिक विवरण: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: अभी कोई auth कॉन्फ़िगर नहीं किया गया।
    - पहचाने गए विकल्पों में से एक डिफ़ॉल्ट मॉडल चुनें (या provider/model मैन्युअली दर्ज करें)।
    - विज़ार्ड मॉडल जाँच चलाता है और यदि कॉन्फ़िगर किया गया मॉडल अज्ञात हो या auth गायब हो तो चेतावनी देता है।
    - OAuth क्रेडेंशियल्स `~/.openclaw/credentials/oauth.json` में रहते हैं; auth प्रोफ़ाइल `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` में (API keys + OAuth)।
    - अधिक विवरण: [/concepts/oauth](/concepts/oauth)
    <Note>
    हेडलेस/सर्वर सुझाव: ब्राउज़र वाली मशीन पर OAuth पूरा करें, फिर
    `~/.openclaw/credentials/oauth.json` (या `$OPENCLAW_STATE_DIR/credentials/oauth.json`) को
    Gateway होस्ट पर कॉपी करें।
    </Note>
  </Step>
  <Step title="Workspace">
    - डिफ़ॉल्ट `~/.openclaw/workspace` (कॉन्फ़िगर करने योग्य)।
    - एजेंट bootstrap ritual के लिए आवश्यक वर्कस्पेस फ़ाइलें तैयार करता है।
    - पूर्ण वर्कस्पेस लेआउट + बैकअप मार्गदर्शिका: [Agent workspace](/concepts/agent-workspace)
  </Step>
  <Step title="Gateway">
    - Port, bind, auth mode, tailscale exposure।
    - Auth सिफारिश: loopback पर भी **Token** रखें ताकि स्थानीय WS क्लाइंट्स को authenticate करना पड़े।
    - केवल तभी auth अक्षम करें जब आप हर स्थानीय प्रक्रिया पर पूर्ण भरोसा करते हों।
    - Non‑loopback bind पर भी auth आवश्यक है।
  </Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): वैकल्पिक QR लॉगिन।
    - [Telegram](/channels/telegram): bot token।
    - [Discord](/channels/discord): bot token।
    - [Google Chat](/channels/googlechat): service account JSON + webhook audience।
    - [Mattermost](/channels/mattermost) (plugin): bot token + base URL।
    - [Signal](/channels/signal): वैकल्पिक `signal-cli` इंस्टॉल + अकाउंट कॉन्फ़िग।
    - [BlueBubbles](/channels/bluebubbles): **iMessage के लिए अनुशंसित**; server URL + password + webhook।
    - [iMessage](/channels/imessage): legacy `imsg` CLI path + DB access।
    - DM सुरक्षा: डिफ़ॉल्ट रूप से pairing। पहला DM एक कोड भेजता है; `openclaw pairing approve <channel> <code>` के ज़रिए अनुमोदित करें या allowlists का उपयोग करें।
  </Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent  
      - लॉग‑इन किया हुआ यूज़र सेशन आवश्यक; हेडलेस के लिए कस्टम LaunchDaemon उपयोग करें (शिप नहीं किया गया)।
    - Linux (और Windows via WSL2): systemd user unit  
      - विज़ार्ड `loginctl enable-linger <user>` के ज़रिए lingering सक्षम करने की कोशिश करता है ताकि लॉगआउट के बाद भी Gateway चालू रहे।  
      - sudo के लिए पूछ सकता है (`/var/lib/systemd/linger` लिखता है); पहले बिना sudo के कोशिश करता है।
    - **Runtime चयन:** Node (अनुशंसित; WhatsApp/Telegram के लिए आवश्यक)। Bun **अनुशंसित नहीं** है।
  </Step>
  <Step title="Health check">
    - Gateway शुरू करता है (यदि आवश्यक हो) और `openclaw health` चलाता है।
    - टिप: `openclaw status --deep` स्टेटस आउटपुट में gateway health probes जोड़ता है (पहुंच योग्य gateway आवश्यक)।
  </Step>
  <Step title="Skills (recommended)">
    - उपलब्ध skills पढ़ता है और आवश्यकताओं की जाँच करता है।
    - आपको node manager चुनने देता है: **npm / pnpm** (bun अनुशंसित नहीं)।
    - वैकल्पिक dependencies इंस्टॉल करता है (कुछ macOS पर Homebrew का उपयोग करती हैं)।
  </Step>
  <Step title="Finish">
    - सारांश + अगले कदम, जिनमें अतिरिक्त फीचर्स के लिए iOS/Android/macOS ऐप्स शामिल हैं।
  </Step>
</Steps>

<Note>
यदि कोई GUI नहीं मिलता, तो विज़ार्ड ब्राउज़र खोलने के बजाय Control UI के लिए SSH port-forward निर्देश प्रिंट करता है।  
यदि Control UI assets गायब हैं, तो विज़ार्ड उन्हें build करने की कोशिश करता है; fallback है `pnpm ui:build` (UI dependencies स्वतः इंस्टॉल करता है)।
</Note>

## Non-interactive मोड

ऑनबोर्डिंग को स्वचालित या स्क्रिप्ट करने के लिए `--non-interactive` का उपयोग करें:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

मशीन‑पठनीय सारांश के लिए `--json` जोड़ें।

<Note>
`--json` का अर्थ **non-interactive मोड** नहीं है। स्क्रिप्ट्स के लिए `--non-interactive` (और `--workspace`) का उपयोग करें।
</Note>

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Z.AI example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Moonshot example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

### एजेंट जोड़ें (non-interactive)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway wizard RPC

Gateway RPC के माध्यम से wizard flow एक्सपोज़ करता है (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`)।  
क्लाइंट्स (macOS ऐप, Control UI) ऑनबोर्डिंग लॉजिक को दोबारा इम्प्लीमेंट किए बिना स्टेप्स रेंडर कर सकते हैं।

## Signal सेटअप (signal-cli)

विज़ार्ड GitHub releases से `signal-cli` इंस्टॉल कर सकता है:

- उपयुक्त release asset डाउनलोड करता है।
- इसे `~/.openclaw/tools/signal-cli/<version>/` के अंतर्गत संग्रहीत करता है।
- आपके config में `channels.signal.cliPath` लिखता है।

नोट्स:

- JVM builds के लिए **Java 21** आवश्यक है।
- जहाँ उपलब्ध हो, native builds का उपयोग किया जाता है।
- Windows WSL2 का उपयोग करता है; signal-cli इंस्टॉल WSL के भीतर Linux flow का पालन करता है।

## विज़ार्ड क्या लिखता है

`~/.openclaw/openclaw.json` में सामान्य फ़ील्ड्स:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (यदि Minimax चुना गया हो)
- `gateway.*` (mode, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- जब आप prompts के दौरान opt‑in करते हैं, तब channel allowlists (Slack/Discord/Matrix/Microsoft Teams) (जहाँ संभव हो नाम IDs में resolve होते हैं)।
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` `agents.list[]` और वैकल्पिक `bindings` लिखता है।

WhatsApp credentials `~/.openclaw/credentials/whatsapp/<accountId>/` के अंतर्गत जाते हैं।  
Sessions `~/.openclaw/agents/<agentId>/sessions/` के अंतर्गत संग्रहीत होते हैं।

कुछ channels plugins के रूप में प्रदान किए जाते हैं। ऑनबोर्डिंग के दौरान जब आप इनमें से किसी को चुनते हैं, तो विज़ार्ड
कॉन्फ़िगरेशन से पहले उसे इंस्टॉल करने (npm या लोकल पाथ) के लिए प्रॉम्प्ट करेगा।

## संबंधित दस्तावेज़

- विज़ार्ड अवलोकन: [Onboarding Wizard](/start/wizard)
- macOS ऐप ऑनबोर्डिंग: [Onboarding](/start/onboarding)
- Config संदर्भ: [Gateway configuration](/gateway/configuration)
- प्रदाता: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (legacy)
- Skills: [Skills](/tools/skills), [Skills config](/tools/skills-config)

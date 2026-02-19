---
read_when:
  - ऑनबोर्डिंग विज़ार्ड चलाते समय या कॉन्फ़िगर करते समय
  - नई मशीन सेटअप करते समय
sidebarTitle: Wizard (CLI)
summary: "CLI ऑनबोर्डिंग विज़ार्ड: Gateway, वर्कस्पेस, चैनल और Skills का इंटरैक्टिव सेटअप"
title: ऑनबोर्डिंग विज़ार्ड (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# ऑनबोर्डिंग विज़ार्ड (CLI)

CLI ऑनबोर्डिंग विज़ार्ड macOS, Linux और Windows (WSL2 के माध्यम से) पर OpenClaw सेटअप करने के लिए अनुशंसित तरीका है। यह स्थानीय या रिमोट Gateway कनेक्शन के साथ-साथ वर्कस्पेस की डिफ़ॉल्ट सेटिंग्स, चैनल और Skills को कॉन्फ़िगर करता है।

```bash
openclaw onboard
```

<Info>
पहली बार सबसे तेज़ी से चैट शुरू करने का तरीका: Control UI खोलें (चैनल कॉन्फ़िगरेशन की आवश्यकता नहीं)। `openclaw dashboard` चलाएँ और ब्राउज़र में चैट करें। दस्तावेज़: [Dashboard](/web/dashboard)।
</Info>

## क्विकस्टार्ट बनाम विस्तृत सेटअप

विज़ार्ड **क्विकस्टार्ट** (डिफ़ॉल्ट सेटिंग्स) या **विस्तृत सेटअप** (पूर्ण नियंत्रण) में से किसी एक को चुनकर शुरू होता है।

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - loopback पर स्थानीय Gateway
    - मौजूदा वर्कस्पेस या डिफ़ॉल्ट वर्कस्पेस
    - Gateway पोर्ट `18789`
    - Gateway प्रमाणीकरण टोकन स्वचालित रूप से जनरेट होता है (loopback पर भी जनरेट होता है)
    - Tailscale प्रकाशन बंद
    - Telegram और WhatsApp DM डिफ़ॉल्ट रूप से अनुमति सूची में (फोन नंबर दर्ज करने के लिए कहा जा सकता है)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - मोड, वर्कस्पेस, Gateway, चैनल, डेमन और Skills के लिए पूर्ण प्रॉम्प्ट फ्लो दिखाता है
  
</Tab>
</Tabs>

## CLI ऑनबोर्डिंग विवरण

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    स्थानीय और रिमोट फ्लो का पूर्ण विवरण, प्रमाणीकरण और मॉडल मैट्रिक्स, कॉन्फ़िगरेशन आउटपुट, विज़ार्ड RPC, और signal-cli का व्यवहार।
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    गैर-इंटरैक्टिव ऑनबोर्डिंग रेसिपी और स्वचालित `agents add` के उदाहरण।
  
</Card>
</Columns>

## अक्सर उपयोग किए जाने वाले फ़ॉलो-अप कमांड

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` का अर्थ गैर-इंटरैक्टिव मोड नहीं है। स्क्रिप्ट में `--non-interactive` का उपयोग करें।
</Note>

<Tip>
अनुशंसित: एजेंट को `web_search` उपयोग करने की अनुमति देने के लिए Brave Search API कुंजी सेट करें (`web_fetch` बिना कुंजी के काम करता है)। सबसे आसान तरीका: `openclaw configure --section web` चलाएँ, जिससे `tools.web.search.apiKey` सहेजा जाएगा। दस्तावेज़: [Webツール](/tools/web)।
</Tip>

## संबंधित दस्तावेज़

- CLI कमांड संदर्भ: [`openclaw onboard`](/cli/onboard)
- macOS ऐप ऑनबोर्डिंग: [ऑनबोर्डिंग](/start/onboarding)
- एजेंट पहली बार शुरू करने के चरण: [एजेंट बूटस्ट्रैपिंग](/start/bootstrapping)

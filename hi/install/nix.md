---
title: "Nix"
---

# Nix इंस्टॉलेशन

Nix के साथ OpenClaw चलाने का अनुशंसित तरीका **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** के माध्यम से है — यह एक batteries-included Home Manager मॉड्यूल है।

## त्वरित प्रारंभ

इसे अपने AI एजेंट (Claude, Cursor, आदि) में पेस्ट करें:

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 पूर्ण मार्गदर्शिका: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> nix-openclaw repo, Nix इंस्टॉलेशन के लिए source of truth है। यह पेज सिर्फ़ एक त्वरित अवलोकन है।

## आपको क्या मिलता है

- Gateway + macOS ऐप + टूल्स (whisper, spotify, cameras) — सभी पिन किए हुए
- रीबूट के बाद भी चलने वाली Launchd सेवा
- घोषणात्मक विन्यास के साथ प्लगइन सिस्टम
- तत्काल रोलबैक: `home-manager switch --rollback`

---

## Nix मोड रनटाइम व्यवहार

जब `OPENCLAW_NIX_MODE=1` सेट होता है (nix-openclaw के साथ स्वतः):

OpenClaw एक **Nix mode** सपोर्ट करता है जो कॉन्फ़िगरेशन को deterministic बनाता है और auto-install फ्लो को निष्क्रिय करता है।
इसे सक्षम करने के लिए export करें:

```bash
OPENCLAW_NIX_MODE=1
```

macOS पर, GUI ऐप शेल env vars को अपने आप inherit नहीं करता। आप
defaults के ज़रिए भी Nix mode सक्षम कर सकते हैं:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### विन्यास + स्थिति पथ

OpenClaw JSON5 विन्यास को `OPENCLAW_CONFIG_PATH` से पढ़ता है और परिवर्तनीय डेटा को `OPENCLAW_STATE_DIR` में संग्रहीत करता है।
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (default precedence: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (डिफ़ॉल्ट: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (डिफ़ॉल्ट: `$OPENCLAW_STATE_DIR/openclaw.json`)

Nix के अंतर्गत चलाते समय, इन्हें Nix-प्रबंधित स्थानों पर स्पष्ट रूप से सेट करें ताकि रनटाइम स्थिति और विन्यास
अपरिवर्तनीय स्टोर से बाहर रहें।

### Nix मोड में रनटाइम व्यवहार

- ऑटो-इंस्टॉल और स्वयं-म्यूटेशन प्रवाह अक्षम होते हैं
- अनुपस्थित निर्भरताएँ Nix-विशिष्ट सुधार संदेशों के साथ दिखाई देती हैं
- UI मौजूद होने पर केवल-पढ़ने योग्य Nix मोड बैनर दिखाता है

## पैकेजिंग नोट (macOS)

macOS पैकेजिंग प्रवाह एक स्थिर Info.plist टेम्पलेट की अपेक्षा करता है, जो यहाँ स्थित है:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) इस टेम्पलेट को ऐप बंडल में कॉपी करता है और dynamic फ़ील्ड्स को patch करता है
(bundle ID, version/build, Git SHA, Sparkle keys)। यह SwiftPM पैकेजिंग और Nix builds (जो पूर्ण Xcode toolchain पर निर्भर नहीं करते) के लिए plist को deterministic रखता है।

## संबंधित

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — पूर्ण सेटअप मार्गदर्शिका
- [Wizard](/start/wizard) — गैर‑Nix CLI सेटअप
- [Docker](/install/docker) — कंटेनरीकृत सेटअप


---
summary: "CLI संदर्भ `openclaw plugins` के लिए (सूची, इंस्टॉल, सक्षम/अक्षम, डॉक्टर)"
read_when:
  - आप इन-प्रोसेस Gateway प्लगइन्स को इंस्टॉल या प्रबंधित करना चाहते हैं
  - आप प्लगइन लोड विफलताओं का डिबग करना चाहते हैं
title: "प्लगइन्स"
---

# `openclaw plugins`

Gateway प्लगइन्स/एक्सटेंशन्स का प्रबंधन करें (इन-प्रोसेस लोड होते हैं)।

संबंधित:

- प्लगइन सिस्टम: [Plugins](/tools/plugin)
- प्लगइन मैनिफ़ेस्ट + स्कीमा: [Plugin manifest](/plugins/manifest)
- सुरक्षा सख़्ती: [Security](/gateway/security)

## कमांड्स

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Bundled plugins ship with OpenClaw but start disabled. 1. उन्हें सक्रिय करने के लिए `plugins enable` का उपयोग करें।

2. सभी प्लगइन्स के साथ एक `openclaw.plugin.json` फ़ाइल होनी चाहिए, जिसमें एक इनलाइन JSON Schema (`configSchema`, भले ही खाली हो) शामिल हो। 3. गायब/अमान्य मैनिफ़ेस्ट या स्कीमा प्लगइन को लोड होने से रोकते हैं और कॉन्फ़िग वैलिडेशन विफल कर देते हैं।

### इंस्टॉल

```bash
openclaw plugins install <path-or-spec>
```

4. सुरक्षा नोट: प्लगइन इंस्टॉल को कोड चलाने जैसा मानें। 5. पिन किए गए संस्करणों को प्राथमिकता दें।

Npm specs केवल **registry-only** हैं (package name + वैकल्पिक version/tag)। Git/URL/file
specs अस्वीकृत हैं। सुरक्षा के लिए dependency installs `--ignore-scripts` के साथ चलाए जाते हैं।

स्थानीय डायरेक्टरी की कॉपी से बचने के लिए `--link` का उपयोग करें (यह `plugins.load.paths` में जोड़ता है):

स्थानीय डायरेक्टरी की कॉपी से बचने के लिए `--link` का उपयोग करें (यह `plugins.load.paths` में जोड़ता है):

```bash
openclaw plugins install -l ./my-plugin
```

### अनइंस्टॉल करें

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` plugin records को `plugins.entries`, `plugins.installs`,
plugin allowlist, और लागू होने पर लिंक किए गए `plugins.load.paths` entries से हटा देता है।
सक्रिय memory plugins के लिए, memory slot `memory-core` पर रीसेट हो जाता है।

डिफ़ॉल्ट रूप से, uninstall सक्रिय state dir extensions root (`$OPENCLAW_STATE_DIR/extensions/<id>`) के अंतर्गत plugin install directory को भी हटा देता है। डिस्क पर फ़ाइलें बनाए रखने के लिए
`--keep-files` का उपयोग करें।

`--keep-config` को `--keep-files` के लिए deprecated alias के रूप में समर्थित किया गया है।

### अपडेट

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

अपडेट केवल npm से इंस्टॉल किए गए प्लगइन्स पर लागू होते हैं (जो `plugins.installs` में ट्रैक किए जाते हैं)।

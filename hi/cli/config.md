---
title: "config"
---

# `openclaw config`

कॉन्फ़िग हेल्पर: पाथ के अनुसार मान get/set/unset करें। बिना सबकमांड के चलाएँ ताकि खुल सके।
the configure wizard (same as `openclaw configure`).

## उदाहरण

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```

## पाथ

पथ डॉट या ब्रैकेट नोटेशन का उपयोग करते हैं:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

किसी विशिष्ट एजेंट को लक्षित करने के लिए एजेंट सूची इंडेक्स का उपयोग करें:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## मान

जहाँ संभव हो, मानों को JSON5 के रूप में पार्स किया जाता है; अन्यथा उन्हें स्ट्रिंग के रूप में माना जाता है।
Use `--json` to require JSON5 parsing.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

संपादन के बाद Gateway को पुनः आरंभ करें।

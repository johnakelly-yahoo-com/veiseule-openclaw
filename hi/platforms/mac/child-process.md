---
title: "Gateway जीवनचक्र"
---

# macOS पर Gateway जीवनचक्र

macOS ऐप डिफ़ॉल्ट रूप से **launchd के ज़रिए Gateway को manage करता है** और Gateway को child process के रूप में spawn नहीं करता। It first tries to attach to an already‑running
Gateway on the configured port; if none is reachable, it enables the launchd
service via the external `openclaw` CLI (no embedded runtime). This gives you
reliable auto‑start at login and restart on crashes.

Child‑process मोड (Gateway को सीधे ऐप द्वारा शुरू किया जाना) आज **उपयोग में नहीं है**।
If you need tighter coupling to the UI, run the Gateway manually in a terminal.

## डिफ़ॉल्ट व्यवहार (launchd)

- ऐप प्रति‑उपयोगकर्ता LaunchAgent इंस्टॉल करता है, जिसका लेबल `bot.molt.gateway` होता है
  (or `bot.molt.<profile>` when using `--profile`/`OPENCLAW_PROFILE`; legacy `com.openclaw.*` is supported).
- जब Local मोड सक्षम होता है, ऐप सुनिश्चित करता है कि LaunchAgent लोड हो और
  आवश्यकता होने पर Gateway शुरू करता है।
- लॉग्स launchd Gateway लॉग पथ पर लिखे जाते हैं (Debug Settings में दिखाई देते हैं)।

सामान्य कमांड:

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

किसी नामित प्रोफ़ाइल को चलाते समय लेबल को `bot.molt.<profile>` से बदलें।

## Unsigned डेवलपर बिल्ड्स

`scripts/restart-mac.sh --no-sign` तेज़ लोकल बिल्ड्स के लिए है, जब आपके पास नहीं होता है
signing keys. To prevent launchd from pointing at an unsigned relay binary, it:

- `~/.openclaw/disable-launchagent` लिखता है।

`scripts/restart-mac.sh` के साइन किए गए रन इस ओवरराइड को हटा देते हैं यदि मार्कर है
present. To reset manually:

```bash
rm ~/.openclaw/disable-launchagent
```

## Attach-only मोड

macOS ऐप को **कभी भी launchd को इंस्टॉल या प्रबंधित न करने** के लिए मजबूर करने हेतु, इसे इस प्रकार लॉन्च करें
`--attach-only` (or `--no-launchd`). This sets `~/.openclaw/disable-launchagent`,
so the app only attaches to an already running Gateway. You can toggle the same
behavior in Debug Settings.

## Remote मोड

Remote मोड कभी भी लोकल Gateway शुरू नहीं करता। ऐप SSH टनल का उपयोग करता है ताकि
remote host and connects over that tunnel.

## हम launchd को क्यों प्राथमिकता देते हैं

- लॉगिन पर ऑटो‑स्टार्ट।
- बिल्ट‑इन रीस्टार्ट/KeepAlive सेमांटिक्स।
- पूर्वानुमेय लॉग्स और सुपरविजन।

यदि भविष्य में फिर से किसी वास्तविक child‑process मोड की आवश्यकता होती है, तो
इसे एक अलग, स्पष्ट केवल‑डेव मोड के रूप में प्रलेखित किया जाना चाहिए।



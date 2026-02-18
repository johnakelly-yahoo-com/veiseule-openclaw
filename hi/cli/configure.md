---
title: "configure"
---

# `openclaw configure`

क्रेडेंशियल्स, डिवाइस, और एजेंट डिफ़ॉल्ट्स सेट अप करने के लिए इंटरैक्टिव प्रॉम्प्ट।

टिप्पणी: **Model** अनुभाग में अब `agents.defaults.models` allowlist के लिए एक मल्टी-सेलेक्ट शामिल है (जो `/model` और मॉडल पिकर में दिखाई देता है)।

टिप: `openclaw config` बिना किसी subcommand के वही wizard खोलता है। उपयोग करें
`openclaw config get|set|unset` for non-interactive edits.

संबंधित:

- Gateway विन्यास संदर्भ: [Configuration](/gateway/configuration)
- Config CLI: [कॉन्फ़िग](/cli/config)

टिप्पणियाँ:

- Gateway कहाँ चलता है, यह चुनने से हमेशा `gateway.mode` अपडेट होता है। यदि आपको केवल यही चाहिए, तो आप अन्य सेक्शनों के बिना "Continue" चुन सकते हैं।
- Channel-उन्मुख सेवाएँ (Slack/Discord/Matrix/Microsoft Teams) सेटअप के दौरान channel/room allowlists के लिए संकेत देती हैं। आप नाम या IDs दर्ज कर सकते हैं; जहाँ संभव हो, wizard नामों को IDs में बदल देता है।

## उदाहरण

```bash
openclaw configure
openclaw configure --section models --section channels
```



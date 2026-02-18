---
title: "macOS साइनिंग"
---

# mac साइनिंग (डिबग बिल्ड्स)

यह ऐप आम तौर पर [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) से बनाया जाता है, जो अब:

- एक स्थिर डिबग बंडल आइडेंटिफ़ायर सेट करता है: `ai.openclaw.mac.debug`
- उसी बंडल आईडी के साथ Info.plist लिखता है ( `BUNDLE_ID=...` के माध्यम से ओवरराइड करें)
- मुख्य बाइनरी और ऐप बंडल पर हस्ताक्षर करने के लिए [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) को कॉल करता है, ताकि macOS हर रीबिल्ड को उसी हस्ताक्षरित बंडल के रूप में माने और TCC अनुमतियाँ (सूचनाएँ, एक्सेसिबिलिटी, स्क्रीन रिकॉर्डिंग, माइक्रोफ़ोन, स्पीच) बरकरार रखे। स्थिर अनुमतियों के लिए वास्तविक signing identity का उपयोग करें; ad-hoc वैकल्पिक है और अस्थिर हो सकता है (देखें [macOS permissions](/platforms/mac/permissions)).
- डिफ़ॉल्ट रूप से `CODESIGN_TIMESTAMP=auto` का उपयोग करता है; यह Developer ID हस्ताक्षरों के लिए विश्वसनीय टाइमस्टैम्प सक्षम करता है। टाइमस्टैम्पिंग छोड़ने के लिए `CODESIGN_TIMESTAMP=off` सेट करें (ऑफ़लाइन डिबग बिल्ड्स).
- Info.plist में बिल्ड मेटाडेटा इंजेक्ट करता है: `OpenClawBuildTimestamp` (UTC) और `OpenClawGitCommit` (शॉर्ट हैश), ताकि About पैन बिल्ड, git, और डिबग/रिलीज़ चैनल दिखा सके।
- **पैकेजिंग के लिए Node 22+ आवश्यक है**: स्क्रिप्ट TS बिल्ड्स और Control UI बिल्ड चलाती है।
- पर्यावरण से `SIGN_IDENTITY` पढ़ता है। अपने शेल rc में `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (या अपना Developer ID Application प्रमाणपत्र) जोड़ें ताकि हमेशा अपने प्रमाणपत्र से साइन हो। Ad-hoc signing के लिए स्पष्ट opt-in आवश्यक है, `ALLOW_ADHOC_SIGNING=1` या `SIGN_IDENTITY="-"` के माध्यम से (permission testing के लिए अनुशंसित नहीं).
- साइन करने के बाद Team ID ऑडिट चलाता है और यदि ऐप बंडल के अंदर कोई भी Mach-O अलग Team ID से साइन किया गया हो तो विफल हो जाता है। बायपास करने के लिए `SKIP_TEAM_ID_CHECK=1` सेट करें.

## उपयोग

```bash
# from repo root
scripts/package-mac-app.sh               # auto-selects identity; errors if none found
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # real cert
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (permissions will not stick)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # explicit ad-hoc (same caveat)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # dev-only Sparkle Team ID mismatch workaround
```

### Ad-hoc साइनिंग नोट

जब `SIGN_IDENTITY="-"` (ad-hoc) के साथ साइन किया जाता है, तो स्क्रिप्ट स्वतः **Hardened Runtime** (`--options runtime`) को अक्षम कर देती है। यह आवश्यक है ताकि ऐप एम्बेडेड फ्रेमवर्क (जैसे Sparkle) लोड करने का प्रयास करते समय क्रैश न हो, जो समान Team ID साझा नहीं करते। Ad-hoc हस्ताक्षर TCC permission persistence को भी बाधित करते हैं; रिकवरी चरणों के लिए [macOS permissions](/platforms/mac/permissions) देखें.

## About के लिए बिल्ड मेटाडेटा

`package-mac-app.sh` बंडल पर निम्न मुहर लगाता है:

- `OpenClawBuildTimestamp`: पैकेज समय पर ISO8601 UTC
- `OpenClawGitCommit`: शॉर्ट git हैश (या अनुपलब्ध होने पर `unknown`)

About टैब इन कुंजियों को पढ़कर संस्करण, बिल्ड तिथि, git commit, और क्या यह डिबग बिल्ड है ( `#if DEBUG` के माध्यम से) दिखाता है। कोड परिवर्तन के बाद इन मानों को रिफ़्रेश करने के लिए packager चलाएँ.

## क्यों

TCC अनुमतियाँ bundle identifier _और_ code signature से जुड़ी होती हैं। बदलते हुए UUID के साथ unsigned debug builds के कारण macOS हर रीबिल्ड के बाद अनुमतियाँ भूल रहा था। बाइनरी पर हस्ताक्षर करना (डिफ़ॉल्ट रूप से ad‑hoc) और एक स्थिर bundle id/path (`dist/OpenClaw.app`) बनाए रखना, बिल्ड्स के बीच अनुमतियाँ संरक्षित रखता है, जो VibeTunnel दृष्टिकोण से मेल खाता है.


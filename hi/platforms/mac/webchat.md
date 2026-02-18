---
title: "WebChat"
---

# WebChat (macOS ऐप)

macOS मेनू बार ऐप WebChat UI को एक नेटिव SwiftUI व्यू के रूप में एम्बेड करता है। यह
connects to the Gateway and defaults to the **main session** for the selected
agent (with a session switcher for other sessions).

- **Local mode**: सीधे स्थानीय Gateway WebSocket से कनेक्ट होता है।
- **Remote mode**: Gateway कंट्रोल पोर्ट को SSH के माध्यम से फ़ॉरवर्ड करता है और उस
  टनल को डेटा प्लेन के रूप में उपयोग करता है।

## लॉन्च और डिबगिंग

- Manual: Lobster मेनू → “Open Chat”.

- परीक्षण के लिए Auto‑open:

  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```

- Logs: `./scripts/clawlog.sh` (subsystem `bot.molt`, category `WebChatSwiftUI`)।

## यह कैसे जुड़ा है

- Data plane: Gateway WS मेथड्स `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` और इवेंट्स `chat`, `agent`, `presence`, `tick`, `health`।
- Session: डिफ़ॉल्ट रूप से प्राथमिक सत्र (`main`, या `global` जब स्कोप
global)। UI सत्रों के बीच स्विच कर सकता है।
- Onboarding पहले‑रन सेटअप को अलग रखने के लिए एक समर्पित सत्र का उपयोग करता है।

## सुरक्षा सतह

- Remote mode में केवल Gateway WebSocket कंट्रोल पोर्ट को SSH के माध्यम से फ़ॉरवर्ड किया जाता है।

## ज्ञात सीमाएँ

- UI चैट सत्रों के लिए अनुकूलित है (पूर्ण ब्राउज़र sandbox नहीं)।

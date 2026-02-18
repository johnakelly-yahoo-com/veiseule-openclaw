---
title: "वॉइस ओवरले"
---

# वॉइस ओवरले जीवनचक्र (macOS)

Audience: macOS ऐप योगदानकर्ता। Goal: wake-word और push-to-talk के ओवरलैप होने पर voice overlay को पूर्वानुमेय बनाए रखना।

## वर्तमान आशय

- यदि overlay पहले से wake-word के कारण दिखाई दे रहा है और उपयोगकर्ता hotkey दबाता है, तो hotkey सत्र मौजूदा टेक्स्ट को _adopt_ करता है, उसे रीसेट नहीं करता। जब तक hotkey दबा रहता है, overlay दिखाई देता रहता है। जब उपयोगकर्ता कुंजी छोड़ता है: यदि trimmed टेक्स्ट मौजूद है तो भेजें, अन्यथा बंद करें।
- केवल वेक‑वर्ड होने पर मौन पर स्वतः भेजा जाता है; पुश‑टू‑टॉक में छोड़ते ही भेजा जाता है।

## कार्यान्वित (9 दिसंबर, 2025)

- अब overlay सत्र प्रत्येक capture (wake-word या push-to-talk) के लिए एक token रखते हैं। जब token मेल नहीं खाता, तो partial/final/send/dismiss/level अपडेट हटा दिए जाते हैं, जिससे stale callbacks से बचा जा सके।
- Push-to-talk adopts any visible overlay text as a prefix (so pressing the hotkey while the wake overlay is up keeps the text and appends new speech). यह अंतिम transcript के लिए 1.5s तक प्रतीक्षा करता है, उसके बाद मौजूदा text पर fallback कर जाता है।
- चाइम/ओवरले लॉगिंग `info` पर श्रेणियों `voicewake.overlay`, `voicewake.ptt`, और `voicewake.chime` में उत्सर्जित होती है (सत्र प्रारंभ, आंशिक, अंतिम, भेजें, बंद करें, चाइम कारण)।

## अगले चरण

1. **VoiceSessionCoordinator (actor)**
   - एक समय में ठीक एक `VoiceSession` का स्वामित्व।
   - API (टोकन‑आधारित): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`।
   - बासी टोकन वाले कॉलबैक को छोड़ता है (पुराने रिकग्नाइज़र द्वारा ओवरले को फिर से खोलने से रोकता है)।
2. **VoiceSession (मॉडल)**
   - फ़ील्ड्स: `token`, `source` (wakeWord|pushToTalk), कमिटेड/वोलेटाइल पाठ, चाइम फ़्लैग्स, टाइमर्स (ऑटो‑सेंड, आइडल), `overlayMode` (display|editing|sending), कूलडाउन डेडलाइन।
3. **ओवरले बाइंडिंग**
   - `VoiceSessionPublisher` (`ObservableObject`) सक्रिय सत्र को SwiftUI में मिरर करता है।
   - `VoiceWakeOverlayView` केवल पब्लिशर के माध्यम से रेंडर करता है; यह कभी भी ग्लोबल सिंगलटन को सीधे म्यूटेट नहीं करता।
   - ओवरले उपयोगकर्ता क्रियाएँ (`sendNow`, `dismiss`, `edit`) सत्र टोकन के साथ कोऑर्डिनेटर में वापस कॉल करती हैं।
4. **एकीकृत भेजने का पथ**
   - `endCapture` पर: यदि ट्रिम किया हुआ पाठ खाली है → बंद करें; अन्यथा `performSend(session:)` (एक बार सेंड चाइम बजाता है, फ़ॉरवर्ड करता है, बंद करता है)।
   - पुश‑टू‑टॉक: कोई देरी नहीं; वेक‑वर्ड: ऑटो‑सेंड के लिए वैकल्पिक देरी।
   - पुश‑टू‑टॉक समाप्त होने के बाद वेक रनटाइम पर एक छोटा कूलडाउन लागू करें ताकि वेक‑वर्ड तुरंत फिर से ट्रिगर न हो।
5. **लॉगिंग**
   - कोऑर्डिनेटर `.info` लॉग्स को सबसिस्टम `bot.molt` में, श्रेणियों `voicewake.overlay` और `voicewake.chime` के अंतर्गत उत्सर्जित करता है।
   - प्रमुख घटनाएँ: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`।

## डिबगिंग चेकलिस्ट

- चिपचिपे ओवरले को पुनः उत्पन्न करते समय स्ट्रीम लॉग्स:

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```

- केवल एक सक्रिय सत्र टोकन की पुष्टि करें; बासी कॉलबैक को कोऑर्डिनेटर द्वारा छोड़ा जाना चाहिए।

- सुनिश्चित करें कि पुश‑टू‑टॉक रिलीज़ हमेशा सक्रिय टोकन के साथ `endCapture` को कॉल करती है; यदि पाठ खाली है, तो बिना चाइम या सेंड के `dismiss` की अपेक्षा करें।

## माइग्रेशन चरण (सुझावित)

1. `VoiceSessionCoordinator`, `VoiceSession`, और `VoiceSessionPublisher` जोड़ें।
2. `VoiceWakeRuntime` को रिफ़ैक्टर करें ताकि `VoiceWakeOverlayController` को सीधे छूने के बजाय सत्र बनाए/अपडेट/समाप्त किए जाएँ।
3. `VoicePushToTalk` को रिफ़ैक्टर करें ताकि मौजूदा सत्रों को अपनाया जा सके और रिलीज़ पर `endCapture` को कॉल किया जा सके; रनटाइम कूलडाउन लागू करें।
4. `VoiceWakeOverlayController` को पब्लिशर से वायर करें; रनटाइम/PTT से सीधे कॉल हटाएँ।
5. सत्र अपनाने, कूलडाउन, और खाली‑पाठ डिसमिसल के लिए एकीकरण परीक्षण जोड़ें।


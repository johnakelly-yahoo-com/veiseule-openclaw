---
title: "कम्पैक्शन"
---

# संदर्भ विंडो और कम्पैक्शन

26. हर model का एक **context window** होता है (अधिकतम tokens जो वह देख सकता है)। 27. Long-running chats messages और tool results जमा करते हैं; जब window tight हो जाती है, OpenClaw सीमाओं के भीतर रहने के लिए पुराने history को **compact** करता है।

## कम्पैक्शन क्या है

28. Compaction पुराने conversation को **summarize** करके एक compact summary entry बनाता है और हाल के messages को intact रखता है। 29. Summary session history में store की जाती है, ताकि future requests इसका उपयोग करें:

- कम्पैक्शन सार
- कम्पैक्शन बिंदु के बाद के हालिया संदेश

कम्पैक्शन सत्र के JSONL इतिहास में **स्थायी** रहता है।

## विन्यास

`agents.defaults.compaction` सेटिंग्स के लिए [Compaction config & modes](/concepts/compaction) देखें।

## स्वचालित कम्पैक्शन (डिफ़ॉल्ट चालू)

जब कोई सत्र मॉडल की संदर्भ विंडो के पास आता है या उसे पार करता है, तो OpenClaw स्वचालित कम्पैक्शन ट्रिगर करता है और कम्पैक्ट किए गए संदर्भ का उपयोग करके मूल अनुरोध को पुनः आज़मा सकता है।

आप देखेंगे:

- verbose मोड में `🧹 Auto-compaction complete`
- `/status` जो `🧹 Compactions: <count>` दिखाता है

30. Compaction से पहले, OpenClaw disk पर durable notes store करने के लिए एक **silent memory flush** turn चला सकता है। 31. विवरण और config के लिए देखें [Memory](/concepts/memory)।

## मैनुअल कम्पैक्शन

कम्पैक्शन पास को बाध्य करने के लिए `/compact` (वैकल्पिक निर्देशों के साथ) का उपयोग करें:

```
/compact Focus on decisions and open questions
```

## संदर्भ विंडो का स्रोत

32. Context window model-specific होता है। 33. OpenClaw limits निर्धारित करने के लिए configured provider catalog से model definition का उपयोग करता है।

## कम्पैक्शन बनाम प्रूनिंग

- **कम्पैक्शन**: सारांशित करता है और JSONL में **स्थायी** रहता है।
- **सत्र प्रूनिंग**: केवल पुराने **टूल परिणामों** को, प्रति अनुरोध, **इन-मेमोरी** ट्रिम करता है।

प्रूनिंग के विवरण के लिए [/concepts/session-pruning](/concepts/session-pruning) देखें।

## सुझाव

- जब सत्र बासी लगें या संदर्भ फूला हुआ हो, तो `/compact` का उपयोग करें।
- बड़े टूल आउटपुट पहले से ही ट्रंकेट किए जाते हैं; प्रूनिंग टूल-परिणामों के जमाव को और कम कर सकती है।
- यदि आपको नई शुरुआत चाहिए, तो `/new` या `/reset` एक नया सत्र आईडी शुरू करता है।



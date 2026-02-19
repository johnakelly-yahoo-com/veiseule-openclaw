---
summary: "योजना: CDP का उपयोग करके browser act:evaluate को Playwright queue से अलग करना, end-to-end deadlines और अधिक सुरक्षित ref resolution के साथ"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor Plan

## संदर्भ

`act:evaluate` पेज में उपयोगकर्ता द्वारा प्रदान किया गया JavaScript निष्पादित करता है। वर्तमान में यह Playwright के माध्यम से चलता है
(`page.evaluate` या `locator.evaluate`)। Playwright प्रति पेज CDP कमांड्स को serialize करता है, इसलिए कोई अटका हुआ या लंबे समय तक चलने वाला evaluate पेज की command queue को ब्लॉक कर सकता है और उस टैब पर बाद की हर क्रिया को "अटका हुआ" दिखा सकता है।

PR #13498 एक व्यावहारिक सुरक्षा उपाय जोड़ता है (bounded evaluate, abort propagation, और best-effort recovery)। यह दस्तावेज़ एक बड़े refactor का वर्णन करता है जो `act:evaluate` को Playwright से मूल रूप से अलग करता है ताकि अटका हुआ evaluate सामान्य Playwright ऑपरेशन्स को प्रभावित न कर सके।

## लक्ष्य

- `act:evaluate` उसी टैब पर बाद की browser क्रियाओं को स्थायी रूप से ब्लॉक नहीं कर सकता।
- Timeouts end-to-end एक ही source of truth हों ताकि कॉलर अपने निर्धारित budget पर भरोसा कर सके।
- Abort और timeout को HTTP और in-process dispatch दोनों में एक ही तरीके से संभाला जाता है।
- evaluate के लिए element targeting का समर्थन हो, बिना सब कुछ Playwright से हटाए।
- मौजूदा callers और payloads के लिए backward compatibility बनाए रखें।

## Non-goals

- सभी browser क्रियाओं (click, type, wait, आदि) को बदलना CDP implementations से।
- PR #13498 में जो मौजूदा safety net जोड़ा गया है उसे हटाना (यह एक उपयोगी fallback बना रहता है)।
- मौजूदा `browser.evaluateEnabled` gate से आगे नई असुरक्षित क्षमताएँ जोड़ना।
- evaluate के लिए process isolation (worker process/thread) जोड़ना। यदि इस refactor के बाद भी कठिन-से-रिकवर होने वाली अटकी हुई स्थितियाँ दिखती हैं,
  तो वह एक follow-up विचार होगा।

## वर्तमान Architecture (यह क्यों अटकता है)

उच्च स्तर पर:

- Callers `act:evaluate` को browser control service को भेजते हैं।
- Route handler JavaScript निष्पादित करने के लिए Playwright को कॉल करता है।
- Playwright पेज कमांड्स को serialize करता है, इसलिए जो evaluate कभी समाप्त नहीं होता वह queue को ब्लॉक कर देता है।
- अटकी हुई queue का मतलब है कि टैब पर बाद की click/type/wait क्रियाएँ अटकी हुई प्रतीत हो सकती हैं।

## प्रस्तावित Architecture

### 1. Deadline Propagation

एक single budget अवधारणा पेश करें और सब कुछ उसी से व्युत्पन्न करें:

- Caller `timeoutMs` सेट करता है (या भविष्य में एक deadline निर्धारित करता है)।
- बाहरी request timeout, route handler logic, और पेज के अंदर execution budget
  सभी उसी budget का उपयोग करते हैं, जहाँ serialization overhead के लिए आवश्यक हो वहाँ थोड़ा headroom रखा जाता है।
- Abort को हर जगह `AbortSignal` के रूप में propagate किया जाता है ताकि cancellation सुसंगत रहे।

कार्यान्वयन दिशा:

- एक छोटा हेल्पर जोड़ें (उदाहरण के लिए `createBudget({ timeoutMs, signal })`) जो निम्नलिखित लौटाए:
  - `signal`: लिंक किया हुआ AbortSignal
  - `deadlineAtMs`: पूर्ण समय-सीमा
  - `remainingMs()`: चाइल्ड ऑपरेशनों के लिए शेष बजट
- इस हेल्पर का उपयोग करें:
  - `src/browser/client-fetch.ts` (HTTP और in-process dispatch)
  - `src/node-host/runner.ts` (proxy path)
  - browser action implementations (Playwright और CDP)

### 2. अलग Evaluate Engine (CDP Path)

एक CDP आधारित evaluate implementation जोड़ें जो Playwright की प्रति-पृष्ठ कमांड
queue को साझा न करे। मुख्य विशेषता यह है कि evaluate transport एक अलग WebSocket कनेक्शन
और target से जुड़ा एक अलग CDP session है।

कार्यान्वयन दिशा:

- नया मॉड्यूल, उदाहरण के लिए `src/browser/cdp-evaluate.ts`, जो:
  - कॉन्फ़िगर किए गए CDP endpoint (browser स्तर के socket) से कनेक्ट करता है।
  - `Target.attachToTarget({ targetId, flatten: true })` का उपयोग करके `sessionId` प्राप्त करता है।
  - इनमें से किसी एक को चलाता है:
    - पेज स्तर evaluate के लिए `Runtime.evaluate`, या
    - element evaluate के लिए `DOM.resolveNode` प्लस `Runtime.callFunctionOn`।
  - timeout या abort होने पर:
    - सेशन के लिए best-effort आधार पर `Runtime.terminateExecution` भेजता है।
    - WebSocket बंद करता है और एक स्पष्ट त्रुटि लौटाता है।

नोट्स:

- यह अभी भी पेज में JavaScript निष्पादित करता है, इसलिए termination के दुष्प्रभाव हो सकते हैं। फायदा
  यह है कि यह Playwright queue को अटकाता नहीं है, और CDP session को समाप्त करके transport
  layer पर इसे रद्द किया जा सकता है।

### 3. Ref स्टोरी (पूर्ण पुनर्लेखन के बिना Element Targeting)

कठिन हिस्सा element targeting है। CDP को DOM handle या `backendDOMNodeId` की आवश्यकता होती है, जबकि
आज अधिकांश browser actions snapshots से प्राप्त refs पर आधारित Playwright locators का उपयोग करते हैं।

अनुशंसित तरीका: मौजूदा refs को बनाए रखें, लेकिन एक वैकल्पिक CDP resolvable id जोड़ें।

#### 3.1 Stored Ref Info का विस्तार करें

संग्रहीत role ref metadata का विस्तार करें ताकि वैकल्पिक रूप से एक CDP id शामिल हो:

- आज: `{ role, name, nth }`
- प्रस्तावित: `{ role, name, nth, backendDOMNodeId?: number }`

यह सभी मौजूदा Playwright आधारित actions को कार्यशील रखता है और CDP evaluate को वही `ref` मान स्वीकार करने की अनुमति देता है जब `backendDOMNodeId` उपलब्ध हो।

#### 3.2 Snapshot समय पर backendDOMNodeId भरें

जब एक role snapshot तैयार किया जा रहा हो:

1. मौजूदा role ref map को आज की तरह उत्पन्न करें (role, name, nth)।
2. CDP के माध्यम से AX tree प्राप्त करें (`Accessibility.getFullAXTree`) और समान duplicate handling नियमों का उपयोग करते हुए
   `(role, name, nth) -> backendDOMNodeId` का एक समानांतर map गणना करें।
3. वर्तमान टैब के लिए संग्रहीत ref जानकारी में id को वापस मर्ज करें।

यदि किसी ref के लिए मैपिंग विफल हो जाती है, तो `backendDOMNodeId` को undefined ही रहने दें। यह इस फीचर को
best-effort और सुरक्षित रूप से रोल आउट करने योग्य बनाता है।

#### 3.3 Ref के साथ व्यवहार का मूल्यांकन करें

`act:evaluate` में:

- यदि `ref` मौजूद है और उसमें `backendDOMNodeId` है, तो CDP के माध्यम से element evaluate चलाएँ।
- यदि `ref` मौजूद है लेकिन उसमें `backendDOMNodeId` नहीं है, तो Playwright पाथ पर वापस जाएँ (सुरक्षा
  नेट के साथ)।

वैकल्पिक एस्केप हैच:

- उन्नत कॉलर्स (और
  डिबगिंग के लिए) के लिए `backendDOMNodeId` को सीधे स्वीकार करने हेतु रिक्वेस्ट संरचना का विस्तार करें, जबकि `ref` को प्राथमिक इंटरफ़ेस के रूप में बनाए रखें।

### 4. एक अंतिम उपाय रिकवरी पाथ बनाए रखें

CDP evaluate के साथ भी, टैब या कनेक्शन के अटकने के अन्य तरीके हो सकते हैं। निम्न के लिए अंतिम उपाय के रूप में
मौजूदा रिकवरी मैकेनिज़्म (execution समाप्त करना + Playwright को डिस्कनेक्ट करना) बनाए रखें:

- legacy कॉलर्स
- वे वातावरण जहाँ CDP attach अवरुद्ध है
- अनपेक्षित Playwright edge cases

## कार्यान्वयन योजना (एकल इटरेशन)

### डिलिवरेबल्स

- एक CDP आधारित evaluate इंजन जो Playwright के प्रति-पेज कमांड क्यू के बाहर चलता है।
- एकल एंड-टू-एंड timeout/abort बजट जिसे कॉलर्स और हैंडलर्स द्वारा लगातार उपयोग किया जाए।
- Ref मेटाडेटा जो वैकल्पिक रूप से element evaluate के लिए `backendDOMNodeId` वहन कर सकता है।
- जहाँ संभव हो `act:evaluate` CDP इंजन को प्राथमिकता देता है और अन्यथा Playwright पर फॉलबैक करता है।
- ऐसे टेस्ट जो साबित करें कि अटका हुआ evaluate बाद की क्रियाओं को अवरुद्ध नहीं करता।
- लॉग्स/मेट्रिक्स जो विफलताओं और फॉलबैक को दृश्य बनाते हैं।

### कार्यान्वयन चेकलिस्ट

1. `timeoutMs` + upstream `AbortSignal` को जोड़ने के लिए एक साझा "budget" हेल्पर जोड़ें:
   - एकल `AbortSignal`
   - एक absolute deadline
   - डाउनस्ट्रीम ऑपरेशनों के लिए `remainingMs()` हेल्पर
2. सभी कॉलर पाथ को उस हेल्पर का उपयोग करने के लिए अपडेट करें ताकि `timeoutMs` का अर्थ हर जगह एक समान हो:
   - `src/browser/client-fetch.ts` (HTTP और in-process dispatch)
   - `src/node-host/runner.ts` (node proxy पाथ)
   - CLI रैपर्स जो `/act` को कॉल करते हैं (`browser evaluate` में `--timeout-ms` जोड़ें)
3. `src/browser/cdp-evaluate.ts` लागू करें:
   - ब्राउज़र-स्तरीय CDP सॉकेट से कनेक्ट करें
   - `sessionId` प्राप्त करने के लिए `Target.attachToTarget`
   - पेज evaluate के लिए `Runtime.evaluate` चलाएँ
   - element evaluate के लिए `DOM.resolveNode` + `Runtime.callFunctionOn` चलाएँ
   - timeout/abort पर: best-effort `Runtime.terminateExecution` फिर सॉकेट बंद करें
4. संग्रहीत role ref मेटाडेटा का विस्तार करें ताकि वैकल्पिक रूप से `backendDOMNodeId` शामिल किया जा सके:
   - Playwright actions के लिए मौजूदा `{ role, name, nth }` व्यवहार बनाए रखें
   - CDP element targeting के लिए `backendDOMNodeId?: number` जोड़ें
5. snapshot बनाते समय `backendDOMNodeId` को populate करें (best-effort):
   - CDP के माध्यम से AX tree प्राप्त करें (`Accessibility.getFullAXTree`)
   - `(role, name, nth) -> backendDOMNodeId` की गणना करें और stored ref map में merge करें
   - यदि mapping अस्पष्ट है या उपलब्ध नहीं है, तो id को undefined छोड़ दें
6. `act:evaluate` routing को अपडेट करें:
   - यदि `ref` नहीं है: हमेशा CDP evaluate का उपयोग करें
   - यदि `ref` किसी `backendDOMNodeId` में resolve होता है: CDP element evaluate का उपयोग करें
   - अन्यथा: Playwright evaluate पर वापस जाएँ (फिर भी सीमित और abortable)
7. मौजूदा "last resort" recovery path को fallback के रूप में रखें, default path के रूप में नहीं।
8. टेस्ट जोड़ें:
   - stuck evaluate निर्धारित समय-सीमा के भीतर timeout हो जाए और अगला click/type सफल हो
   - abort evaluate को रद्द करे (client disconnect या timeout) और बाद की actions को unblock करे
   - mapping failures साफ़ तौर पर Playwright पर fallback हों
9. observability जोड़ें:
   - evaluate अवधि और timeout counters
   - terminateExecution उपयोग
   - fallback दर (CDP -> Playwright) और उसके कारण

### स्वीकृति मानदंड

- जानबूझकर अटकाया गया `act:evaluate` caller के budget के भीतर लौटे और बाद की actions के लिए
  tab को अटकाए नहीं।
- `timeoutMs` CLI, agent tool, node proxy, और in-process calls में समान रूप से व्यवहार करे।
- यदि `ref` को `backendDOMNodeId` में map किया जा सकता है, तो element evaluate CDP का उपयोग करे; अन्यथा
  fallback path फिर भी सीमित और पुनर्प्राप्त करने योग्य हो।

## परीक्षण योजना

- यूनिट टेस्ट:
  - role refs और AX tree nodes के बीच `(role, name, nth)` matching logic।
  - Budget helper का व्यवहार (headroom, शेष समय की गणना)।
- इंटीग्रेशन टेस्ट:
  - CDP evaluate timeout budget के भीतर लौटे और अगली action को block न करे।
  - Abort evaluate को रद्द करे और termination को best-effort आधार पर ट्रिगर करे।
- कॉन्ट्रैक्ट टेस्ट:
  - सुनिश्चित करें कि `BrowserActRequest` और `BrowserActResponse` संगत बने रहें।

## जोखिम और शमन

- Mapping पूर्णतः सटीक नहीं है:
  - शमन: best-effort mapping, Playwright evaluate पर fallback, और debug tooling जोड़ें।
- `Runtime.terminateExecution` के side effects होते हैं:
  - शमन: केवल timeout/abort पर उपयोग करें और errors में इस व्यवहार का दस्तावेज़ करें।
- अतिरिक्त ओवरहेड:
  - शमन: केवल तब AX tree प्राप्त करें जब snapshots का अनुरोध किया गया हो, प्रति target cache करें, और
    CDP session को अल्पकालिक रखें।
- Extension relay सीमाएँ:
  - निवारण: जब प्रति-पेज सॉकेट उपलब्ध न हों तो ब्राउज़र-स्तरीय attach APIs का उपयोग करें, और
    वर्तमान Playwright पथ को fallback के रूप में बनाए रखें।

## खुले प्रश्न

- क्या नए इंजन को `playwright`, `cdp`, या `auto` के रूप में कॉन्फ़िगर करने योग्य होना चाहिए?
- क्या हम उन्नत उपयोगकर्ताओं के लिए नया "nodeRef" फ़ॉर्मेट उपलब्ध कराना चाहते हैं, या केवल `ref` ही रखना चाहिए?
- फ्रेम स्नैपशॉट और सेलेक्टर-स्कोप्ड स्नैपशॉट्स को AX मैपिंग में कैसे शामिल होना चाहिए?

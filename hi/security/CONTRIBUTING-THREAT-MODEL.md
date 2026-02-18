# OpenClaw Threat Model में योगदान

OpenClaw को अधिक सुरक्षित बनाने में मदद करने के लिए धन्यवाद। यह threat model एक जीवंत दस्तावेज़ है और हम किसी भी व्यक्ति के योगदान का स्वागत करते हैं — आपको सुरक्षा विशेषज्ञ होने की आवश्यकता नहीं है।

## योगदान करने के तरीके

### एक Threat जोड़ें

क्या आपने कोई attack vector या जोखिम देखा है जिसे हमने शामिल नहीं किया? [openclaw/trust](https://github.com/openclaw/trust/issues) पर एक issue खोलें और इसे अपने शब्दों में वर्णित करें। आपको किसी framework की जानकारी होना या हर फ़ील्ड भरना आवश्यक नहीं है — बस परिदृश्य समझाएँ।

**शामिल करना उपयोगी होगा (लेकिन अनिवार्य नहीं):**

- आक्रमण परिदृश्य और उसका दुरुपयोग कैसे किया जा सकता है
- OpenClaw के कौन से भाग प्रभावित होते हैं (CLI, gateway, channels, ClawHub, MCP servers, आदि)
- आपके अनुसार इसकी गंभीरता (low / medium / high / critical)
- संबंधित शोध, CVEs, या वास्तविक उदाहरणों के लिंक

समीक्षा के दौरान हम ATLAS mapping, threat IDs, और risk assessment संभाल लेंगे। यदि आप ये विवरण जोड़ना चाहें तो बहुत अच्छा — लेकिन यह अपेक्षित नहीं है।

> **यह threat model में जोड़ने के लिए है, लाइव vulnerabilities की रिपोर्टिंग के लिए नहीं।** यदि आपको कोई exploitable vulnerability मिली है, तो responsible disclosure निर्देशों के लिए हमारी [Trust page](https://trust.openclaw.ai) देखें।

### एक Mitigation सुझाएँ

किसी मौजूदा threat को संबोधित करने का कोई विचार है? उस threat का संदर्भ देते हुए एक issue या PR खोलें। उपयोगी mitigations विशिष्ट और क्रियान्वित करने योग्य होते हैं — उदाहरण के लिए, "gateway पर प्रति-प्रेषक 10 संदेश/मिनट की rate limiting" कहना, "rate limiting लागू करें" से बेहतर है।

### एक Attack Chain प्रस्तावित करें

Attack chains दिखाती हैं कि कैसे कई threats मिलकर एक वास्तविक आक्रमण परिदृश्य बनाते हैं। यदि आपको कोई खतरनाक संयोजन दिखता है, तो चरणों का वर्णन करें और बताएँ कि एक हमलावर उन्हें कैसे जोड़ सकता है। व्यवहार में हमला कैसे घटित होता है, इसका एक संक्षिप्त वर्णन किसी औपचारिक टेम्पलेट से अधिक मूल्यवान है।

### मौजूदा सामग्री को ठीक या बेहतर बनाएँ

टाइपो, स्पष्टीकरण, पुरानी जानकारी, बेहतर उदाहरण — PR का स्वागत है, issue की आवश्यकता नहीं।

## हम क्या उपयोग करते हैं

### MITRE ATLAS

यह threat model [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems) पर आधारित है, जो विशेष रूप से AI/ML खतरों जैसे prompt injection, tool misuse, और agent exploitation के लिए बनाया गया एक framework है। योगदान करने के लिए आपको ATLAS जानना आवश्यक नहीं है — हम समीक्षा के दौरान submissions को framework से मैप करते हैं।

### खतरा आईडी

प्रत्येक threat को `T-EXEC-003` जैसा एक ID दिया जाता है। श्रेणियाँ इस प्रकार हैं:

| कोड    | श्रेणी                                   |
| ------- | ------------------------------------------ |
| RECON   | Reconnaissance - जानकारी एकत्र करना       |
| ACCESS  | Initial access - प्रारंभिक प्रवेश प्राप्त करना |
| EXEC    | Execution - दुर्भावनापूर्ण क्रियाएँ चलाना |
| PERSIST | Persistence - पहुँच बनाए रखना             |
| EVADE   | Defense evasion - पहचान से बचना           |
| DISC    | Discovery - वातावरण के बारे में सीखना    |
| EXFIL   | Exfiltration - डेटा चोरी करना             |
| IMPACT  | Impact - नुकसान या व्यवधान                |

IDs समीक्षा के दौरान maintainers द्वारा सौंपे जाते हैं। आपको स्वयं कोई चुनने की आवश्यकता नहीं है।

### जोखिम स्तर

| स्तर        | अर्थ                                                           |
| ------------ | ----------------------------------------------------------------- |
| **गंभीर** | पूर्ण सिस्टम समझौता, या उच्च संभावना + गंभीर प्रभाव             |
| **उच्च**     | महत्वपूर्ण नुकसान की संभावना, या मध्यम संभावना + गंभीर प्रभाव   |
| **मध्यम**   | मध्यम जोखिम, या कम संभावना + उच्च प्रभाव                        |
| **Low**      | कम संभावना और सीमित प्रभाव                                       |

यदि आपको risk level के बारे में संदेह है, तो केवल प्रभाव का वर्णन करें और हम उसका आकलन कर लेंगे।

## समीक्षा प्रक्रिया

1. **Triage** - हम 48 घंटे के भीतर नई submissions की समीक्षा करते हैं  
2. **Assessment** - हम व्यवहार्यता की पुष्टि करते हैं, ATLAS mapping और threat ID सौंपते हैं, risk level सत्यापित करते हैं  
3. **Documentation** - हम सुनिश्चित करते हैं कि सब कुछ सही प्रारूप में और पूर्ण हो  
4. **Merge** - threat model और visualization में जोड़ा जाता है  

## संसाधन

- [ATLAS Website](https://atlas.mitre.org/)
- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [OpenClaw Threat Model](./THREAT-MODEL-ATLAS.md)

## संपर्क

- **Security vulnerabilities:** रिपोर्टिंग निर्देशों के लिए हमारी [Trust page](https://trust.openclaw.ai) देखें  
- **Threat model से संबंधित प्रश्न:** [openclaw/trust](https://github.com/openclaw/trust/issues) पर एक issue खोलें  
- **सामान्य चर्चा:** Discord #security चैनल  

## मान्यता

Threat model में योगदान देने वालों को threat model acknowledgments, release notes, और महत्वपूर्ण योगदानों के लिए OpenClaw security hall of fame में मान्यता दी जाती है।

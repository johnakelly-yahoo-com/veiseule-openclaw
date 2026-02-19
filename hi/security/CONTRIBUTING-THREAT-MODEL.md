# OpenClaw Threat Model में योगदान

Thanks for helping make OpenClaw more secure. This threat model is a living document and we welcome contributions from anyone - you don't need to be a security expert.

## योगदान करने के तरीके

### एक Threat जोड़ें

Spotted an attack vector or risk we haven't covered? Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues) and describe it in your own words. You don't need to know any frameworks or fill in every field - just describe the scenario.

**शामिल करना उपयोगी होगा (लेकिन अनिवार्य नहीं):**

- आक्रमण परिदृश्य और उसका दुरुपयोग कैसे किया जा सकता है
- OpenClaw के कौन से भाग प्रभावित होते हैं (CLI, gateway, channels, ClawHub, MCP servers, आदि)
- आपके अनुसार इसकी गंभीरता (low / medium / high / critical)
- संबंधित शोध, CVEs, या वास्तविक उदाहरणों के लिंक

We'll handle the ATLAS mapping, threat IDs, and risk assessment during review. If you want to include those details, great - but it's not expected.

> **यह threat model में जोड़ने के लिए है, लाइव vulnerabilities की रिपोर्टिंग के लिए नहीं।** यदि आपको कोई exploitable vulnerability मिली है, तो responsible disclosure निर्देशों के लिए हमारी [Trust page](https://trust.openclaw.ai) देखें।

### एक Mitigation सुझाएँ

Have an idea for how to address an existing threat? Open an issue or PR referencing the threat. Useful mitigations are specific and actionable - for example, "per-sender rate limiting of 10 messages/minute at the gateway" is better than "implement rate limiting."

### एक Attack Chain प्रस्तावित करें

Attack chains show how multiple threats combine into a realistic attack scenario. If you see a dangerous combination, describe the steps and how an attacker would chain them together. A short narrative of how the attack unfolds in practice is more valuable than a formal template.

### मौजूदा सामग्री को ठीक या बेहतर बनाएँ

टाइपो, स्पष्टीकरण, पुरानी जानकारी, बेहतर उदाहरण — PR का स्वागत है, issue की आवश्यकता नहीं।

## हम क्या उपयोग करते हैं

### MITRE ATLAS

This threat model is built on [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), a framework designed specifically for AI/ML threats like prompt injection, tool misuse, and agent exploitation. You don't need to know ATLAS to contribute - we map submissions to the framework during review.

### खतरा आईडी

Each threat gets an ID like `T-EXEC-003`. The categories are:

| कोड     | श्रेणी                                         |
| ------- | ---------------------------------------------- |
| RECON   | Reconnaissance - जानकारी एकत्र करना            |
| ACCESS  | Initial access - प्रारंभिक प्रवेश प्राप्त करना |
| EXEC    | Execution - दुर्भावनापूर्ण क्रियाएँ चलाना      |
| PERSIST | Persistence - पहुँच बनाए रखना                  |
| EVADE   | Defense evasion - पहचान से बचना                |
| DISC    | Discovery - वातावरण के बारे में सीखना          |
| EXFIL   | Exfiltration - डेटा चोरी करना                  |
| IMPACT  | Impact - नुकसान या व्यवधान                     |

IDs are assigned by maintainers during review. You don't need to pick one.

### जोखिम स्तर

| स्तर      | अर्थ                                                          |
| --------- | ------------------------------------------------------------- |
| **गंभीर** | पूर्ण सिस्टम समझौता, या उच्च संभावना + गंभीर प्रभाव           |
| **उच्च**  | महत्वपूर्ण नुकसान की संभावना, या मध्यम संभावना + गंभीर प्रभाव |
| **मध्यम** | मध्यम जोखिम, या कम संभावना + उच्च प्रभाव                      |
| **Low**   | कम संभावना और सीमित प्रभाव                                    |

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


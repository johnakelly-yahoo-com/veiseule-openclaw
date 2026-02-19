# OpenClaw تھریٹ ماڈل میں حصہ لینا

Thanks for helping make OpenClaw more secure. This threat model is a living document and we welcome contributions from anyone - you don't need to be a security expert.

## حصہ لینے کے طریقے

### کوئی خطرہ (Threat) شامل کریں

Spotted an attack vector or risk we haven't covered? Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues) and describe it in your own words. You don't need to know any frameworks or fill in every field - just describe the scenario.

**درج ذیل معلومات مددگار ہو سکتی ہیں (لیکن لازمی نہیں):**

- حملے کا منظرنامہ اور اسے کیسے استعمال کیا جا سکتا ہے
- OpenClaw کے کون سے حصے متاثر ہوتے ہیں (CLI, gateway, channels, ClawHub, MCP servers, وغیرہ)
- آپ کے خیال میں اس کی شدت کیا ہے (low / medium / high / critical)
- متعلقہ تحقیق، CVEs، یا حقیقی دنیا کی مثالوں کے لنکس

We'll handle the ATLAS mapping, threat IDs, and risk assessment during review. If you want to include those details, great - but it's not expected.

> **یہ تھریٹ ماڈل میں اضافہ کرنے کے لیے ہے، براہِ راست کمزوریوں کی رپورٹنگ کے لیے نہیں۔** اگر آپ کو کوئی قابلِ استحصال کمزوری ملی ہے تو ذمہ دارانہ انکشاف کی ہدایات کے لیے ہمارا [Trust page](https://trust.openclaw.ai) دیکھیں۔

### کسی حفاظتی اقدام (Mitigation) کی تجویز دیں

Have an idea for how to address an existing threat? Open an issue or PR referencing the threat. Useful mitigations are specific and actionable - for example, "per-sender rate limiting of 10 messages/minute at the gateway" is better than "implement rate limiting."

### اٹیک چین (Attack Chain) کی تجویز دیں

Attack chains show how multiple threats combine into a realistic attack scenario. If you see a dangerous combination, describe the steps and how an attacker would chain them together. A short narrative of how the attack unfolds in practice is more valuable than a formal template.

### موجودہ مواد کی اصلاح یا بہتری

املا کی غلطیاں، وضاحتیں، پرانی معلومات، بہتر مثالیں — PR کا خیرمقدم ہے، ایشو کھولنا ضروری نہیں۔

## ہم کیا استعمال کرتے ہیں

### MITRE ATLAS

This threat model is built on [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), a framework designed specifically for AI/ML threats like prompt injection, tool misuse, and agent exploitation. You don't need to know ATLAS to contribute - we map submissions to the framework during review.

### خطرات کے شناختی نمبرز

ہر خطرے کو `T-EXEC-003` جیسی ایک ID دی جاتی ہے۔ The categories are:

| کوڈ     | زمرہ                                        |
| ------- | ------------------------------------------- |
| RECON   | Reconnaissance - معلومات جمع کرنا           |
| ACCESS  | Initial access - ابتدائی رسائی حاصل کرنا    |
| EXEC    | Execution - بدنیتی پر مبنی کارروائیاں چلانا |
| PERSIST | Persistence - رسائی برقرار رکھنا            |
| EVADE   | Defense evasion - شناخت سے بچنا             |
| DISC    | Discovery - ماحول کے بارے میں جاننا         |
| EXFIL   | Exfiltration - ڈیٹا چوری کرنا               |
| IMPACT  | Impact - نقصان یا خلل ڈالنا                 |

IDs are assigned by maintainers during review. You don't need to pick one.

### رسک لیولز

| سطح             | مطلب                                                   |
| --------------- | ------------------------------------------------------ |
| **نہایت سنگین** | مکمل سسٹم پر سمجھوتہ، یا زیادہ امکان + نہایت سنگین اثر |
| **سنگین**       | نمایاں نقصان کا امکان، یا درمیانہ امکان + سنگین اثر    |
| **درمیانہ**     | معتدل خطرہ، یا کم امکان + زیادہ اثر                    |
| **کم**          | کم امکان اور محدود اثر                                 |

اگر آپ کو رسک لیول کے بارے میں یقین نہیں، تو صرف اثرات بیان کریں — ہم اس کا جائزہ لے لیں گے۔

## جائزے کا عمل

1. **Triage** - ہم نئی submissions کا 48 گھنٹوں کے اندر جائزہ لیتے ہیں
2. **Assessment** - ہم عملی امکان کی تصدیق کرتے ہیں، ATLAS میپنگ اور تھریٹ ID تفویض کرتے ہیں، اور رسک لیول کی توثیق کرتے ہیں
3. **Documentation** - ہم یقینی بناتے ہیں کہ سب کچھ درست فارمیٹ اور مکمل ہے
4. **Merge** - تھریٹ ماڈل اور visualization میں شامل کر دیا جاتا ہے

## وسائل

- [ATLAS Website](https://atlas.mitre.org/)
- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [OpenClaw Threat Model](./THREAT-MODEL-ATLAS.md)

## رابطہ

- **Security vulnerabilities:** رپورٹنگ کی ہدایات کے لیے ہمارا [Trust page](https://trust.openclaw.ai) دیکھیں
- **Threat model questions:** [openclaw/trust](https://github.com/openclaw/trust/issues) پر ایشو کھولیں
- **General chat:** Discord #security چینل

## اعتراف

تھریٹ ماڈل میں حصہ لینے والوں کو تھریٹ ماڈل کے acknowledgments، ریلیز نوٹس، اور نمایاں خدمات پر OpenClaw سیکیورٹی ہال آف فیم میں تسلیم کیا جاتا ہے۔

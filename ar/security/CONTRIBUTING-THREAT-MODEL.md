# المساهمة في نموذج تهديدات OpenClaw

شكراً لمساعدتك في جعل OpenClaw أكثر أماناً. نموذج التهديد هذا وثيقة حيّة ونرحب بالمساهمات من أي شخص — لا تحتاج لأن تكون خبيراً أمنياً.

## طرق المساهمة

### إضافة تهديد

هل لاحظت متجه هجوم أو خطراً لم نقم بتغطيته؟ 31. افتح مشكلة على [openclaw/trust](https://github.com/openclaw/trust/issues) ووصفها بكلماتك الخاصة. لا تحتاج إلى معرفة أي أطر عمل أو ملء كل حقل — فقط صف السيناريو.

**مفيد تضمينه (لكن غير مطلوب):**

- سيناريو الهجوم وكيف يمكن استغلاله
- أي أجزاء من OpenClaw متأثرة (CLI، البوابة، القنوات، ClawHub، خوادم MCP، إلخ.)
- مدى شدته برأيك (منخفض / متوسط / مرتفع / حرج)
- أي روابط لأبحاث ذات صلة أو CVEs أو أمثلة من العالم الحقيقي

سنتولى مطابقة ATLAS ومعرّفات التهديد وتقييم المخاطر أثناء المراجعة. إذا رغبت في تضمين تلك التفاصيل، فهذا رائع — لكنه غير متوقع.

> **هذا مخصّص للإضافة إلى نموذج التهديد، وليس للإبلاغ عن ثغرات حية.** إذا عثرت على ثغرة قابلة للاستغلال، راجع [صفحة الثقة](https://trust.openclaw.ai) لدينا للحصول على إرشادات الإفصاح المسؤول.

### اقترح إجراء تخفيف

هل لديك فكرة لمعالجة تهديد قائم؟ 43. افتح مشكلة أو طلب سحب (PR) مع الإشارة إلى التهديد. إجراءات التخفيف المفيدة تكون محددة وقابلة للتنفيذ — على سبيل المثال، "تحديد معدل لكل مُرسِل بواقع 10 رسائل/دقيقة عند البوابة" أفضل من "تطبيق تحديد المعدل".

### اقترح سلسلة هجوم

تُظهر سلاسل الهجوم كيف تتكامل عدة تهديدات لتشكّل سيناريو هجوم واقعي. إذا رأيت توليفة خطيرة، صف الخطوات وكيف سيقوم المهاجم بربطها معاً. سردٌ قصير لكيفية تطور الهجوم عملياً أكثر قيمة من قالب رسمي.

### إصلاح أو تحسين المحتوى الحالي

أخطاء مطبعية، توضيحات، معلومات قديمة، أمثلة أفضل — نرحب بطلبات السحب، ولا حاجة لفتح مشكلة.

## ما الذي نستخدمه

### MITRE ATLAS

يعتمد نموذج التهديد هذا على [MITRE ATLAS](https://atlas.mitre.org/) (مشهد التهديدات العدائية لأنظمة الذكاء الاصطناعي)، وهو إطار عمل مُصمَّم خصيصًا لتهديدات الذكاء الاصطناعي/تعلّم الآلة مثل حقن الأوامر (prompt injection)، وإساءة استخدام الأدوات، واستغلال الوكلاء. لا تحتاج إلى معرفة ATLAS للمساهمة — نقوم بمواءمة المشاركات مع الإطار أثناء المراجعة.

### معرّفات التهديد

يحصل كل تهديد على معرّف مثل `T-EXEC-003`. الفئات هي:

| الشيفرة | الفئة                                      |
| ------- | ------------------------------------------ |
| RECON   | الاستطلاع - جمع المعلومات                  |
| ACCESS  | الوصول الأولي - الحصول على دخول            |
| EXEC    | التنفيذ - تشغيل إجراءات خبيثة              |
| PERSIST | الاستمرارية - الحفاظ على الوصول            |
| EVADE   | تجنّب الدفاع - تفادي الاكتشاف              |
| DISC    | Discovery - learning about the environment |
| EXFIL   | Exfiltration - stealing data               |
| IMPACT  | Impact - damage or disruption              |

IDs are assigned by maintainers during review. You don't need to pick one.

### Risk Levels

| Level        | Meaning                                                           |
| ------------ | ----------------------------------------------------------------- |
| **Critical** | Full system compromise, or high likelihood + critical impact      |
| **High**     | Significant damage likely, or medium likelihood + critical impact |
| **Medium**   | Moderate risk, or low likelihood + high impact                    |
| **Low**      | Unlikely and limited impact                                       |

If you're unsure about the risk level, just describe the impact and we'll assess it.

## Review Process

1. **Triage** - We review new submissions within 48 hours
2. **Assessment** - We verify feasibility, assign ATLAS mapping and threat ID, validate risk level
3. **Documentation** - We ensure everything is formatted and complete
4. **Merge** - Added to the threat model and visualization

## Resources

- [ATLAS Website](https://atlas.mitre.org/)
- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [OpenClaw Threat Model](./THREAT-MODEL-ATLAS.md)

## Contact

- **Security vulnerabilities:** See our [Trust page](https://trust.openclaw.ai) for reporting instructions
- **Threat model questions:** Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues)
- **General chat:** Discord #security channel

## Recognition

Contributors to the threat model are recognized in the threat model acknowledgments, release notes, and the OpenClaw security hall of fame for significant contributions.


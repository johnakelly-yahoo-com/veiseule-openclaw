# OpenClaw Threat Model v1.0

## MITRE ATLAS Framework

**Version:** 1.0-draft
**Last Updated:** 2026-02-04
**Methodology:** MITRE ATLAS + Data Flow Diagrams
**Framework:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Framework Attribution

This threat model is built on [MITRE ATLAS](https://atlas.mitre.org/), the industry-standard framework for documenting adversarial threats to AI/ML systems. ATLAS is maintained by [MITRE](https://www.mitre.org/) in collaboration with the AI security community.

**Key ATLAS Resources:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contributing to ATLAS](https://atlas.mitre.org/resources/contribute)

### Contributing to This Threat Model

This is a living document maintained by the OpenClaw community. See [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) for guidelines on contributing:

- Reporting new threats
- Updating existing threats
- Proposing attack chains
- Suggesting mitigations

---

## 1. Introduction

### 1.1 Purpose

This threat model documents adversarial threats to the OpenClaw AI agent platform and ClawHub skill marketplace, using the MITRE ATLAS framework designed specifically for AI/ML systems.

### 1.2 Scope

| المكون                 | Included | الملاحظات                                                        |
| ---------------------- | -------- | ---------------------------------------------------------------- |
| OpenClaw Agent Runtime | نعم      | Core agent execution, tool calls, sessions                       |
| Gateway                | نعم      | Authentication, routing, channel integration                     |
| Channel Integrations   | نعم      | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| ClawHub Marketplace    | نعم      | Skill publishing, moderation, distribution                       |
| MCP Servers            | نعم      | External tool providers                                          |
| User Devices           | Partial  | Mobile apps, desktop clients                                     |

### 1.3 Out of Scope

Nothing is explicitly out of scope for this threat model.

---

## 2. System Architecture

### 2.1 Trust Boundaries

```
┌─────────────────────────────────────────────────────────────────┐
│                    UNTRUSTED ZONE                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 1: Channel Access                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Device Pairing (30s grace period)                      │   │
│  │  • AllowFrom / AllowList validation                       │   │
│  │  • Token/Password/Tailscale auth                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 2: Session Isolation              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENT SESSIONS                          │   │
│  │  • Session key = agent:channel:peer                       │   │
│  │  • Tool policies per agent                                │   │
│  │  • Transcript logging                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 3: Tool Execution                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  EXECUTION SANDBOX                        │   │
│  │  • Docker sandbox OR Host (exec-approvals)                │   │
│  │  • Node remote execution                                  │   │
│  │  • SSRF protection (DNS pinning + IP blocking)            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 4: External Content               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              FETCHED URLs / EMAILS / WEBHOOKS             │   │
│  │  • External content wrapping (XML tags)                   │   │
│  │  • Security notice injection                              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 5: Supply Chain                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Skill publishing (semver, SKILL.md required)           │   │
│  │  • Pattern-based moderation flags                         │   │
│  │  • VirusTotal scanning (coming soon)                      │   │
│  │  • GitHub account age verification                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Flows

| Flow | Source  | Destination | Data                                    | Protection        |
| ---- | ------- | ----------- | --------------------------------------- | ----------------- |
| F1   | قناة    | Gateway     | رسائل المستخدم                          | TLS، AllowFrom    |
| F2   | Gateway | وكيل        | رسائل مُوجَّهة                          | Session isolation |
| F3   | وكيل    | الأدوات     | استدعاءات الأدوات                       | فرض السياسات      |
| F4   | وكيل    | خارجي       | web_fetch requests | حظر SSRF          |
| F5   | ClawHub | وكيل        | شفرة المهارة                            | الإشراف، الفحص    |
| F6   | وكيل    | قناة        | الاستجابات                              | تصفية المخرجات    |

---

## 3. تحليل التهديدات وفق تكتيك ATLAS

### 3.1 الاستطلاع (AML.TA0002)

#### T-RECON-001: اكتشاف نقاط نهاية الوكيل

| Attribute               | القيمة                                                        |
| ----------------------- | ------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - المسح النشط                       |
| **Description**         | يقوم المهاجم بمسح نقاط نهاية بوابة OpenClaw المكشوفة          |
| **Attack Vector**       | مسح الشبكة، استعلامات shodan، تعداد DNS                       |
| **Affected Components** | البوابة، نقاط نهاية واجهة برمجة التطبيقات المكشوفة            |
| **Current Mitigations** | خيار مصادقة Tailscale، الربط على loopback افتراضيًا           |
| **Residual Risk**       | متوسط - البوابات العامة قابلة للاكتشاف                        |
| **توصيات**              | توثيق النشر الآمن، إضافة تحديد المعدل على نقاط نهاية الاكتشاف |

#### T-RECON-002: استكشاف تكامل القنوات

| Attribute               | القيمة                                                                          |
| ----------------------- | ------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - المسح النشط                                         |
| **Description**         | يقوم المهاجم باستكشاف قنوات المراسلة لتحديد الحسابات المُدارة بالذكاء الاصطناعي |
| **Attack Vector**       | إرسال رسائل اختبار، وملاحظة أنماط الاستجابة                                     |
| **Affected Components** | جميع تكاملات القنوات                                                            |
| **Current Mitigations** | None specific                                                                   |
| **Residual Risk**       | Low - Limited value from discovery alone                                        |
| **توصيات**              | النظر في عشوائية توقيت الاستجابة                                                |

---

### 3.2 الوصول الأولي (AML.TA0004)

#### T-ACCESS-001: اعتراض رمز الإقران

| Attribute               | القيمة                                                      |
| ----------------------- | ----------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access   |
| **Description**         | يعترض المهاجم رمز الإقران خلال فترة السماح البالغة 30 ثانية |
| **Attack Vector**       | التلصص البصري، التنصت على الشبكة، الهندسة الاجتماعية        |
| **Affected Components** | نظام إقران الأجهزة                                          |
| **Current Mitigations** | انتهاء الصلاحية خلال 30 ثانية، إرسال الرموز عبر قناة موجودة |
| **Residual Risk**       | متوسطة - فترة السماح قابلة للاستغلال                        |
| **توصيات**              | تقليل فترة السماح، إضافة خطوة تأكيد                         |

#### T-ACCESS-002: انتحال AllowFrom

| Attribute               | القيمة                                                      |
| ----------------------- | ----------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access   |
| **Description**         | يقوم المهاجم بانتحال هوية المُرسِل المسموح به في القناة     |
| **Attack Vector**       | يعتمد على القناة - انتحال رقم الهاتف، انتحال اسم المستخدم   |
| **Affected Components** | التحقق من AllowFrom لكل قناة                                |
| **Current Mitigations** | التحقق من الهوية الخاص بكل قناة                             |
| **Residual Risk**       | متوسطة - بعض القنوات عرضة للانتحال                          |
| **توصيات**              | توثيق المخاطر الخاصة بكل قناة، إضافة تحقق تشفيري حيثما أمكن |

#### T-ACCESS-003: سرقة الرموز المميِّزة

| Attribute               | القيمة                                                                                          |
| ----------------------- | ----------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                                       |
| **Description**         | يسرق المهاجم رموز المصادقة من ملفات الإعداد                                                     |
| **Attack Vector**       | برمجيات خبيثة، وصول غير مصرح به إلى الجهاز، تعرّض نسخ إعدادات احتياطية                          |
| **Affected Components** | ~/.openclaw/credentials/، تخزين الإعدادات                       |
| **Current Mitigations** | أذونات الملفات                                                                                  |
| **Residual Risk**       | مرتفع - يتم تخزين الرموز المميزة بنص صريح                                                       |
| **توصيات**              | 2. تنفيذ تشفير الرموز المميزة أثناء التخزين، وإضافة تدوير للرموز المميزة |

---

### 3.3 التنفيذ (AML.TA0005)

#### 4. T-EXEC-001: حقن الموجّه المباشر

| Attribute               | القيمة                                                                                              |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - حقن موجّه نموذج اللغة الكبير: مباشر |
| **Description**         | يرسل المهاجم موجّهات مُعدّة خصيصًا للتلاعب بسلوك الوكيل                                             |
| **Attack Vector**       | رسائل القناة التي تحتوي على تعليمات عدائية                                                          |
| **Affected Components** | 13. وكيل نموذج اللغة الكبير، جميع أسطح الإدخال                               |
| **Current Mitigations** | Pattern detection, external content wrapping                                                        |
| **Residual Risk**       | حرجة - كشف فقط دون حظر؛ الهجمات المتقدمة تتجاوز ذلك                                                 |
| **توصيات**              | Implement multi-layer defense, output validation, user confirmation for sensitive actions           |

#### T-EXEC-002: حقن الموجّه غير المباشر

| Attribute               | القيمة                                                                                                  |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - حقن موجّه نموذج اللغة الكبير: غير مباشر |
| **Description**         | يضمّن المهاجم تعليمات خبيثة في المحتوى الذي يتم جلبه                                                    |
| **Attack Vector**       | عناوين URL خبيثة، رسائل بريد إلكتروني مسمومة، Webhooks مخترقة                                           |
| **Affected Components** | web_fetch، استيعاب البريد الإلكتروني، مصادر البيانات الخارجية                      |
| **Current Mitigations** | تغليف المحتوى بعلامات XML وإشعار أمني                                                                   |
| **Residual Risk**       | 32. مرتفع - قد يتجاهل نموذج اللغة الكبير تعليمات التغليف                         |
| **توصيات**              | تنفيذ تنقية المحتوى، فصل سياقات التنفيذ                                                                 |

#### T-EXEC-003: Tool Argument Injection

| Attribute               | القيمة                                                                                              |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - حقن موجّه نموذج اللغة الكبير: مباشر |
| **Description**         | 39. يتلاعب المهاجم بوسائط الأداة عبر حقن الموجّه                             |
| **Attack Vector**       | موجّهات مُعدّة تؤثر على قيم معاملات الأداة                                                          |
| **Affected Components** | جميع استدعاءات الأدوات                                                                              |
| **Current Mitigations** | موافقات التنفيذ للأوامر الخطِرة                                                                     |
| **Residual Risk**       | مرتفع - يعتمد على حُكم المستخدم                                                                     |
| **توصيات**              | 48. تنفيذ التحقق من الوسائط، واستدعاءات أدوات مُعلّمة بالمعاملات             |

#### T-EXEC-004: تجاوز موافقة التنفيذ

| Attribute               | القيمة                                                     |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data         |
| **Description**         | Attacker crafts commands that bypass approval allowlist    |
| **Attack Vector**       | Command obfuscation, alias exploitation, path manipulation |
| **Affected Components** | exec-approvals.ts, command allowlist       |
| **Current Mitigations** | Allowlist + ask mode                                       |
| **Residual Risk**       | High - No command sanitization                             |
| **توصيات**              | Implement command normalization, expand blocklist          |

---

### 3.4 Persistence (AML.TA0006)

#### T-PERSIST-001: Malicious Skill Installation

| Attribute               | القيمة                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker publishes malicious skill to ClawHub                                                        |
| **Attack Vector**       | Create account, publish skill with hidden malicious code                                             |
| **Affected Components** | ClawHub, skill loading, agent execution                                                              |
| **Current Mitigations** | GitHub account age verification, pattern-based moderation flags                                      |
| **Residual Risk**       | Critical - No sandboxing, limited review                                                             |
| **توصيات**              | VirusTotal integration (in progress), skill sandboxing, community review          |

#### T-PERSIST-002: Skill Update Poisoning

| Attribute               | القيمة                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker compromises popular skill and pushes malicious update                                       |
| **Attack Vector**       | Account compromise, social engineering of skill owner                                                |
| **Affected Components** | ClawHub versioning, auto-update flows                                                                |
| **Current Mitigations** | Version fingerprinting                                                                               |
| **Residual Risk**       | High - Auto-updates may pull malicious versions                                                      |
| **توصيات**              | Implement update signing, rollback capability, version pinning                                       |

#### T-PERSIST-003: Agent Configuration Tampering

| Attribute               | القيمة                                                                                        |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Supply Chain Compromise: Data |
| **Description**         | Attacker modifies agent configuration to persist access                                       |
| **Attack Vector**       | Config file modification, settings injection                                                  |
| **Affected Components** | Agent config, tool policies                                                                   |
| **Current Mitigations** | أذونات الملفات                                                                                |
| **Residual Risk**       | Medium - Requires local access                                                                |
| **توصيات**              | Config integrity verification, audit logging for config changes                               |

---

### 3.5 Defense Evasion (AML.TA0007)

#### T-EVADE-001: Moderation Pattern Bypass

| Attribute               | القيمة                                                                                    |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                        |
| **Description**         | Attacker crafts skill content to evade moderation patterns                                |
| **Attack Vector**       | Unicode homoglyphs, encoding tricks, dynamic loading                                      |
| **Affected Components** | ClawHub moderation.ts                                                     |
| **Current Mitigations** | Pattern-based FLAG_RULES                                             |
| **Residual Risk**       | High - Simple regex easily bypassed                                                       |
| **توصيات**              | Add behavioral analysis (VirusTotal Code Insight), AST-based detection |

#### T-EVADE-002: Content Wrapper Escape

| Attribute               | القيمة                                                    |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data        |
| **Description**         | Attacker crafts content that escapes XML wrapper context  |
| **Attack Vector**       | Tag manipulation, context confusion, instruction override |
| **Affected Components** | External content wrapping                                 |
| **Current Mitigations** | XML tags + security notice                                |
| **Residual Risk**       | Medium - Novel escapes discovered regularly               |
| **توصيات**              | Multiple wrapper layers, output-side validation           |

---

### 3.6 Discovery (AML.TA0008)

#### T-DISC-001: Tool Enumeration

| Attribute               | القيمة                                                    |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker enumerates available tools through prompting     |
| **Attack Vector**       | "What tools do you have?" style queries                   |
| **Affected Components** | Agent tool registry                                       |
| **Current Mitigations** | None specific                                             |
| **Residual Risk**       | Low - Tools generally documented                          |
| **توصيات**              | Consider tool visibility controls                         |

#### T-DISC-002: Session Data Extraction

| Attribute               | القيمة                                                    |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker extracts sensitive data from session context     |
| **Attack Vector**       | "What did we discuss?" queries, context probing           |
| **Affected Components** | Session transcripts, context window                       |
| **Current Mitigations** | Session isolation per sender                              |
| **Residual Risk**       | Medium - Within-session data accessible                   |
| **توصيات**              | Implement sensitive data redaction in context             |

---

### 3.7 Collection & Exfiltration (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Data Theft via web_fetch

| Attribute               | القيمة                                                                 |
| ----------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                 |
| **Description**         | Attacker exfiltrates data by instructing agent to send to external URL |
| **Attack Vector**       | Prompt injection causing agent to POST data to attacker server         |
| **Affected Components** | web_fetch tool                                    |
| **Current Mitigations** | SSRF blocking for internal networks                                    |
| **Residual Risk**       | High - External URLs permitted                                         |
| **توصيات**              | Implement URL allowlisting, data classification awareness              |

#### T-EXFIL-002: Unauthorized Message Sending

| Attribute               | القيمة                                                           |
| ----------------------- | ---------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                           |
| **Description**         | Attacker causes agent to send messages containing sensitive data |
| **Attack Vector**       | Prompt injection causing agent to message attacker               |
| **Affected Components** | Message tool, channel integrations                               |
| **Current Mitigations** | Outbound messaging gating                                        |
| **Residual Risk**       | Medium - Gating may be bypassed                                  |
| **توصيات**              | Require explicit confirmation for new recipients                 |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | القيمة                                                  |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                  |
| **Description**         | Malicious skill harvests credentials from agent context |
| **Attack Vector**       | Skill code reads environment variables, config files    |
| **Affected Components** | Skill execution environment                             |
| **Current Mitigations** | None specific to skills                                 |
| **Residual Risk**       | Critical - Skills run with agent privileges             |
| **توصيات**              | Skill sandboxing, credential isolation                  |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001: Unauthorized Command Execution

| Attribute               | القيمة                                               |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker executes arbitrary commands on user system  |
| **Attack Vector**       | Prompt injection combined with exec approval bypass  |
| **Affected Components** | Bash tool, command execution                         |
| **Current Mitigations** | Exec approvals, Docker sandbox option                |
| **Residual Risk**       | Critical - Host execution without sandbox            |
| **توصيات**              | Default to sandbox, improve approval UX              |

#### T-IMPACT-002: Resource Exhaustion (DoS)

| Attribute               | القيمة                                               |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker exhausts API credits or compute resources   |
| **Attack Vector**       | Automated message flooding, expensive tool calls     |
| **Affected Components** | Gateway, agent sessions, API provider                |
| **Current Mitigations** | None                                                 |
| **Residual Risk**       | High - No rate limiting                              |
| **توصيات**              | Implement per-sender rate limits, cost budgets       |

#### T-IMPACT-003: Reputation Damage

| Attribute               | القيمة                                               |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | يتسبب المهاجم في جعل الوكيل يرسل محتوى ضارًا/مسيئًا  |
| **Attack Vector**       | حقن المطالبات الذي يؤدي إلى استجابات غير لائقة       |
| **Affected Components** | توليد المخرجات، مراسلة القنوات                       |
| **Current Mitigations** | سياسات محتوى مزود نماذج اللغة الكبيرة                |
| **Residual Risk**       | متوسطة - فلاتر المزود غير مثالية                     |
| **توصيات**              | طبقة تصفية المخرجات، عناصر تحكم المستخدم             |

---

## 4. T-EXEC-001: حقن الموجّه المباشر

### 4.1 ضوابط الأمان الحالية

| 15. الضبط        | التنفيذ                                                          | الفعالية                                    |
| --------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------- |
| عمر حساب GitHub                         | `requireGitHubAccountAge()`                                      | متوسطة - ترفع العتبة أمام المهاجمين الجدد   |
| تعقيم المسار                            | `sanitizePath()`                                                 | High - Prevents path traversal              |
| التحقق من نوع الملف                     | `isTextFile()`                                                   | متوسطة - ملفات نصية فقط، لكنها قد تظل خبيثة |
| حدود الحجم                              | إجمالي الحزمة 50MB                                               | عالية - تمنع استنزاف الموارد                |
| ملف SKILL.md مطلوب      | 30. ملف README إلزامي                     | قيمة أمنية منخفضة - معلوماتية فقط           |
| إشراف الأنماط                           | FLAG_RULES في moderation.ts | منخفضة - يسهل تجاوزها                       |
| 35. حالة الإشراف | الحقل `moderationStatus`                                         | متوسطة - المراجعة اليدوية ممكنة             |

### 4.2 أنماط أعلام الإشراف

الأنماط الحالية في `moderation.ts`:

```javascript
40. // Known-bad identifiers
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i

// Suspicious keywords
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+|\s*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i
```

**القيود:**

- يتحقق فقط من slug وdisplayName والملخص وfrontmatter والبيانات الوصفية ومسارات الملفات
- لا يحلل محتوى كود المهارة الفعلي
- تعبيرات regex بسيطة يسهل تجاوزها عبر الإخفاء
- لا يوجد تحليل سلوكي

### 4.3 التحسينات المخطط لها

| التحسين              | الحالة                                                | الأثر                                                                |
| -------------------- | ----------------------------------------------------- | -------------------------------------------------------------------- |
| تكامل VirusTotal     | 50. قيد التنفيذ                | مرتفع - تحليل سلوكي لرؤية الشيفرة                                    |
| الإبلاغ المجتمعي     | جزئي (`skillReports` table exists) | Medium                                                               |
| تسجيل عمليات التدقيق | جزئي (`auditLogs` table exists)    | Medium                                                               |
| نظام الشارات         | قيد التنفيذ                                           | متوسط - `highlighted`, `official`, `deprecated`, `redactionApproved` |

---

## 5. مصفوفة المخاطر

### 5.1 الاحتمالية مقابل التأثير

| معرّف التهديد | الاحتمالية | التأثير | Risk Level   | الأولوية |
| ------------- | ---------- | ------- | ------------ | -------- |
| T-EXEC-001    | High       | حرج     | **Critical** | P0       |
| T-PERSIST-001 | High       | حرج     | **Critical** | P0       |
| T-EXFIL-003   | Medium     | حرج     | **Critical** | P0       |
| T-IMPACT-001  | Medium     | حرج     | **High**     | P1       |
| T-EXEC-002    | High       | High    | **High**     | P1       |
| T-EXEC-004    | Medium     | High    | **High**     | P1       |
| T-ACCESS-003  | Medium     | High    | **High**     | P1       |
| T-EXFIL-001   | Medium     | High    | **High**     | P1       |
| T-IMPACT-002  | High       | Medium  | **High**     | P1       |
| T-EVADE-001   | High       | Medium  | **Medium**   | P2       |
| T-ACCESS-001  | Low        | High    | **Medium**   | P2       |
| T-ACCESS-002  | Low        | High    | **Medium**   | P2       |
| T-PERSIST-002 | Low        | High    | **Medium**   | P2       |

### 5.2 Critical Path Attack Chains

**Attack Chain 1: Skill-Based Data Theft**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(Publish malicious skill) → (Evade moderation) → (Harvest credentials)
```

**Attack Chain 2: Prompt Injection to RCE**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Inject prompt) → (Bypass exec approval) → (Execute commands)
```

**Attack Chain 3: Indirect Injection via Fetched Content**

```
T-EXEC-002 → T-EXFIL-001 → External exfiltration
(Poison URL content) → (Agent fetches & follows instructions) → (Data sent to attacker)
```

---

## 6. Recommendations Summary

### 6.1 Immediate (P0)

| ID    | Recommendation                              | Addresses                  |
| ----- | ------------------------------------------- | -------------------------- |
| R-001 | Complete VirusTotal integration             | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implement skill sandboxing                  | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Add output validation for sensitive actions | T-EXEC-001, T-EXEC-002     |

### 6.2 Short-term (P1)

| ID    | Recommendation                                                | Addresses    |
| ----- | ------------------------------------------------------------- | ------------ |
| R-004 | Implement rate limiting                                       | T-IMPACT-002 |
| R-005 | Add token encryption at rest                                  | T-ACCESS-003 |
| R-006 | Improve exec approval UX and validation                       | T-EXEC-004   |
| R-007 | Implement URL allowlisting for web_fetch | T-EXFIL-001  |

### 6.3 Medium-term (P2)

| ID    | Recommendation                                        | Addresses     |
| ----- | ----------------------------------------------------- | ------------- |
| R-008 | Add cryptographic channel verification where possible | T-ACCESS-002  |
| R-009 | Implement config integrity verification               | T-PERSIST-003 |
| R-010 | Add update signing and version pinning                | T-PERSIST-002 |

---

## 7. Appendices

### 7.1 ATLAS Technique Mapping

| ATLAS ID                                      | Technique Name                                 | OpenClaw Threats                                                 |
| --------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Active Scanning                                | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Collection                                     | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Supply Chain: AI Software      | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Supply Chain: Data             | T-PERSIST-003                                                    |
| AML.T0031                     | Erode AI Model Integrity                       | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | AI Model Inference API Access                  | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Craft Adversarial Data                         | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM Prompt Injection: Direct   | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM Prompt Injection: Indirect | T-EXEC-002                                                       |

### 7.2 Key Security Files

| المسار                              | الغرض                       | Risk Level   |
| ----------------------------------- | --------------------------- | ------------ |
| `src/infra/exec-approvals.ts`       | Command approval logic      | **Critical** |
| `src/gateway/auth.ts`               | Gateway authentication      | **Critical** |
| `src/web/inbound/access-control.ts` | Channel access control      | **Critical** |
| `src/infra/net/ssrf.ts`             | SSRF protection             | **Critical** |
| `src/security/external-content.ts`  | Prompt injection mitigation | **Critical** |
| `src/agents/sandbox/tool-policy.ts` | Tool policy enforcement     | **Critical** |
| `convex/lib/moderation.ts`          | ClawHub moderation          | **High**     |
| `convex/lib/skillPublish.ts`        | Skill publishing flow       | **High**     |
| `src/routing/resolve-route.ts`      | Session isolation           | **Medium**   |

### 7.3 Glossary

| Term              | Definition                                          |
| ----------------- | --------------------------------------------------- |
| **ATLAS**         | MITRE's Adversarial Threat Landscape for AI Systems |
| **ClawHub**       | OpenClaw's skill marketplace                        |
| **Gateway**       | OpenClaw's message routing and authentication layer |
| **MCP**           | بروتوكول سياق النموذج - واجهة موفّر الأدوات         |
| **حقن التوجيهات** | هجوم يتم فيه تضمين تعليمات خبيثة داخل الإدخال       |
| **Skill**         | امتداد قابل للتنزيل لوكلاء OpenClaw                 |
| **SSRF**          | تزوير الطلبات من جانب الخادم                        |

---

_هذا نموذج تهديد حي ومتجدد. أبلِغ عن مشكلات الأمان إلى security@openclaw.ai_


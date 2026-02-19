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

| Bileşen                | Included | Notes                                                            |
| ---------------------- | -------- | ---------------------------------------------------------------- |
| OpenClaw Agent Runtime | Evet     | Core agent execution, tool calls, sessions                       |
| Gateway                | Evet     | Authentication, routing, channel integration                     |
| Channel Integrations   | Evet     | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| ClawHub Marketplace    | Evet     | Skill publishing, moderation, distribution                       |
| MCP Servers            | Evet     | External tool providers                                          |
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

| Flow | Source  | Destination | Data                                    | Protection           |
| ---- | ------- | ----------- | --------------------------------------- | -------------------- |
| F1   | Channel | Gateway     | User messages                           | TLS, AllowFrom       |
| F2   | Gateway | Agent       | Routed messages                         | Session isolation    |
| F3   | Agent   | Araçlar     | Tool invocations                        | Policy enforcement   |
| F4   | Agent   | External    | web_fetch requests | SSRF blocking        |
| F5   | ClawHub | Agent       | Skill code                              | Moderation, scanning |
| F6   | Agent   | Channel     | Yanıtlar                                | Output filtering     |

---

## 3. Threat Analysis by ATLAS Tactic

### 3.1 Reconnaissance (AML.TA0002)

#### T-RECON-001: Agent Endpoint Discovery

| Attribute               | Değer                                                                |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                          |
| **Description**         | Attacker scans for exposed OpenClaw gateway endpoints                |
| **Attack Vector**       | Network scanning, shodan queries, DNS enumeration                    |
| **Affected Components** | Gateway, exposed API endpoints                                       |
| **Current Mitigations** | Tailscale auth option, bind to loopback by default                   |
| **Residual Risk**       | Medium - Public gateways discoverable                                |
| **Öneriler**            | Document secure deployment, add rate limiting on discovery endpoints |

#### T-RECON-002: Channel Integration Probing

| Attribute               | Değer                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Active Scanning                        |
| **Description**         | Attacker probes messaging channels to identify AI-managed accounts |
| **Attack Vector**       | Sending test messages, observing response patterns                 |
| **Affected Components** | Tüm kanal entegrasyonları                                          |
| **Current Mitigations** | Özel bir önlem yok                                                 |
| **Residual Risk**       | Düşük - Yalnızca keşiften elde edilen değer sınırlı                |
| **Öneriler**            | Yanıt zamanlaması için rastgeleleştirme düşünün                    |

---

### 3.2 İlk Erişim (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | Değer                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Omuz sörfü, ağ dinleme, sosyal mühendislik                |
| **Affected Components** | Cihaz eşleştirme sistemi                                  |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Orta - Ek süre istismar edilebilir                        |
| **Öneriler**            | Ek süreyi azaltın, bir onay adımı ekleyin                 |

#### T-ACCESS-002: AllowFrom Sahteciliği

| Attribute               | Değer                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                            |
| **Description**         | Saldırgan, kanalda izin verilen gönderen kimliğini taklit eder                       |
| **Attack Vector**       | Kanala bağlıdır - telefon numarası sahteciliği, kullanıcı adı taklidi                |
| **Affected Components** | Kanal başına AllowFrom doğrulaması                                                   |
| **Current Mitigations** | Kanala özgü kimlik doğrulaması                                                       |
| **Residual Risk**       | Orta - Bazı kanallar sahteciliğe açıktır                                             |
| **Öneriler**            | Kanala özgü riskleri belgeleyin, mümkün olan yerlerde kriptografik doğrulama ekleyin |

#### T-ACCESS-003: Belirteç Hırsızlığı

| Attribute               | Değer                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                            |
| **Description**         | Saldırgan, yapılandırma dosyalarından kimlik doğrulama belirteçlerini çalar          |
| **Attack Vector**       | Kötü amaçlı yazılım, yetkisiz cihaz erişimi, yapılandırma yedeklerinin açığa çıkması |
| **Affected Components** | ~/.openclaw/credentials/, yapılandırma depolaması    |
| **Current Mitigations** | Dosya izinleri                                                                       |
| **Residual Risk**       | Yüksek - Tokenlar düz metin olarak saklanıyor                                        |
| **Öneriler**            | Depolamada token şifrelemesini uygulayın, token rotasyonu ekleyin                    |

---

### 3.3 Yürütme (AML.TA0005)

#### T-EXEC-001: Doğrudan Prompt Enjeksiyonu

| Attribute               | Değer                                                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Enjeksiyonu: Doğrudan |
| **Description**         | Attacker sends crafted prompts to manipulate agent behavior                                      |
| **Attack Vector**       | Düşmanca talimatlar içeren kanal mesajları                                                       |
| **Affected Components** | Ajan LLM, tüm girdi yüzeyleri                                                                    |
| **Current Mitigations** | Desen tespiti, harici içerik sarmalama                                                           |
| **Residual Risk**       | Kritik - Yalnızca tespit, engelleme yok; gelişmiş saldırılar aşabilir                            |
| **Öneriler**            | Çok katmanlı savunma uygulayın, çıktı doğrulaması, hassas eylemler için kullanıcı onayı          |

#### T-EXEC-002: Dolaylı Prompt Enjeksiyonu

| Attribute               | Değer                                                                                           |
| ----------------------- | ----------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - LLM Prompt Enjeksiyonu: Dolaylı |
| **Description**         | Saldırgan, alınan içerik içine kötü amaçlı talimatlar gömer                                     |
| **Attack Vector**       | Kötü amaçlı URL’ler, zehirlenmiş e-postalar, ele geçirilmiş webhook’lar                         |
| **Affected Components** | web_fetch, e-posta alımı, harici veri kaynakları                           |
| **Current Mitigations** | XML etiketleri ve güvenlik bildirimi ile içerik sarmalama                                       |
| **Residual Risk**       | Yüksek - LLM sarmalayıcı talimatları yok sayabilir                                              |
| **Öneriler**            | İçerik sanitizasyonu uygulayın, ayrı yürütme bağlamları                                         |

#### T-EXEC-003: Araç Argümanı Enjeksiyonu

| Attribute               | Değer                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Enjeksiyonu: Doğrudan  |
| **Description**         | 39. Saldırgan, prompt enjeksiyonu yoluyla araç argümanlarını manipüle eder |
| **Attack Vector**       | Araç parametre değerlerini etkileyen hazırlanmış promptlar                                        |
| **Affected Components** | Tüm araç çağrıları                                                                                |
| **Current Mitigations** | Tehlikeli komutlar için yürütme onayları                                                          |
| **Residual Risk**       | Yüksek - Kullanıcı muhakemesine dayanır                                                           |
| **Öneriler**            | Argüman doğrulaması uygulayın, parametreli araç çağrıları                                         |

#### T-EXEC-004: Yürütme Onayı Atlama

| Attribute               | Değer                                                      |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data         |
| **Description**         | Attacker crafts commands that bypass approval allowlist    |
| **Attack Vector**       | Command obfuscation, alias exploitation, path manipulation |
| **Affected Components** | exec-approvals.ts, command allowlist       |
| **Current Mitigations** | Allowlist + ask mode                                       |
| **Residual Risk**       | High - No command sanitization                             |
| **Öneriler**            | Implement command normalization, expand blocklist          |

---

### 3.4 Persistence (AML.TA0006)

#### T-PERSIST-001: Malicious Skill Installation

| Attribute               | Değer                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker publishes malicious skill to ClawHub                                                        |
| **Attack Vector**       | Create account, publish skill with hidden malicious code                                             |
| **Affected Components** | ClawHub, skill loading, agent execution                                                              |
| **Current Mitigations** | GitHub account age verification, pattern-based moderation flags                                      |
| **Residual Risk**       | Critical - No sandboxing, limited review                                                             |
| **Öneriler**            | VirusTotal integration (in progress), skill sandboxing, community review          |

#### T-PERSIST-002: Skill Update Poisoning

| Attribute               | Değer                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker compromises popular skill and pushes malicious update                                       |
| **Attack Vector**       | Account compromise, social engineering of skill owner                                                |
| **Affected Components** | ClawHub versioning, auto-update flows                                                                |
| **Current Mitigations** | Version fingerprinting                                                                               |
| **Residual Risk**       | High - Auto-updates may pull malicious versions                                                      |
| **Öneriler**            | Implement update signing, rollback capability, version pinning                                       |

#### T-PERSIST-003: Agent Configuration Tampering

| Attribute               | Değer                                                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Supply Chain Compromise: Data |
| **Description**         | Attacker modifies agent configuration to persist access                                       |
| **Attack Vector**       | Yapılandırma dosyası değişikliği, ayar enjeksiyonu                                            |
| **Affected Components** | Ajan yapılandırması, araç politikaları                                                        |
| **Current Mitigations** | Dosya izinleri                                                                                |
| **Residual Risk**       | Orta - Yerel erişim gerektirir                                                                |
| **Öneriler**            | Yapılandırma bütünlüğü doğrulaması, yapılandırma değişiklikleri için denetim günlükleri       |

---

### 3.5 Savunmadan Kaçınma (AML.TA0007)

#### T-EVADE-001: Moderasyon Kalıbı Atlama

| Attribute               | Değer                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                          |
| **Description**         | Saldırgan, moderasyon kalıplarından kaçınmak için beceri içeriği oluşturur                  |
| **Attack Vector**       | Unicode homoglifleri, kodlama hileleri, dinamik yükleme                                     |
| **Affected Components** | ClawHub moderation.ts                                                       |
| **Current Mitigations** | Kalıp tabanlı FLAG_RULES                                               |
| **Residual Risk**       | Yüksek - Basit regex kolayca aşılabilir                                                     |
| **Öneriler**            | Davranışsal analiz ekleyin (VirusTotal Code Insight), AST tabanlı tespit |

#### T-EVADE-002: İçerik Sarmalayıcıdan Kaçış

| Attribute               | Değer                                                          |
| ----------------------- | -------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data             |
| **Description**         | Attacker crafts content that escapes XML wrapper context       |
| **Attack Vector**       | Etiket manipülasyonu, bağlam karmaşası, talimat geçersiz kılma |
| **Affected Components** | 34. Harici içerik sarmalama             |
| **Current Mitigations** | XML etiketleri + güvenlik bildirimi                            |
| **Residual Risk**       | Orta - Yeni kaçışlar düzenli olarak keşfediliyor               |
| **Öneriler**            | Çoklu sarmalayıcı katmanları, çıktı tarafı doğrulama           |

---

### 3.6 Keşif (AML.TA0008)

#### T-DISC-001: Araçların Listelenmesi

| Attribute               | Değer                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Saldırgan, yönlendirme yoluyla mevcut araçları listeler   |
| **Attack Vector**       | "What tools do you have?" style queries                   |
| **Affected Components** | Agent tool registry                                       |
| **Current Mitigations** | Özel bir önlem yok                                        |
| **Residual Risk**       | Düşük - Araçlar genellikle belgelenmiştir                 |
| **Öneriler**            | Araç görünürlüğü kontrollerini değerlendirin              |

#### T-DISC-002: Oturum Verisi Çıkarımı

| Attribute               | Değer                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Saldırgan, oturum bağlamından hassas verileri çıkarır     |
| **Attack Vector**       | "Ne konuştuk?" sorguları, bağlam yoklama                  |
| **Affected Components** | Oturum transkriptleri, bağlam penceresi                   |
| **Current Mitigations** | Gönderici başına oturum yalıtımı                          |
| **Residual Risk**       | Orta - Oturum içindeki veriler erişilebilir               |
| **Öneriler**            | Bağlam içinde hassas veri maskelemesi uygulayın           |

---

### 3.7 Toplama ve Sızdırma (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: web_fetch üzerinden Veri Hırsızlığı

| Attribute               | Değer                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                             |
| **Description**         | Saldırgan, ajanı harici bir URL'ye göndermesi için yönlendirerek verileri sızdırır |
| **Attack Vector**       | Ajanın saldırgan sunucusuna veri POST etmesine neden olan istem enjeksiyonu        |
| **Affected Components** | web_fetch aracı                                               |
| **Current Mitigations** | Dahili ağlar için SSRF engellemesi                                                 |
| **Residual Risk**       | Yüksek - Harici URL'lere izin veriliyor                                            |
| **Öneriler**            | URL izin listesi, veri sınıflandırma farkındalığı uygulayın                        |

#### T-EXFIL-002: Yetkisiz Mesaj Gönderimi

| Attribute               | Değer                                                                    |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0009 - Collection                                   |
| **Description**         | Saldırgan, ajanın hassas veriler içeren mesajlar göndermesine neden olur |
| **Attack Vector**       | Ajanın saldırgana mesaj göndermesine neden olan istem enjeksiyonu        |
| **Affected Components** | Mesaj aracı, kanal entegrasyonları                                       |
| **Current Mitigations** | Outbound messaging gating                                                |
| **Residual Risk**       | Orta - Geçitleme aşılabilir                                              |
| **Öneriler**            | Require explicit confirmation for new recipients                         |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | Değer                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                  |
| **Description**         | Malicious skill harvests credentials from agent context |
| **Attack Vector**       | Skill code reads environment variables, config files    |
| **Affected Components** | Skill execution environment                             |
| **Current Mitigations** | None specific to skills                                 |
| **Residual Risk**       | Critical - Skills run with agent privileges             |
| **Öneriler**            | Skill sandboxing, credential isolation                  |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001: Unauthorized Command Execution

| Attribute               | Değer                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker executes arbitrary commands on user system  |
| **Attack Vector**       | Prompt injection combined with exec approval bypass  |
| **Affected Components** | Bash tool, command execution                         |
| **Current Mitigations** | Exec approvals, Docker sandbox option                |
| **Residual Risk**       | Critical - Host execution without sandbox            |
| **Öneriler**            | Default to sandbox, improve approval UX              |

#### T-IMPACT-002: Resource Exhaustion (DoS)

| Attribute               | Değer                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker exhausts API credits or compute resources   |
| **Attack Vector**       | Automated message flooding, expensive tool calls     |
| **Affected Components** | Gateway, agent sessions, API provider                |
| **Current Mitigations** | None                                                 |
| **Residual Risk**       | High - No rate limiting                              |
| **Öneriler**            | Implement per-sender rate limits, cost budgets       |

#### T-IMPACT-003: Reputation Damage

| Attribute               | Değer                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity    |
| **Description**         | Attacker causes agent to send harmful/offensive content |
| **Attack Vector**       | Prompt injection causing inappropriate responses        |
| **Affected Components** | Output generation, channel messaging                    |
| **Current Mitigations** | LLM provider content policies                           |
| **Residual Risk**       | Medium - Provider filters imperfect                     |
| **Öneriler**            | Output filtering layer, user controls                   |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Uygulama                                                         | Effectiveness                                        |
| --------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------- |
| GitHub Account Age                | `requireGitHubAccountAge()`                                      | Medium - Raises bar for new attackers                |
| Path Sanitization                 | `sanitizePath()`                                                 | High - Prevents path traversal                       |
| File Type Validation              | `isTextFile()`                                                   | Medium - Only text files, but can still be malicious |
| Size Limits                       | 50MB total bundle                                                | High - Prevents resource exhaustion                  |
| Required SKILL.md | Mandatory readme                                                 | Low security value - Informational only              |
| Pattern Moderation                | FLAG_RULES in moderation.ts | Low - Easily bypassed                                |
| Moderation Status                 | `moderationStatus` field                                         | Medium - Manual review possible                      |

### 4.2 Moderation Flag Patterns

Current patterns in `moderation.ts`:

```javascript
// Known-bad identifiers
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i

// Suspicious keywords
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+|\s*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i
```

**Limitations:**

- Only checks slug, displayName, summary, frontmatter, metadata, file paths
- Does not analyze actual skill code content
- Simple regex easily bypassed with obfuscation
- No behavioral analysis

### 4.3 Planned Improvements

| Improvement                                           | Status                                                   | Impact                                                              |
| ----------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------- |
| VirusTotal Integration                                | In Progress                                              | Topluluk Raporlaması                                                |
| Topluluk Raporlaması                                  | Kısmi (`skillReports` tablosu mevcut) | Orta                                                                |
| Kısmi (`auditLogs` tablosu mevcut) | Denetim Günlüğü                                          | Orta                                                                |
| Rozet Sistemi                                         | Uygulandı                                                | Orta - `highlighted`, `official`, `deprecated`, `redactionApproved` |

---

## 6. **Artık Risk**

### 5.1 Olasılık ve Etki

| Tehdit Kimliği | Olasılık | Impact | Risk Seviyesi | Öncelik |
| -------------- | -------- | ------ | ------------- | ------- |
| T-EXEC-001     | High     | Kritik | **Kritik**    | P0      |
| T-PERSIST-001  | High     | Kritik | **Kritik**    | P0      |
| T-EXFIL-003    | Orta     | Kritik | **Kritik**    | P0      |
| T-IMPACT-001   | Orta     | Kritik | **High**      | P1      |
| T-EXEC-002     | High     | High   | **High**      | P1      |
| T-EXEC-004     | Orta     | High   | **High**      | P1      |
| T-ACCESS-003   | Orta     | High   | **High**      | P1      |
| T-EXFIL-001    | Orta     | High   | **High**      | P1      |
| T-IMPACT-002   | High     | Orta   | **High**      | P1      |
| T-EVADE-001    | High     | Orta   | **Orta**      | P2      |
| T-ACCESS-001   | Düşük    | High   | **Orta**      | P2      |
| T-ACCESS-002   | Düşük    | High   | **Orta**      | P2      |
| T-PERSIST-002  | Düşük    | High   | **Orta**      | P2      |

### 5.2 Kritik Yol Saldırı Zincirleri

**Saldırı Zinciri 1: Yetenek Tabanlı Veri Hırsızlığı**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(Kötü amaçlı bir skill yayımlama) → (Moderasyondan kaçma) → (Kimlik bilgilerini toplama)
```

**Saldırı Zinciri 2: Prompt Enjeksiyonundan RCE'ye**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Prompt enjekte etme) → (Yürütme onayını aşma) → (Komutları çalıştırma)
```

**Saldırı Zinciri 3: Getirilen İçerik Üzerinden Dolaylı Enjeksiyon**

```
T-EXEC-002 → T-EXFIL-001 → Harici sızdırma
(URL içeriğini zehirleme) → (Ajan içeriği getirir ve talimatları izler) → (Saldırgana veri gönderilir)
```

---

## 6. Öneriler Özeti

### 6.1 Acil (P0)

| ID    | Öneri                                       | Ele Aldıkları              |
| ----- | ------------------------------------------- | -------------------------- |
| R-001 | VirusTotal entegrasyonunu tamamlayın        | T-PERSIST-001, T-EVADE-001 |
| R-002 | Skill sandboxing'i uygulayın                | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Add output validation for sensitive actions | T-EXEC-001, T-EXEC-002     |

### 6.2 Kısa vadeli (P1)

| ID    | Öneri                                                          | Ele Aldığı   |
| ----- | -------------------------------------------------------------- | ------------ |
| R-004 | Hız sınırlaması uygulayın                                      | T-IMPACT-002 |
| R-005 | Dinlenme halinde token şifrelemesi ekleyin                     | T-ACCESS-003 |
| R-006 | Yürütme onayı UX'ini ve doğrulamayı iyileştirin                | T-EXEC-004   |
| R-007 | web_fetch için URL izin listesi uygulayın | T-EXFIL-001  |

### 6.3 Orta vadeli (P2)

| ID    | Öneri                                                       | Ele Aldığı    |
| ----- | ----------------------------------------------------------- | ------------- |
| R-008 | Mümkün olan yerlerde kriptografik kanal doğrulaması ekleyin | T-ACCESS-002  |
| R-009 | Yapılandırma bütünlüğü doğrulaması uygulayın                | T-PERSIST-003 |
| R-010 | Güncelleme imzalama ve sürüm sabitleme ekleyin              | T-PERSIST-002 |

---

## 7. Ekler

### 7.1 ATLAS Teknik Eşlemesi

| ATLAS ID                                      | Teknik Adı                                                | OpenClaw Tehditleri                                              |
| --------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Aktif Tarama                                              | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Toplama                                                   | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Tedarik Zinciri: Yapay Zekâ Yazılımı      | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Tedarik Zinciri: Veri                     | T-PERSIST-003                                                    |
| AML.T0031                     | Yapay Zekâ Modeli Bütünlüğünü Aşındırma                   | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | AML.T0040 - AI Modeli Çıkarım API Erişimi | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Düşmanca Veri Oluşturma                                   | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM Prompt Enjeksiyonu: Doğrudan          | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM Prompt Enjeksiyonu: Dolaylı           | T-EXEC-002                                                       |

### 7.2 Temel Güvenlik Dosyaları

| Yol                                 | Amaç                         | Risk Seviyesi |
| ----------------------------------- | ---------------------------- | ------------- |
| `src/infra/exec-approvals.ts`       | Komut onay mantığı           | **Kritik**    |
| `src/gateway/auth.ts`               | Ağ geçidi kimlik doğrulaması | **Kritik**    |
| `src/web/inbound/access-control.ts` | Kanal erişim denetimi        | **Kritik**    |
| `src/infra/net/ssrf.ts`             | SSRF koruması                | **Kritik**    |
| `src/security/external-content.ts`  | Prompt enjeksiyonu azaltımı  | **Kritik**    |
| `src/agents/sandbox/tool-policy.ts` | Araç politikası uygulaması   | **Kritik**    |
| `convex/lib/moderation.ts`          | ClawHub moderasyonu          | **High**      |
| `convex/lib/skillPublish.ts`        | Beceri yayımlama akışı       | **High**      |
| `src/routing/resolve-route.ts`      | Session isolation            | **Orta**      |

### 7.3 Sözlük

| Terim                | Tanım                                                          |
| -------------------- | -------------------------------------------------------------- |
| **ATLAS**            | MITRE'nin Yapay Zekâ Sistemleri için Düşmanca Tehdit Manzarası |
| **ClawHub**          | OpenClaw'un beceri pazaryeri                                   |
| **Gateway**          | OpenClaw'un mesaj yönlendirme ve kimlik doğrulama katmanı      |
| **MCP**              | Model Context Protocol - araç sağlayıcı arayüzü                |
| **Prompt Injection** | Kötü amaçlı talimatların girdiye gömüldüğü saldırı             |
| **Skill**            | OpenClaw ajanları için indirilebilir uzantı                    |
| **SSRF**             | Sunucu Taraflı İstek Sahteciliği                               |

---

_Bu tehdit modeli yaşayan bir belgedir._ Güvenlik sorunlarını security@openclaw.ai adresine bildirin Güvenlik sorunlarını security@openclaw.ai adresine bildirin


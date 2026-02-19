# OpenClaw Hotmodell v1.0

## MITRE ATLAS-ramverket

**Version:** 1.0-utkast  
**Senast uppdaterad:** 2026-02-04  
**Metodik:** MITRE ATLAS + Dataflödesdiagram  
**Ramverk:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Ramverksattribution

This threat model is built on [MITRE ATLAS](https://atlas.mitre.org/), the industry-standard framework for documenting adversarial threats to AI/ML systems. ATLAS is maintained by [MITRE](https://www.mitre.org/) in collaboration with the AI security community.

**Viktiga ATLAS-resurser:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contributing to ATLAS](https://atlas.mitre.org/resources/contribute)

### Bidra till denna hotmodell

This is a living document maintained by the OpenClaw community. See [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) for guidelines on contributing:

- Rapportera nya hot
- Uppdatera befintliga hot
- Föreslå attackkedjor
- Föreslå skyddsåtgärder

---

## 1. Introduction

### 1.1 Syfte

Denna hotmodell dokumenterar adversariella hot mot OpenClaw AI-agentplattformen och ClawHub-marknadsplatsen för skills, med hjälp av MITRE ATLAS-ramverket som är särskilt utformat för AI/ML-system.

### 1.2 Omfattning

| Komponent                   | Ingår  | Kommentar                                                        |
| --------------------------- | ------ | ---------------------------------------------------------------- |
| OpenClaw Agent-körtidsmiljö | Ja     | Kärnexekvering av agent, verktygsanrop, sessioner                |
| Gateway                     | Ja     | Autentisering, routing, kanalintegrationer                       |
| Kanalkopplingar             | Ja     | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| ClawHub-marknadsplats       | Ja     | Publicering, moderering, distribution av skills                  |
| MCP-servrar                 | Ja     | Externa verktygsleverantörer                                     |
| Användarenheter             | Delvis | Mobilappar, skrivbordsklienter                                   |

### 1.3 Utanför omfattning

Inget är uttryckligen utanför omfattningen för denna hotmodell.

---

## 2. System Architecture

### 2.1 Förtroendegränser

```
┌─────────────────────────────────────────────────────────────────┐
│                    ICKE BETRODD ZON                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│             FÖRTROENDEGRÄNS 1: Kanalåtkomst                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Enhetsparning (30 s respitperiod)                      │   │
│  │  • AllowFrom / AllowList-validering                       │   │
│  │  • Token-/lösenords-/Tailscale-autentisering              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           FÖRTROENDEGRÄNS 2: Sessionsisolering                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENTSESSIONER                         │   │
│  │  • Sessionsnyckel = agent:channel:peer                   │   │
│  │  • Verktygspolicyer per agent                            │   │
│  │  • Loggning av transkript                                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            FÖRTROENDEGRÄNS 3: Verktygsexekvering                │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  EXEKVERINGSSANDBOX                      │   │
│  │  • Docker-sandbox ELLER Host (exec-approvals)            │   │
│  │  • Fjärrkörning via Node                                 │   │
│  │  • SSRF-skydd (DNS-pinning + IP-blockering)              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            FÖRTROENDEGRÄNS 4: Externt innehåll                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │            HÄMTADE URL:ER / E-POST / WEBHOOKS           │   │
│  │  • Inpackning av externt innehåll (XML-taggar)           │   │
│  │  • Injektion av säkerhetsmeddelande                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            FÖRTROENDEGRÄNS 5: Leverantörskedja                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Publicering av skills (semver, SKILL.md krävs)        │   │
│  │  • Mönsterbaserade modereringsflaggor                    │   │
│  │  • VirusTotal-skanning (kommer snart)                    │   │
│  │  • Verifiering av GitHub-kontots ålder                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Dataflöden

| Flöde | Källa   | Destination | Data                                         | Skydd                |
| ----- | ------- | ----------- | -------------------------------------------- | -------------------- |
| F1    | Kanal   | Gateway     | Användarmeddelanden                          | TLS, AllowFrom       |
| F2    | Gateway | Agent       | Routade meddelanden                          | Sessionsisolering    |
| F3    | Agent   | Verktyg     | Verktygsanrop                                | Policytillämpning    |
| F4    | Agent   | Extern      | web_fetch-förfrågningar | SSRF-blockering      |
| F5    | ClawHub | Agent       | Skill-kod                                    | Moderering, skanning |
| F6    | Agent   | Kanal       | Svar                                         | Utdatafiltrering     |

---

## 3. 5. Hotanalys efter ATLAS-taktik

### 3.1 Rekognosering (AML.TA0002)

#### T-RECON-001: Upptäckt av agentändpunkt

| Attribut                  | Värde                                                                           |
| ------------------------- | ------------------------------------------------------------------------------- |
| **ATLAS ID**              | AML.T0006 - Active Scanning                                     |
| **Beskrivning**           | Angripare skannar efter exponerade OpenClaw gateway-ändpunkter                  |
| **Attackvektor**          | Nätverksskanning, shodan-frågor, DNS-enumerering                                |
| **Påverkade komponenter** | Gateway, exponerade API-ändpunkter                                              |
| **Nuvarande skydd**       | Tailscale-autentisering som alternativ, binder till loopback som standard       |
| **Kvarstående risk**      | Medel - Publika gateways kan upptäckas                                          |
| **Rekommendationer**      | Dokumentera säker driftsättning, lägg till rate limiting på upptäcktsändpunkter |

#### T-RECON-002: Sondering av kanalintegration

| Attribut                  | Värde                                                                        |
| ------------------------- | ---------------------------------------------------------------------------- |
| **ATLAS ID**              | AML.T0006 - Active Scanning                                  |
| **Beskrivning**           | Angripare sonderar meddelandekanaler för att identifiera AI-hanterade konton |
| **Attackvektor**          | Skicka testmeddelanden, observera svarsmönster                               |
| **Påverkade komponenter** | Alla kanalintegrationer                                                      |
| **Nuvarande skydd**       | Inga specifika                                                               |
| **Kvarstående risk**      | Låg - Begränsat värde enbart från upptäckt                                   |
| **Rekommendationer**      | Överväg slumpmässig variation av svarstider                                  |

---

### 3.2 Initial Access (AML.TA0004)

#### 7. T-ACCESS-001: Avlyssning av parningskod

| Attribute               | Värde                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **Rekommendationer**    | Reduce grace period, add confirmation step                |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | Värde                                                                          |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker spoofs allowed sender identity in channel                             |
| **Attack Vector**       | Depends on channel - phone number spoofing, username impersonation             |
| **Affected Components** | AllowFrom validation per channel                                               |
| **Current Mitigations** | Channel-specific identity verification                                         |
| **Residual Risk**       | Medium - Some channels vulnerable to spoofing                                  |
| **Rekommendationer**    | Document channel-specific risks, add cryptographic verification where possible |

#### T-ACCESS-003: Token Theft

| Attribute               | Värde                                                                    |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                |
| **Description**         | Attacker steals authentication tokens from config files                  |
| **Attack Vector**       | Malware, unauthorized device access, config backup exposure              |
| **Affected Components** | ~/.openclaw/credentials/, config storage |
| **Current Mitigations** | Filbehörigheter                                                          |
| **Residual Risk**       | High - Tokens stored in plaintext                                        |
| **Rekommendationer**    | Implement token encryption at rest, add token rotation                   |

---

### 11. 3.3 Exekvering (AML.TA0005)

#### T-EXEC-001: Direct Prompt Injection

| Attribute               | Värde                                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct          |
| **Description**         | 13. Angriparen skickar utformade prompter för att manipulera agentens beteende |
| **Attack Vector**       | Channel messages containing adversarial instructions                                                  |
| **Affected Components** | Agent LLM, all input surfaces                                                                         |
| **Current Mitigations** | Pattern detection, external content wrapping                                                          |
| **Residual Risk**       | Critical - Detection only, no blocking; sophisticated attacks bypass                                  |
| **Rekommendationer**    | Implement multi-layer defense, output validation, user confirmation for sensitive actions             |

#### T-EXEC-002: Indirect Prompt Injection

| Attribute               | Värde                                                                                                                    |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **ATLAS ID**            | 15. AML.T0051.001 – LLM-promptinjektion: Indirekt |
| **Description**         | Attacker embeds malicious instructions in fetched content                                                                |
| **Attack Vector**       | Malicious URLs, poisoned emails, compromised webhooks                                                                    |
| **Affected Components** | web_fetch, email ingestion, external data sources                                                   |
| **Current Mitigations** | Content wrapping with XML tags and security notice                                                                       |
| **Residual Risk**       | High - LLM may ignore wrapper instructions                                                                               |
| **Rekommendationer**    | Implement content sanitization, separate execution contexts                                                              |

#### T-EXEC-003: Tool Argument Injection

| Attribute               | Värde                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct |
| **Description**         | Attacker manipulates tool arguments through prompt injection                                 |
| **Attack Vector**       | Crafted prompts that influence tool parameter values                                         |
| **Affected Components** | All tool invocations                                                                         |
| **Current Mitigations** | Exec approvals for dangerous commands                                                        |
| **Residual Risk**       | High - Relies on user judgment                                                               |
| **Rekommendationer**    | Implement argument validation, parameterized tool calls                                      |

#### T-EXEC-004: Exec Approval Bypass

| Attribute               | Värde                                                      |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data         |
| **Description**         | Attacker crafts commands that bypass approval allowlist    |
| **Attack Vector**       | Command obfuscation, alias exploitation, path manipulation |
| **Affected Components** | exec-approvals.ts, command allowlist       |
| **Current Mitigations** | 19. Tillåtelselista + frågeläge     |
| **Residual Risk**       | High - No command sanitization                             |
| **Rekommendationer**    | Implement command normalization, expand blocklist          |

---

### 3.4 Persistence (AML.TA0006)

#### T-PERSIST-001: Malicious Skill Installation

| Attribute               | Värde                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker publishes malicious skill to ClawHub                                                        |
| **Attack Vector**       | Create account, publish skill with hidden malicious code                                             |
| **Affected Components** | ClawHub, skill loading, agent execution                                                              |
| **Current Mitigations** | 20. Verifiering av GitHub-kontots ålder, mönsterbaserade modereringsflaggor   |
| **Residual Risk**       | 21. Kritisk – Ingen sandboxing, begränsad granskning                          |
| **Rekommendationer**    | VirusTotal integration (in progress), skill sandboxing, community review          |

#### T-PERSIST-002: Skill Update Poisoning

| Attribute               | Värde                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker compromises popular skill and pushes malicious update                                       |
| **Attack Vector**       | Account compromise, social engineering of skill owner                                                |
| **Affected Components** | ClawHub versioning, auto-update flows                                                                |
| **Current Mitigations** | Version fingerprinting                                                                               |
| **Residual Risk**       | High - Auto-updates may pull malicious versions                                                      |
| **Rekommendationer**    | Implement update signing, rollback capability, version pinning                                       |

#### T-PERSIST-003: Agent Configuration Tampering

| Attribute               | Värde                                                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Supply Chain Compromise: Data |
| **Description**         | Attacker modifies agent configuration to persist access                                       |
| **Attack Vector**       | 23. Modifiering av konfigurationsfiler, injicering av inställningar    |
| **Affected Components** | Agentkonfiguration, verktygspolicyer                                                          |
| **Current Mitigations** | Filbehörigheter                                                                               |
| **Residual Risk**       | 24. Medel – Kräver lokal åtkomst                                       |
| **Rekommendationer**    | Verifiering av konfigurationsintegritet, revisionsloggning för konfigurationsändringar        |

---

### 3.5 Försvarsförbigående (AML.TA0007)

#### T-EVADE-001: Kringgående av modereringsmönster

| Attribute               | Värde                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                             |
| **Description**         | Angriparen utformar färdighetsinnehåll för att undvika modereringsmönster                      |
| **Attack Vector**       | 26. Unicode-homoglyfer, kodningstrick, dynamisk inläsning               |
| **Affected Components** | ClawHub moderation.ts                                                          |
| **Current Mitigations** | Mönsterbaserade FLAG_RULES                                                |
| **Residual Risk**       | Hög – Enkla regex kan lätt kringgås                                                            |
| **Rekommendationer**    | Lägg till beteendeanalys (VirusTotal Code Insight), AST-baserad detektering |

#### T-EVADE-002: Undanflykt från innehållsomslag

| Attribute               | Värde                                                               |
| ----------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                  |
| **Description**         | Angriparen utformar innehåll som undkommer XML-omslagskontexten     |
| **Attack Vector**       | Taggmanipulation, kontextförvirring, åsidosättning av instruktioner |
| **Affected Components** | Extern innehållsomslagning                                          |
| **Current Mitigations** | XML-taggar + säkerhetsmeddelande                                    |
| **Residual Risk**       | Medel – Nya undanflykter upptäcks regelbundet                       |
| **Rekommendationer**    | Flera omslagslager, validering på utgångssidan                      |

---

### 3.6 Upptäckt (AML.TA0008)

#### T-DISC-001: Verktygsinventering

| Attribute               | Värde                                                               |
| ----------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access           |
| **Description**         | Angriparen inventerar tillgängliga verktyg genom prompting          |
| **Attack Vector**       | "Vilka verktyg har du?"-liknande frågor                             |
| **Affected Components** | Agentens verktygsregister                                           |
| **Current Mitigations** | None specific                                                       |
| **Residual Risk**       | 4. Låg – Verktyg är generellt dokumenterade  |
| **Rekommendationer**    | 31. Överväg kontroller för verktygssynlighet |

#### 32. T-DISC-002: Extrahering av sessionsdata

| Attribute               | Värde                                                                             |
| ----------------------- | --------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                         |
| **Description**         | 11. Angripare extraherar känslig data från sessionskontext |
| **Attack Vector**       | 13. Frågor som ”Vad diskuterade vi?”, kontextsondering     |
| **Affected Components** | 15. Sessionsutskrifter, kontextfönster                     |
| **Current Mitigations** | 35. Sessionsisolering per avsändare                        |
| **Residual Risk**       | 36. Medel – Data inom sessionen är åtkomlig                |
| **Rekommendationer**    | 37. Implementera maskering av känslig data i kontext       |

---

### 38. 3.7 Insamling och exfiltrering (AML.TA0009, AML.TA0010)

#### 22. T-EXFIL-001: Datastöld via web_fetch

| Attribute               | Värde                                                                                                               |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 25. AML.T0009 – Insamling                                                    |
| **Description**         | 27. Angripare exfiltrerar data genom att instruera agenten att skicka till extern URL        |
| **Attack Vector**       | 29. Prompt injection som får agenten att POST:a data till angriparens server |
| **Affected Components** | 31. web_fetch-verktyget                                                 |
| **Current Mitigations** | 33. SSRF-blockering för interna nätverk                                                      |
| **Residual Risk**       | 35. Hög – Externa URL:er är tillåtna                                         |
| **Rekommendationer**    | 36. Implementera URL-allowlisting, medvetenhet om dataklassificering                         |

#### 37. T-EXFIL-002: Obehörig meddelandesändning

| Attribute               | Värde                                                                                               |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 40. AML.T0009 – Insamling                                    |
| **Description**         | 42. Angripare får agenten att skicka meddelanden som innehåller känslig data |
| **Attack Vector**       | 44. Prompt injection som får agenten att skicka meddelanden till angriparen  |
| **Affected Components** | 46. Meddelandeverktyg, kanalintegrationer                                    |
| **Current Mitigations** | 48. Styrning av utgående meddelanden                                         |
| **Residual Risk**       | 50. Medel – Styrningen kan kringgås                                          |
| **Rekommendationer**    | 39. Kräv uttryckligt godkännande för nya mottagare                           |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | Värde                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 40. AML.T0009 – Insamling                                     |
| **Description**         | 42. Skadlig färdighet samlar in autentiseringsuppgifter från agentens kontext |
| **Attack Vector**       | 44. Färdighetskod läser miljövariabler, konfigurationsfiler                   |
| **Affected Components** | Skill execution environment                                                                          |
| **Current Mitigations** | None specific to skills                                                                              |
| **Residual Risk**       | Critical - Skills run with agent privileges                                                          |
| **Rekommendationer**    | Skill sandboxing, credential isolation                                                               |

---

### 45. 3.8 Påverkan (AML.TA0011)

#### T-IMPACT-001: Unauthorized Command Execution

| Attribute               | Värde                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker executes arbitrary commands on user system  |
| **Attack Vector**       | Prompt injection combined with exec approval bypass  |
| **Affected Components** | Bash tool, command execution                         |
| **Current Mitigations** | Exec approvals, Docker sandbox option                |
| **Residual Risk**       | Critical - Host execution without sandbox            |
| **Rekommendationer**    | Default to sandbox, improve approval UX              |

#### T-IMPACT-002: Resource Exhaustion (DoS)

| Attribute               | Värde                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker exhausts API credits or compute resources   |
| **Attack Vector**       | Automated message flooding, expensive tool calls     |
| **Affected Components** | Gateway, agent sessions, API provider                |
| **Current Mitigations** | None                                                 |
| **Residual Risk**       | High - No rate limiting                              |
| **Rekommendationer**    | Implement per-sender rate limits, cost budgets       |

#### T-IMPACT-003: Reputation Damage

| Attribute               | Värde                                                          |
| ----------------------- | -------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity           |
| **Description**         | Attacker causes agent to send harmful/offensive content        |
| **Attack Vector**       | Prompt injection causing inappropriate responses               |
| **Affected Components** | Output generation, channel messaging                           |
| **Current Mitigations** | 48. LLM-leverantörens innehållspolicyer |
| **Residual Risk**       | Medium - Provider filters imperfect                            |
| **Rekommendationer**    | Output filtering layer, user controls                          |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Implementering                                                   | Effectiveness                                                |
| --------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------ |
| GitHub Account Age                | `requireGitHubAccountAge()`                                      | Medium - Raises bar for new attackers                        |
| Path Sanitization                 | `sanitizePath()`                                                 | High - Prevents path traversal                               |
| File Type Validation              | `isTextFile()`                                                   | Medium - Only text files, but can still be malicious         |
| Size Limits                       | 50MB total bundle                                                | 49. Hög – Förhindrar resursutmattning |
| Required SKILL.md | Mandatory readme                                                 | Low security value - Informational only                      |
| Pattern Moderation                | FLAG_RULES in moderation.ts | Low - Easily bypassed                                        |
| Moderation Status                 | `moderationStatus` field                                         | Medium - Manual review possible                              |

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

| Improvement            | Status                                                                               | Impact                                                                |
| ---------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| VirusTotal Integration | In Progress                                                                          | High - Code Insight behavioral analysis                               |
| Community Reporting    | 50. Delvis (`skillReports`-tabellen finns) | Medium                                                                |
| Audit Logging          | Partial (`auditLogs` table exists)                                | Medium                                                                |
| Badge System           | Implemented                                                                          | Medium - `highlighted`, `official`, `deprecated`, `redactionApproved` |

---

## 5. Risk Matrix

### 5.1 Likelihood vs Impact

| Threat ID     | Likelihood | Impact   | Risk Level   | Priority |
| ------------- | ---------- | -------- | ------------ | -------- |
| T-EXEC-001    | High       | Critical | **Critical** | P0       |
| T-PERSIST-001 | High       | Critical | **Critical** | P0       |
| T-EXFIL-003   | Medium     | Critical | **Critical** | P0       |
| T-IMPACT-001  | Medium     | Critical | **High**     | P1       |
| T-EXEC-002    | High       | High     | **High**     | P1       |
| T-EXEC-004    | Medium     | High     | **High**     | P1       |
| T-ACCESS-003  | Medium     | High     | **High**     | P1       |
| T-EXFIL-001   | Medium     | High     | **High**     | P1       |
| T-IMPACT-002  | High       | Medium   | **High**     | P1       |
| T-EVADE-001   | High       | Medium   | **Medel**    | P2       |
| T-ACCESS-001  | Låg        | High     | **Medel**    | P2       |
| T-ACCESS-002  | Låg        | High     | **Medel**    | P2       |
| T-PERSIST-002 | Låg        | High     | **Medel**    | P2       |

### 5.2 Kritiska attackkedjor

**Attackkedja 1: Färdighetsbaserad datastöld**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(Publicera skadlig färdighet) → (Undvik moderering) → (Skörda autentiseringsuppgifter)
```

**Attackkedja 2: Prompt-injektion till RCE**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Injectera prompt) → (Kringgå körningsgodkännande) → (Kör kommandon)
```

**Attackkedja 3: Indirekt injektion via hämtat innehåll**

```
T-EXEC-002 → T-EXFIL-001 → Extern exfiltrering
(Förgifta URL-innehåll) → (Agenten hämtar och följer instruktioner) → (Data skickas till angripare)
```

---

## 6. Sammanfattning av rekommendationer

### 6.1 Omedelbart (P0)

| ID    | Rekommendation                                    | Adresserar                 |
| ----- | ------------------------------------------------- | -------------------------- |
| R-001 | Slutför VirusTotal-integration                    | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implementera sandboxning av färdigheter           | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Lägg till utdata­validering för känsliga åtgärder | T-EXEC-001, T-EXEC-002     |

### 6.2 Kortsiktigt (P1)

| ID    | Rekommendation                                                   | Adresser     |
| ----- | ---------------------------------------------------------------- | ------------ |
| R-004 | Implement rate limiting                                          | T-IMPACT-002 |
| R-005 | Lägg till tokenkryptering i vila                                 | T-ACCESS-003 |
| R-006 | Förbättra UX för exec-godkännande och validering                 | T-EXEC-004   |
| R-007 | Implementera URL-tillåtslista för web_fetch | T-EXFIL-001  |

### 6.3 Medellång sikt (P2)

| ID    | Rekommendation                                              | Adresser      |
| ----- | ----------------------------------------------------------- | ------------- |
| R-008 | Lägg till kryptografisk kanalverifiering där det är möjligt | T-ACCESS-002  |
| R-009 | Implementera verifiering av konfigurationsintegritet        | T-PERSIST-003 |
| R-010 | Lägg till signering av uppdateringar och versionslåsning    | T-PERSIST-002 |

---

## 7. Bilagor

### 7.1 ATLAS-teknikkartläggning

| ATLAS-ID                                      | Tekniknamn                                    | OpenClaw-hot                                                     |
| --------------------------------------------- | --------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Aktiv skanning                                | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Insamling                                     | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Leveranskedja: AI-programvara | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Leveranskedja: Data           | T-PERSIST-003                                                    |
| AML.T0031                     | Underminera AI-modellens integritet           | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | Åtkomst till AI-modellens inferens-API        | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Skapa adversariella data                      | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM-promptinjektion: Direkt   | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM-promptinjektion: Indirekt | T-EXEC-002                                                       |

### 7.2 Viktiga säkerhetsfiler

| Sökväg                              | Syfte                          | Risk Level   |
| ----------------------------------- | ------------------------------ | ------------ |
| `src/infra/exec-approvals.ts`       | Logik för kommando-godkännande | **Critical** |
| `src/gateway/auth.ts`               | Gateway-autentisering          | **Critical** |
| `src/web/inbound/access-control.ts` | Åtkomstkontroll för kanaler    | **Critical** |
| `src/infra/net/ssrf.ts`             | SSRF-skydd                     | **Critical** |
| `src/security/external-content.ts`  | Åtgärder mot promptinjektion   | **Critical** |
| `src/agents/sandbox/tool-policy.ts` | Efterlevnad av verktygspolicy  | **Critical** |
| `convex/lib/moderation.ts`          | ClawHub-moderering             | **High**     |
| `convex/lib/skillPublish.ts`        | Skill publishing flow          | **High**     |
| `src/routing/resolve-route.ts`      | Session isolation              | **Medel**    |

### 7.3 Ordlista

| Term                 | Definition                                                |
| -------------------- | --------------------------------------------------------- |
| **ATLAS**            | MITRE's Adversarial Threat Landscape for AI Systems       |
| **ClawHub**          | OpenClaws marknadsplats för färdigheter                   |
| **Gateway**          | OpenClaws meddelanderoutning och autentiseringslager      |
| **MCP**              | Model Context Protocol - tool provider interface          |
| **Prompt Injection** | Attack where malicious instructions are embedded in input |
| **Skill**            | Downloadable extension for OpenClaw agents                |
| **SSRF**             | Server-Side Request Forgery                               |

---

_This threat model is a living document. Report security issues to security@openclaw.ai_


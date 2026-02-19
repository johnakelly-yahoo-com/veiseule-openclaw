# OpenClaw Trusselsmodel v1.0

## MITRE ATLAS-rammeværk

**Version:** 1.0-draft  
**Sidst opdateret:** 2026-02-04  
**Metode:** MITRE ATLAS + Data Flow Diagrams  
**Rammeværk:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Rammeværkstilskrivning

This threat model is built on [MITRE ATLAS](https://atlas.mitre.org/), the industry-standard framework for documenting adversarial threats to AI/ML systems. ATLAS is maintained by [MITRE](https://www.mitre.org/) in collaboration with the AI security community.

**Centrale ATLAS-ressourcer:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Bidrag til ATLAS](https://atlas.mitre.org/resources/contribute)

### Bidrag til denne trusselsmodel

This is a living document maintained by the OpenClaw community. See [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) for guidelines on contributing:

- Rapportering af nye trusler
- Opdatering af eksisterende trusler
- Foreslå angrebskæder
- Foreslå afværgeforanstaltninger

---

## 1. Introduction

### 1.1 Formål

Denne trusselsmodel dokumenterer adversarielle trusler mod OpenClaw AI-agentplatformen og ClawHub skill-markedspladsen ved brug af MITRE ATLAS-rammeværket, som er specifikt designet til AI/ML-systemer.

### 1.2 Omfang

| Komponent            | Inkluderet | Bemærkninger                                                                     |
| -------------------- | ---------- | -------------------------------------------------------------------------------- |
| OpenClaw Agentkørsel | Ja         | Kerneafvikling af agent, tool-kald, sessioner                                    |
| Gateway              | Ja         | Autentifikation, routing, kanal-integration                                      |
| Kanal-integrationer  | Ja         | WhatsApp, Telegram, Discord, Signal, Slack m.fl. |
| ClawHub Markedsplads | Ja         | Udgivelse, moderation, distribution af skills                                    |
| MCP-servere          | Ja         | Eksterne tool-udbydere                                                           |
| Brugerens enheder    | Delvist    | Mobilapps, desktopklienter                                                       |

### 1.3 Uden for omfang

Intet er eksplicit uden for omfanget for denne trusselsmodel.

---

## 2. System Architecture

### 2.1 Tillidsgrænser

```
┌─────────────────────────────────────────────────────────────────┐
│                    IKKE-TILLIDSSONE                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│              TILLIDSGRÆNSE 1: Kanaladgang                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Enhedsparring (30 sek. grace-periode)                  │   │
│  │  • AllowFrom / AllowList-validering                       │   │
│  │  • Token/Password/Tailscale-autentifikation               │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           TILLIDSGRÆNSE 2: Sessionsisolering                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENTSESSIONER                          │   │
│  │  • Sessionsnøgle = agent:channel:peer                     │   │
│  │  • Tool-politikker pr. agent                              │   │
│  │  • Logning af transskriptioner                            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            TILLIDSGRÆNSE 3: Tool-eksekvering                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                 EKSEKVERINGSSANDBOX                       │   │
│  │  • Docker-sandbox ELLER vært (exec-approvals)            │   │
│  │  • Node remote execution                                  │   │
│  │  • SSRF-beskyttelse (DNS-pinning + IP-blokering)         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│         TILLIDSGRÆNSE 4: Eksternt indhold                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           HENTEDE URL'ER / EMAILS / WEBHOOKS             │   │
│  │  • Indpakning af eksternt indhold (XML-tags)             │   │
│  │  • Indsættelse af sikkerhedsadvarsel                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          TILLIDSGRÆNSE 5: Supply chain                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Udgivelse af skills (semver, SKILL.md påkrævet)       │   │
│  │  • Mønsterbaserede moderationsflag                       │   │
│  │  • VirusTotal-scanning (kommer snart)                    │   │
│  │  • Verifikation af GitHub-kontoens alder                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Dataflows

| Forløb | Kilde   | Destination | Data                                         | Beskyttelse          |
| ------ | ------- | ----------- | -------------------------------------------- | -------------------- |
| F1     | Kanal   | Gateway     | Brugerbeskeder                               | TLS, AllowFrom       |
| F2     | Gateway | Agent       | Routede beskeder                             | Sessionsisolering    |
| F3     | Agent   | Tools       | Tool-kald                                    | Politik-håndhævelse  |
| F4     | Agent   | Ekstern     | web_fetch-forespørgsler | SSRF-blokering       |
| F5     | ClawHub | Agent       | Skill-kode                                   | Moderation, scanning |
| F6     | Agent   | Kanal       | Svar                                         | Output-filtrering    |

---

## 3. Trusselsanalyse efter ATLAS-taktik

### 3.1 Rekognoscering (AML.TA0002)

#### T-RECON-001: Opdagelse af agent-endpoint

| Attribut                | Værdi                                                                         |
| ----------------------- | ----------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                   |
| **Beskrivelse**         | Angriber scanner efter eksponerede OpenClaw gateway-endpoints                 |
| **Angrebsvektor**       | Netværksscanning, shodan-forespørgsler, DNS-enumeration                       |
| **Berørte komponenter** | Gateway, eksponerede API-endpoints                                            |
| **Nuværende afværge**   | Tailscale-autentifikation, binding til loopback som standard                  |
| **Resterende risiko**   | Middel - Offentlige gateways kan opdages                                      |
| **Anbefalinger**        | Dokumentér sikker implementering, tilføj rate limiting på opdagelsesendpoints |

#### T-RECON-002: Afsøgning af kanal-integrationer

| Attribut                | Værdi                                                                |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                          |
| **Beskrivelse**         | Angriber sonderer beskedkanaler for at identificere AI-styrede konti |
| **Angrebsvektor**       | Afsendelse af testbeskeder, observation af svarmønstre               |
| **Berørte komponenter** | Alle kanal-integrationer                                             |
| **Nuværende afværge**   | Ingen specifikke                                                     |
| **Resterende risiko**   | Lav - Begrænset værdi alene ved opdagelse                            |
| **Anbefalinger**        | Overvej randomisering af svartider                                   |

---

### 3.2 Indledende adgang (AML.TA0004)

#### T-ACCESS-001: Aflytning af parringskode

| Attribute               | Værdi                                                             |
| ----------------------- | ----------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access         |
| **Description**         | Angriber opfanger parringskode i løbet af 30 sekunders frist      |
| **Attack Vector**       | Skulderkigning, netværkssniffing, social engineering              |
| **Affected Components** | Enhedsparringssystem                                              |
| **Current Mitigations** | 30 sek. udløb, koder sendt via eksisterende kanal |
| **Residual Risk**       | Medium - Grace period exploitable                                 |
| **Anbefalinger**        | Reducer fristen, tilføj bekræftelsestrin                          |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | Værdi                                                                             |
| ----------------------- | --------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                         |
| **Description**         | Attacker spoofs allowed sender identity in channel                                |
| **Attack Vector**       | Afhænger af kanal – telefonnummerforfalskning, brugernavns-efterligning           |
| **Affected Components** | AllowFrom-validering pr. kanal                                    |
| **Current Mitigations** | Kanal-specifik identitetsverifikation                                             |
| **Residual Risk**       | Middel – Nogle kanaler er sårbare over for forfalskning                           |
| **Anbefalinger**        | Dokumentér kanal-specifikke risici, tilføj kryptografisk verifikation hvor muligt |

#### T-ACCESS-003: Token-tyveri

| Attribute               | Værdi                                                                               |
| ----------------------- | ----------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                           |
| **Description**         | Angriber stjæler autentifikationstokens fra konfigurationsfiler                     |
| **Attack Vector**       | Malware, uautoriseret enhedsadgang, eksponering af konfigurationsbackups            |
| **Affected Components** | ~/.openclaw/credentials/, konfigurationslager       |
| **Current Mitigations** | Filtilladelser                                                                      |
| **Residual Risk**       | High - Tokens stored in plaintext                                                   |
| **Anbefalinger**        | 2. Implementér tokenkryptering i hvile, tilføj tokenrotation |

---

### 3. 3.3 Eksekvering (AML.TA0005)

#### T-EXEC-001: Direct Prompt Injection

| Attribute               | Værdi                                                                                                                   |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 7. AML.T0051.000 - LLM Prompt Injection: Direkte |
| **Description**         | 9. Angriber sender specialudformede prompts for at manipulere agentens adfærd                    |
| **Attack Vector**       | 11. Kanalmeddelelser indeholdende modstridende instruktioner                                     |
| **Affected Components** | Agent LLM, all input surfaces                                                                                           |
| **Current Mitigations** | 15. Mønstergenkendelse, indpakning af eksternt indhold                                           |
| **Residual Risk**       | 17. Kritisk - Kun detektion, ingen blokering; sofistikerede angreb omgås                         |
| **Anbefalinger**        | 18. Implementér flerlagsforsvar, outputvalidering, brugerbekræftelse for følsomme handlinger     |

#### 19. T-EXEC-002: Indirekte Prompt Injection

| Attribute               | Værdi                                                                                                                      |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 22. AML.T0051.001 - LLM Prompt Injection: Indirekte |
| **Description**         | 24. Angriber indlejrer ondsindede instruktioner i hentet indhold                                    |
| **Attack Vector**       | 26. Ondsindede URL'er, forgiftede e-mails, kompromitterede webhooks                                 |
| **Affected Components** | 28. web_fetch, e-mailindtagelse, eksterne datakilder                           |
| **Current Mitigations** | 30. Indpakning af indhold med XML-tags og sikkerhedsnotits                                          |
| **Residual Risk**       | 32. Høj - LLM kan ignorere indpakningsinstruktioner                                                 |
| **Anbefalinger**        | 33. Implementér indholdssanitering, separate eksekveringskontekster                                 |

#### 34. T-EXEC-003: Tool Argument Injection

| Attribute               | Værdi                                                                                                                    |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **ATLAS ID**            | 37. AML.T0051.000 - LLM Prompt Injection: Direkte |
| **Description**         | 39. Angriber manipulerer værktøjsargumenter via prompt injection                                  |
| **Attack Vector**       | 41. Specialudformede prompts, der påvirker værdier for værktøjsparametre                          |
| **Affected Components** | 43. Alle værktøjsinvokationer                                                                     |
| **Current Mitigations** | 45. Eksekveringsgodkendelser for farlige kommandoer                                               |
| **Residual Risk**       | 47. Høj - Afhænger af brugerens dømmekraft                                                        |
| **Anbefalinger**        | 48. Implementér argumentvalidering, parameteriserede værktøjskald                                 |

#### 49. T-EXEC-004: Exec Approval Bypass

| Attribute               | Værdi                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                   |
| **Description**         | Attacker crafts commands that bypass approval allowlist                              |
| **Attack Vector**       | 6. Kommandoobfuskering, udnyttelse af aliaser, stihåndtering  |
| **Affected Components** | 8. exec-approvals.ts, kommando-allowlist      |
| **Current Mitigations** | 10. Allowlist + spørg-tilstand                                |
| **Residual Risk**       | 12. Høj - Ingen kommandosanitetskontrol                       |
| **Anbefalinger**        | 13. Implementér kommandonormalisering, udvid blokeringslisten |

---

### 14. 3.4 Persistens (AML.TA0006)

#### 15. T-PERSIST-001: Installation af ondsindet skill

| Attribute               | Værdi                                                                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 18. AML.T0010.001 - Forsyningskædekompromittering: AI-software |
| **Description**         | 20. Angriber udgiver ondsindet skill på ClawHub                                                                |
| **Attack Vector**       | 22. Opret konto, udgiv skill med skjult ondsindet kode                                                         |
| **Affected Components** | 24. ClawHub, indlæsning af skills, agenteksekvering                                                            |
| **Current Mitigations** | 26. Verifikation af GitHub-kontoens alder, mønsterbaserede moderationsflag                                     |
| **Residual Risk**       | 28. Kritisk - Ingen sandboxing, begrænset gennemgang                                                           |
| **Anbefalinger**        | 29. VirusTotal-integration (under udvikling), skill-sandboxing, community-gennemgang        |

#### 30. T-PERSIST-002: Forgiftning af skill-opdatering

| Attribute               | Værdi                                                                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 33. AML.T0010.001 - Forsyningskædekompromittering: AI-software |
| **Description**         | 35. Angriber kompromitterer populær skill og udsender ondsindet opdatering                                     |
| **Attack Vector**       | 37. Kontokompromittering, social engineering af skill-ejer                                                     |
| **Affected Components** | 39. ClawHub-versionering, automatiske opdateringsflows                                                         |
| **Current Mitigations** | 41. Versionsfingeraftryk                                                                                       |
| **Residual Risk**       | 43. Høj - Automatiske opdateringer kan hente ondsindede versioner                                              |
| **Anbefalinger**        | 44. Implementér opdateringssignering, rollback-kapacitet, versionsfastlåsning                                  |

#### 45. T-PERSIST-003: Manipulation af agentkonfiguration

| Attribute               | Værdi                                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **ATLAS ID**            | 48. AML.T0010.002 - Forsyningskædekompromittering: Data |
| **Description**         | 50. Angriber ændrer agentkonfiguration for at opretholde adgang                                         |
| **Attack Vector**       | Config file modification, settings injection                                                                                   |
| **Affected Components** | Agent config, tool policies                                                                                                    |
| **Current Mitigations** | Filtilladelser                                                                                                                 |
| **Residual Risk**       | Medium - Requires local access                                                                                                 |
| **Anbefalinger**        | Config integrity verification, audit logging for config changes                                                                |

---

### 3.5 Defense Evasion (AML.TA0007)

#### T-EVADE-001: Moderation Pattern Bypass

| Attribute               | Værdi                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                        |
| **Description**         | Attacker crafts skill content to evade moderation patterns                                |
| **Attack Vector**       | Unicode homoglyphs, encoding tricks, dynamic loading                                      |
| **Affected Components** | ClawHub moderation.ts                                                     |
| **Current Mitigations** | Pattern-based FLAG_RULES                                             |
| **Residual Risk**       | High - Simple regex easily bypassed                                                       |
| **Anbefalinger**        | Add behavioral analysis (VirusTotal Code Insight), AST-based detection |

#### T-EVADE-002: Content Wrapper Escape

| Attribute               | Værdi                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data        |
| **Description**         | Attacker crafts content that escapes XML wrapper context  |
| **Attack Vector**       | Tag manipulation, context confusion, instruction override |
| **Affected Components** | External content wrapping                                 |
| **Current Mitigations** | XML tags + security notice                                |
| **Residual Risk**       | Medium - Novel escapes discovered regularly               |
| **Anbefalinger**        | Multiple wrapper layers, output-side validation           |

---

### 3.6 Discovery (AML.TA0008)

#### T-DISC-001: Tool Enumeration

| Attribute               | Værdi                                                               |
| ----------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access           |
| **Description**         | Attacker enumerates available tools through prompting               |
| **Attack Vector**       | "What tools do you have?" style queries                             |
| **Affected Components** | Agent tool registry                                                 |
| **Current Mitigations** | None specific                                                       |
| **Residual Risk**       | 4. Lav - Værktøjer er generelt dokumenterede |
| **Anbefalinger**        | 5. Overvej kontroller for værktøjssynlighed  |

#### 6. T-DISC-002: Udtrækning af sessionsdata

| Attribute               | Værdi                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                          |
| **Description**         | 11. Angriber udtrækker følsomme data fra sessionskonteksten |
| **Attack Vector**       | 13. "Hvad diskuterede vi?"-forespørgsler, kontekstsondering |
| **Affected Components** | Session transcripts, context window                                                |
| **Current Mitigations** | 17. Sessionsisolering pr. afsender          |
| **Residual Risk**       | 19. Middel - Data inden for sessionen er tilgængelige       |
| **Anbefalinger**        | 20. Implementér redigering af følsomme data i konteksten    |

---

### 21. 3.7 Indsamling & Eksfiltration (AML.TA0009, AML.TA0010)

#### 22. T-EXFIL-001: Datatyveri via web_fetch

| Attribute               | Værdi                                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                                                |
| **Description**         | Attacker exfiltrates data by instructing agent to send to external URL                                |
| **Attack Vector**       | 29. Prompt injection, der får agenten til at POSTe data til angriberens server |
| **Affected Components** | 31. web_fetch-værktøj                                     |
| **Current Mitigations** | 33. SSRF-blokering for interne netværk                                         |
| **Residual Risk**       | 35. Høj - Eksterne URL'er er tilladt                                           |
| **Anbefalinger**        | 36. Implementér URL-allowlisting og bevidsthed om dataklassificering           |

#### 37. T-EXFIL-002: Uautoriseret afsendelse af beskeder

| Attribute               | Værdi                                                                                               |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                                              |
| **Description**         | 42. Angriber får agenten til at sende beskeder, der indeholder følsomme data |
| **Attack Vector**       | 44. Prompt injection, der får agenten til at sende beskeder til angriberen   |
| **Affected Components** | 46. Beskedværktøj, kanal-integrationer                                       |
| **Current Mitigations** | 48. Begrænsning af udgående beskeder                                         |
| **Residual Risk**       | 50. Middel - Begrænsninger kan omgås                                         |
| **Anbefalinger**        | Require explicit confirmation for new recipients                                                    |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | Værdi                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                  |
| **Description**         | Malicious skill harvests credentials from agent context |
| **Attack Vector**       | Skill code reads environment variables, config files    |
| **Affected Components** | Skill execution environment                             |
| **Current Mitigations** | None specific to skills                                 |
| **Residual Risk**       | Critical - Skills run with agent privileges             |
| **Anbefalinger**        | Skill sandboxing, credential isolation                  |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001: Unauthorized Command Execution

| Attribute               | Værdi                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker executes arbitrary commands on user system  |
| **Attack Vector**       | Prompt injection combined with exec approval bypass  |
| **Affected Components** | Bash tool, command execution                         |
| **Current Mitigations** | Exec approvals, Docker sandbox option                |
| **Residual Risk**       | Critical - Host execution without sandbox            |
| **Anbefalinger**        | Default to sandbox, improve approval UX              |

#### T-IMPACT-002: Resource Exhaustion (DoS)

| Attribute               | Værdi                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker exhausts API credits or compute resources   |
| **Attack Vector**       | Automated message flooding, expensive tool calls     |
| **Affected Components** | Gateway, agent sessions, API provider                |
| **Current Mitigations** | None                                                 |
| **Residual Risk**       | High - No rate limiting                              |
| **Anbefalinger**        | Implement per-sender rate limits, cost budgets       |

#### T-IMPACT-003: Reputation Damage

| Attribute               | Værdi                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity    |
| **Description**         | Attacker causes agent to send harmful/offensive content |
| **Attack Vector**       | Prompt injection causing inappropriate responses        |
| **Affected Components** | Output generation, channel messaging                    |
| **Current Mitigations** | LLM provider content policies                           |
| **Residual Risk**       | Medium - Provider filters imperfect                     |
| **Anbefalinger**        | Output filtering layer, user controls                   |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Implementering                                                   | Effectiveness                                        |
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

| Improvement            | Status                                                   | Impact                                                                |
| ---------------------- | -------------------------------------------------------- | --------------------------------------------------------------------- |
| VirusTotal Integration | In Progress                                              | High - Code Insight behavioral analysis                               |
| Community Reporting    | Partial (`skillReports` table exists) | Medium                                                                |
| Audit Logging          | Partial (`auditLogs` table exists)    | Medium                                                                |
| Badge System           | Implemented                                              | Medium - `highlighted`, `official`, `deprecated`, `redactionApproved` |

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
| T-EVADE-001   | High       | Medium   | **Medium**   | P2       |
| T-ACCESS-001  | Low        | High     | **Medium**   | P2       |
| T-ACCESS-002  | Low        | High     | **Medium**   | P2       |
| T-PERSIST-002 | Low        | High     | **Medium**   | P2       |

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

| ID    | Recommendation                                  | Addresses                  |
| ----- | ----------------------------------------------- | -------------------------- |
| R-001 | Complete VirusTotal integration                 | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implement skill sandboxing                      | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Tilføj outputvalidering for følsomme handlinger | T-EXEC-001, T-EXEC-002     |

### 6.2 Kort sigt (P1)

| ID    | Recommendation                                                  | Addresses    |
| ----- | --------------------------------------------------------------- | ------------ |
| R-004 | Implementér hastighedsbegrænsning                               | T-IMPACT-002 |
| R-005 | Tilføj kryptering af tokens i hvile                             | T-ACCESS-003 |
| R-006 | Forbedr UX og validering for exec-godkendelse                   | T-EXEC-004   |
| R-007 | Implementér URL-allowlisting for web_fetch | T-EXFIL-001  |

### 6.3 Mellemlang sigt (P2)

| ID    | Recommendation                                             | Addresses     |
| ----- | ---------------------------------------------------------- | ------------- |
| R-008 | Tilføj kryptografisk kanalverifikation, hvor det er muligt | T-ACCESS-002  |
| R-009 | Implementér verifikation af konfigurationsintegritet       | T-PERSIST-003 |
| R-010 | Tilføj signering af opdateringer og versionsfastlåsning    | T-PERSIST-002 |

---

## 7. Bilag

### 7.1 ATLAS-teknikkortlægning

| ATLAS-ID                                      | Tekniknavn                                     | OpenClaw-trusler                                                 |
| --------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Aktiv scanning                                 | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Indsamling                                     | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Forsyningskæde: AI-software    | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Supply Chain: Data             | T-PERSIST-003                                                    |
| AML.T0031                     | Erode AI Model Integrity                       | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | AI Model Inference API Access                  | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Craft Adversarial Data                         | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM Prompt Injection: Direct   | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM Prompt Injection: Indirect | T-EXEC-002                                                       |

### 7.2 Key Security Files

| Sti                                 | Formål                      | Risk Level   |
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

| Term                 | Definition                                              |
| -------------------- | ------------------------------------------------------- |
| **ATLAS**            | MITRE's Adversarial Threat Landscape for AI Systems     |
| **ClawHub**          | OpenClaw's skill marketplace                            |
| **Gateway**          | OpenClaw's message routing and authentication layer     |
| **MCP**              | Model Context Protocol – værktøjsudbydergrænseflade     |
| **Prompt Injection** | Angreb, hvor ondsindede instruktioner indlejres i input |
| **Skill**            | Downloadbar udvidelse til OpenClaw-agenter              |
| **SSRF**             | Server-Side Request Forgery                             |

---

_This threat model is a living document. Rapportér sikkerhedsproblemer til security@openclaw.ai

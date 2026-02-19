# OpenClaw Dreigingsmodel v1.0

## MITRE ATLAS-framework

**Versie:** 1.0-draft  
**Laatst bijgewerkt:** 2026-02-04  
**Methodologie:** MITRE ATLAS + Data Flow Diagrams  
**Framework:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Framework-toeschrijving

Dit dreigingsmodel is gebaseerd op [MITRE ATLAS](https://atlas.mitre.org/), het industriestandaardframework voor het documenteren van adversariële dreigingen voor AI/ML-systemen. ATLAS wordt onderhouden door [MITRE](https://www.mitre.org/) in samenwerking met de AI-beveiligingsgemeenschap.

**Belangrijke ATLAS-bronnen:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Bijdragen aan ATLAS](https://atlas.mitre.org/resources/contribute)

### Bijdragen aan dit dreigingsmodel

Dit is een levend document dat wordt onderhouden door de OpenClaw-community. Zie [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) voor richtlijnen om bij te dragen:

- Nieuwe dreigingen melden
- Bestaande dreigingen bijwerken
- Proposing attack chains
- Mitigaties suggereren

---

## 1. Inleiding

### 1.1 Doel

Dit dreigingsmodel documenteert adversariële dreigingen voor het OpenClaw AI-agentplatform en de ClawHub-skillmarktplaats, met gebruik van het MITRE ATLAS-framework dat specifiek is ontworpen voor AI/ML-systemen.

### 1.2 Reikwijdte

| Component              | Inbegrepen   | Opmerkingen                                                      |
| ---------------------- | ------------ | ---------------------------------------------------------------- |
| OpenClaw Agent Runtime | Ja           | Kernuitvoering van agenten, toolaanroepen, sessies               |
| Gateway                | Ja           | Authenticatie, routering, kanaalintegratie                       |
| Kanaalintegraties      | Ja           | WhatsApp, Telegram, Discord, Signal, Slack, enz. |
| ClawHub Marktplaats    | Ja           | Publicatie, moderatie, distributie van skills                    |
| MCP-servers            | Ja           | Externe toolproviders                                            |
| Gebruikersapparaten    | Gedeeltelijk | Mobiele apps, desktopclients                                     |

### 1.3 Buiten scope

Er is niets expliciet buiten scope voor dit dreigingsmodel.

---

## 2. Systeemarchitectuur

### 2.1 Vertrouwensgrenzen

```
┌─────────────────────────────────────────────────────────────────┐
│                    NIET-VERTROUWDE ZONE                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│            VERTROUWENSGRENS 1: Kanaaltoegang                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Apparaatkoppeling (30s respijtperiode)                 │   │
│  │  • AllowFrom / AllowList-validatie                        │   │
│  │  • Token/Password/Tailscale-authenticatie                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          VERTROUWENSGRENS 2: Sessie-isolatie                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENTSESSIES                            │   │
│  │  • Sessiesleutel = agent:channel:peer                     │   │
│  │  • Toolbeleid per agent                                   │   │
│  │  • Transcriptlogging                                       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          VERTROUWENSGRENS 3: Tooluitvoering                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                UITVOERINGSSANDBOX                         │   │
│  │  • Docker-sandbox OF Host (exec-approvals)                │   │
│  │  • Node remote execution                                   │   │
│  │  • SSRF-bescherming (DNS-pinning + IP-blokkering)         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        VERTROUWENSGRENS 4: Externe content                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │        OPGEHAALDE URL's / E-MAILS / WEBHOOKS             │   │
│  │  • Verpakken van externe content (XML-tags)              │   │
│  │  • Injectie van beveiligingsmelding                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        VERTROUWENSGRENS 5: Supply chain                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Skillpublicatie (semver, SKILL.md verplicht)          │   │
│  │  • Patroon-gebaseerde moderatievlaggen                   │   │
│  │  • VirusTotal-scanning (binnenkort beschikbaar)          │   │
│  │  • Verificatie van GitHub-accountleeftijd                │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Datastromen

| Flow | Source  | Bestemming | Data                                     | Beveiliging         |
| ---- | ------- | ---------- | ---------------------------------------- | ------------------- |
| F1   | Kanaal  | Gateway    | Gebruikersberichten                      | TLS, AllowFrom      |
| F2   | Gateway | Agent      | Gerouteerde berichten                    | Sessie-isolatie     |
| F3   | Agent   | Tools      | Toolaanroepen                            | Beleidsafdwinging   |
| F4   | Agent   | Extern     | web_fetch-verzoeken | SSRF-blokkering     |
| F5   | ClawHub | Agent      | Skillcode                                | Moderatie, scanning |
| F6   | Agent   | Kanaal     | Antwoorden                               | Outputfiltering     |

---

## 3. Dreigingsanalyse per ATLAS-tactiek

### 3.1 Verkenning (AML.TA0002)

#### T-RECON-001: Ontdekking van agent-endpoints

| Attribuut                 | Waarde                                                                    |
| ------------------------- | ------------------------------------------------------------------------- |
| **ATLAS ID**              | AML.T0006 - Active Scanning                               |
| **Beschrijving**          | Aanvaller scant naar blootgestelde OpenClaw-gateway-endpoints             |
| **Aanvalsvector**         | Netwerkscans, shodan-queries, DNS-enumeratie                              |
| **Getroffen componenten** | Gateway, blootgestelde API-endpoints                                      |
| **Huidige mitigaties**    | Tailscale-authoptie, standaard binden aan loopback                        |
| **Restrisico**            | Middel - Publieke gateways zijn vindbaar                                  |
| **Aanbevelingen**         | Documenteer veilige uitrol, voeg rate limiting toe op discovery-endpoints |

#### T-RECON-002: Probing van kanaalintegraties

| Attribuut                 | Waarde                                                                 |
| ------------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**              | AML.T0006 - Active Scanning                            |
| **Beschrijving**          | Aanvaller test berichtkanalen om AI-beheerde accounts te identificeren |
| **Aanvalsvector**         | Testberichten sturen, responspatronen observeren                       |
| **Getroffen componenten** | Alle kanaalintegraties                                                 |
| **Huidige mitigaties**    | Geen specifieke                                                        |
| **Restrisico**            | Laag - Beperkte waarde van enkel ontdekking                            |
| **Aanbevelingen**         | Overweeg randomisatie van responstiming                                |

---

### 3.2 Initial Access (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | Waarde                                                    |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **Aanbevelingen**       | Reduce grace period, add confirmation step                |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | Waarde                                                                         |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker spoofs allowed sender identity in channel                             |
| **Attack Vector**       | Depends on channel - phone number spoofing, username impersonation             |
| **Affected Components** | AllowFrom validation per channel                                               |
| **Current Mitigations** | Channel-specific identity verification                                         |
| **Residual Risk**       | Medium - Some channels vulnerable to spoofing                                  |
| **Aanbevelingen**       | Document channel-specific risks, add cryptographic verification where possible |

#### T-ACCESS-003: Token Theft

| Attribute               | Waarde                                                                   |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                |
| **Description**         | Attacker steals authentication tokens from config files                  |
| **Attack Vector**       | Malware, unauthorized device access, config backup exposure              |
| **Affected Components** | ~/.openclaw/credentials/, config storage |
| **Current Mitigations** | Bestandsrechten                                                          |
| **Residual Risk**       | Hoog - Tokens worden in platte tekst opgeslagen                          |
| **Aanbevelingen**       | Implementeer tokenversleuteling in rust, voeg tokenrotatie toe           |

---

### 3.3 Uitvoering (AML.TA0005)

#### T-EXEC-001: Directe Promptinjectie

| Attribute               | Waarde                                                                                              |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct        |
| **Description**         | Aanvaller stuurt zorgvuldig opgestelde prompts om het gedrag van de agent te manipuleren            |
| **Attack Vector**       | Kanaalberichten met adversariële instructies                                                        |
| **Affected Components** | Agent LLM, alle invoeroppervlakken                                                                  |
| **Current Mitigations** | Patroondetectie, omhulling van externe inhoud                                                       |
| **Residual Risk**       | Kritiek - Alleen detectie, geen blokkering; geavanceerde aanvallen omzeilen dit                     |
| **Aanbevelingen**       | Implementeer meerlaagse verdediging, uitvoer­validatie, gebruikersbevestiging voor gevoelige acties |

#### T-EXEC-002: Indirecte Promptinjectie

| Attribute               | Waarde                                                                                         |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - LLM Prompt Injection: Indirect |
| **Description**         | Aanvaller verbergt kwaadaardige instructies in opgehaalde inhoud                               |
| **Attack Vector**       | Kwaadaardige URL's, vergiftigde e-mails, gecompromitteerde webhooks                            |
| **Affected Components** | web_fetch, e-mailinname, externe gegevensbronnen                          |
| **Current Mitigations** | Inhoudsomsluiting met XML-tags en beveiligingsmelding                                          |
| **Residual Risk**       | Hoog - LLM kan wrapper-instructies negeren                                                     |
| **Aanbevelingen**       | Implementeer inhoudssanitisatie, gescheiden uitvoeringscontexten                               |

#### T-EXEC-003: Toolargument-injectie

| Attribute               | Waarde                                                                                       |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct |
| **Description**         | Aanvaller manipuleert toolargumenten via promptinjectie                                      |
| **Attack Vector**       | Zorgvuldig opgestelde prompts die de waarden van toolparameters beïnvloeden                  |
| **Affected Components** | Alle toolaanroepen                                                                           |
| **Current Mitigations** | Exec-goedkeuringen voor gevaarlijke opdrachten                                               |
| **Residual Risk**       | Hoog - Vertrouwt op gebruikersoordeel                                                        |
| **Aanbevelingen**       | Implementeer argumentvalidatie, geparameteriseerde toolaanroepen                             |

#### T-EXEC-004: Omzeiling van Exec-goedkeuring

| Attribute               | Waarde                                                     |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data         |
| **Description**         | Attacker crafts commands that bypass approval allowlist    |
| **Attack Vector**       | Command obfuscation, alias exploitation, path manipulation |
| **Affected Components** | exec-approvals.ts, command allowlist       |
| **Current Mitigations** | Allowlist + ask mode                                       |
| **Residual Risk**       | High - No command sanitization                             |
| **Aanbevelingen**       | Implement command normalization, expand blocklist          |

---

### 3.4 Persistence (AML.TA0006)

#### T-PERSIST-001: Malicious Skill Installation

| Attribute               | Waarde                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker publishes malicious skill to ClawHub                                                        |
| **Attack Vector**       | Create account, publish skill with hidden malicious code                                             |
| **Affected Components** | ClawHub, skill loading, agent execution                                                              |
| **Current Mitigations** | GitHub account age verification, pattern-based moderation flags                                      |
| **Residual Risk**       | Critical - No sandboxing, limited review                                                             |
| **Aanbevelingen**       | VirusTotal integration (in progress), skill sandboxing, community review          |

#### T-PERSIST-002: Skill Update Poisoning

| Attribute               | Waarde                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker compromises popular skill and pushes malicious update                                       |
| **Attack Vector**       | Account compromise, social engineering of skill owner                                                |
| **Affected Components** | ClawHub versioning, auto-update flows                                                                |
| **Current Mitigations** | Version fingerprinting                                                                               |
| **Residual Risk**       | High - Auto-updates may pull malicious versions                                                      |
| **Aanbevelingen**       | Implement update signing, rollback capability, version pinning                                       |

#### T-PERSIST-003: Agent Configuration Tampering

| Attribute               | Waarde                                                                                        |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Supply Chain Compromise: Data |
| **Description**         | Attacker modifies agent configuration to persist access                                       |
| **Attack Vector**       | Config file modification, settings injection                                                  |
| **Affected Components** | Agent config, tool policies                                                                   |
| **Current Mitigations** | Bestandsrechten                                                                               |
| **Residual Risk**       | Medium - Requires local access                                                                |
| **Aanbevelingen**       | Config integrity verification, audit logging for config changes                               |

---

### 3.5 Defense Evasion (AML.TA0007)

#### T-EVADE-001: Moderation Pattern Bypass

| Attribute               | Waarde                                                                                    |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                        |
| **Description**         | Attacker crafts skill content to evade moderation patterns                                |
| **Attack Vector**       | Unicode homoglyphs, encoding tricks, dynamic loading                                      |
| **Affected Components** | ClawHub moderation.ts                                                     |
| **Current Mitigations** | Pattern-based FLAG_RULES                                             |
| **Residual Risk**       | High - Simple regex easily bypassed                                                       |
| **Aanbevelingen**       | Add behavioral analysis (VirusTotal Code Insight), AST-based detection |

#### T-EVADE-002: Content Wrapper Escape

| Attribute               | Waarde                                                    |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data        |
| **Description**         | Attacker crafts content that escapes XML wrapper context  |
| **Attack Vector**       | Tag manipulation, context confusion, instruction override |
| **Affected Components** | External content wrapping                                 |
| **Current Mitigations** | XML tags + security notice                                |
| **Residual Risk**       | Medium - Novel escapes discovered regularly               |
| **Aanbevelingen**       | Multiple wrapper layers, output-side validation           |

---

### 3.6 Discovery (AML.TA0008)

#### T-DISC-001: Tool Enumeration

| Attribute               | Waarde                                                    |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker enumerates available tools through prompting     |
| **Attack Vector**       | "What tools do you have?" style queries                   |
| **Affected Components** | Agent tool registry                                       |
| **Current Mitigations** | None specific                                             |
| **Residual Risk**       | Low - Tools generally documented                          |
| **Aanbevelingen**       | Overweeg controles voor toolzichtbaarheid                 |

#### T-DISC-002: Extractie van Sessiedata

| Attribute               | Waarde                                                     |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access  |
| **Description**         | Aanvaller extraheert gevoelige data uit de sessiecontext   |
| **Attack Vector**       | "Waar hebben we het over gehad?"-vragen, contextverkenning |
| **Affected Components** | Sessietranscripten, contextvenster                         |
| **Current Mitigations** | Session isolation per sender                               |
| **Residual Risk**       | Gemiddeld - Data binnen de sessie is toegankelijk          |
| **Aanbevelingen**       | Implement sensitive data redaction in context              |

---

### 3.7 Verzameling & Exfiltratie (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Datadiefstal via web_fetch

| Attribute               | Waarde                                                                 |
| ----------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Verzameling                                |
| **Description**         | Attacker exfiltrates data by instructing agent to send to external URL |
| **Attack Vector**       | Promptinjectie waardoor de agent data POST naar een aanvallerserver    |
| **Affected Components** | web_fetch-tool                                    |
| **Current Mitigations** | SSRF-blokkering voor interne netwerken                                 |
| **Residual Risk**       | Hoog - Externe URL’s toegestaan                                        |
| **Aanbevelingen**       | Implementeer URL-allowlisting en bewustzijn van dataclassificatie      |

#### T-EXFIL-002: Ongeautoriseerd Berichten Verzenden

| Attribute               | Waarde                                                                   |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0009 - Verzameling                                  |
| **Description**         | Aanvaller veroorzaakt dat de agent berichten met gevoelige data verzendt |
| **Attack Vector**       | Promptinjectie waardoor de agent de aanvaller een bericht stuurt         |
| **Affected Components** | Berichtentool, kanaalintegraties                                         |
| **Current Mitigations** | Beperking van uitgaande berichten                                        |
| **Residual Risk**       | Gemiddeld - Beperkingen kunnen worden omzeild                            |
| **Aanbevelingen**       | Expliciete bevestiging vereisen voor nieuwe ontvangers                   |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | Waarde                                                     |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Verzameling                    |
| **Description**         | Malicious skill harvests credentials from agent context    |
| **Attack Vector**       | Skillcode leest omgevingsvariabelen, configuratiebestanden |
| **Affected Components** | Skill-uitvoeringsomgeving                                  |
| **Current Mitigations** | None specific to skills                                    |
| **Residual Risk**       | Kritiek - Skills draaien met agentprivileges               |
| **Aanbevelingen**       | Skill-sandboxing, isolatie van referentiegegevens          |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001: Ongeautoriseerde commando-uitvoering

| Attribute               | Waarde                                                               |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity                 |
| **Description**         | Aanvaller voert willekeurige commando’s uit op het gebruikerssysteem |
| **Attack Vector**       | Promptinjectie gecombineerd met het omzeilen van exec-goedkeuring    |
| **Affected Components** | Bash-tool, commando-uitvoering                                       |
| **Current Mitigations** | Exec-goedkeuringen, Docker-sandboxoptie                              |
| **Residual Risk**       | Kritiek - Hostuitvoering zonder sandbox                              |
| **Aanbevelingen**       | Standaard naar sandbox, verbeter de goedkeurings-UX                  |

#### T-IMPACT-002: Uitputting van middelen (DoS)

| Attribute               | Waarde                                                   |
| ----------------------- | -------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity     |
| **Description**         | Aanvaller put API-tegoed of rekenresources uit           |
| **Attack Vector**       | Geautomatiseerde berichtoverstroming, dure toolaanroepen |
| **Affected Components** | Gateway, agentsessies, API-provider                      |
| **Current Mitigations** | Geen                                                     |
| **Residual Risk**       | Hoog - Geen rate limiting                                |
| **Aanbevelingen**       | Implementeer rate limits per afzender, kostenbudgetten   |

#### T-IMPACT-003: Reputatieschade

| Attribute               | Waarde                                                  |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity    |
| **Description**         | Attacker causes agent to send harmful/offensive content |
| **Attack Vector**       | Prompt injection causing inappropriate responses        |
| **Affected Components** | Output generation, channel messaging                    |
| **Current Mitigations** | LLM provider content policies                           |
| **Residual Risk**       | Medium - Provider filters imperfect                     |
| **Aanbevelingen**       | Output filtering layer, user controls                   |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Implementatie                                                    | Effectiveness                                        |
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

| ID    | Waarde                                         | Addresses                  |
| ----- | ---------------------------------------------- | -------------------------- |
| R-001 | Complete VirusTotal integration                | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implement skill sandboxing                     | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Voeg outputvalidatie toe voor gevoelige acties | T-EXEC-001, T-EXEC-002     |

### 6.2 Kortetermijn (P1)

| ID    | Aanvalsketens voorstellen                                         | Addresses    |
| ----- | ----------------------------------------------------------------- | ------------ |
| R-004 | Implementeer rate limiting                                        | T-IMPACT-002 |
| R-005 | Voeg tokenversleuteling in rust toe                               | T-ACCESS-003 |
| R-006 | Verbeter de UX en validatie voor exec-goedkeuring                 | T-EXEC-004   |
| R-007 | Implementeer URL-allowlisting voor web_fetch | T-EXFIL-001  |

### 6.3 Middellangetermijn (P2)

| ID    | Waarde                                                   | Addresses     |
| ----- | -------------------------------------------------------- | ------------- |
| R-008 | Voeg cryptografische kanaalverificatie toe waar mogelijk | T-ACCESS-002  |
| R-009 | Implementeer verificatie van configuratie-integriteit    | T-PERSIST-003 |
| R-010 | Voeg updatesigning en versiepinning toe                  | T-PERSIST-002 |

---

## 7. Bijlagen

### 7.1 ATLAS-techniekmapping

| ATLAS-ID                                      | Technieknaam                                   | OpenClaw-bedreigingen                                            |
| --------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Actief scannen                                 | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Verzameling                                    | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Toeleveringsketen: AI-software | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Toeleveringsketen: Data        | T-PERSIST-003                                                    |
| AML.T0031                     | Aantasting van de integriteit van AI-modellen  | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | Toegang tot AI-modelinference-API              | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Craft Adversarial Data                         | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM Prompt Injection: Direct   | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM Prompt Injection: Indirect | T-EXEC-002                                                       |

### 7.2 Belangrijke beveiligingsbestanden

| Pad                                 | Doel                                   | Risk Level   |
| ----------------------------------- | -------------------------------------- | ------------ |
| `src/infra/exec-approvals.ts`       | Logica voor goedkeuring van opdrachten | **Critical** |
| `src/gateway/auth.ts`               | Gateway-authenticatie                  | **Critical** |
| `src/web/inbound/access-control.ts` | Toegangscontrole per kanaal            | **Critical** |
| `src/infra/net/ssrf.ts`             | SSRF-bescherming                       | **Critical** |
| `src/security/external-content.ts`  | Mitigatie van prompt injection         | **Critical** |
| `src/agents/sandbox/tool-policy.ts` | Handhaving van toolbeleid              | **Critical** |
| `convex/lib/moderation.ts`          | ClawHub-moderatie                      | **High**     |
| `convex/lib/skillPublish.ts`        | Proces voor het publiceren van skills  | **High**     |
| `src/routing/resolve-route.ts`      | Sessiescheiding                        | **Medium**   |

### 7.3 Woordenlijst

| Term                 | Definitie                                                 |
| -------------------- | --------------------------------------------------------- |
| **ATLAS**            | Het Adversarial Threat Landscape for AI Systems van MITRE |
| **ClawHub**          | De skillmarktplaats van OpenClaw                          |
| **Gateway**          | De berichtroutering- en authenticatielaag van OpenClaw    |
| **MCP**              | Model Context Protocol - tool provider interface          |
| **Prompt Injection** | Attack where malicious instructions are embedded in input |
| **Skill**            | Downloadable extension for OpenClaw agents                |
| **SSRF**             | Server-Side Request Forgery                               |

---

_This threat model is a living document. Report security issues to security@openclaw.ai_

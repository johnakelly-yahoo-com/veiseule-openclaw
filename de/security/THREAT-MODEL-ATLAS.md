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

| Komponente             | Included | Notes                                                            |
| ---------------------- | -------- | ---------------------------------------------------------------- |
| OpenClaw Agent Runtime | Ja       | Core agent execution, tool calls, sessions                       |
| Gateway                | Ja       | Authentication, routing, channel integration                     |
| Channel Integrations   | Ja       | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| ClawHub Marketplace    | Ja       | Skill publishing, moderation, distribution                       |
| MCP Servers            | Ja       | External tool providers                                          |
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
| F3   | Agent   | „Werkzeuge“ | Tool invocations                        | Policy enforcement   |
| F4   | Agent   | External    | web_fetch requests | SSRF blocking        |
| F5   | ClawHub | Agent       | Skill code                              | Moderation, scanning |
| F6   | Agent   | Channel     | Responses                               | Output filtering     |

---

## 3. Threat Analysis by ATLAS Tactic

### 3.1 Reconnaissance (AML.TA0002)

#### T-RECON-001: Agent Endpoint Discovery

| Attribute               | Wert                                                                 |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                          |
| **Description**         | Attacker scans for exposed OpenClaw gateway endpoints                |
| **Attack Vector**       | Network scanning, shodan queries, DNS enumeration                    |
| **Affected Components** | Gateway, exposed API endpoints                                       |
| **Current Mitigations** | Tailscale auth option, bind to loopback by default                   |
| **Residual Risk**       | Medium - Public gateways discoverable                                |
| **Empfehlungen**        | Document secure deployment, add rate limiting on discovery endpoints |

#### T-RECON-002: Channel Integration Probing

| Attribute               | Wert                                                               |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Active Scanning                        |
| **Description**         | Attacker probes messaging channels to identify AI-managed accounts |
| **Attack Vector**       | Sending test messages, observing response patterns                 |
| **Affected Components** | All channel integrations                                           |
| **Current Mitigations** | None specific                                                      |
| **Residual Risk**       | Low - Limited value from discovery alone                           |
| **Empfehlungen**        | Consider response timing randomization                             |

---

### 3.2 Initial Access (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | Wert                                                      |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **Empfehlungen**        | Reduce grace period, add confirmation step                |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | Wert                                                                           |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker spoofs allowed sender identity in channel                             |
| **Attack Vector**       | Depends on channel - phone number spoofing, username impersonation             |
| **Affected Components** | AllowFrom validation per channel                                               |
| **Current Mitigations** | Channel-specific identity verification                                         |
| **Residual Risk**       | Medium - Some channels vulnerable to spoofing                                  |
| **Empfehlungen**        | Document channel-specific risks, add cryptographic verification where possible |

#### T-ACCESS-003: Token Theft

| Attribute               | Wert                                                                           |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker steals authentication tokens from config files                        |
| **Attack Vector**       | Malware, unauthorized device access, config backup exposure                    |
| **Affected Components** | ~/.openclaw/credentials/, config storage       |
| **Current Mitigations** | Dateiberechtigungen                                                            |
| **Residual Risk**       | Hoch - Tokens werden im Klartext gespeichert                                   |
| **Empfehlungen**        | Token-Verschlüsselung im Ruhezustand implementieren, Token-Rotation hinzufügen |

---

### 3.3 Ausführung (AML.TA0005)

#### T-EXEC-001: Direkte Prompt-Injektion

| Attribute               | Wert                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt-Injektion: Direkt              |
| **Description**         | Angreifer sendet manipulierte Prompts, um das Verhalten des Agenten zu beeinflussen                       |
| **Attack Vector**       | Kanalnachrichten mit gegnerischen Anweisungen                                                             |
| **Affected Components** | Agent LLM, alle Eingabeoberflächen                                                                        |
| **Current Mitigations** | Mustererkennung, Einbettung externer Inhalte                                                              |
| **Residual Risk**       | Kritisch - Nur Erkennung, keine Blockierung; ausgefeilte Angriffe umgehen dies                            |
| **Empfehlungen**        | Mehrschichtige Verteidigung, Ausgabevalidierung, Benutzerbestätigung für sensible Aktionen implementieren |

#### 19. T-EXEC-002: Indirekte Prompt-Injektion

| Attribute               | Wert                                                                                           |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - LLM Prompt Injection: Indirect |
| **Description**         | Angreifer bettet bösartige Anweisungen in abgerufene Inhalte ein                               |
| **Attack Vector**       | Bösartige URLs, manipulierte E-Mails, kompromittierte Webhooks                                 |
| **Affected Components** | web_fetch, email ingestion, external data sources                         |
| **Current Mitigations** | Content wrapping with XML tags and security notice                                             |
| **Residual Risk**       | Hoch - LLM könnte Wrapper-Anweisungen ignorieren                                               |
| **Empfehlungen**        | Inhaltssanitisierung implementieren, getrennte Ausführungskontexte                             |

#### T-EXEC-003: Tool-Argument-Injektion

| Attribute               | Wert                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt-Injektion: Direkt |
| **Description**         | Angreifer manipuliert Tool-Argumente durch Prompt-Injektion                                  |
| **Attack Vector**       | Manipulierte Prompts, die Tool-Parameterwerte beeinflussen                                   |
| **Affected Components** | Alle Tool-Aufrufe                                                                            |
| **Current Mitigations** | Ausführungsfreigaben für gefährliche Befehle                                                 |
| **Residual Risk**       | Hoch - Abhängigkeit vom Urteilsvermögen des Benutzers                                        |
| **Empfehlungen**        | Argumentvalidierung implementieren, parametrisierte Tool-Aufrufe                             |

#### T-EXEC-004: Umgehung der Ausführungsfreigabe

| Attribute               | Wert                                                               |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                 |
| **Description**         | Angreifer erstellt Befehle, die die Genehmigungs-Allowlist umgehen |
| **Attack Vector**       | Befehlsverschleierung, Alias-Ausnutzung, Pfadmanipulation          |
| **Affected Components** | exec-approvals.ts, Befehls-Allowlist               |
| **Current Mitigations** | Allowlist + Ask-Modus                                              |
| **Residual Risk**       | Hoch - Keine Befehlsbereinigung                                    |
| **Empfehlungen**        | Implement command normalization, expand blocklist                  |

---

### 3.4 Persistenz (AML.TA0006)

#### T-PERSIST-001: Installation bösartiger Skills

| Attribute               | Wert                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Lieferkettenkompromittierung: KI-Software |
| **Description**         | Angreifer veröffentlicht bösartigen Skill auf ClawHub                                                     |
| **Attack Vector**       | Konto erstellen, Skill mit verstecktem bösartigem Code veröffentlichen                                    |
| **Affected Components** | ClawHub, Skill-Ladevorgang, Agentenausführung                                                             |
| **Current Mitigations** | Überprüfung des GitHub-Kontoalters, musterbasierte Moderations-Flags                                      |
| **Residual Risk**       | Kritisch - Kein Sandboxing, eingeschränkte Überprüfung                                                    |
| **Empfehlungen**        | VirusTotal-Integration (in Arbeit), Skill-Sandboxing, Community-Review                 |

#### T-PERSIST-002: Vergiftung von Skill-Updates

| Attribute               | Wert                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Lieferkettenkompromittierung: KI-Software |
| **Description**         | Angreifer kompromittiert einen beliebten Skill und veröffentlicht ein bösartiges Update                   |
| **Attack Vector**       | Kontokompromittierung, Social Engineering des Skill-Besitzers                                             |
| **Affected Components** | ClawHub-Versionierung, Auto-Update-Abläufe                                                                |
| **Current Mitigations** | Versions-Fingerprinting                                                                                   |
| **Residual Risk**       | Hoch - Auto-Updates könnten bösartige Versionen beziehen                                                  |
| **Empfehlungen**        | Update-Signierung implementieren, Rollback-Fähigkeit, Versionsfixierung                                   |

#### T-PERSIST-003: Manipulation der Agentenkonfiguration

| Attribute               | Wert                                                                                                |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Lieferkettenkompromittierung: Daten |
| **Description**         | Angreifer verändert die Agentenkonfiguration, um Zugriff dauerhaft zu erhalten                      |
| **Attack Vector**       | Config file modification, settings injection                                                        |
| **Affected Components** | Agent config, tool policies                                                                         |
| **Current Mitigations** | Dateiberechtigungen                                                                                 |
| **Residual Risk**       | Medium - Requires local access                                                                      |
| **Empfehlungen**        | Config integrity verification, audit logging for config changes                                     |

---

### 3.5 Defense Evasion (AML.TA0007)

#### T-EVADE-001: Moderation Pattern Bypass

| Attribute               | Wert                                                                                      |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                        |
| **Description**         | Attacker crafts skill content to evade moderation patterns                                |
| **Attack Vector**       | Unicode homoglyphs, encoding tricks, dynamic loading                                      |
| **Affected Components** | ClawHub moderation.ts                                                     |
| **Current Mitigations** | Pattern-based FLAG_RULES                                             |
| **Residual Risk**       | High - Simple regex easily bypassed                                                       |
| **Empfehlungen**        | Add behavioral analysis (VirusTotal Code Insight), AST-based detection |

#### T-EVADE-002: Content Wrapper Escape

| Attribute               | Wert                                                      |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data        |
| **Description**         | Attacker crafts content that escapes XML wrapper context  |
| **Attack Vector**       | Tag manipulation, context confusion, instruction override |
| **Affected Components** | External content wrapping                                 |
| **Current Mitigations** | XML tags + security notice                                |
| **Residual Risk**       | Medium - Novel escapes discovered regularly               |
| **Empfehlungen**        | Multiple wrapper layers, output-side validation           |

---

### 3.6 Discovery (AML.TA0008)

#### T-DISC-001: Tool Enumeration

| Attribute               | Wert                                                      |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker enumerates available tools through prompting     |
| **Attack Vector**       | "What tools do you have?" style queries                   |
| **Affected Components** | Agent tool registry                                       |
| **Current Mitigations** | None specific                                             |
| **Residual Risk**       | Niedrig – Werkzeuge sind in der Regel dokumentiert        |
| **Empfehlungen**        | Werkzeug-Sichtbarkeitskontrollen in Betracht ziehen       |

#### T-DISC-002: Extraktion von Sitzungsdaten

| Attribute               | Wert                                                        |
| ----------------------- | ----------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access   |
| **Description**         | Angreifer extrahiert sensible Daten aus dem Sitzungskontext |
| **Attack Vector**       | "Worüber haben wir gesprochen?"-Anfragen, Kontextsondierung |
| **Affected Components** | Sitzungsprotokolle, Kontextfenster                          |
| **Current Mitigations** | Sitzungsisolierung pro Absender                             |
| **Residual Risk**       | Mittel – Daten innerhalb der Sitzung zugänglich             |
| **Empfehlungen**        | Implementierung einer Schwärzung sensibler Daten im Kontext |

---

### 3.7 Collection & Exfiltration (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Datendiebstahl über web_fetch

| Attribute               | Wert                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 – Sammlung                                                         |
| **Description**         | Angreifer exfiltriert Daten, indem er den Agenten anweist, sie an eine externe URL zu senden |
| **Attack Vector**       | Prompt-Injection veranlasst den Agenten, Daten per POST an den Angreifer-Server zu senden    |
| **Affected Components** | web_fetch-Werkzeug                                                      |
| **Current Mitigations** | SSRF-Blockierung für interne Netzwerke                                                       |
| **Residual Risk**       | High - External URLs permitted                                                               |
| **Empfehlungen**        | Implementierung von URL-Allowlisting und Bewusstsein für Datenklassifizierung                |

#### T-EXFIL-002: Unautorisierter Nachrichtenversand

| Attribute               | Wert                                                                         |
| ----------------------- | ---------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 – Sammlung                                         |
| **Description**         | Angreifer veranlasst den Agenten, Nachrichten mit sensiblen Daten zu senden  |
| **Attack Vector**       | Prompt-Injection veranlasst den Agenten, dem Angreifer Nachrichten zu senden |
| **Affected Components** | Nachrichtenwerkzeug, Kanal-Integrationen                                     |
| **Current Mitigations** | Steuerung des ausgehenden Nachrichtenversands                                |
| **Residual Risk**       | Mittel – Steuerung kann umgangen werden                                      |
| **Empfehlungen**        | Require explicit confirmation for new recipients                             |

#### T-EXFIL-003: Abgreifen von Zugangsdaten

| Attribute               | Wert                                                            |
| ----------------------- | --------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 – Sammlung                            |
| **Description**         | Bösartige Fähigkeit sammelt Zugangsdaten aus dem Agentenkontext |
| **Attack Vector**       | Fähigkeitscode liest Umgebungsvariablen, Konfigurationsdateien  |
| **Affected Components** | Ausführungsumgebung der Fähigkeit                               |
| **Current Mitigations** | Keine speziell für Fähigkeiten                                  |
| **Residual Risk**       | Kritisch – Fähigkeiten laufen mit Agentenprivilegien            |
| **Empfehlungen**        | Sandboxing von Fähigkeiten, Isolierung von Zugangsdaten         |

---

### 3.8 Auswirkung (AML.TA0011)

#### T-IMPACT-001: Unautorisierte Befehlsausführung

| Attribute               | Wert                                                             |
| ----------------------- | ---------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity             |
| **Description**         | Angreifer führt beliebige Befehle auf dem Benutzersystem aus     |
| **Attack Vector**       | Prompt-Injection kombiniert mit Umgehung der Ausführungsfreigabe |
| **Affected Components** | Bash-Tool, Befehlsausführung                                     |
| **Current Mitigations** | Ausführungsfreigaben, Docker-Sandbox-Option                      |
| **Residual Risk**       | Kritisch – Host-Ausführung ohne Sandbox                          |
| **Empfehlungen**        | Standardmäßig Sandbox verwenden, Freigabe-UX verbessern          |

#### T-IMPACT-002: Ressourcenerschöpfung (DoS)

| Attribute               | Wert                                                   |
| ----------------------- | ------------------------------------------------------ |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity   |
| **Description**         | Angreifer erschöpft API-Guthaben oder Rechenressourcen |
| **Attack Vector**       | Automatisiertes Nachrichtenfluten, teure Tool-Aufrufe  |
| **Affected Components** | Gateway, Agentensitzungen, API-Anbieter                |
| **Current Mitigations** | Keine                                                  |
| **Residual Risk**       | Hoch – Keine Ratenbegrenzung                           |
| **Empfehlungen**        | Pro-Absender-Ratenlimits, Kostenbudgets implementieren |

#### T-IMPACT-003: Rufschädigung

| Attribute               | Wert                                                    |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity    |
| **Description**         | Attacker causes agent to send harmful/offensive content |
| **Attack Vector**       | Prompt injection causing inappropriate responses        |
| **Affected Components** | Output generation, channel messaging                    |
| **Current Mitigations** | LLM provider content policies                           |
| **Residual Risk**       | Medium - Provider filters imperfect                     |
| **Empfehlungen**        | Output filtering layer, user controls                   |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Implementierung                                                  | Effectiveness                                        |
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

| Improvement            | Status                                                     | Impact                                                                |
| ---------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------- |
| VirusTotal Integration | In Progress                                                | Hoch – Code-Insight-Verhaltensanalyse                                 |
| Community-Meldungen    | Teilweise (`skillReports` table exists) | Medium                                                                |
| Audit-Protokollierung  | Partial (`auditLogs` table exists)      | Medium                                                                |
| Badge System           | Implemented                                                | Mittel – `highlighted`, `official`, `deprecated`, `redactionApproved` |

---

## 5. Risikomatrix

### 5.1 Eintrittswahrscheinlichkeit vs. Auswirkung

| Bedrohungs-ID | Likelihood | Impact   | Risikostufe  | Priorität |
| ------------- | ---------- | -------- | ------------ | --------- |
| T-EXEC-001    | High       | Kritisch | **Critical** | P0        |
| T-PERSIST-001 | High       | Kritisch | **Critical** | P0        |
| T-EXFIL-003   | Medium     | Kritisch | **Critical** | P0        |
| T-IMPACT-001  | Medium     | Kritisch | **High**     | P1        |
| T-EXEC-002    | High       | High     | **High**     | P1        |
| T-EXEC-004    | Medium     | High     | **High**     | P1        |
| T-ACCESS-003  | Medium     | High     | **High**     | P1        |
| T-EXFIL-001   | Medium     | High     | **High**     | P1        |
| T-IMPACT-002  | High       | Medium   | **High**     | P1        |
| T-EVADE-001   | High       | Medium   | **Medium**   | P2        |
| T-ACCESS-001  | Low        | High     | **Medium**   | P2        |
| T-ACCESS-002  | Low        | High     | **Medium**   | P2        |
| T-PERSIST-002 | Low        | High     | **Medium**   | P2        |

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

| ID    | Recommendation                                      | Addresses                  |
| ----- | --------------------------------------------------- | -------------------------- |
| R-001 | Complete VirusTotal integration                     | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implement skill sandboxing                          | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Ausgabevalidierung für sensible Aktionen hinzufügen | T-EXEC-001, T-EXEC-002     |

### 6.2 Short-term (P1)

| ID    | Recommendation                                                     | Addresses    |
| ----- | ------------------------------------------------------------------ | ------------ |
| R-004 | Ratenbegrenzung implementieren                                     | T-IMPACT-002 |
| R-005 | Add token encryption at rest                                       | T-ACCESS-003 |
| R-006 | UX für Exec-Freigaben verbessern und Validierung erweitern         | T-EXEC-004   |
| R-007 | URL-Allowlisting für web_fetch implementieren | T-EXFIL-001  |

### 6.3 Medium-term (P2)

| ID    | Recommendation                                           | Addresses     |
| ----- | -------------------------------------------------------- | ------------- |
| R-008 | Kryptografische Kanalverifikation hinzufügen, wo möglich | T-ACCESS-002  |
| R-009 | Konfigurationsintegritätsprüfung implementieren          | T-PERSIST-003 |
| R-010 | Update-Signierung und Versionsfixierung hinzufügen       | T-PERSIST-002 |

---

## 7. Anhänge

### 7.1 ATLAS-Technikzuordnung

| ATLAS-ID                                      | Technikname                                    | OpenClaw-Bedrohungen                                             |
| --------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Aktives Scannen                                | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Sammlung                                       | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Lieferkette: KI-Software       | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Lieferkette: Daten             | T-PERSIST-003                                                    |
| AML.T0031                     | Integrität von KI-Modellen untergraben         | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | Zugriff auf die Inferenz-API des KI-Modells    | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Adversarielle Daten erstellen                  | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM-Prompt-Injection: Direkt   | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM-Prompt-Injection: Indirekt | T-EXEC-002                                                       |

### 7.2 Zentrale Sicherheitsdateien

| Pfad                                | Zweck                                 | Risikostufe  |
| ----------------------------------- | ------------------------------------- | ------------ |
| `src/infra/exec-approvals.ts`       | Logik zur Befehlsfreigabe             | **Critical** |
| `src/gateway/auth.ts`               | Gateway-Authentifizierung             | **Critical** |
| `src/web/inbound/access-control.ts` | Kanalzugriffskontrolle                | **Critical** |
| `src/infra/net/ssrf.ts`             | SSRF-Schutz                           | **Critical** |
| `src/security/external-content.ts`  | Gegenmaßnahmen gegen Prompt-Injection | **Critical** |
| `src/agents/sandbox/tool-policy.ts` | Durchsetzung der Tool-Richtlinie      | **Critical** |
| `convex/lib/moderation.ts`          | ClawHub-Moderation                    | **High**     |
| `convex/lib/skillPublish.ts`        | Ablauf der Skill-Veröffentlichung     | **High**     |
| `src/routing/resolve-route.ts`      | Session isolation                     | **Medium**   |

### 7.3 Glossar

| Begriff              | Definition                                                            |
| -------------------- | --------------------------------------------------------------------- |
| **ATLAS**            | MITREs adversarische Bedrohungslandschaft für KI-Systeme              |
| **ClawHub**          | OpenClaws Skill-Marktplatz                                            |
| **Gateway**          | OpenClaws Nachrichten-Routing- und Authentifizierungsschicht          |
| **MCP**              | Model Context Protocol – Schnittstelle für Tool-Anbieter              |
| **Prompt Injection** | Angriff, bei dem bösartige Anweisungen in Eingaben eingebettet werden |
| **Skill**            | Herunterladbare Erweiterung für OpenClaw-Agenten                      |
| **SSRF**             | Server-Side Request Forgery                                           |

---

_Dieses Bedrohungsmodell ist ein lebendes Dokument. Sicherheitsprobleme an security@openclaw.ai melden_


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

| Componente             | Included | Notas                                                            |
| ---------------------- | -------- | ---------------------------------------------------------------- |
| OpenClaw Agent Runtime | Sí       | Core agent execution, tool calls, sessions                       |
| Gateway                | Sí       | Authentication, routing, channel integration                     |
| Channel Integrations   | Sí       | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| ClawHub Marketplace    | Sí       | Skill publishing, moderation, distribution                       |
| MCP Servers            | Sí       | External tool providers                                          |
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

| Flow | Source  | Destination  | Data                                    | Protection           |
| ---- | ------- | ------------ | --------------------------------------- | -------------------- |
| F1   | Channel | Gateway      | User messages                           | TLS, AllowFrom       |
| F2   | Gateway | Agent        | Routed messages                         | Session isolation    |
| F3   | Agent   | Herramientas | Tool invocations                        | Policy enforcement   |
| F4   | Agent   | External     | web_fetch requests | SSRF blocking        |
| F5   | ClawHub | Agent        | Skill code                              | Moderation, scanning |
| F6   | Agent   | Channel      | Respuestas                              | Output filtering     |

---

## 3. Threat Analysis by ATLAS Tactic

### 3.1 Reconnaissance (AML.TA0002)

#### T-RECON-001: Agent Endpoint Discovery

| Attribute               | Valor                                                                |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                          |
| **Description**         | Attacker scans for exposed OpenClaw gateway endpoints                |
| **Attack Vector**       | Network scanning, shodan queries, DNS enumeration                    |
| **Affected Components** | Gateway, exposed API endpoints                                       |
| **Current Mitigations** | Tailscale auth option, bind to loopback by default                   |
| **Residual Risk**       | Medium - Public gateways discoverable                                |
| **Recomendaciones**     | Document secure deployment, add rate limiting on discovery endpoints |

#### T-RECON-002: Channel Integration Probing

| Attribute               | Valor                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Active Scanning                        |
| **Description**         | Attacker probes messaging channels to identify AI-managed accounts |
| **Attack Vector**       | Sending test messages, observing response patterns                 |
| **Affected Components** | All channel integrations                                           |
| **Current Mitigations** | Ninguna específica                                                 |
| **Residual Risk**       | Bajo - Valor limitado solo por el descubrimiento                   |
| **Recomendaciones**     | Consider response timing randomization                             |

---

### 3.2 Acceso Inicial (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | Valor                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **Recomendaciones**     | Reduce grace period, add confirmation step                |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | Valor                                                                          |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker spoofs allowed sender identity in channel                             |
| **Attack Vector**       | Depends on channel - phone number spoofing, username impersonation             |
| **Affected Components** | AllowFrom validation per channel                                               |
| **Current Mitigations** | Channel-specific identity verification                                         |
| **Residual Risk**       | Medium - Some channels vulnerable to spoofing                                  |
| **Recomendaciones**     | Document channel-specific risks, add cryptographic verification where possible |

#### T-ACCESS-003: Token Theft

| Attribute               | Valor                                                                    |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                |
| **Description**         | Attacker steals authentication tokens from config files                  |
| **Attack Vector**       | Malware, unauthorized device access, config backup exposure              |
| **Affected Components** | ~/.openclaw/credentials/, config storage |
| **Current Mitigations** | Permisos de archivos                                                     |
| **Residual Risk**       | Alto - Tokens almacenados en texto plano                                 |
| **Recomendaciones**     | Implementar cifrado de tokens en reposo, agregar rotación de tokens      |

---

### 3.3 Ejecución (AML.TA0005)

#### T-EXEC-001: Inyección directa de prompts

| Attribute               | Valor                                                                                                  |
| ----------------------- | ------------------------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0051.000 - Inyección de prompts en LLM: Directa   |
| **Description**         | El atacante envía prompts elaborados para manipular el comportamiento del agente                       |
| **Attack Vector**       | Mensajes del canal que contienen instrucciones adversarias                                             |
| **Affected Components** | LLM del agente, todas las superficies de entrada                                                       |
| **Current Mitigations** | Detección de patrones, encapsulación de contenido externo                                              |
| **Residual Risk**       | Crítico - Solo detección, sin bloqueo; los ataques sofisticados lo eluden                              |
| **Recomendaciones**     | Implementar defensa multicapa, validación de salidas, confirmación del usuario para acciones sensibles |

#### T-EXEC-002: Inyección indirecta de prompt

| Attribute               | Valor                                                                                                  |
| ----------------------- | ------------------------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0051.001 - Inyección de prompts en LLM: Indirecta |
| **Description**         | El atacante incrusta instrucciones maliciosas en contenido obtenido                                    |
| **Attack Vector**       | URLs maliciosas, correos electrónicos envenenados, webhooks comprometidos                              |
| **Affected Components** | web_fetch, ingesta de correo electrónico, fuentes de datos externas               |
| **Current Mitigations** | Encapsulación de contenido con etiquetas XML y aviso de seguridad                                      |
| **Residual Risk**       | Alto - El LLM puede ignorar las instrucciones del contenedor                                           |
| **Recomendaciones**     | Implementar sanitización de contenido, contextos de ejecución separados                                |

#### T-EXEC-003: Inyección de argumentos de herramientas

| Attribute               | Valor                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - Inyección de prompts en LLM: Directa |
| **Description**         | El atacante manipula los argumentos de las herramientas mediante inyección de prompts                |
| **Attack Vector**       | Prompts elaborados que influyen en los valores de los parámetros de las herramientas                 |
| **Affected Components** | Todas las invocaciones de herramientas                                                               |
| **Current Mitigations** | Aprobaciones de ejecución para comandos peligrosos                                                   |
| **Residual Risk**       | Alto - Depende del criterio del usuario                                                              |
| **Recomendaciones**     | Implementar validación de argumentos, llamadas a herramientas parametrizadas                         |

#### T-EXEC-004: Bypass de aprobación de ejecución

| Attribute               | Valor                                                                     |
| ----------------------- | ------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Crear datos adversarios                       |
| **Description**         | El atacante crea comandos que eluden la lista de permitidos de aprobación |
| **Attack Vector**       | Ofuscación de comandos, explotación de alias, manipulación de rutas       |
| **Affected Components** | exec-approvals.ts, lista de permitidos de comandos        |
| **Current Mitigations** | Lista de permitidos + modo de confirmación                                |
| **Residual Risk**       | Alto - Sin sanitización de comandos                                       |
| **Recomendaciones**     | Implementar normalización de comandos, ampliar la lista de bloqueo        |

---

### 3.4 Persistencia (AML.TA0006)

#### T-PERSIST-001: Instalación de habilidad maliciosa

| Attribute               | Valor                                                                                                                 |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Compromiso de la cadena de suministro: software de IA |
| **Description**         | El atacante publica una habilidad maliciosa en ClawHub                                                                |
| **Attack Vector**       | Crear una cuenta, publicar una habilidad con código malicioso oculto                                                  |
| **Affected Components** | ClawHub, carga de habilidades, ejecución del agente                                                                   |
| **Current Mitigations** | Verificación de antigüedad de cuentas de GitHub, indicadores de moderación basados en patrones                        |
| **Residual Risk**       | Crítico - Sin aislamiento (sandboxing), revisión limitada                                          |
| **Recomendaciones**     | Integración con VirusTotal (en progreso), aislamiento de habilidades, revisión comunitaria         |

#### T-PERSIST-002: Envenenamiento de actualizaciones de habilidades

| Attribute               | Valor                                                                                                                 |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Compromiso de la cadena de suministro: software de IA |
| **Description**         | El atacante compromete una habilidad popular y envía una actualización maliciosa                                      |
| **Attack Vector**       | Compromiso de la cuenta, ingeniería social del propietario de la habilidad                                            |
| **Affected Components** | Versionado de ClawHub, flujos de actualización automática                                                             |
| **Current Mitigations** | Huellas digitales de versión                                                                                          |
| **Residual Risk**       | Alto - Las actualizaciones automáticas pueden descargar versiones maliciosas                                          |
| **Recomendaciones**     | Implementar firma de actualizaciones, capacidad de reversión, fijación de versiones                                   |

#### T-PERSIST-003: Manipulación de la configuración del agente

| Attribute               | Valor                                                                                                        |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0010.002 - Compromiso de la cadena de suministro: datos |
| **Description**         | El atacante modifica la configuración del agente para mantener el acceso                                     |
| **Attack Vector**       | Modificación de archivos de configuración, inyección de ajustes                                              |
| **Affected Components** | Configuración del agente, políticas de herramientas                                                          |
| **Current Mitigations** | Permisos de archivos                                                                                         |
| **Residual Risk**       | Medio - Requiere acceso local                                                                                |
| **Recomendaciones**     | Verificación de integridad de configuración, registro de auditoría para cambios de configuración             |

---

### 3.5 Evasión de Defensa (AML.TA0007)

#### T-EVADE-001: Omisión de Patrones de Moderación

| Attribute               | Valor                                                                                                   |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Elaborar Datos Adversarios                                                  |
| **Description**         | El atacante elabora contenido de habilidades para evadir patrones de moderación                         |
| **Attack Vector**       | Homoglifos Unicode, trucos de codificación, carga dinámica                                              |
| **Affected Components** | ClawHub moderation.ts                                                                   |
| **Current Mitigations** | FLAG_RULES basadas en patrones                                                     |
| **Residual Risk**       | Alto - Expresiones regulares simples se evaden fácilmente                                               |
| **Recomendaciones**     | Añadir análisis de comportamiento (VirusTotal Code Insight), detección basada en AST |

#### T-EVADE-002: Escape del Envoltorio de Contenido

| Attribute               | Valor                                                                        |
| ----------------------- | ---------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Elaborar Datos Adversarios                       |
| **Description**         | El atacante elabora contenido que escapa al contexto del envoltorio XML      |
| **Attack Vector**       | Manipulación de etiquetas, confusión de contexto, anulación de instrucciones |
| **Affected Components** | Envoltorio de contenido externo                                              |
| **Current Mitigations** | Etiquetas XML + aviso de seguridad                                           |
| **Residual Risk**       | Medio - Se descubren escapes novedosos con regularidad                       |
| **Recomendaciones**     | Múltiples capas de envoltura, validación del lado de salida                  |

---

### 3.6 Descubrimiento (AML.TA0008)

#### T-DISC-001: Enumeración de Herramientas

| Attribute               | Valor                                                             |
| ----------------------- | ----------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access         |
| **Description**         | El atacante enumera las herramientas disponibles mediante prompts |
| **Attack Vector**       | "¿Qué herramientas tienes?" consultas de estilo                   |
| **Affected Components** | Registro de herramientas del agente                               |
| **Current Mitigations** | Ninguna específica                                                |
| **Residual Risk**       | Bajo - Las herramientas generalmente están documentadas           |
| **Recomendaciones**     | Considerar controles de visibilidad de herramientas               |

#### T-DISC-002: Extracción de datos de sesión

| Attribute               | Valor                                                        |
| ----------------------- | ------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access    |
| **Description**         | El atacante extrae datos sensibles del contexto de la sesión |
| **Attack Vector**       | Consultas del tipo "¿Qué discutimos?", sondeo del contexto   |
| **Affected Components** | Transcripciones de sesión, ventana de contexto               |
| **Current Mitigations** | Aislamiento de sesión por remitente                          |
| **Residual Risk**       | Medio - Los datos dentro de la sesión son accesibles         |
| **Recomendaciones**     | Implementar la redacción de datos sensibles en el contexto   |

---

### 3.7 Recopilación y Exfiltración (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Robo de Datos mediante web_fetch

| Attribute               | Valor                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Recolección                                                   |
| **Description**         | El atacante exfiltra datos instruyendo al agente para enviarlos a una URL externa         |
| **Attack Vector**       | Inyección de prompt que provoca que el agente haga POST de datos al servidor del atacante |
| **Affected Components** | Herramienta web_fetch                                                |
| **Current Mitigations** | Bloqueo SSRF para redes internas                                                          |
| **Residual Risk**       | Alto - Se permiten URLs externas                                                          |
| **Recomendaciones**     | Implementar listas blancas de URLs, concienciación sobre clasificación de datos           |

#### T-EXFIL-002: Envío de Mensajes No Autorizado

| Attribute               | Valor                                                                          |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0009 - Recolección                                        |
| **Description**         | El atacante provoca que el agente envíe mensajes que contienen datos sensibles |
| **Attack Vector**       | Inyección de prompt que provoca que el agente envíe mensajes al atacante       |
| **Affected Components** | Herramienta de mensajería, integraciones de canales                            |
| **Current Mitigations** | 48. Control del envío de mensajes salientes             |
| **Residual Risk**       | Medio - El control puede ser eludido                                           |
| **Recomendaciones**     | Requerir confirmación explícita para nuevos destinatarios                      |

#### T-EXFIL-003: Recolección de credenciales

| Attribute               | Valor                                                                     |
| ----------------------- | ------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Recolección                                   |
| **Description**         | La skill maliciosa recolecta credenciales del contexto del agente         |
| **Attack Vector**       | El código de la skill lee variables de entorno, archivos de configuración |
| **Affected Components** | Entorno de ejecución de la skill                                          |
| **Current Mitigations** | Ninguna específica para skills                                            |
| **Residual Risk**       | Crítico - Las skills se ejecutan con privilegios del agente               |
| **Recomendaciones**     | Aislamiento de skills, aislamiento de credenciales                        |

---

### 3.8 Impacto (AML.TA0011)

#### T-IMPACT-001: Ejecución de comandos no autorizada

| Attribute               | Valor                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity               |
| **Description**         | El atacante ejecuta comandos arbitrarios en el sistema del usuario |
| **Attack Vector**       | Inyección de prompts combinada con omisión de aprobación de exec   |
| **Affected Components** | Herramienta Bash, ejecución de comandos                            |
| **Current Mitigations** | Aprobaciones de exec, opción de sandbox Docker                     |
| **Residual Risk**       | Crítico - Ejecución en el host sin sandbox                         |
| **Recomendaciones**     | Usar sandbox por defecto, mejorar la UX de aprobación              |

#### T-IMPACT-002: Agotamiento de recursos (DoS)

| Attribute               | Valor                                                                 |
| ----------------------- | --------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity                  |
| **Description**         | El atacante agota los créditos de API o los recursos de cómputo       |
| **Attack Vector**       | Inundación automatizada de mensajes, llamadas costosas a herramientas |
| **Affected Components** | Gateway, sesiones del agente, proveedor de API                        |
| **Current Mitigations** | Ninguna                                                               |
| **Residual Risk**       | Alto - Sin limitación de tasa                                         |
| **Recomendaciones**     | Implementar límites de tasa por remitente, presupuestos de costos     |

#### T-IMPACT-003: Daño reputacional

| Attribute               | Valor                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity    |
| **Description**         | Attacker causes agent to send harmful/offensive content |
| **Attack Vector**       | Prompt injection causing inappropriate responses        |
| **Affected Components** | Output generation, channel messaging                    |
| **Current Mitigations** | LLM provider content policies                           |
| **Residual Risk**       | Medium - Provider filters imperfect                     |
| **Recomendaciones**     | Output filtering layer, user controls                   |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Implementación                                                   | Efectividad                                          |
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

| Improvement            | Estado                                                   | Impact                                                                |
| ---------------------- | -------------------------------------------------------- | --------------------------------------------------------------------- |
| VirusTotal Integration | In Progress                                              | High - Code Insight behavioral analysis                               |
| Community Reporting    | Partial (`skillReports` table exists) | Medium                                                                |
| Audit Logging          | Partial (`auditLogs` table exists)    | Medium                                                                |
| Badge System           | Implementado                                             | Medium - `highlighted`, `official`, `deprecated`, `redactionApproved` |

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

| ID    | Recommendation                                       | Addresses                  |
| ----- | ---------------------------------------------------- | -------------------------- |
| R-001 | Complete VirusTotal integration                      | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implement skill sandboxing                           | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Agregar validación de salida para acciones sensibles | T-EXEC-001, T-EXEC-002     |

### 6.2 Corto plazo (P1)

| ID    | Recommendation                                                              | Addresses    |
| ----- | --------------------------------------------------------------------------- | ------------ |
| R-004 | Implementar limitación de tasa                                              | T-IMPACT-002 |
| R-005 | Agregar cifrado de tokens en reposo                                         | T-ACCESS-003 |
| R-006 | Mejorar la UX de aprobación de exec y la validación                         | T-EXEC-004   |
| R-007 | Implementar listas de permitidos de URL para web_fetch | T-EXFIL-001  |

### 6.3 Medio plazo (P2)

| ID    | Recommendation                                                 | Addresses     |
| ----- | -------------------------------------------------------------- | ------------- |
| R-008 | Agregar verificación criptográfica de canal cuando sea posible | T-ACCESS-002  |
| R-009 | Implementar verificación de integridad de la configuración     | T-PERSIST-003 |
| R-010 | Agregar firma de actualizaciones y fijación de versiones       | T-PERSIST-002 |

---

## 7. Apéndices

### 7.1 Mapeo de técnicas ATLAS

| ID de ATLAS                                   | Nombre de la técnica                                 | Amenazas de OpenClaw                                             |
| --------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Escaneo activo                                       | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Recolección                                          | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Cadena de suministro: software de IA | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Cadena de suministro: datos          | T-PERSIST-003                                                    |
| AML.T0031                     | Erosionar la integridad del modelo de IA             | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | AI Model Inference API Access                        | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Craft Adversarial Data                               | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM Prompt Injection: Direct         | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM Prompt Injection: Indirect       | T-EXEC-002                                                       |

### 7.2 Key Security Files

| Ruta                                | Propósito                   | Risk Level   |
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

| Term                 | Definition                                                |
| -------------------- | --------------------------------------------------------- |
| **ATLAS**            | MITRE's Adversarial Threat Landscape for AI Systems       |
| **ClawHub**          | OpenClaw's skill marketplace                              |
| **Gateway**          | OpenClaw's message routing and authentication layer       |
| **MCP**              | Model Context Protocol - tool provider interface          |
| **Prompt Injection** | Attack where malicious instructions are embedded in input |
| **Skill**            | Downloadable extension for OpenClaw agents                |
| **SSRF**             | Server-Side Request Forgery                               |

---

_This threat model is a living document. Report security issues to security@openclaw.ai_


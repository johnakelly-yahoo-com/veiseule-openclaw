# Modelo de Ameaças do OpenClaw v1.0

## 5. Framework MITRE ATLAS

**Versão:** 1.0-draft
**Última atualização:** 2026-02-04
**Metodologia:** MITRE ATLAS + Diagramas de Fluxo de Dados
**Framework:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### 7. Atribuição do Framework

Este modelo de ameaças é construído com base no [MITRE ATLAS](https://atlas.mitre.org/), o framework padrão da indústria para documentar ameaças adversariais a sistemas de IA/ML. O ATLAS é mantido pela [MITRE](https://www.mitre.org/) em colaboração com a comunidade de segurança em IA.

**Principais Recursos do ATLAS:**

- [Técnicas ATLAS](https://atlas.mitre.org/techniques/)
- [Táticas ATLAS](https://atlas.mitre.org/tactics/)
- [Estudos de Caso ATLAS](https://atlas.mitre.org/studies/)
- [ATLAS no GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contribuindo para o ATLAS](https://atlas.mitre.org/resources/contribute)

### Contribuindo para Este Modelo de Ameaças

Este é um documento vivo mantido pela comunidade OpenClaw. Consulte [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) para diretrizes sobre como contribuir:

- Relatar novas ameaças
- Atualizar ameaças existentes
- Propor cadeias de ataque
- Sugerir mitigações

---

## 6. Recommendations Summary

### 1.1 Propósito

Este modelo de ameaças documenta ameaças adversariais à plataforma de agentes de IA OpenClaw e ao marketplace de skills ClawHub, utilizando o framework MITRE ATLAS projetado especificamente para sistemas de IA/ML.

### 1.2 Escopo

| Componente                                       | Incluído | Observações                                                      |
| ------------------------------------------------ | -------- | ---------------------------------------------------------------- |
| Runtime de Agentes OpenClaw                      | Sim      | Execução central de agentes, chamadas de ferramentas, sessões    |
| Gateway                                          | Sim      | Autenticação, roteamento, integração de canais                   |
| 31. Integrações de Canais | Sim      | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| Marketplace ClawHub                              | Sim      | Publicação de skills, moderação, distribuição                    |
| Servidores MCP                                   | Sim      | Provedores de ferramentas externas                               |
| Dispositivos do Usuário                          | Atributo | Aplicativos móveis, clientes desktop                             |

### 1.3 Fora do Escopo

Nada está explicitamente fora do escopo deste modelo de ameaças.

---

## 5. Framework MITRE ATLAS

### 2.1 Limites de Confiança

```
44. ┌─────────────────────────────────────────────────────────────────┐
│                    ZONA NÃO CONFIÁVEL                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│           LIMITE DE CONFIANÇA 1: Acesso ao Canal                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Pareamento de Dispositivo (período de carência de 30s) │   │
│  │  • Validação AllowFrom / AllowList                        │   │
│  │  • Autenticação por Token/Senha/Tailscale                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           LIMITE DE CONFIANÇA 2: Isolamento de Sessão             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   SESSÕES DE AGENTE                       │
│  │  • Chave de sessão = agente:canal:par                     │
│  │  • Políticas de ferramentas por agente                    │
│  │  • Registro de transcrições                               │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           LIMITE DE CONFIANÇA 3: Execução de Ferramentas          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  SANDBOX DE EXECUÇÃO                      │
│  │  • Sandbox Docker OU Host (aprovações de exec)            │
│  │  • Execução remota de Node                                │
│  │  • Proteção SSRF (fixação de DNS + bloqueio de IP)        │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           LIMITE DE CONFIANÇA 4: Conteúdo Externo                │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              URLs / EMAILS / WEBHOOKS OBTIDOS             │
│  │  • Encapsulamento de conteúdo externo (tags XML)         │
│  │  • Injeção de aviso de segurança                          │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           LIMITE DE CONFIANÇA 5: Cadeia de Suprimentos            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │
│  │  • Publicação de skills (semver, SKILL.md obrigatório)   │
│  │  • Sinalizadores de moderação baseados em padrões         │
│  │  • Varredura VirusTotal (em breve)                        │
│  │  • Verificação da idade da conta GitHub                  │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Fluxos de Dados

| Baixo | Origem  | Atributo | Dados                                      | Proteção               |
| ----- | ------- | -------- | ------------------------------------------ | ---------------------- |
| F1    | Canal   | Gateway  | Mensagens do usuário                       | TLS, AllowFrom         |
| F2    | Gateway | Agente   | Mensagens roteadas                         | Session isolation      |
| F3    | Agente  | Tools    | Invocações de ferramentas                  | Aplicação de políticas |
| F4    | Agente  | Externo  | requisições web_fetch | Bloqueio de SSRF       |
| F5    | ClawHub | Agente   | Código de skill                            | Moderação, varredura   |
| F6    | Agente  | Canal    | Respostas                                  | Filtragem de saída     |

---

## 3. Análise de Ameaças por Tática ATLAS

### 3.1 Reconhecimento (AML.TA0002)

#### T-RECON-001: Descoberta de Endpoints do Agente

| Attribute               | Valor                                                                                  |
| ----------------------- | -------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Varredura Ativa                                            |
| **Description**         | Atacante realiza varredura em busca de endpoints expostos do gateway OpenClaw          |
| **Attack Vector**       | Varredura de rede, consultas no Shodan, enumeração de DNS                              |
| **Affected Components** | Gateway, endpoints de API expostos                                                     |
| **Current Mitigations** | Opção de autenticação Tailscale, vincular ao loopback por padrão                       |
| **Residual Risk**       | Médio - Gateways públicos descobríveis                                                 |
| **Recomendações**       | Documentar implantação segura, adicionar limitação de taxa nos endpoints de descoberta |

#### T-RECON-002: Sondagem de Integração de Canal

| Attribute               | Valor                                                                         |
| ----------------------- | ----------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Varredura Ativa                                   |
| **Description**         | Atacante sonda canais de mensagens para identificar contas gerenciadas por IA |
| **Attack Vector**       | Envio de mensagens de teste, observação de padrões de resposta                |
| **Affected Components** | All channel integrations                                                      |
| **Current Mitigations** | None specific                                                                 |
| **Residual Risk**       | Low - Limited value from discovery alone                                      |
| **Recomendações**       | Consider response timing randomization                                        |

---

### 3.2 Initial Access (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | Valor                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **Recomendações**       | Reduce grace period, add confirmation step                |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | Valor                                                                          |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker spoofs allowed sender identity in channel                             |
| **Attack Vector**       | Depends on channel - phone number spoofing, username impersonation             |
| **Affected Components** | AllowFrom validation per channel                                               |
| **Current Mitigations** | Channel-specific identity verification                                         |
| **Residual Risk**       | Medium - Some channels vulnerable to spoofing                                  |
| **Recomendações**       | Document channel-specific risks, add cryptographic verification where possible |

#### T-ACCESS-003: Token Theft

| Attribute               | Valor                                                                      |
| ----------------------- | -------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                  |
| **Description**         | Attacker steals authentication tokens from config files                    |
| **Attack Vector**       | Malware, unauthorized device access, config backup exposure                |
| **Affected Components** | ~/.openclaw/credentials/, config storage   |
| **Current Mitigations** | Permissões de arquivo                                                      |
| **Residual Risk**       | Alto - Tokens armazenados em texto simples                                 |
| **Recomendações**       | Implementar criptografia de tokens em repouso, adicionar rotação de tokens |

---

### AML.T0009 - Coleta

#### T-EXEC-001: Injeção Direta de Prompt

| Attribute               | Valor                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - Acesso à API de Inferência de Modelo de IA                                   |
| **Description**         | Atacante envia prompts elaborados para manipular o comportamento do agente                               |
| **Attack Vector**       | Mensagens de canal contendo instruções adversárias                                                       |
| **Affected Components** | LLM do agente, todas as superfícies de entrada                                                           |
| **Current Mitigations** | Detecção de padrões, encapsulamento de conteúdo externo                                                  |
| **Residual Risk**       | Crítico - Apenas detecção, sem bloqueio; ataques sofisticados burlam                                     |
| **Recomendações**       | Implementar defesa em múltiplas camadas, validação de saída, confirmação do usuário para ações sensíveis |

#### T-EXEC-002: Injeção Indireta de Prompt

| Attribute               | Valor                                                                        |
| ----------------------- | ---------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - Acesso à API de Inferência de Modelo de IA       |
| **Description**         | Atacante incorpora instruções maliciosas em conteúdo obtido                  |
| **Attack Vector**       | URLs maliciosas, e-mails envenenados, webhooks comprometidos                 |
| **Affected Components** | web_fetch, ingestão de e-mail, fontes de dados externas |
| **Current Mitigations** | Encapsulamento de conteúdo com tags XML e aviso de segurança                 |
| **Residual Risk**       | Alto - O LLM pode ignorar as instruções do encapsulamento                    |
| **Recomendações**       | Implementar sanitização de conteúdo, contextos de execução separados         |

#### T-EXEC-003: Injeção de Argumentos de Ferramenta

| Attribute               | Valor                                                                       |
| ----------------------- | --------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Coleta                                          |
| **Description**         | Atacante manipula argumentos de ferramentas por meio de injeção de prompt   |
| **Attack Vector**       | Prompts elaborados que influenciam valores de parâmetros das ferramentas    |
| **Affected Components** | Nenhuma específica                                                          |
| **Current Mitigations** | Aprovações de execução para comandos perigosos                              |
| **Residual Risk**       | Alto - Depende do julgamento do usuário                                     |
| **Recomendações**       | Implementar validação de argumentos, chamadas de ferramentas parametrizadas |

#### T-EXEC-004: Bypass de Aprovação de Execução

| Attribute               | Valor                                                      |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data         |
| **Description**         | Attacker crafts commands that bypass approval allowlist    |
| **Attack Vector**       | Command obfuscation, alias exploitation, path manipulation |
| **Affected Components** | exec-approvals.ts, command allowlist       |
| **Current Mitigations** | Allowlist + ask mode                                       |
| **Residual Risk**       | High - No command sanitization                             |
| **Recomendações**       | Implement command normalization, expand blocklist          |

---

### 3.4 Persistence (AML.TA0006)

#### T-PERSIST-001: Malicious Skill Installation

| Attribute               | Valor                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Atacante publica habilidade maliciosa no ClawHub                                                     |
| **Attack Vector**       | Create account, publish skill with hidden malicious code                                             |
| **Affected Components** | ClawHub, skill loading, agent execution                                                              |
| **Current Mitigations** | GitHub account age verification, pattern-based moderation flags                                      |
| **Residual Risk**       | Critical - No sandboxing, limited review                                                             |
| **Recomendações**       | VirusTotal integration (in progress), skill sandboxing, community review          |

#### T-PERSIST-002: Skill Update Poisoning

| Attribute               | Valor                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker compromises popular skill and pushes malicious update                                       |
| **Attack Vector**       | Account compromise, social engineering of skill owner                                                |
| **Affected Components** | ClawHub versioning, auto-update flows                                                                |
| **Current Mitigations** | Version fingerprinting                                                                               |
| **Residual Risk**       | High - Auto-updates may pull malicious versions                                                      |
| **Recomendações**       | Implement update signing, rollback capability, version pinning                                       |

#### T-PERSIST-003: Agent Configuration Tampering

| Attribute               | Valor                                                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Supply Chain Compromise: Data |
| **Description**         | Attacker modifies agent configuration to persist access                                       |
| **Attack Vector**       | Config file modification, settings injection                                                  |
| **Affected Components** | Agent config, tool policies                                                                   |
| **Current Mitigations** | Permissões de arquivo                                                                         |
| **Residual Risk**       | Medium - Requires local access                                                                |
| **Recomendações**       | Config integrity verification, audit logging for config changes                               |

---

### 3.5 Defense Evasion (AML.TA0007)

#### T-EVADE-001: Moderation Pattern Bypass

| Attribute               | Valor                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                        |
| **Description**         | Attacker crafts skill content to evade moderation patterns                                |
| **Attack Vector**       | Unicode homoglyphs, encoding tricks, dynamic loading                                      |
| **Affected Components** | ClawHub moderation.ts                                                     |
| **Current Mitigations** | Pattern-based FLAG_RULES                                             |
| **Residual Risk**       | High - Simple regex easily bypassed                                                       |
| **Recomendações**       | Add behavioral analysis (VirusTotal Code Insight), AST-based detection |

#### T-EVADE-002: Content Wrapper Escape

| Attribute               | Valor                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data        |
| **Description**         | Attacker crafts content that escapes XML wrapper context  |
| **Attack Vector**       | Tag manipulation, context confusion, instruction override |
| **Affected Components** | External content wrapping                                 |
| **Current Mitigations** | XML tags + security notice                                |
| **Residual Risk**       | Medium - Novel escapes discovered regularly               |
| **Recomendações**       | Multiple wrapper layers, output-side validation           |

---

### 3.6 Discovery (AML.TA0008)

#### T-DISC-001: Tool Enumeration

| Attribute               | Valor                                                          |
| ----------------------- | -------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access      |
| **Description**         | Atacante enumera ferramentas disponíveis por meio de prompting |
| **Attack Vector**       | Consultas do tipo "Quais ferramentas você tem?"                |
| **Affected Components** | Registro de ferramentas do agente                              |
| **Current Mitigations** | None specific                                                  |
| **Residual Risk**       | Baixo - Ferramentas geralmente documentadas                    |
| **Recomendações**       | Considerar controles de visibilidade de ferramentas            |

#### T-DISC-002: Extração de Dados de Sessão

| Attribute               | Valor                                                       |
| ----------------------- | ----------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access   |
| **Description**         | Atacante extrai dados sensíveis do contexto da sessão       |
| **Attack Vector**       | Consultas do tipo "O que discutimos?", sondagem de contexto |
| **Affected Components** | Transcrições de sessão, janela de contexto                  |
| **Current Mitigations** | Isolamento de sessão por remetente                          |
| **Residual Risk**       | Médio - Dados dentro da sessão acessíveis                   |
| **Recomendações**       | Implementar redação de dados sensíveis no contexto          |

---

### 3.7 Coleta e Exfiltração (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Roubo de Dados via web_fetch

| Attribute               | Valor                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                                |
| **Description**         | Atacante exfiltra dados instruindo o agente a enviar para uma URL externa             |
| **Attack Vector**       | Injeção de prompt fazendo o agente realizar POST de dados para o servidor do atacante |
| **Affected Components** | Ferramenta web_fetch                                             |
| **Current Mitigations** | Bloqueio de SSRF para redes internas                                                  |
| **Residual Risk**       | Alto - URLs externas permitidas                                                       |
| **Recomendações**       | Implementar allowlisting de URLs e conscientização sobre classificação de dados       |

#### T-EXFIL-002: Envio de Mensagens Não Autorizado

| Attribute               | Valor                                                            |
| ----------------------- | ---------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                           |
| **Description**         | Attacker causes agent to send messages containing sensitive data |
| **Attack Vector**       | Prompt injection causing agent to message attacker               |
| **Affected Components** | Message tool, channel integrations                               |
| **Current Mitigations** | Outbound messaging gating                                        |
| **Residual Risk**       | Medium - Gating may be bypassed                                  |
| **Recomendações**       | Require explicit confirmation for new recipients                 |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | Valor                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                  |
| **Description**         | Malicious skill harvests credentials from agent context |
| **Attack Vector**       | Skill code reads environment variables, config files    |
| **Affected Components** | Skill execution environment                             |
| **Current Mitigations** | None specific to skills                                 |
| **Residual Risk**       | Critical - Skills run with agent privileges             |
| **Recomendações**       | Skill sandboxing, credential isolation                  |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001: Unauthorized Command Execution

| Attribute               | Valor                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker executes arbitrary commands on user system  |
| **Attack Vector**       | Prompt injection combined with exec approval bypass  |
| **Affected Components** | Bash tool, command execution                         |
| **Current Mitigations** | Exec approvals, Docker sandbox option                |
| **Residual Risk**       | Critical - Host execution without sandbox            |
| **Recomendações**       | Default to sandbox, improve approval UX              |

#### T-IMPACT-002: Resource Exhaustion (DoS)

| Attribute               | Valor                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker exhausts API credits or compute resources   |
| **Attack Vector**       | Automated message flooding, expensive tool calls     |
| **Affected Components** | Gateway, agent sessions, API provider                |
| **Current Mitigations** | None                                                 |
| **Residual Risk**       | High - No rate limiting                              |
| **Recomendações**       | Implement per-sender rate limits, cost budgets       |

#### T-IMPACT-003: Reputation Damage

| Attribute               | Valor                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity    |
| **Description**         | Attacker causes agent to send harmful/offensive content |
| **Attack Vector**       | Prompt injection causing inappropriate responses        |
| **Affected Components** | Output generation, channel messaging                    |
| **Current Mitigations** | LLM provider content policies                           |
| **Residual Risk**       | Medium - Provider filters imperfect                     |
| **Recomendações**       | Output filtering layer, user controls                   |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Implementação                                                    | Effectiveness                                        |
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

## 7. Apêndices

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

| ID    | Recommendation                                           | Addresses     |
| ----- | -------------------------------------------------------- | ------------- |
| R-008 | Add cryptographic channel verification where possible    | T-ACCESS-002  |
| R-009 | Implementar verificação de integridade da configuração   | T-PERSIST-003 |
| R-010 | Adicionar assinatura de atualizações e fixação de versão | T-PERSIST-002 |

---

## 7. Atribuição do Framework

### 7.1 Mapeamento de Técnicas ATLAS

| ID ATLAS                                      | Nome da Técnica                                       | Ameaças OpenClaw                                                 |
| --------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Varredura Ativa                                       | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Coleta                                                | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Cadeia de Suprimentos: Software de IA | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Cadeia de Suprimentos: Dados          | T-PERSIST-003                                                    |
| AML.T0031                     | Erosão da Integridade do Modelo de IA                 | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | Acesso à API de Inferência de Modelo de IA            | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Criar Dados Adversariais                              | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | Injeção de Prompt em LLM: Direta      | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | Injeção de Prompt em LLM: Indireta    | T-EXEC-002                                                       |

### 7.2 Arquivos de Segurança Principais

| Caminho                             | Propósito                       | Risk Level   |
| ----------------------------------- | ------------------------------- | ------------ |
| `src/infra/exec-approvals.ts`       | Lógica de aprovação de comandos | **Critical** |
| `src/gateway/auth.ts`               | Autenticação do gateway         | **Critical** |
| `src/web/inbound/access-control.ts` | Channel access control          | **Critical** |
| `src/infra/net/ssrf.ts`             | SSRF protection                 | **Critical** |
| `src/security/external-content.ts`  | Prompt injection mitigation     | **Critical** |
| `src/agents/sandbox/tool-policy.ts` | Tool policy enforcement         | **Critical** |
| `convex/lib/moderation.ts`          | ClawHub moderation              | **High**     |
| `convex/lib/skillPublish.ts`        | Skill publishing flow           | **High**     |
| `src/routing/resolve-route.ts`      | Session isolation               | **Medium**   |

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

_Este modelo de ameaças é um documento vivo. Relate problemas de segurança para security@openclaw.ai_


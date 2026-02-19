# OpenClaw 脅威モデル v1.0

## MITRE ATLAS フレームワーク

**バージョン:** 1.0-draft
**最終更新日:** 2026-02-04
**方法論:** MITRE ATLAS + データフロー図
**フレームワーク:** [MITRE ATLAS](https://atlas.mitre.org/)（AI システムのための敵対的脅威ランドスケープ）

### フレームワークの帰属

この脅威モデルは、AI/ML システムに対する敵対的脅威を文書化する業界標準フレームワークである [MITRE ATLAS](https://atlas.mitre.org/) に基づいて構築されています。 ATLAS は、AI セキュリティコミュニティと協力して [MITRE](https://www.mitre.org/) により維持されています。 ATLAS は、AI セキュリティコミュニティと協力して [MITRE](https://www.mitre.org/) により維持されています。

**主要な ATLAS リソース:**

- [ATLAS 技術](https://atlas.mitre.org/techniques/)
- [ATLAS 戦術](https://atlas.mitre.org/tactics/)
- [ATLAS ケーススタディ](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [ATLAS への貢献](https://atlas.mitre.org/resources/contribute)

### この脅威モデルへの貢献

これは OpenClaw コミュニティによって維持されるリビングドキュメントです。 これは OpenClaw コミュニティによって維持されるリビングドキュメントです。 貢献に関するガイドラインについては [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) を参照してください:

- 新しい脅威の報告
- 既存の脅威の更新
- 攻撃チェーンの提案
- 緩和策の提案

---

## 1. はじめに

### 1.1 目的

この脅威モデルは、AI/ML システム向けに特別に設計された MITRE ATLAS フレームワークを使用して、OpenClaw AI エージェントプラットフォームおよび ClawHub スキルマーケットプレイスに対する敵対的脅威を文書化します。

### 1.2 スコープ

| コンポーネント              | 対象      | 備考                                        |
| -------------------- | ------- | ----------------------------------------- |
| OpenClaw エージェントランタイム | Yes     | コアとなるエージェント実行、ツール呼び出し、セッション               |
| Gateway              | Yes     | 認証、ルーティング、チャネル統合                          |
| チャネル統合               | Yes     | WhatsApp、Telegram、Discord、Signal、Slack など |
| ClawHub マーケットプレイス    | Yes     | スキルの公開、モデレーション、配布                         |
| MCP サーバー             | Yes     | External tool providers                   |
| User Devices         | Partial | Mobile apps, desktop clients              |

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
| F3   | Agent   | Tools       | Tool invocations                        | Policy enforcement   |
| F4   | Agent   | External    | web_fetch requests | SSRF blocking        |
| F5   | ClawHub | Agent       | Skill code                              | Moderation, scanning |
| F6   | Agent   | Channel     | Responses                               | Output filtering     |

---

## 3. Threat Analysis by ATLAS Tactic

### 3.1 Reconnaissance (AML.TA0002)

#### T-RECON-001: Agent Endpoint Discovery

| Attribute               | Value                                                                |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                          |
| **Description**         | Attacker scans for exposed OpenClaw gateway endpoints                |
| **Attack Vector**       | Network scanning, shodan queries, DNS enumeration                    |
| **Affected Components** | Gateway, exposed API endpoints                                       |
| **Current Mitigations** | Tailscale auth option, bind to loopback by default                   |
| **Residual Risk**       | Medium - Public gateways discoverable                                |
| **Recommendations**     | Document secure deployment, add rate limiting on discovery endpoints |

#### T-RECON-002: Channel Integration Probing

| Attribute               | Value                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Active Scanning                        |
| **Description**         | Attacker probes messaging channels to identify AI-managed accounts |
| **Attack Vector**       | Sending test messages, observing response patterns                 |
| **Affected Components** | All channel integrations                                           |
| **Current Mitigations** | None specific                                                      |
| **Residual Risk**       | Low - Limited value from discovery alone                           |
| **Recommendations**     | Consider response timing randomization                             |

---

### 3.2 Initial Access (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | Value                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **Recommendations**     | 猶予期間を短縮し、確認ステップを追加する                                      |

#### T-ACCESS-002: AllowFrom スプーフィング

| Attribute               | Value                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | 攻撃者がチャネル内で許可された送信者のアイデンティティを偽装する                          |
| **Attack Vector**       | チャネルに依存 - 電話番号のスプーフィング、ユーザー名のなりすまし                        |
| **Affected Components** | チャネルごとの AllowFrom 検証                                      |
| **Current Mitigations** | チャネル固有のアイデンティティ検証                                         |
| **Residual Risk**       | 中 - 一部のチャネルはスプーフィングに対して脆弱                                 |
| **Recommendations**     | チャネル固有のリスクを文書化し、可能な場合は暗号学的検証を追加する                         |

#### T-ACCESS-003: トークン盗難

| Attribute               | Value                                                             |
| ----------------------- | ----------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access         |
| **Description**         | 攻撃者が設定ファイルから認証トークンを盗む                                             |
| **Attack Vector**       | マルウェア、不正なデバイスアクセス、設定バックアップの露出                                     |
| **Affected Components** | ~/.openclaw/credentials/, 設定ストレージ |
| **Current Mitigations** | ファイル権限                                                            |
| **Residual Risk**       | 高 - トークンが平文で保存されている                                               |
| **Recommendations**     | 保存時のトークン暗号化を実装し、トークンローテーションを追加する                                  |

---

### 3.3 実行 (AML.TA0005)

#### T-EXEC-001: 直接プロンプトインジェクション

| Attribute               | Value                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct |
| **Description**         | 攻撃者が細工されたプロンプトを送信してエージェントの挙動を操作する                                                            |
| **Attack Vector**       | 敵対的な指示を含むチャネルメッセージ                                                                           |
| **Affected Components** | エージェント LLM、すべての入力サーフェス                                                                       |
| **Current Mitigations** | パターン検出、外部コンテンツのラッピング                                                                         |
| **Residual Risk**       | Critical - Detection only, no blocking; sophisticated attacks bypass                         |
| **Recommendations**     | Implement multi-layer defense, output validation, user confirmation for sensitive actions    |

#### T-EXEC-002: Indirect Prompt Injection

| Attribute               | Value                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - LLM Prompt Injection: Indirect |
| **Description**         | Attacker embeds malicious instructions in fetched content                                      |
| **Attack Vector**       | Malicious URLs, poisoned emails, compromised webhooks                                          |
| **Affected Components** | web_fetch, email ingestion, external data sources                         |
| **Current Mitigations** | Content wrapping with XML tags and security notice                                             |
| **Residual Risk**       | High - LLM may ignore wrapper instructions                                                     |
| **Recommendations**     | Implement content sanitization, separate execution contexts                                    |

#### T-EXEC-003: Tool Argument Injection

| Attribute               | Value                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct |
| **Description**         | Attacker manipulates tool arguments through prompt injection                                 |
| **Attack Vector**       | Crafted prompts that influence tool parameter values                                         |
| **Affected Components** | All tool invocations                                                                         |
| **Current Mitigations** | Exec approvals for dangerous commands                                                        |
| **Residual Risk**       | High - Relies on user judgment                                                               |
| **Recommendations**     | Implement argument validation, parameterized tool calls                                      |

#### T-EXEC-004: Exec Approval Bypass

| Attribute               | Value                                                      |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data         |
| **Description**         | Attacker crafts commands that bypass approval allowlist    |
| **Attack Vector**       | Command obfuscation, alias exploitation, path manipulation |
| **Affected Components** | exec-approvals.ts, command allowlist       |
| **Current Mitigations** | 許可リスト + 確認モード                                              |
| **Residual Risk**       | 高 - コマンドのサニタイズなし                                           |
| **Recommendations**     | コマンド正規化の実装、ブロックリストの拡張                                      |

---

### 3.4 永続化 (AML.TA0006)

#### T-PERSIST-001: 悪意のあるスキルのインストール

| Attribute               | Value                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0010.001 - サプライチェーン侵害: AIソフトウェア |
| **Description**         | 攻撃者が悪意のあるスキルをClawHubに公開                                                              |
| **Attack Vector**       | アカウントを作成し、隠された悪意のあるコードを含むスキルを公開                                                      |
| **Affected Components** | ClawHub、スキル読み込み、エージェント実行                                                             |
| **Current Mitigations** | GitHubアカウントの経過年数検証、パターンベースのモデレーションフラグ                                                |
| **Residual Risk**       | 重大 - サンドボックスなし、レビューが限定的                                                              |
| **Recommendations**     | VirusTotal連携（進行中）、スキルのサンドボックス化、コミュニティレビュー                                            |

#### T-PERSIST-002: スキル更新の汚染

| Attribute               | Value                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0010.001 - サプライチェーン侵害: AIソフトウェア |
| **Description**         | 攻撃者が人気のあるスキルを侵害し、悪意のある更新を配信                                                          |
| **Attack Vector**       | Account compromise, social engineering of skill owner                                |
| **Affected Components** | ClawHubのバージョニング、自動更新フロー                                                              |
| **Current Mitigations** | バージョンフィンガープリンティング                                                                    |
| **Residual Risk**       | 高 - 自動更新により悪意のあるバージョンが取得される可能性                                                       |
| **Recommendations**     | 更新署名の実装、ロールバック機能、バージョン固定                                                             |

#### T-PERSIST-003: エージェント設定の改ざん

| Attribute               | Value                                                                           |
| ----------------------- | ------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - サプライチェーン侵害: データ |
| **Description**         | 攻撃者がエージェント設定を変更してアクセスを永続化                                                       |
| **Attack Vector**       | 設定ファイルの変更、設定の注入                                                                 |
| **Affected Components** | エージェント設定、ツールポリシー                                                                |
| **Current Mitigations** | ファイル権限                                                                          |
| **Residual Risk**       | 中 — ローカルアクセスが必要                                                                 |
| **Recommendations**     | 設定整合性の検証、設定変更の監査ログ                                                              |

---

### 3.5 防御回避（AML.TA0007）

#### T-EVADE-001: モデレーションパターン回避

| Attribute               | Value                                              |
| ----------------------- | -------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data |
| **Description**         | 攻撃者がモデレーションパターンを回避するためにスキルコンテンツを作成する               |
| **Attack Vector**       | Unicode同形異字、エンコーディングのトリック、動的ロード                    |
| **Affected Components** | ClawHub moderation.ts              |
| **Current Mitigations** | パターンベースの FLAG_RULES           |
| **Residual Risk**       | 高 — 単純な正規表現は容易に回避される                               |
| **Recommendations**     | 行動分析（VirusTotal Code Insight）、ASTベースの検出を追加         |

#### T-EVADE-002: コンテンツラッパーエスケープ

| Attribute               | Value                                              |
| ----------------------- | -------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data |
| **Description**         | 攻撃者がXMLラッパーのコンテキストをエスケープするコンテンツを作成する               |
| **Attack Vector**       | タグ操作、コンテキスト混乱、命令上書き                                |
| **Affected Components** | 外部コンテンツのラッピング                                      |
| **Current Mitigations** | XMLタグ＋セキュリティ通知                                     |
| **Residual Risk**       | 中 — 新しいエスケープ手法が定期的に発見されている                         |
| **Recommendations**     | 複数のラッパー層、出力側の検証                                    |

---

### 3.6 探索（AML.TA0008）

#### T-DISC-001: ツール列挙

| Attribute               | Value                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | 攻撃者がプロンプトを通じて利用可能なツールを列挙する                                |
| **Attack Vector**       | "What tools do you have?" style queries                   |
| **Affected Components** | Agent tool registry                                       |
| **Current Mitigations** | None specific                                             |
| **Residual Risk**       | Low - Tools generally documented                          |
| **Recommendations**     | Consider tool visibility controls                         |

#### T-DISC-002: Session Data Extraction

| Attribute               | Value                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker extracts sensitive data from session context     |
| **Attack Vector**       | "What did we discuss?" queries, context probing           |
| **Affected Components** | Session transcripts, context window                       |
| **Current Mitigations** | Session isolation per sender                              |
| **Residual Risk**       | Medium - Within-session data accessible                   |
| **Recommendations**     | Implement sensitive data redaction in context             |

---

### 3.7 Collection & Exfiltration (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Data Theft via web_fetch

| Attribute               | Value                                                                  |
| ----------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                 |
| **Description**         | Attacker exfiltrates data by instructing agent to send to external URL |
| **Attack Vector**       | Prompt injection causing agent to POST data to attacker server         |
| **Affected Components** | web_fetch tool                                    |
| **Current Mitigations** | SSRF blocking for internal networks                                    |
| **Residual Risk**       | High - External URLs permitted                                         |
| **Recommendations**     | Implement URL allowlisting, data classification awareness              |

#### T-EXFIL-002: Unauthorized Message Sending

| Attribute               | Value                                                            |
| ----------------------- | ---------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                           |
| **Description**         | Attacker causes agent to send messages containing sensitive data |
| **Attack Vector**       | Prompt injection causing agent to message attacker               |
| **Affected Components** | Message tool, channel integrations                               |
| **Current Mitigations** | Outbound messaging gating                                        |
| **Residual Risk**       | Medium - Gating may be bypassed                                  |
| **Recommendations**     | Require explicit confirmation for new recipients                 |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | Value                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                  |
| **Description**         | Malicious skill harvests credentials from agent context |
| **Attack Vector**       | Skill code reads environment variables, config files    |
| **Affected Components** | Skill execution environment                             |
| **Current Mitigations** | None specific to skills                                 |
| **Residual Risk**       | Critical - Skills run with agent privileges             |
| **Recommendations**     | Skill sandboxing, credential isolation                  |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001: Unauthorized Command Execution

| Attribute               | Value                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker executes arbitrary commands on user system  |
| **Attack Vector**       | Prompt injection combined with exec approval bypass  |
| **Affected Components** | Bash tool, command execution                         |
| **Current Mitigations** | Exec approvals, Docker sandbox option                |
| **Residual Risk**       | Critical - Host execution without sandbox            |
| **Recommendations**     | Default to sandbox, improve approval UX              |

#### T-IMPACT-002: Resource Exhaustion (DoS)

| Attribute               | Value                                                |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker exhausts API credits or compute resources   |
| **Attack Vector**       | Automated message flooding, expensive tool calls     |
| **Affected Components** | Gateway, agent sessions, API provider                |
| **Current Mitigations** | None                                                 |
| **Residual Risk**       | High - No rate limiting                              |
| **Recommendations**     | Implement per-sender rate limits, cost budgets       |

#### T-IMPACT-003: Reputation Damage

| Attribute               | Value                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity    |
| **Description**         | Attacker causes agent to send harmful/offensive content |
| **Attack Vector**       | Prompt injection causing inappropriate responses        |
| **Affected Components** | Output generation, channel messaging                    |
| **Current Mitigations** | LLM provider content policies                           |
| **Residual Risk**       | Medium - Provider filters imperfect                     |
| **Recommendations**     | Output filtering layer, user controls                   |

---

## 4. ClawHub Supply Chain Analysis

### 4.1 Current Security Controls

| Control                           | Implementation                                                   | Effectiveness                                        |
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

### 5.2 クリティカルパス攻撃チェーン

**攻撃チェーン 1: スキルベースのデータ窃取**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(悪意のあるスキルを公開) → (モデレーションを回避) → (認証情報を収集)
```

**攻撃チェーン 2: プロンプトインジェクションによる RCE**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Inject prompt) → (Bypass exec approval) → (Execute commands)
```

**攻撃チェーン 3: 取得コンテンツ経由の間接インジェクション**

```
T-EXEC-002 → T-EXFIL-001 → 外部への流出
(URL コンテンツを汚染) → (エージェントが取得して指示に従う) → (データが攻撃者に送信される)
```

---

## 6. Recommendations Summary

### 6.1 即時対応 (P0)

| ID    | 推奨事項                | 対応対象                       |
| ----- | ------------------- | -------------------------- |
| R-001 | VirusTotal との完全な統合  | T-PERSIST-001, T-EVADE-001 |
| R-002 | スキルのサンドボックス化を実装     | T-PERSIST-001, T-EXFIL-003 |
| R-003 | 機密性の高い操作に対する出力検証を追加 | T-EXEC-001, T-EXEC-002     |

### 6.2 短期 (P1)

| ID    | 推奨事項                                             | 対応対象         |
| ----- | ------------------------------------------------ | ------------ |
| R-004 | レート制限を実装                                         | T-IMPACT-002 |
| R-005 | 保存時のトークン暗号化を追加                                   | T-ACCESS-003 |
| R-006 | 実行承認の UX と検証を改善                                  | T-EXEC-004   |
| R-007 | web_fetch に対する URL 許可リストを実装 | T-EXFIL-001  |

### 6.3 中期 (P2)

| ID    | 推奨事項                 | 対応対象          |
| ----- | -------------------- | ------------- |
| R-008 | 可能な場合は暗号学的なチャネル検証を追加 | T-ACCESS-002  |
| R-009 | 設定の整合性検証を実装          | T-PERSIST-003 |
| R-010 | アップデート署名とバージョン固定を追加  | T-PERSIST-002 |

---

## 7. 付録

### 7.1 ATLAS 技法マッピング

| ATLAS ID                                      | 技法名                  | OpenClaw の脅威                                                     |
| --------------------------------------------- | -------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | アクティブスキャン            | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | 収集                   | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | サプライチェーン：AI ソフトウェア   | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | サプライチェーン：データ         | T-PERSIST-003                                                    |
| AML.T0031                     | AI モデルの整合性を侵食        | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | AI モデル推論 API アクセス    | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | 敵対的データの作成            | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | LLM プロンプトインジェクション：直接 | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | LLM プロンプトインジェクション：間接 | T-EXEC-002                                                       |

### 7.2 主要セキュリティファイル

| パス                                  | 目的                          | Risk Level   |
| ----------------------------------- | --------------------------- | ------------ |
| `src/infra/exec-approvals.ts`       | コマンド承認ロジック                  | **Critical** |
| `src/gateway/auth.ts`               | ゲートウェイ認証                    | **Critical** |
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


# Mô hình Đe doạ OpenClaw v1.0

## Khung MITRE ATLAS

**Phiên bản:** 1.0-draft  
**Cập nhật lần cuối:** 2026-02-04  
**Phương pháp luận:** MITRE ATLAS + Sơ đồ Luồng Dữ liệu  
**Khung:** [MITRE ATLAS](https://atlas.mitre.org/) (Bức tranh Đe doạ Đối kháng cho Hệ thống AI)

### Ghi nhận khung

This threat model is built on [MITRE ATLAS](https://atlas.mitre.org/), the industry-standard framework for documenting adversarial threats to AI/ML systems. ATLAS is maintained by [MITRE](https://www.mitre.org/) in collaboration with the AI security community.

**Tài nguyên ATLAS chính:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Đóng góp cho ATLAS](https://atlas.mitre.org/resources/contribute)

### Đóng góp cho mô hình đe doạ này

This is a living document maintained by the OpenClaw community. See [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) for guidelines on contributing:

- Báo cáo mối đe doạ mới
- Cập nhật mối đe doạ hiện có
- Đề xuất chuỗi tấn công
- Đề xuất biện pháp giảm thiểu

---

## 8. 1. Introduction

### 1.1 Mục đích

Mô hình đe doạ này ghi nhận các mối đe doạ đối kháng đối với nền tảng AI agent OpenClaw và chợ kỹ năng ClawHub, sử dụng khung MITRE ATLAS được thiết kế chuyên biệt cho hệ thống AI/ML.

### 1.2 Phạm vi

| Thành phần                        | Bao gồm  | Ghi chú                                                                          |
| --------------------------------- | -------- | -------------------------------------------------------------------------------- |
| Môi trường chạy Tác nhân OpenClaw | Có       | Thực thi agent cốt lõi, gọi công cụ, phiên làm việc                              |
| Gateway                           | Có       | Xác thực, định tuyến, tích hợp kênh                                              |
| Tích hợp Kênh                     | Có       | WhatsApp, Telegram, Discord, Signal, Slack, v.v. |
| Chợ ClawHub                       | Có       | Xuất bản kỹ năng, kiểm duyệt, phân phối                                          |
| MCP Servers                       | Có       | Nhà cung cấp công cụ bên ngoài                                                   |
| Thiết bị Người dùng               | Một phần | Ứng dụng di động, ứng dụng desktop                                               |

### 1.3 Ngoài phạm vi

Không có nội dung nào được loại trừ rõ ràng khỏi mô hình đe doạ này.

---

## 7. 2. System Architecture

### 2.1 Ranh giới Tin cậy

```
┌─────────────────────────────────────────────────────────────────┐
│                    VÙNG KHÔNG TIN CẬY                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 1: Truy cập Kênh                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Ghép nối thiết bị (thời gian chờ 30s)                  │   │
│  │  • Xác thực AllowFrom / AllowList                         │   │
│  │  • Xác thực Token/Password/Tailscale                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 2: Cô lập Phiên                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENT SESSIONS                          │   │
│  │  • Khóa phiên = agent:channel:peer                       │   │
│  │  • Chính sách công cụ theo từng agent                    │   │
│  │  • Ghi log hội thoại                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 3: Thực thi Công cụ                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  EXECUTION SANDBOX                        │   │
│  │  • Docker sandbox HOẶC Host (exec-approvals)             │   │
│  │  • Thực thi từ xa qua Node                               │   │
│  │  • Bảo vệ SSRF (ghim DNS + chặn IP)                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 4: Nội dung Bên ngoài               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │          URL / EMAIL / WEBHOOK ĐƯỢC TẢI VỀ               │   │
│  │  • Bao bọc nội dung bên ngoài (thẻ XML)                  │   │
│  │  • Chèn thông báo bảo mật                                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 5: Chuỗi Cung ứng                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Xuất bản kỹ năng (semver, bắt buộc SKILL.md)          │   │
│  │  • Cờ kiểm duyệt dựa trên mẫu                             │   │
│  │  • Quét VirusTotal (sắp ra mắt)                          │   │
│  │  • Xác minh tuổi tài khoản GitHub                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Luồng Dữ liệu

| Luồng | Nguồn   | Đích      | Dữ liệu                                | Bảo vệ              |
| ----- | ------- | --------- | -------------------------------------- | ------------------- |
| F1    | Kênh    | Gateway   | Tin nhắn người dùng                    | TLS, AllowFrom      |
| F2    | Gateway | Agent     | Tin nhắn đã định tuyến                 | Cô lập phiên        |
| F3    | Agent   | Công cụ   | Lời gọi công cụ                        | Thực thi chính sách |
| F4    | Agent   | Bên ngoài | Yêu cầu web_fetch | Chặn SSRF           |
| F5    | ClawHub | Agent     | Mã kỹ năng                             | Kiểm duyệt, quét    |
| F6    | Agent   | Kênh      | Phản hồi                               | Lọc đầu ra          |

---

## 38. 3. Phân tích mối đe dọa theo chiến thuật ATLAS

### 3.1 Trinh sát (AML.TA0002)

#### T-RECON-001: Phát hiện Endpoint Agent

| Thuộc tính                  | Giá trị                                                                |
| --------------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**                | AML.T0006 - Quét Chủ động                              |
| **Mô tả**                   | Kẻ tấn công quét các endpoint gateway OpenClaw bị lộ                   |
| **Vector tấn công**         | Quét mạng, truy vấn shodan, liệt kê DNS                                |
| **Thành phần bị ảnh hưởng** | Gateway, các API endpoint công khai                                    |
| **Giảm thiểu hiện tại**     | Tùy chọn xác thực Tailscale, mặc định bind loopback                    |
| **Rủi ro còn lại**          | Trung bình - Gateway công khai có thể bị phát hiện                     |
| **Khuyến nghị**             | Tài liệu triển khai an toàn, thêm rate limiting cho endpoint phát hiện |

#### T-RECON-002: Thăm dò Tích hợp Kênh

| Thuộc tính                  | Giá trị                                                               |
| --------------------------- | --------------------------------------------------------------------- |
| **ATLAS ID**                | AML.T0006 - Quét Chủ động                             |
| **Mô tả**                   | Kẻ tấn công thăm dò kênh nhắn tin để xác định tài khoản do AI quản lý |
| **Vector tấn công**         | Gửi tin nhắn thử nghiệm, quan sát mẫu phản hồi                        |
| **Thành phần bị ảnh hưởng** | Tất cả tích hợp kênh                                                  |
| **Giảm thiểu hiện tại**     | Không có biện pháp cụ thể                                             |
| **Rủi ro còn lại**          | Thấp - Giá trị hạn chế chỉ từ việc phát hiện                          |
| **Khuyến nghị**             | Cân nhắc ngẫu nhiên hóa thời gian phản hồi                            |

---

### 3.2 Initial Access (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | Giá trị                                                   |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **Khuyến nghị**         | Reduce grace period, add confirmation step                |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | Giá trị                                                                        |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker spoofs allowed sender identity in channel                             |
| **Attack Vector**       | Depends on channel - phone number spoofing, username impersonation             |
| **Affected Components** | AllowFrom validation per channel                                               |
| **Current Mitigations** | Channel-specific identity verification                                         |
| **Residual Risk**       | Medium - Some channels vulnerable to spoofing                                  |
| **Khuyến nghị**         | Document channel-specific risks, add cryptographic verification where possible |

#### T-ACCESS-003: Token Theft

| Attribute               | Giá trị                                                                  |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                |
| **Description**         | Attacker steals authentication tokens from config files                  |
| **Attack Vector**       | Malware, unauthorized device access, config backup exposure              |
| **Affected Components** | ~/.openclaw/credentials/, config storage |
| **Current Mitigations** | Quyền tệp                                                                |
| **Residual Risk**       | Cao - Token được lưu trữ ở dạng văn bản thuần                            |
| **Khuyến nghị**         | Implement token encryption at rest, add token rotation                   |

---

### 3.3 Execution (AML.TA0005)

#### T-EXEC-001: Direct Prompt Injection

| Attribute               | Giá trị                                                                                      |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct |
| **Description**         | Kẻ tấn công gửi các prompt được chế tác để thao túng hành vi của tác nhân                    |
| **Attack Vector**       | Channel messages containing adversarial instructions                                         |
| **Affected Components** | Agent LLM, all input surfaces                                                                |
| **Current Mitigations** | Phát hiện mẫu, bao bọc nội dung bên ngoài                                                    |
| **Residual Risk**       | Critical - Detection only, no blocking; sophisticated attacks bypass                         |
| **Khuyến nghị**         | Implement multi-layer defense, output validation, user confirmation for sensitive actions    |

#### T-EXEC-002: Indirect Prompt Injection

| Attribute               | Giá trị                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - LLM Prompt Injection: Indirect |
| **Description**         | Attacker embeds malicious instructions in fetched content                                      |
| **Attack Vector**       | Malicious URLs, poisoned emails, compromised webhooks                                          |
| **Affected Components** | web_fetch, email ingestion, external data sources                         |
| **Current Mitigations** | Content wrapping with XML tags and security notice                                             |
| **Residual Risk**       | High - LLM may ignore wrapper instructions                                                     |
| **Khuyến nghị**         | Implement content sanitization, separate execution contexts                                    |

#### T-EXEC-003: Tool Argument Injection

| Attribute               | Giá trị                                                                                      |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - LLM Prompt Injection: Direct |
| **Description**         | Attacker manipulates tool arguments through prompt injection                                 |
| **Attack Vector**       | Crafted prompts that influence tool parameter values                                         |
| **Affected Components** | All tool invocations                                                                         |
| **Current Mitigations** | Exec approvals for dangerous commands                                                        |
| **Residual Risk**       | High - Relies on user judgment                                                               |
| **Khuyến nghị**         | Implement argument validation, parameterized tool calls                                      |

#### T-EXEC-004: Exec Approval Bypass

| Attribute               | Giá trị                                                                                  |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                       |
| **Description**         | 4. Kẻ tấn công tạo các lệnh vượt qua danh sách cho phép phê duyệt |
| **Attack Vector**       | 6. Làm rối lệnh, khai thác bí danh, thao túng đường dẫn           |
| **Affected Components** | 8. exec-approvals.ts, danh sách cho phép lệnh     |
| **Current Mitigations** | Danh sách cho phép + chế độ hỏi                                                          |
| **Residual Risk**       | 12. Cao - Không có chuẩn hóa lệnh                                 |
| **Khuyến nghị**         | 13. Triển khai chuẩn hóa lệnh, mở rộng danh sách chặn             |

---

### 14. 3.4 Duy trì (AML.TA0006)

#### 15. T-PERSIST-001: Cài đặt kỹ năng độc hại

| Attribute               | Giá trị                                                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Xâm phạm chuỗi cung ứng: Phần mềm AI                    |
| **Description**         | 20. Kẻ tấn công phát hành kỹ năng độc hại lên ClawHub                                            |
| **Attack Vector**       | 22. Tạo tài khoản, phát hành kỹ năng có mã độc ẩn                                                |
| **Affected Components** | ClawHub, tải kỹ năng, thực thi tác nhân                                                                                 |
| **Current Mitigations** | 26. Xác minh tuổi tài khoản GitHub, cờ kiểm duyệt dựa trên mẫu                                   |
| **Residual Risk**       | 28. Nghiêm trọng - Không có sandboxing, kiểm duyệt hạn chế                                       |
| **Khuyến nghị**         | 29. Tích hợp VirusTotal (đang tiến hành), sandbox kỹ năng, đánh giá cộng đồng |

#### 30. T-PERSIST-002: Đầu độc cập nhật kỹ năng

| Attribute               | Giá trị                                                                                                                         |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 33. AML.T0010.001 - Xâm phạm chuỗi cung ứng: Phần mềm AI |
| **Description**         | 35. Kẻ tấn công xâm nhập kỹ năng phổ biến và đẩy bản cập nhật độc hại                                    |
| **Attack Vector**       | 37. Xâm phạm tài khoản, kỹ nghệ xã hội đối với chủ sở hữu kỹ năng                                        |
| **Affected Components** | 39. Phiên bản hóa ClawHub, luồng tự động cập nhật                                                        |
| **Current Mitigations** | 41. Lấy dấu vân tay phiên bản                                                                            |
| **Residual Risk**       | 43. Cao - Tự động cập nhật có thể tải về phiên bản độc hại                                               |
| **Khuyến nghị**         | 44. Triển khai ký cập nhật, khả năng hoàn nguyên, ghim phiên bản                                         |

#### 45. T-PERSIST-003: Can thiệp cấu hình tác nhân

| Attribute               | Giá trị                                                                                                                     |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | 48. AML.T0010.002 - Xâm phạm chuỗi cung ứng: Dữ liệu |
| **Description**         | 50. Kẻ tấn công sửa đổi cấu hình tác nhân để duy trì quyền truy cập                                  |
| **Attack Vector**       | Chỉnh sửa tệp cấu hình, chèn cài đặt                                                                                        |
| **Affected Components** | Agent config, tool policies                                                                                                 |
| **Current Mitigations** | Quyền tệp                                                                                                                   |
| **Residual Risk**       | Medium - Requires local access                                                                                              |
| **Khuyến nghị**         | Config integrity verification, audit logging for config changes                                                             |

---

### 3.5 Defense Evasion (AML.TA0007)

#### T-EVADE-001: Moderation Pattern Bypass

| Attribute               | Giá trị                                                                                   |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                        |
| **Description**         | Kẻ tấn công chế tác nội dung kỹ năng để né tránh các mẫu kiểm duyệt                       |
| **Attack Vector**       | Unicode homoglyphs, encoding tricks, dynamic loading                                      |
| **Affected Components** | ClawHub moderation.ts                                                     |
| **Current Mitigations** | Pattern-based FLAG_RULES                                             |
| **Residual Risk**       | High - Simple regex easily bypassed                                                       |
| **Khuyến nghị**         | Add behavioral analysis (VirusTotal Code Insight), AST-based detection |

#### T-EVADE-002: Content Wrapper Escape

| Attribute               | Giá trị                                                   |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data        |
| **Description**         | Attacker crafts content that escapes XML wrapper context  |
| **Attack Vector**       | Tag manipulation, context confusion, instruction override |
| **Affected Components** | External content wrapping                                 |
| **Current Mitigations** | XML tags + security notice                                |
| **Residual Risk**       | Medium - Novel escapes discovered regularly               |
| **Khuyến nghị**         | Multiple wrapper layers, output-side validation           |

---

### 3.6 Discovery (AML.TA0008)

#### T-DISC-001: Tool Enumeration

| Attribute               | Giá trị                                                   |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker enumerates available tools through prompting     |
| **Attack Vector**       | "What tools do you have?" style queries                   |
| **Affected Components** | Agent tool registry                                       |
| **Current Mitigations** | None specific                                             |
| **Residual Risk**       | Low - Tools generally documented                          |
| **Khuyến nghị**         | Consider tool visibility controls                         |

#### T-DISC-002: Session Data Extraction

| Attribute               | Giá trị                                                   |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker extracts sensitive data from session context     |
| **Attack Vector**       | "What did we discuss?" queries, context probing           |
| **Affected Components** | Session transcripts, context window                       |
| **Current Mitigations** | Session isolation per sender                              |
| **Residual Risk**       | Medium - Within-session data accessible                   |
| **Khuyến nghị**         | Implement sensitive data redaction in context             |

---

### 3.7 Collection & Exfiltration (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Data Theft via web_fetch

| Attribute               | Giá trị                                                                |
| ----------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                 |
| **Description**         | Attacker exfiltrates data by instructing agent to send to external URL |
| **Attack Vector**       | Prompt injection causing agent to POST data to attacker server         |
| **Affected Components** | web_fetch tool                                    |
| **Current Mitigations** | SSRF blocking for internal networks                                    |
| **Residual Risk**       | High - External URLs permitted                                         |
| **Khuyến nghị**         | Implement URL allowlisting, data classification awareness              |

#### T-EXFIL-002: Unauthorized Message Sending

| Attribute               | Giá trị                                                          |
| ----------------------- | ---------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                           |
| **Description**         | Attacker causes agent to send messages containing sensitive data |
| **Attack Vector**       | Prompt injection causing agent to message attacker               |
| **Affected Components** | Message tool, channel integrations                               |
| **Current Mitigations** | Outbound messaging gating                                        |
| **Residual Risk**       | Medium - Gating may be bypassed                                  |
| **Khuyến nghị**         | Require explicit confirmation for new recipients                 |

#### T-EXFIL-003: Credential Harvesting

| Attribute               | Giá trị                                                 |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                  |
| **Description**         | Malicious skill harvests credentials from agent context |
| **Attack Vector**       | Skill code reads environment variables, config files    |
| **Affected Components** | Skill execution environment                             |
| **Current Mitigations** | Không có gì cụ thể cho kỹ năng                          |
| **Residual Risk**       | Critical - Skills run with agent privileges             |
| **Khuyến nghị**         | Skill sandboxing, credential isolation                  |

---

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001: Unauthorized Command Execution

| Attribute               | Giá trị                                              |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker executes arbitrary commands on user system  |
| **Attack Vector**       | Prompt injection combined with exec approval bypass  |
| **Affected Components** | Bash tool, command execution                         |
| **Current Mitigations** | Exec approvals, Docker sandbox option                |
| **Residual Risk**       | Critical - Host execution without sandbox            |
| **Khuyến nghị**         | Default to sandbox, improve approval UX              |

#### T-IMPACT-002: Resource Exhaustion (DoS)

| Attribute               | Giá trị                                              |
| ----------------------- | ---------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity |
| **Description**         | Attacker exhausts API credits or compute resources   |
| **Attack Vector**       | Automated message flooding, expensive tool calls     |
| **Affected Components** | Gateway, agent sessions, API provider                |
| **Current Mitigations** | None                                                 |
| **Residual Risk**       | High - No rate limiting                              |
| **Khuyến nghị**         | Implement per-sender rate limits, cost budgets       |

#### T-IMPACT-003: Reputation Damage

| Attribute               | Giá trị                                                  |
| ----------------------- | -------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity     |
| **Description**         | Kẻ tấn công khiến tác nhân gửi nội dung gây hại/xúc phạm |
| **Attack Vector**       | Prompt injection gây ra phản hồi không phù hợp           |
| **Affected Components** | Tạo đầu ra, nhắn tin qua kênh                            |
| **Current Mitigations** | Chính sách nội dung của nhà cung cấp LLM                 |
| **Residual Risk**       | Trung bình - Bộ lọc của nhà cung cấp chưa hoàn hảo       |
| **Khuyến nghị**         | Lớp lọc đầu ra, kiểm soát người dùng                     |

---

## 9. 4. Phân tích chuỗi cung ứng ClawHub

### 4.1 Các kiểm soát bảo mật hiện tại

| Kiểm soát                        | Triển khai                                                          | Hiệu quả                                                        |
| -------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------- |
| Tuổi tài khoản GitHub            | `requireGitHubAccountAge()`                                         | Trung bình - Nâng cao rào cản cho kẻ tấn công mới               |
| Chuẩn hóa đường dẫn              | `sanitizePath()`                                                    | Cao - Ngăn chặn path traversal                                  |
| Xác thực loại tệp                | `isTextFile()`                                                      | Trung bình - Chỉ cho phép tệp văn bản, nhưng vẫn có thể độc hại |
| Giới hạn kích thước              | Tổng gói 50MB                                                       | Cao - Ngăn chặn cạn kiệt tài nguyên                             |
| Yêu cầu SKILL.md | Readme bắt buộc                                                     | Giá trị bảo mật thấp - Chỉ mang tính thông tin                  |
| Điều tiết theo mẫu               | FLAG_RULES trong moderation.ts | Thấp - Dễ bị vượt qua                                           |
| Trạng thái điều tiết             | `moderationStatus` field                                            | Trung bình - Có thể xem xét thủ công                            |

### 4.2 Các mẫu cờ điều tiết

Các mẫu hiện tại trong `moderation.ts`:

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

**Hạn chế:**

- Chỉ kiểm tra slug, displayName, summary, frontmatter, metadata, đường dẫn tệp
- Không phân tích nội dung mã kỹ năng thực tế
- Regex đơn giản dễ bị vượt qua bằng cách làm rối
- Không có phân tích hành vi

### 4.3 Các cải tiến dự kiến

| Cải tiến            | Trạng thái                                               | Impact                                                                |
| ------------------- | -------------------------------------------------------- | --------------------------------------------------------------------- |
| Tích hợp VirusTotal | Đang triển khai                                          | High - Code Insight behavioral analysis                               |
| Community Reporting | Partial (`skillReports` table exists) | Medium                                                                |
| Audit Logging       | Một phần (`auditLogs` table tồn tại)  | Medium                                                                |
| Badge System        | Implemented                                              | Medium - `highlighted`, `official`, `deprecated`, `redactionApproved` |

---

## 40. 5. Risk Matrix

### 5.1 Khả năng xảy ra so với tác động

| ID mối đe dọa | Khả năng xảy ra | Impact   | Mức độ rủi ro | Ưu tiên |
| ------------- | --------------- | -------- | ------------- | ------- |
| T-EXEC-001    | High            | Critical | **Critical**  | P0      |
| T-PERSIST-001 | High            | Critical | **Critical**  | P0      |
| T-EXFIL-003   | Medium          | Critical | **Critical**  | P0      |
| T-IMPACT-001  | Medium          | Critical | **High**      | P1      |
| T-EXEC-002    | High            | High     | **High**      | P1      |
| T-EXEC-004    | Medium          | High     | **High**      | P1      |
| T-ACCESS-003  | Medium          | High     | **High**      | P1      |
| T-EXFIL-001   | Medium          | High     | **High**      | P1      |
| T-IMPACT-002  | High            | Medium   | **High**      | P1      |
| T-EVADE-001   | High            | Medium   | **Medium**    | P2      |
| T-ACCESS-001  | Low             | High     | **Medium**    | P2      |
| T-ACCESS-002  | Low             | High     | **Medium**    | P2      |
| T-PERSIST-002 | Low             | High     | **Medium**    | P2      |

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

| ATLAS ID                                      | Technique Name                               | OpenClaw Threats                                                 |
| --------------------------------------------- | -------------------------------------------- | ---------------------------------------------------------------- |
| AML.T0006                     | Active Scanning                              | T-RECON-001, T-RECON-002                                         |
| AML.T0009                     | Collection                                   | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Supply Chain: AI Software    | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Supply Chain: Data           | T-PERSIST-003                                                    |
| AML.T0031                     | Làm xói mòn tính toàn vẹn của mô hình AI     | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040                     | Truy cập API suy luận mô hình AI             | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043                     | Tạo dữ liệu đối nghịch                       | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | Tiêm lời nhắc LLM: Trực tiếp | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | Tiêm lời nhắc LLM: Gián tiếp | T-EXEC-002                                                       |

### 7.2 Các tệp bảo mật then chốt

| dung lượng lưu trữ của bạn, không phải mô hình. | Mục đích                    | Mức độ rủi ro |
| --------------------------------------------------------------- | --------------------------- | ------------- |
| `src/infra/exec-approvals.ts`                                   | Logic phê duyệt lệnh        | **Critical**  |
| `src/gateway/auth.ts`                                           | Xác thực cổng               | **Critical**  |
| `src/web/inbound/access-control.ts`                             | Kiểm soát truy cập kênh     | **Critical**  |
| `src/infra/net/ssrf.ts`                                         | Bảo vệ SSRF                 | **Critical**  |
| `src/security/external-content.ts`                              | Giảm thiểu tiêm lời nhắc    | **Critical**  |
| `src/agents/sandbox/tool-policy.ts`                             | Thực thi chính sách công cụ | **Critical**  |
| `convex/lib/moderation.ts`                                      | Kiểm duyệt ClawHub          | **High**      |
| `convex/lib/skillPublish.ts`                                    | Quy trình phát hành kỹ năng | **High**      |
| `src/routing/resolve-route.ts`                                  | Cách ly phiên               | **Medium**    |

### 7.3 Thuật ngữ

| Thuật ngữ            | Định nghĩa                                                   |
| -------------------- | ------------------------------------------------------------ |
| **ATLAS**            | Bản đồ mối đe dọa đối nghịch cho hệ thống AI của MITRE       |
| **ClawHub**          | Chợ kỹ năng của OpenClaw                                     |
| **Gateway**          | Lớp định tuyến thông điệp và xác thực của OpenClaw           |
| **MCP**              | Giao thức Ngữ cảnh Mô hình - giao diện nhà cung cấp công cụ  |
| **Prompt Injection** | Tấn công trong đó các chỉ dẫn độc hại được nhúng vào đầu vào |
| **Skill**            | Tiện ích mở rộng có thể tải xuống cho các tác nhân OpenClaw  |
| **SSRF**             | Giả mạo yêu cầu phía máy chủ                                 |

---

_Mô hình mối đe dọa này là một tài liệu luôn được cập nhật._ Báo cáo các vấn đề bảo mật tới security@openclaw.ai_

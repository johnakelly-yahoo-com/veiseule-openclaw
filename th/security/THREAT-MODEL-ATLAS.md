# 4. แบบจำลองภัยคุกคาม OpenClaw v1.0

## กรอบงาน MITRE ATLAS

**เวอร์ชัน:** 1.0-draft
**อัปเดตล่าสุด:** 2026-02-04
**ระเบียบวิธี:** MITRE ATLAS + แผนภาพการไหลของข้อมูล
**กรอบงาน:** [MITRE ATLAS](https://atlas.mitre.org/) (ภูมิทัศน์ภัยคุกคามเชิงปฏิปักษ์สำหรับระบบ AI)

### 7. การระบุแหล่งที่มาของกรอบงาน

แบบจำลองภัยคุกคามนี้สร้างขึ้นบน [MITRE ATLAS](https://atlas.mitre.org/) ซึ่งเป็นกรอบงานมาตรฐานอุตสาหกรรมสำหรับการจัดทำเอกสารภัยคุกคามเชิงปฏิปักษ์ต่อระบบ AI/ML 9. ATLAS ได้รับการดูแลโดย [MITRE](https://www.mitre.org/) ร่วมกับชุมชนความปลอดภัยด้าน AI

**ทรัพยากร ATLAS หลัก:**

- [เทคนิค ATLAS](https://atlas.mitre.org/techniques/)
- [ยุทธวิธี ATLAS](https://atlas.mitre.org/tactics/)
- [กรณีศึกษา ATLAS](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [การมีส่วนร่วมกับ ATLAS](https://atlas.mitre.org/resources/contribute)

### การมีส่วนร่วมในแบบจำลองภัยคุกคามนี้

เอกสารนี้เป็นเอกสารที่มีการปรับปรุงอย่างต่อเนื่อง โดยได้รับการดูแลจากชุมชน OpenClaw 18. ดู [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) สำหรับแนวทางการมีส่วนร่วม:

- การรายงานภัยคุกคามใหม่
- การอัปเดตภัยคุกคามที่มีอยู่
- การเสนอห่วงโซ่การโจมตี
- การเสนอแนวทางการบรรเทา

---

## 1. F1

### 1.1 วัตถุประสงค์

แบบจำลองภัยคุกคามนี้จัดทำเอกสารภัยคุกคามเชิงปฏิปักษ์ต่อแพลตฟอร์มเอเจนต์ AI ของ OpenClaw และตลาดทักษะ ClawHub โดยใช้กรอบงาน MITRE ATLAS ซึ่งออกแบบมาเฉพาะสำหรับระบบ AI/ML

### 1.2 ขอบเขต

| Component         | รวมอยู่ | หมายเหตุ                                            |
| ----------------- | ------- | --------------------------------------------------- |
| แอตทริบิวต์       | ใช่     | การทำงานหลักของเอเจนต์ การเรียกใช้เครื่องมือ เซสชัน |
| Gateway           | ใช่     | การยืนยันตัวตน การกำหนดเส้นทาง การผสานรวมช่องทาง    |
| การผสานรวมช่องทาง | ใช่     | WhatsApp, Telegram, Discord, Signal, Slack ฯลฯ      |
| วิกฤต             | ใช่     | การเผยแพร่ทักษะ การกลั่นกรอง การกระจาย              |
| เซิร์ฟเวอร์ MCP   | ใช่     | ผู้ให้บริการเครื่องมือภายนอก                        |
| อุปกรณ์ผู้ใช้     | บางส่วน | แอปมือถือ ไคลเอนต์เดสก์ท็อป                         |

### 1.3 นอกขอบเขต

ไม่มีสิ่งใดถูกระบุว่าอยู่นอกขอบเขตอย่างชัดเจนสำหรับแบบจำลองภัยคุกคามนี้

---

## 2. ช่องทาง

### 2.1 ขอบเขตความเชื่อถือ

```
44. ┌─────────────────────────────────────────────────────────────────┐
│                    โซนที่ไม่ไว้วางใจ                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 ขอบเขตความเชื่อถือ 1: การเข้าถึงช่องทาง          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      เกตเวย์                              │   │
│  │  • การจับคู่อุปกรณ์ (ช่วงผ่อนผัน 30 วินาที)               │   │
│  │  • การตรวจสอบ AllowFrom / AllowList                      │   │
│  │  • การยืนยันตัวตนด้วยโทเคน/รหัสผ่าน/Tailscale            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 ขอบเขตความเชื่อถือ 2: การแยกเซสชัน              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   เซสชันเอเจนต์                          │   │
│  │  • คีย์เซสชัน = agent:channel:peer                       │   │
│  │  • นโยบายเครื่องมือต่อเอเจนต์                           │   │
│  │  • การบันทึกทรานสคริปต์                                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 ขอบเขตความเชื่อถือ 3: การเรียกใช้เครื่องมือ      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  แซนด์บ็อกซ์การทำงาน                    │   │
│  │  • Docker sandbox หรือโฮสต์ (ต้องอนุมัติการ exec)        │   │
│  │  • การเรียกใช้ระยะไกลของโหนด                           │   │
│  │  • การป้องกัน SSRF (DNS pinning + การบล็อก IP)          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 ขอบเขตความเชื่อถือ 4: เนื้อหาภายนอก             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              URL / อีเมล / เว็บฮุคที่ดึงมา               │   │
│  │  • การห่อหุ้มเนื้อหาภายนอก (แท็ก XML)                  │   │
│  │  • การแทรกประกาศด้านความปลอดภัย                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 ขอบเขตความเชื่อถือ 5: ซัพพลายเชน               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • การเผยแพร่ทักษะ (semver, ต้องมี SKILL.md)            │   │
│  │  • ธงการกลั่นกรองตามรูปแบบ                             │   │
│  │  • การสแกน VirusTotal (เร็ว ๆ นี้)                      │   │
│  │  • การตรวจสอบอายุบัญชี GitHub                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 การไหลของข้อมูล

| การไหล                       | แหล่งที่มา                        | สูง        | ข้อมูล                              | การป้องกัน           |
| ---------------------------- | --------------------------------- | ---------- | ----------------------------------- | -------------------- |
| 1. F1 | 2. ช่องทาง | Gateway    | ข้อความผู้ใช้                       | TLS, AllowFrom       |
| F2                           | Gateway                           | เอเจนต์    | ข้อความที่ถูกกำหนดเส้นทาง           | Session isolation    |
| F3                           | เอเจนต์                           | เครื่องมือ | คุณลักษณะ                           | การบังคับใช้นโยบาย   |
| F4                           | เอเจนต์                           | ปานกลาง    | คำขอ web_fetch | การป้องกัน SSRF      |
| F5                           | ClawHub                           | เอเจนต์    | โค้ดสกิล                            | การกลั่นกรอง การสแกน |
| F6                           | เอเจนต์                           | ช่องทาง    | Responses                           | การกรองเอาต์พุต      |

---

## 4. แบบจำลองภัยคุกคาม OpenClaw v1.0

### 3.1 การลาดตระเวน (AML.TA0002)

#### T-RECON-001: การค้นหาเอนด์พอยต์ของเอเจนต์

| Attribute               | ค่า                                                                      |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | การกลั่นกรองตามแพตเทิร์น                                                 |
| **Description**         | ผู้โจมตีสแกนหาเอ็นด์พอยต์เกตเวย์ OpenClaw ที่เปิดเผย                     |
| **Attack Vector**       | การสแกนเครือข่าย, การค้นหาด้วย Shodan, การสำรวจ DNS                      |
| **Affected Components** | เกตเวย์, เอ็นด์พอยต์ API ที่เปิดเผย                                      |
| **Current Mitigations** | ตัวเลือกการยืนยันตัวตนด้วย Tailscale, ผูกกับ loopback เป็นค่าเริ่มต้น    |
| **Residual Risk**       | ปานกลาง - เกตเวย์สาธารณะสามารถถูกค้นพบได้                                |
| **คำแนะนำ**             | จัดทำเอกสารการปรับใช้ที่ปลอดภัย, เพิ่มการจำกัดอัตราบนเอ็นด์พอยต์การค้นพบ |

#### T-RECON-002: การทดสอบการผสานรวมช่องทาง

| Attribute               | ค่า                                                               |
| ----------------------- | ----------------------------------------------------------------- |
| **ATLAS ID**            | README บังคับ                                                     |
| **Description**         | ผู้โจมตีทดสอบช่องทางการส่งข้อความเพื่อระบุบัญชีที่ถูกจัดการโดย AI |
| **Attack Vector**       | การส่งข้อความทดสอบ, การสังเกตรูปแบบการตอบสนอง                     |
| **Affected Components** | All channel integrations                                          |
| **Current Mitigations** | None specific                                                     |
| **Residual Risk**       | Low - Limited value from discovery alone                          |
| **คำแนะนำ**             | Consider response timing randomization                            |

---

### 3.2 Initial Access (AML.TA0004)

#### T-ACCESS-001: Pairing Code Interception

| Attribute               | ค่า                                                       |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | Attacker intercepts pairing code during 30s grace period  |
| **Attack Vector**       | Shoulder surfing, network sniffing, social engineering    |
| **Affected Components** | Device pairing system                                     |
| **Current Mitigations** | 30s expiry, codes sent via existing channel               |
| **Residual Risk**       | Medium - Grace period exploitable                         |
| **คำแนะนำ**             | Reduce grace period, add confirmation step                |

#### T-ACCESS-002: AllowFrom Spoofing

| Attribute               | ค่า                                                                            |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                      |
| **Description**         | Attacker spoofs allowed sender identity in channel                             |
| **Attack Vector**       | Depends on channel - phone number spoofing, username impersonation             |
| **Affected Components** | AllowFrom validation per channel                                               |
| **Current Mitigations** | Channel-specific identity verification                                         |
| **Residual Risk**       | Medium - Some channels vulnerable to spoofing                                  |
| **คำแนะนำ**             | Document channel-specific risks, add cryptographic verification where possible |

#### T-ACCESS-003: Token Theft

| Attribute               | ค่า                                                                         |
| ----------------------- | --------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access                   |
| **Description**         | Attacker steals authentication tokens from config files                     |
| **Attack Vector**       | มัลแวร์ การเข้าถึงอุปกรณ์โดยไม่ได้รับอนุญาต การเปิดเผยข้อมูลสำรองการตั้งค่า |
| **Affected Components** | ~/.openclaw/credentials/, config storage    |
| **Current Mitigations** | File permissions                                                            |
| **Residual Risk**       | สูง - โทเคนถูกจัดเก็บเป็นข้อความล้วน                                        |
| **คำแนะนำ**             | นำการเข้ารหัสโทเคนขณะจัดเก็บมาใช้ เพิ่มการหมุนเวียนโทเคน                    |

---

### 3.3 การดำเนินการ (AML.TA0005)

#### T-EXEC-001: การฉีดคำสั่งโดยตรง (Direct Prompt Injection)

| Attribute               | ค่า                                                                                      |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - การฉีดคำสั่ง LLM: โดยตรง |
| **Description**         | ผู้โจมตีส่งพรอมป์ที่ถูกออกแบบมาเพื่อบิดเบือนพฤติกรรมของเอเจนต์                           |
| **Attack Vector**       | ข้อความในแชนเนลที่มีคำสั่งเชิงปฏิปักษ์                                                   |
| **Affected Components** | Agent LLM ทุกพื้นผิวอินพุต                                                               |
| **Current Mitigations** | การตรวจจับรูปแบบ, การครอบเนื้อหาภายนอก                                                   |
| **Residual Risk**       | วิกฤต - ตรวจจับได้เท่านั้น ไม่มีการบล็อก; การโจมตีขั้นสูงสามารถเลี่ยงได้                 |
| **คำแนะนำ**             | นำการป้องกันหลายชั้นมาใช้ การตรวจสอบเอาต์พุต การยืนยันจากผู้ใช้สำหรับการกระทำที่อ่อนไหว  |

#### T-EXEC-002: การฉีดคำสั่งทางอ้อม (Indirect Prompt Injection)

| Attribute               | ค่า                                                                                       |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - การฉีดคำสั่ง LLM: ทางอ้อม |
| **Description**         | ผู้โจมตีฝังคำสั่งที่เป็นอันตรายในเนื้อหาที่ถูกดึงมา                                       |
| **Attack Vector**       | URL ที่เป็นอันตราย อีเมลที่ถูกวางยาพิษ เว็บฮุคที่ถูกบุกรุก                                |
| **Affected Components** | web_fetch, การนำเข้าอีเมล, แหล่งข้อมูลภายนอก                         |
| **Current Mitigations** | การครอบเนื้อหาด้วยแท็ก XML และประกาศด้านความปลอดภัย                                       |
| **Residual Risk**       | สูง - LLM อาจเพิกเฉยต่อคำสั่งการครอบ                                                      |
| **คำแนะนำ**             | นำการทำความสะอาดเนื้อหา การแยกบริบทการทำงาน                                               |

#### T-EXEC-003: การฉีดอาร์กิวเมนต์ของเครื่องมือ

| Attribute               | ค่า                                                                                      |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - การฉีดคำสั่ง LLM: โดยตรง |
| **Description**         | ผู้โจมตีบิดเบือนอาร์กิวเมนต์ของเครื่องมือผ่านการฉีดคำสั่ง                                |
| **Attack Vector**       | พรอมป์ที่ถูกออกแบบมาเพื่อมีอิทธิพลต่อค่าพารามิเตอร์ของเครื่องมือ                         |
| **Affected Components** | การเรียกใช้เครื่องมือทั้งหมด                                                             |
| **Current Mitigations** | การอนุมัติการรันสำหรับคำสั่งที่เป็นอันตราย                                               |
| **Residual Risk**       | สูง - พึ่งพาดุลยพินิจของผู้ใช้                                                           |
| **คำแนะนำ**             | นำการตรวจสอบอาร์กิวเมนต์ การเรียกใช้เครื่องมือแบบกำหนดพารามิเตอร์มาใช้                   |

#### T-EXEC-004: การข้ามการอนุมัติการรัน (Exec Approval Bypass)

| Attribute               | ค่า                                                        |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data         |
| **Description**         | Attacker crafts commands that bypass approval allowlist    |
| **Attack Vector**       | Command obfuscation, alias exploitation, path manipulation |
| **Affected Components** | exec-approvals.ts, command allowlist       |
| **Current Mitigations** | Allowlist + ask mode                                       |
| **Residual Risk**       | High - No command sanitization                             |
| **คำแนะนำ**             | Implement command normalization, expand blocklist          |

---

### 3.4 Persistence (AML.TA0006)

#### T-PERSIST-001: Malicious Skill Installation

| Attribute               | ค่า                                                                                                  |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker publishes malicious skill to ClawHub                                                        |
| **Attack Vector**       | Create account, publish skill with hidden malicious code                                             |
| **Affected Components** | ClawHub, skill loading, agent execution                                                              |
| **Current Mitigations** | GitHub account age verification, pattern-based moderation flags                                      |
| **Residual Risk**       | Critical - No sandboxing, limited review                                                             |
| **คำแนะนำ**             | VirusTotal integration (in progress), skill sandboxing, community review          |

#### T-PERSIST-002: Skill Update Poisoning

| Attribute               | ค่า                                                                                                  |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software |
| **Description**         | Attacker compromises popular skill and pushes malicious update                                       |
| **Attack Vector**       | Account compromise, social engineering of skill owner                                                |
| **Affected Components** | ClawHub versioning, auto-update flows                                                                |
| **Current Mitigations** | Version fingerprinting                                                                               |
| **Residual Risk**       | High - Auto-updates may pull malicious versions                                                      |
| **คำแนะนำ**             | Implement update signing, rollback capability, version pinning                                       |

#### T-PERSIST-003: Agent Configuration Tampering

| Attribute               | ค่า                                                                                           |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Supply Chain Compromise: Data |
| **Description**         | ผู้โจมตีแก้ไขการกำหนดค่าเอเจนต์เพื่อคงการเข้าถึงไว้                                           |
| **Attack Vector**       | การแก้ไขไฟล์กำหนดค่า การแทรกการตั้งค่า                                                        |
| **Affected Components** | การกำหนดค่าเอเจนต์ นโยบายเครื่องมือ                                                           |
| **Current Mitigations** | File permissions                                                                              |
| **Residual Risk**       | ปานกลาง - ต้องการการเข้าถึงในเครื่อง                                                          |
| **คำแนะนำ**             | การตรวจสอบความสมบูรณ์ของการกำหนดค่า การบันทึกการตรวจสอบสำหรับการเปลี่ยนแปลงการกำหนดค่า        |

---

### 3.5 การหลบเลี่ยงการป้องกัน (AML.TA0007)

#### T-EVADE-001: การหลบเลี่ยงรูปแบบการกลั่นกรอง

| Attribute               | ค่า                                                                                             |
| ----------------------- | ----------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                              |
| **Description**         | ผู้โจมตีสร้างเนื้อหาทักษะเพื่อหลบเลี่ยงรูปแบบการกลั่นกรอง                                       |
| **Attack Vector**       | โฮโมกลิฟ Unicode เทคนิคการเข้ารหัส การโหลดแบบไดนามิก                                            |
| **Affected Components** | ClawHub moderation.ts                                                           |
| **Current Mitigations** | FLAG_RULES แบบอิงรูปแบบ                                                    |
| **Residual Risk**       | สูง - regex แบบง่ายถูกหลบเลี่ยงได้ง่าย                                                          |
| **คำแนะนำ**             | เพิ่มการวิเคราะห์เชิงพฤติกรรม (VirusTotal Code Insight) การตรวจจับแบบอิง AST |

#### T-EVADE-002: การหลบหนีจากตัวห่อหุ้มเนื้อหา

| Attribute               | ค่า                                                     |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data      |
| **Description**         | ผู้โจมตีสร้างเนื้อหาที่หลบหนีบริบทของตัวห่อหุ้ม XML     |
| **Attack Vector**       | การจัดการแท็ก ความสับสนของบริบท การเขียนทับคำสั่ง       |
| **Affected Components** | การห่อหุ้มเนื้อหาภายนอก                                 |
| **Current Mitigations** | แท็ก XML + ประกาศความปลอดภัย                            |
| **Residual Risk**       | ปานกลาง - การหลบหนีรูปแบบใหม่ถูกค้นพบเป็นประจำ          |
| **คำแนะนำ**             | หลายชั้นของตัวห่อหุ้ม การตรวจสอบความถูกต้องฝั่งเอาต์พุต |

---

### 3.6 การค้นหา (AML.TA0008)

#### T-DISC-001: การไล่รายการเครื่องมือ

| Attribute               | ค่า                                                       |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | ผู้โจมตีไล่รายการเครื่องมือที่มีอยู่ผ่านการตั้งคำถาม      |
| **Attack Vector**       | "คุณมีเครื่องมืออะไรบ้าง?" ลักษณะของคำถาม                 |
| **Affected Components** | รีจิสทรีเครื่องมือของเอเจนต์                              |
| **Current Mitigations** | None specific                                             |
| **Residual Risk**       | ต่ำ - เครื่องมือส่วนใหญ่มีการบันทึกเอกสารไว้แล้ว          |
| **คำแนะนำ**             | พิจารณาการควบคุมการมองเห็นของเครื่องมือ                   |

#### สูง - โทเคนถูกจัดเก็บเป็นข้อความล้วน

| Attribute               | ค่า                                                       |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - AI Model Inference API Access |
| **Description**         | ผู้โจมตีดึงข้อมูลอ่อนไหวจากบริบทของเซสชัน                 |
| **Attack Vector**       | คำถาม "เราคุยอะไรกันไป?", การสำรวจบริบท                   |
| **Affected Components** | บันทึกถอดความเซสชัน, หน้าต่างบริบท                        |
| **Current Mitigations** | การแยกเซสชันตามผู้ส่ง                                     |
| **Residual Risk**       | ปานกลาง - สามารถเข้าถึงข้อมูลภายในเซสชันได้               |
| **คำแนะนำ**             | ดำเนินการปกปิดข้อมูลอ่อนไหวในบริบท                        |

---

### 3.7 การรวบรวมและการส่งออกข้อมูล (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: การขโมยข้อมูลผ่าน web_fetch

| Attribute               | ค่า                                                                  |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - การสร้างข้อมูลเชิงปฏิปักษ์               |
| **Description**         | ผู้โจมตีส่งข้อมูลออกโดยสั่งให้เอเจนต์ส่งไปยัง URL ภายนอก             |
| **Attack Vector**       | การฉีดพรอมป์ต์ที่ทำให้เอเจนต์ POST ข้อมูลไปยังเซิร์ฟเวอร์ของผู้โจมตี |
| **Affected Components** | การปรับปรุง                                                          |
| **Current Mitigations** | การบล็อก SSRF สำหรับเครือข่ายภายใน                                   |
| **Residual Risk**       | สูง - อนุญาต URL ภายนอก                                              |
| **คำแนะนำ**             | ดำเนินการทำ allowlist ของ URL และการตระหนักรู้การจัดประเภทข้อมูล     |

#### T-EXFIL-002: การส่งข้อความโดยไม่ได้รับอนุญาต

| Attribute               | ค่า                                                    |
| ----------------------- | ------------------------------------------------------ |
| **ATLAS ID**            | AML.T0009 - การรวบรวมข้อมูล            |
| **Description**         | ผู้โจมตีทำให้เอเจนต์ส่งข้อความที่มีข้อมูลอ่อนไหว       |
| **Attack Vector**       | การฉีดพรอมป์ต์ที่ทำให้เอเจนต์ส่งข้อความถึงผู้โจมตี     |
| **Affected Components** | เครื่องมือข้อความ, การเชื่อมต่อช่องทาง                 |
| **Current Mitigations** | 50. การควบคุมการส่งข้อความขาออก |
| **Residual Risk**       | ปานกลาง - การกั้นอาจถูกหลีกเลี่ยงได้                   |
| **คำแนะนำ**             | ต้องการการยืนยันอย่างชัดเจนสำหรับผู้รับรายใหม่         |

#### สิทธิ์ของไฟล์

| Attribute               | ค่า                                                        |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - การสร้างข้อมูลเชิงปฏิปักษ์     |
| **Description**         | สกิลที่เป็นอันตรายเก็บเกี่ยวข้อมูลรับรองจากบริบทของเอเจนต์ |
| **Attack Vector**       | โค้ดสกิลอ่านตัวแปรสภาพแวดล้อมและไฟล์คอนฟิก                 |
| **Affected Components** | สภาพแวดล้อมการรันสกิล                                      |
| **Current Mitigations** | ไม่มีมาตรการเฉพาะสำหรับสกิล                                |
| **Residual Risk**       | วิกฤต - สกิลรันด้วยสิทธิ์ของเอเจนต์                        |
| **คำแนะนำ**             | แซนด์บ็อกซ์สกิล, การแยกข้อมูลรับรอง                        |

---

### **องค์ประกอบที่ได้รับผลกระทบ**

#### ข้อความในแชนเนลที่มีคำสั่งเชิงปฏิปักษ์

| Attribute               | ค่า                                                         |
| ----------------------- | ----------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - บ่อนทำลายความสมบูรณ์ของโมเดล AI |
| **Description**         | ผู้โจมตีรันคำสั่งใด ๆ บนระบบของผู้ใช้                       |
| **Attack Vector**       | การฉีดพรอมต์ร่วมกับการเลี่ยงการอนุมัติการรันคำสั่ง          |
| **Affected Components** | เครื่องมือ Bash, การรันคำสั่ง                               |
| **Current Mitigations** | การอนุมัติการรันคำสั่ง, ตัวเลือกแซนด์บ็อกซ์ Docker          |
| **Residual Risk**       | วิกฤต - รันบนโฮสต์โดยไม่มีแซนด์บ็อกซ์                       |
| **คำแนะนำ**             | ตั้งค่าเริ่มต้นเป็นแซนด์บ็อกซ์, ปรับปรุง UX การอนุมัติ      |

#### T-IMPACT-002: การใช้ทรัพยากรจนหมด (DoS)

| Attribute               | ค่า                                                                                                                                           |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **ATLAS ID**            | _โมเดลภัยคุกคามนี้เป็นเอกสารที่มีการปรับปรุงอย่างต่อเนื่อง รายงานปัญหาด้านความปลอดภัยไปที่ security@openclaw.ai_ |
| **Description**         | ผู้โจมตีทำให้เครดิต API หรือทรัพยากรคอมพิวต์หมด                                                                                               |
| **Attack Vector**       | การส่งข้อความอัตโนมัติถี่ ๆ, การเรียกใช้เครื่องมือที่มีค่าใช้จ่ายสูง                                                                          |
| **Affected Components** | เกตเวย์, เซสชันเอเจนต์, ผู้ให้บริการ API                                                                                                      |
| **Current Mitigations** | ไม่มี                                                                                                                                         |
| **Residual Risk**       | สูง - ไม่มีการจำกัดอัตรา                                                                                                                      |
| **คำแนะนำ**             | ใช้การจำกัดอัตราต่อผู้ส่ง, งบประมาณค่าใช้จ่าย                                                                                                 |

#### T-IMPACT-003: ความเสียหายต่อชื่อเสียง

| Attribute               | ค่า                                                             |
| ----------------------- | --------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - การเข้าถึง API การอนุมานของโมเดล AI |
| **Description**         | ผู้โจมตีทำให้เอเจนต์ส่งเนื้อหาที่เป็นอันตรายหรือไม่เหมาะสม      |
| **Attack Vector**       | การฉีดพรอมป์ต์ทำให้เกิดการตอบสนองที่ไม่เหมาะสม                  |
| **Affected Components** | การสร้างเอาต์พุต, การส่งข้อความผ่านช่องทาง                      |
| **Current Mitigations** | นโยบายเนื้อหาของผู้ให้บริการ LLM                                |
| **Residual Risk**       | ปานกลาง - ตัวกรองของผู้ให้บริการยังไม่สมบูรณ์                   |
| **คำแนะนำ**             | ชั้นการกรองเอาต์พุต, การควบคุมโดยผู้ใช้                         |

---

## 4. การวิเคราะห์ซัพพลายเชนของ ClawHub

### 4.1 การควบคุมความปลอดภัยปัจจุบัน

| การควบคุม                       | การนำไปใช้งาน                                                    | ประสิทธิผล                                               |
| ------------------------------- | ---------------------------------------------------------------- | -------------------------------------------------------- |
| อายุบัญชี GitHub                | `requireGitHubAccountAge()`                                      | ปานกลาง - เพิ่มอุปสรรคให้กับผู้โจมตีรายใหม่              |
| การทำความสะอาดพาธ               | `sanitizePath()`                                                 | สูง - ป้องกันการไต่ระดับพาธ                              |
| การตรวจสอบประเภทไฟล์            | `isTextFile()`                                                   | ปานกลาง - อนุญาตเฉพาะไฟล์ข้อความ แต่ยังอาจเป็นอันตรายได้ |
| ขีดจำกัดขนาด                    | รวมแพ็กเกจ 50MB                                                  | สูง - ป้องกันการใช้ทรัพยากรจนหมด                         |
| ต้องมี SKILL.md | README บังคับ                                                    | คุณค่าด้านความปลอดภัยต่ำ - ให้ข้อมูลเท่านั้น             |
| การกลั่นกรองตามแพตเทิร์น        | FLAG_RULES ใน moderation.ts | ต่ำ - หลีกเลี่ยงได้ง่าย                                  |
| สถานะการกลั่นกรอง               | `moderationStatus` ฟิลด์                                         | ปานกลาง - สามารถตรวจทานด้วยตนเองได้                      |

### 4.2 แพตเทิร์นแฟล็กการกลั่นกรอง

แพตเทิร์นปัจจุบันใน `moderation.ts`:// Known-bad identifiers
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i// Suspicious keywords
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+|\s\*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i

```javascript
แพตเทิร์นปัจจุบันใน `moderation.ts`:// Known-bad identifiers
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i// Suspicious keywords
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+|\s\*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i
```

**ข้อจำกัด:**

- ตรวจสอบเฉพาะ slug, displayName, summary, frontmatter, metadata, พาธไฟล์
- ไม่วิเคราะห์เนื้อหาโค้ดของสกิลจริง
- รีเจ็กซ์แบบง่ายสามารถหลีกเลี่ยงได้ด้วยการทำให้คลุมเครือ
- ไม่มีการวิเคราะห์เชิงพฤติกรรม

### 4.3 การปรับปรุงที่วางแผนไว้

| Improvement           | Status                                                | ผลกระทบ                                                                |
| --------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------- |
| การผสานรวม VirusTotal | อยู่ระหว่างดำเนินการ                                  | สูง - การวิเคราะห์พฤติกรรมจากข้อมูลเชิงลึกของโค้ด                      |
| การรายงานโดยชุมชน     | บางส่วน (มีตาราง `skillReports`)   | Medium                                                                 |
| การบันทึกการตรวจสอบ   | บางส่วน (`auditLogs` table exists) | Medium                                                                 |
| ระบบป้าย              | นำไปใช้แล้ว                                           | ปานกลาง - `highlighted`, `official`, `deprecated`, `redactionApproved` |

---

## 5. เมทริกซ์ความเสี่ยง

### 5.1 ความน่าจะเป็น เทียบกับ ผลกระทบ

| รหัสภัยคุกคาม | ความน่าจะเป็น | ผลกระทบ | Risk Level   | ลำดับความสำคัญ |
| ------------- | ------------- | ------- | ------------ | -------------- |
| T-EXEC-001    | High          | วิกฤต   | **Critical** | P0             |
| T-PERSIST-001 | High          | วิกฤต   | **Critical** | P0             |
| T-EXFIL-003   | Medium        | วิกฤต   | **Critical** | P0             |
| T-IMPACT-001  | Medium        | วิกฤต   | **High**     | P1             |
| T-EXEC-002    | High          | High    | **High**     | P1             |
| T-EXEC-004    | Medium        | High    | **High**     | P1             |
| T-ACCESS-003  | Medium        | High    | **High**     | P1             |
| T-EXFIL-001   | Medium        | High    | **High**     | P1             |
| T-IMPACT-002  | High          | Medium  | **High**     | P1             |
| T-EVADE-001   | High          | Medium  | **Medium**   | P2             |
| T-ACCESS-001  | Low           | High    | **Medium**   | P2             |
| T-ACCESS-002  | Low           | High    | **Medium**   | P2             |
| T-PERSIST-002 | Low           | High    | **Medium**   | P2             |

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

### 7.2 ไฟล์ความปลอดภัยหลัก

| พาธ                                 | วัตถุประสงค์                | Risk Level   |
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

| Term                 | Definition                                                 |
| -------------------- | ---------------------------------------------------------- |
| **ATLAS**            | MITRE's Adversarial Threat Landscape for AI Systems        |
| **ClawHub**          | ตลาดสกิลของ OpenClaw                                       |
| **Gateway**          | เลเยอร์การกำหนดเส้นทางข้อความและการยืนยันตัวตนของ OpenClaw |
| **MCP**              | Model Context Protocol - อินเทอร์เฟซผู้ให้บริการเครื่องมือ |
| **Prompt Injection** | การโจมตีที่ฝังคำสั่งที่เป็นอันตรายไว้ในอินพุต              |
| **Skill**            | ส่วนขยายที่ดาวน์โหลดได้สำหรับเอเจนต์ OpenClaw              |
| **SSRF**             | Server-Side Request Forgery                                |

---

_โมเดลภัยคุกคามนี้เป็นเอกสารที่มีการปรับปรุงอย่างต่อเนื่อง รายงานปัญหาด้านความปลอดภัยไปที่ security@openclaw.ai_


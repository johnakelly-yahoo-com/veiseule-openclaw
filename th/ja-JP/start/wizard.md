---
read_when:
  - เมื่อเรียกใช้หรือกำหนดค่าオンボーディングウィザード
  - เมื่อตั้งค่าเครื่องใหม่
sidebarTitle: Wizard (CLI)
summary: "CLIオンボーディングウィザード: การตั้งค่า Gateway, เวิร์กสเปซ, ช่องทาง และ Skills แบบโต้ตอบ"
title: オンボーディングウィザード (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# オンボーディングウィザード (CLI)

CLIオンボーディングウィザードเป็นเส้นทางที่แนะนำสำหรับการตั้งค่า OpenClaw บน macOS, Linux และ Windows (ผ่าน WSL2) โดยจะกำหนดค่า Gateway ภายในเครื่องหรือการเชื่อมต่อ Gateway ระยะไกล พร้อมทั้งตั้งค่าเริ่มต้นของเวิร์กสเปซ ช่องทาง และ Skills

```bash
openclaw onboard
```

<Info>
วิธีที่เร็วที่สุดในการเริ่มแชทครั้งแรก: เปิด Control UI (ไม่ต้องตั้งค่าช่องทาง) รัน `openclaw dashboard` แล้วคุณสามารถแชทผ่านเบราว์เซอร์ได้ เอกสาร: [Dashboard](/web/dashboard)。
</Info>

## การเริ่มต้นอย่างรวดเร็ว vs การตั้งค่าขั้นสูง

วิซาร์ดจะเริ่มต้นโดยให้คุณเลือก **การเริ่มต้นอย่างรวดเร็ว** (การตั้งค่าเริ่มต้น) หรือ **การตั้งค่าขั้นสูง** (ควบคุมได้อย่างเต็มรูปแบบ)

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">- Gateway ภายในเครื่องบน loopback
- เวิร์กสเปซที่มีอยู่หรือเวิร์กสเปซเริ่มต้น
- พอร์ต Gateway `18789`
- โทเค็นการยืนยันตัวตนของ Gateway ถูกสร้างอัตโนมัติ (สร้างบน loopback เช่นกัน)
- ปิดการเผยแพร่ผ่าน Tailscale
- DM ของ Telegram และ WhatsApp ถูกตั้งค่าเป็น allowlist โดยค่าเริ่มต้น (อาจมีการขอให้ป้อนหมายเลขโทรศัพท์)
</Tab>
  <Tab title="詳細設定（完全な制御）">- แสดงโฟลว์พรอมป์แบบครบถ้วนสำหรับโหมด เวิร์กสเปซ Gateway ช่องทาง เดมอน และ Skills
</Tab>
</Tabs>

## รายละเอียดการเริ่มต้นใช้งาน CLI

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">คำอธิบายโฟลว์ทั้งแบบโลคัลและรีโมตอย่างครบถ้วน รวมถึงการยืนยันตัวตนและเมทริกซ์ของโมเดล เอาต์พุตการตั้งค่า Wizard RPC และการทำงานของ signal-cli
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">ขั้นตอนสำหรับการเริ่มต้นแบบไม่โต้ตอบและตัวอย่าง `agents add` แบบอัตโนมัติ
</Card>
</Columns>

## คำสั่งติดตามที่ใช้บ่อย

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` ไม่ได้หมายถึงโหมดไม่โต้ตอบ สำหรับสคริปต์ให้ใช้ `--non-interactive`
</Note>

<Tip>
คำแนะนำ: ตั้งค่า Brave Search API key เพื่อให้เอเจนต์สามารถใช้ `web_search` ได้ (`web_fetch` ทำงานได้โดยไม่ต้องใช้คีย์) วิธีที่ง่ายที่สุด: รัน `openclaw configure --section web` เพื่อบันทึก `tools.web.search.apiKey` ดูเอกสาร: [Webツール](/tools/web)
</Tip>

## เอกสารที่เกี่ยวข้อง

- คู่มืออ้างอิงคำสั่ง CLI: [`openclaw onboard`](/cli/onboard)
- การเริ่มต้นใช้งานแอป macOS: [オンボーディング](/start/onboarding)
- ขั้นตอนการเปิดใช้งานครั้งแรกของเอเจนต์: [エージェントブートストラップ](/start/bootstrapping)

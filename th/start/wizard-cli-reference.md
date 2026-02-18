---
title: "เอกสารอ้างอิงการเริ่มต้นใช้งานผ่านCLI"
sidebarTitle: "เอกสารอ้างอิง CLI"
---

# เอกสารอ้างอิงการเริ่มต้นใช้งานผ่านCLI

หน้านี้คือเอกสารอ้างอิงฉบับเต็มสำหรับ `openclaw onboard`  
สำหรับคู่มือฉบับย่อโปรดดู [Onboarding Wizard (CLI)](/start/wizard)

## สิ่งที่วิซาร์ดทำ

โหมด Local (ค่าเริ่มต้น) จะพาคุณทำตามขั้นตอนต่อไปนี้:

- การตั้งค่าโมเดลและการยืนยันตัวตน (OpenAI Code subscription OAuth, Anthropic API key หรือ setup token รวมถึงตัวเลือก MiniMax, GLM, Moonshot และ AI Gateway)
- ตำแหน่ง Workspace และไฟล์บูตสแตรป
- การตั้งค่า Gateway (พอร์ต, การ bind, การยืนยันตัวตน, tailscale)
- ช่องทางและผู้ให้บริการ (Telegram, WhatsApp, Discord, Google Chat, Mattermost plugin, Signal)
- การติดตั้งเดมอน (LaunchAgent หรือ systemd user unit)
- การตรวจสุขภาพ
- การตั้งค่า Skills

โหมด Remote จะกำหนดค่าเครื่องนี้ให้เชื่อมต่อกับ Gateway ที่อยู่อื่น  
โหมดนี้จะไม่ติดตั้งหรือแก้ไขสิ่งใดบนโฮสต์ระยะไกล

## รายละเอียดโฟลว์แบบ Local

<Steps>
  <Step title="Existing config detection">
    - หากมี `~/.openclaw/openclaw.json` ให้เลือก เก็บไว้ (Keep), แก้ไข (Modify) หรือ รีเซ็ต (Reset)
    - การรันวิซาร์ดซ้ำจะไม่ลบข้อมูลใด ๆ เว้นแต่คุณจะเลือก Reset อย่างชัดเจน (หรือส่ง `--reset`)
    - หากคอนฟิกไม่ถูกต้องหรือมีคีย์แบบเดิม (legacy) วิซาร์ดจะหยุดและขอให้คุณรัน `openclaw doctor` ก่อนดำเนินการต่อ
    - การ Reset ใช้ `trash` และมีขอบเขตให้เลือก:
      - เฉพาะคอนฟิก
      - คอนฟิก + ข้อมูลรับรอง + เซสชัน
      - รีเซ็ตทั้งหมด (ลบ workspace ด้วย)
  </Step>
  <Step title="Model and auth">
    - เมทริกซ์ตัวเลือกแบบเต็มอยู่ที่ [Auth and model options](#auth-and-model-options)
  </Step>
  <Step title="Workspace">
    - ค่าเริ่มต้น `~/.openclaw/workspace` (ปรับได้)
    - สร้างไฟล์ Workspace ที่จำเป็นสำหรับพิธี bootstrap ครั้งแรก
    - โครงสร้าง Workspace: [Agent workspace](/concepts/agent-workspace)
  </Step>
  <Step title="Gateway">
    - ถามค่า port, bind, โหมดการยืนยันตัวตน และการเปิดใช้งานผ่าน tailscale
    - แนะนำ: คงการยืนยันตัวตนด้วยโทเคนไว้ แม้จะเป็น loopback เพื่อให้ไคลเอนต์ WS ภายในเครื่องต้องยืนยันตัวตน
    - ปิดการยืนยันตัวตนเฉพาะเมื่อคุณเชื่อถือทุกโปรเซสภายในเครื่องอย่างสมบูรณ์
    - การ bind ที่ไม่ใช่ loopback ยังคงต้องมีการยืนยันตัวตน
  </Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): การล็อกอินด้วย QR แบบไม่บังคับ
    - [Telegram](/channels/telegram): โทเคนบอต
    - [Discord](/channels/discord): โทเคนบอต
    - [Google Chat](/channels/googlechat): service account JSON + webhook audience
    - [Mattermost](/channels/mattermost) plugin: โทเคนบอต + base URL
    - [Signal](/channels/signal): การติดตั้ง `signal-cli` แบบไม่บังคับ + การตั้งค่าบัญชี
    - [BlueBubbles](/channels/bluebubbles): แนะนำสำหรับ iMessage; server URL + รหัสผ่าน + webhook
    - [iMessage](/channels/imessage): เส้นทาง CLI รุ่นเก่า `imsg` + การเข้าถึง DB
    - ความปลอดภัยของ DM: ค่าเริ่มต้นคือการจับคู่ (pairing) DM แรกจะส่งโค้ด; อนุมัติผ่าน
      `openclaw pairing approve <channel> <code>` หรือใช้ allowlists
  </Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - ต้องมีเซสชันผู้ใช้ที่ล็อกอินอยู่; สำหรับ headless ให้ใช้ LaunchDaemon แบบกำหนดเอง (ไม่ได้จัดส่ง)
    - Linux และ Windows ผ่าน WSL2: systemd user unit
      - วิซาร์ดพยายาม `loginctl enable-linger <user>` เพื่อให้ gateway ยังทำงานหลังจากออกจากระบบ
      - อาจขอ sudo (เขียน `/var/lib/systemd/linger`); จะพยายามโดยไม่ใช้ sudo ก่อน
    - การเลือกรันไทม์: Node (แนะนำ; จำเป็นสำหรับ WhatsApp และ Telegram) ไม่แนะนำ Bun
  </Step>
  <Step title="Health check">
    - เริ่ม gateway (หากจำเป็น) และรัน `openclaw health`
    - `openclaw status --deep` จะเพิ่ม gateway health probes ในเอาต์พุตสถานะ
  </Step>
  <Step title="Skills">
    - อ่าน Skills ที่มีและตรวจสอบข้อกำหนด
    - ให้คุณเลือกตัวจัดการแพ็กเกจ Node: npm หรือ pnpm (ไม่แนะนำ bun)
    - ติดตั้ง dependency เสริมแบบไม่บังคับ (บางรายการใช้ Homebrew บน macOS)
  </Step>
  <Step title="Finish">
    - สรุปและขั้นตอนถัดไป รวมถึงตัวเลือกแอป iOS, Android และ macOS
  </Step>
</Steps>

<Note>
หากไม่ตรวจพบ GUI วิซาร์ดจะพิมพ์คำแนะนำการทำ SSH port-forward สำหรับ Control UI แทนการเปิดเบราว์เซอร์  
หากไม่มี Control UI assets วิซาร์ดจะพยายามสร้างให้; ทางเลือกสำรองคือ `pnpm ui:build` (ติดตั้ง UI deps อัตโนมัติ)
</Note>

## รายละเอียดโหมด Remote

โหมด Remote จะกำหนดค่าเครื่องนี้ให้เชื่อมต่อกับ Gateway ที่อยู่อื่น

<Info>
โหมด Remote จะไม่ติดตั้งหรือแก้ไขสิ่งใดบนโฮสต์ระยะไกล
</Info>

สิ่งที่คุณตั้งค่า:

- URL ของ Remote gateway (`ws://...`)
- โทเคน หาก remote gateway ต้องการการยืนยันตัวตน (แนะนำ)

<Note>
- หาก gateway เป็น loopback เท่านั้น ให้ใช้การทำอุโมงค์ SSH หรือ tailnet
- คำใบ้การค้นหา (Discovery):
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)
</Note>

## ตัวเลือกการยืนยันตัวตนและโมเดล

<AccordionGroup>
  <Accordion title="Anthropic API key (recommended)">
    ใช้ `ANTHROPIC_API_KEY` หากมีอยู่ หรือจะขอคีย์แล้วบันทึกไว้เพื่อใช้กับเดมอน
  </Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: ตรวจสอบรายการ Keychain ชื่อ "Claude Code-credentials"
    - Linux และ Windows: ใช้ `~/.claude/.credentials.json` ซ้ำหากมีอยู่

    บน macOS ให้เลือก "Always Allow" เพื่อไม่ให้การเริ่มต้นด้วย launchd ถูกบล็อก
  </Accordion>
  <Accordion title="Anthropic token (setup-token paste)">
    รัน `claude setup-token` บนเครื่องใดก็ได้ แล้ววางโทเคน  
    คุณสามารถตั้งชื่อได้; เว้นว่างจะใช้ค่าเริ่มต้น
  </Accordion>
  <Accordion title="OpenAI Code subscription (Codex CLI reuse)">
    หากมี `~/.codex/auth.json` วิซาร์ดสามารถนำมาใช้ซ้ำได้
  </Accordion>
  <Accordion title="OpenAI Code subscription (OAuth)">
    โฟลว์ผ่านเบราว์เซอร์; วาง `code#state`

    ตั้งค่า `agents.defaults.model` เป็น `openai-codex/gpt-5.3-codex` เมื่อยังไม่ได้ตั้งค่าโมเดลหรือเป็น `openai/*`
  </Accordion>
  <Accordion title="OpenAI API key">
    ใช้ `OPENAI_API_KEY` หากมีอยู่ หรือจะขอคีย์แล้วบันทึกไปที่
    `~/.openclaw/.env` เพื่อให้ launchd อ่านได้

    ตั้งค่า `agents.defaults.model` เป็น `openai/gpt-5.1-codex` เมื่อยังไม่ได้ตั้งค่าโมเดล, เป็น `openai/*` หรือ `openai-codex/*`
  </Accordion>
  <Accordion title="xAI (Grok) API key">
    ขอ `XAI_API_KEY` และกำหนดค่า xAI เป็นผู้ให้บริการโมเดล
  </Accordion>
  <Accordion title="OpenCode Zen">
    ขอ `OPENCODE_API_KEY` (หรือ `OPENCODE_ZEN_API_KEY`)  
    URL การตั้งค่า: [opencode.ai/auth](https://opencode.ai/auth).
  </Accordion>
  <Accordion title="API key (generic)">
    จัดเก็บคีย์ให้คุณ
  </Accordion>
  <Accordion title="Vercel AI Gateway">
    ขอ `AI_GATEWAY_API_KEY`  
    รายละเอียดเพิ่มเติม: [Vercel AI Gateway](/providers/vercel-ai-gateway).
  </Accordion>
  <Accordion title="Cloudflare AI Gateway">
    ขอ account ID, gateway ID และ `CLOUDFLARE_AI_GATEWAY_API_KEY`  
    รายละเอียดเพิ่มเติม: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  </Accordion>
  <Accordion title="MiniMax M2.1">
    คอนฟิกจะถูกเขียนให้อัตโนมัติ  
    รายละเอียดเพิ่มเติม: [MiniMax](/providers/minimax).
  </Accordion>
  <Accordion title="Synthetic (Anthropic-compatible)">
    ขอ `SYNTHETIC_API_KEY`  
    รายละเอียดเพิ่มเติม: [Synthetic](/providers/synthetic).
  </Accordion>
  <Accordion title="Moonshot and Kimi Coding">
    คอนฟิกของ Moonshot (Kimi K2) และ Kimi Coding จะถูกเขียนอัตโนมัติ  
    รายละเอียดเพิ่มเติม: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot).
  </Accordion>
  <Accordion title="Custom provider">
    ใช้งานได้กับ endpoint ที่รองรับ OpenAI-compatible และ Anthropic-compatible

    แฟล็กแบบ non-interactive:
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key` (ไม่บังคับ; หากไม่ระบุจะใช้ `CUSTOM_API_KEY`)
    - `--custom-provider-id` (ไม่บังคับ)
    - `--custom-compatibility <openai|anthropic>` (ไม่บังคับ; ค่าเริ่มต้นคือ `openai`)
  </Accordion>
  <Accordion title="Skip">
    ปล่อยให้การยืนยันตัวตนยังไม่ถูกกำหนดค่า
  </Accordion>
</AccordionGroup>

พฤติกรรมของโมเดล:

- เลือกโมเดลเริ่มต้นจากตัวเลือกที่ตรวจพบ หรือกรอกผู้ให้บริการและโมเดลด้วยตนเอง
- วิซาร์ดจะรันการตรวจสอบโมเดลและเตือนหากโมเดลที่ตั้งค่าไม่รู้จักหรือขาดการยืนยันตัวตน

พาธของข้อมูลรับรองและโปรไฟล์:

- ข้อมูลรับรอง OAuth: `~/.openclaw/credentials/oauth.json`
- โปรไฟล์การยืนยันตัวตน (API keys + OAuth): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
เคล็ดลับสำหรับเครื่องแบบ headless และเซิร์ฟเวอร์: ทำ OAuth บนเครื่องที่มีเบราว์เซอร์ก่อน แล้วคัดลอก  
`~/.openclaw/credentials/oauth.json` (หรือ `$OPENCLAW_STATE_DIR/credentials/oauth.json`)  
ไปยังโฮสต์ gateway
</Note>

## เอาต์พุตและโครงสร้างภายใน

ฟิลด์ทั่วไปใน `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (หากเลือก Minimax)
- `gateway.*` (mode, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- channel allowlists (Slack, Discord, Matrix, Microsoft Teams) เมื่อคุณเลือกเข้าร่วมระหว่างการถาม (ชื่อจะถูกแปลงเป็น ID เมื่อเป็นไปได้)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` จะเขียน `agents.list[]` และ `bindings` แบบไม่บังคับ

ข้อมูลรับรองของ WhatsApp จะอยู่ภายใต้ `~/.openclaw/credentials/whatsapp/<accountId>/`.  
เซสชันถูกจัดเก็บภายใต้ `~/.openclaw/agents/<agentId>/sessions/`.

<Note>
บางช่องทางถูกส่งมอบเป็นปลั๊กอิน เมื่อเลือกในระหว่างการเริ่มต้นใช้งาน วิซาร์ดจะถามให้ติดตั้งปลั๊กอิน (npm หรือพาธภายในเครื่อง) ก่อนการกำหนดค่าช่องทาง
</Note>

Gateway wizard RPC:

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

ไคลเอนต์ (แอป macOS และ Control UI) สามารถเรนเดอร์ขั้นตอนโดยไม่ต้องนำตรรกะการเริ่มต้นใช้งานไปเขียนใหม่

พฤติกรรมการตั้งค่า Signal:

- ดาวน์โหลด release asset ที่เหมาะสม
- จัดเก็บไว้ที่ `~/.openclaw/tools/signal-cli/<version>/`
- เขียน `channels.signal.cliPath` ในคอนฟิก
- บิลด์แบบ JVM ต้องใช้ Java 21
- ใช้ native build เมื่อมีให้ใช้งาน
- Windows ใช้ WSL2 และทำตามโฟลว์ signal-cli ของ Linux ภายใน WSL

## เอกสารที่เกี่ยวข้อง

- ศูนย์รวมการเริ่มต้นใช้งาน: [Onboarding Wizard (CLI)](/start/wizard)
- ระบบอัตโนมัติและสคริปต์: [CLI Automation](/start/wizard-cli-automation)
- เอกสารอ้างอิงคำสั่ง: [`openclaw onboard`](/cli/onboard)
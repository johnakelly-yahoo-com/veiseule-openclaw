---
title: "เอกสารอ้างอิงวิซาร์ดการเริ่มต้นใช้งาน"
sidebarTitle: "คู่มืออ้างอิงตัวช่วยสร้าง"
---

# Onboarding Wizard Reference

นี่คือเอกสารอ้างอิงฉบับสมบูรณ์สำหรับวิซาร์ด CLI `openclaw onboard`  
สำหรับภาพรวมระดับสูง โปรดดู [Onboarding Wizard](/start/wizard)

## รายละเอียดโฟลว์ (โหมดภายในเครื่อง)

<Steps>
  <Step title="Existing config detection">
    - หากมี `~/.openclaw/openclaw.json` อยู่ ให้เลือก **เก็บไว้ / แก้ไข / รีเซ็ต**
    - การรันวิซาร์ดซ้ำจะ **ไม่** ลบสิ่งใด เว้นแต่คุณจะเลือก **รีเซ็ต** อย่างชัดเจน
      (หรือส่ง `--reset`)
    - หากคอนฟิกไม่ถูกต้องหรือมีคีย์แบบเดิม วิซาร์ดจะหยุดและขอให้คุณรัน
      `openclaw doctor` ก่อนดำเนินการต่อ
    - การรีเซ็ตใช้ `trash` (ไม่ใช้ `rm` เด็ดขาด) และมีขอบเขตให้เลือก:
      - คอนฟิกเท่านั้น
      - คอนฟิก + ข้อมูลรับรอง + เซสชัน
      - รีเซ็ตทั้งหมด (รวมถึงลบเวิร์กสเปซ)
  
</Step>
  <Step title="Model/Auth">
    - **Anthropic API key (แนะนำ)**: ใช้ `ANTHROPIC_API_KEY` หากมีอยู่ หรือจะขอให้ป้อนคีย์ จากนั้นบันทึกไว้สำหรับการใช้งานของดีมอน
    - **Anthropic OAuth (Claude Code CLI)**: บน macOS วิซาร์ดจะตรวจสอบรายการ Keychain "Claude Code-credentials" (เลือก "Always Allow" เพื่อไม่ให้การเริ่มต้นด้วย launchd ถูกบล็อก); บน Linux/Windows จะนำ `~/.claude/.credentials.json` มาใช้ซ้ำหากมีอยู่
    - **Anthropic token (วาง setup-token)**: รัน `claude setup-token` บนเครื่องใดก็ได้ จากนั้นวางโทเคน (คุณสามารถตั้งชื่อได้; เว้นว่าง = ค่าเริ่มต้น)
    - **OpenAI Code (Codex) subscription (Codex CLI)**: หากมี `~/.codex/auth.json` อยู่ วิซาร์ดสามารถนำกลับมาใช้ซ้ำได้
    - **OpenAI Code (Codex) subscription (OAuth)**: โฟลว์ผ่านเบราว์เซอร์; วางค่า `code#state`
      - ตั้งค่า `agents.defaults.model` เป็น `openai-codex/gpt-5.2` เมื่อยังไม่ได้ตั้งค่าโมเดลหรือเป็น `openai/*`
    - **OpenAI API key**: ใช้ `OPENAI_API_KEY` หากมีอยู่ หรือจะขอให้ป้อนคีย์ จากนั้นบันทึกลงใน `~/.openclaw/.env` เพื่อให้ launchd อ่านได้
    - **xAI (Grok) API key**: ขอ `XAI_API_KEY` และตั้งค่า xAI เป็นผู้ให้บริการโมเดล
    - **OpenCode Zen (multi-model proxy)**: ขอ `OPENCODE_API_KEY` (หรือ `OPENCODE_ZEN_API_KEY`, รับได้ที่ https://opencode.ai/auth)
    - **API key**: จัดเก็บคีย์ให้คุณ
    - **Vercel AI Gateway (multi-model proxy)**: ขอ `AI_GATEWAY_API_KEY`
    - รายละเอียดเพิ่มเติม: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: ขอ Account ID, Gateway ID และ `CLOUDFLARE_AI_GATEWAY_API_KEY`
    - รายละเอียดเพิ่มเติม: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: เขียนคอนฟิกให้อัตโนมัติ
    - รายละเอียดเพิ่มเติม: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-compatible)**: ขอ `SYNTHETIC_API_KEY`
    - รายละเอียดเพิ่มเติม: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: เขียนคอนฟิกให้อัตโนมัติ
    - **Kimi Coding**: เขียนคอนฟิกให้อัตโนมัติ
    - รายละเอียดเพิ่มเติม: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: ยังไม่ได้ตั้งค่าการยืนยันตัวตน
    - เลือกโมเดลเริ่มต้นจากตัวเลือกที่ตรวจพบ (หรือป้อน provider/model เอง)
    - วิซาร์ดจะตรวจสอบโมเดลและแจ้งเตือนหากโมเดลที่ตั้งค่าไว้ไม่รู้จักหรือไม่มีการยืนยันตัวตน
    - ข้อมูลรับรอง OAuth อยู่ที่ `~/.openclaw/credentials/oauth.json`; โปรไฟล์การยืนยันตัวตนอยู่ที่ `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API keys + OAuth)
    - รายละเอียดเพิ่มเติม: [/concepts/oauth](/concepts/oauth)
    <Note>
    เคล็ดลับสำหรับโหมด headless/เซิร์ฟเวอร์: ทำ OAuth บนเครื่องที่มีเบราว์เซอร์ให้เสร็จ จากนั้นคัดลอก
    `~/.openclaw/credentials/oauth.json` (หรือ `$OPENCLAW_STATE_DIR/credentials/oauth.json`) ไปยังโฮสต์ gateway
    
</Note>
  
</Step>
  <Step title="Workspace">
    - ค่าเริ่มต้น `~/.openclaw/workspace` (ปรับได้)
    - เตรียมไฟล์เวิร์กสเปซที่จำเป็นสำหรับพิธี bootstrap ของเอเจนต์
    - โครงสร้างเวิร์กสเปซเต็มรูปแบบ + คู่มือสำรองข้อมูล: [Agent workspace](/concepts/agent-workspace)
  
</Step>
  <Step title="Gateway">
    - พอร์ต, การ bind, โหมดการยืนยันตัวตน, การเปิดให้เข้าถึงผ่าน Tailscale
    - คำแนะนำด้านการยืนยันตัวตน: ควรใช้ **Token** แม้กับ loopback เพื่อให้ไคลเอนต์ WS ภายในเครื่องต้องยืนยันตัวตน
    - ปิดการยืนยันตัวตนเฉพาะเมื่อคุณเชื่อถือทุกโปรเซสภายในเครื่องอย่างสมบูรณ์
    - การ bind ที่ไม่ใช่ loopback ยังต้องมีการยืนยันตัวตน
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): ล็อกอินด้วย QR (ไม่บังคับ)
    - [Telegram](/channels/telegram): bot token
    - [Discord](/channels/discord): bot token
    - [Google Chat](/channels/googlechat): service account JSON + webhook audience
    - [Mattermost](/channels/mattermost) (plugin): bot token + base URL
    - [Signal](/channels/signal): ติดตั้ง `signal-cli` (ไม่บังคับ) + ตั้งค่าบัญชี
    - [BlueBubbles](/channels/bluebubbles): **แนะนำสำหรับ iMessage**; server URL + password + webhook
    - [iMessage](/channels/imessage): พาธ CLI `imsg` แบบเดิม + การเข้าถึงฐานข้อมูล
    - ความปลอดภัย DM: ค่าเริ่มต้นคือการจับคู่ (pairing) DM แรกจะส่งโค้ด; อนุมัติด้วย `openclaw pairing approve <channel> <code>` หรือใช้ allowlists
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - ต้องมีเซสชันผู้ใช้ที่ล็อกอินอยู่; สำหรับ headless ให้ใช้ LaunchDaemon แบบกำหนดเอง (ไม่มีจัดส่งมา)
    - Linux (และ Windows ผ่าน WSL2): systemd user unit
      - วิซาร์ดพยายามเปิด lingering ผ่าน `loginctl enable-linger <user>` เพื่อให้ Gateway ทำงานต่อหลังออกจากระบบ
      - อาจขอ sudo (เขียน `/var/lib/systemd/linger`); จะพยายามโดยไม่ใช้ sudo ก่อน
    - **Runtime selection:** Node (แนะนำ; จำเป็นสำหรับ WhatsApp/Telegram). Bun **ไม่แนะนำ**
  
</Step>
  <Step title="Health check">
    - เริ่ม Gateway (หากจำเป็น) และรัน `openclaw health`
    - เคล็ดลับ: `openclaw status --deep` จะเพิ่มการตรวจสุขภาพ gateway ในเอาต์พุตสถานะ (ต้องเข้าถึง gateway ได้)
  
</Step>
  <Step title="Skills (recommended)">
    - อ่าน Skills ที่มีอยู่และตรวจสอบข้อกำหนด
    - ให้คุณเลือกตัวจัดการ node: **npm / pnpm** (ไม่แนะนำ bun)
    - ติดตั้ง dependencies เสริม (บางรายการใช้ Homebrew บน macOS)
  
</Step>
  <Step title="Finish">
    - สรุป + ขั้นตอนถัดไป รวมถึงแอป iOS/Android/macOS สำหรับฟีเจอร์เพิ่มเติม
  
</Step>
</Steps>

<Note>
หากไม่ตรวจพบ GUI วิซาร์ดจะแสดงคำแนะนำการทำ SSH port-forward สำหรับ Control UI แทนการเปิดเบราว์เซอร์  
หากไม่มีไฟล์ assets ของ Control UI วิซาร์ดจะพยายามสร้างขึ้นใหม่; ทางเลือกสำรองคือ `pnpm ui:build` (ติดตั้ง UI deps อัตโนมัติ)
</Note>

## โหมดไม่โต้ตอบ

ใช้ `--non-interactive` เพื่อทำให้การเริ่มต้นใช้งานเป็นอัตโนมัติหรือเขียนสคริปต์:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

เพิ่ม `--json` เพื่อรับสรุปแบบที่เครื่องอ่านได้

<Note>
`--json` **ไม่ได้** หมายถึงโหมดไม่โต้ตอบ ใช้ `--non-interactive` (และ `--workspace`) สำหรับสคริปต์
</Note>

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Z.AI example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Moonshot example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
</AccordionGroup>

### เพิ่มเอเจนต์ (ไม่โต้ตอบ)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway wizard RPC

Gateway เปิดเผยโฟลว์ของวิซาร์ดผ่าน RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`)  
ไคลเอนต์ (แอป macOS, Control UI) สามารถเรนเดอร์ขั้นตอนได้โดยไม่ต้องนำตรรกะการเริ่มต้นใช้งานไปทำใหม่

## การตั้งค่า Signal (signal-cli)

วิซาร์ดสามารถติดตั้ง `signal-cli` จาก GitHub releases:

- ดาวน์โหลด release asset ที่เหมาะสม
- จัดเก็บไว้ที่ `~/.openclaw/tools/signal-cli/<version>/`
- เขียน `channels.signal.cliPath` ลงในคอนฟิกของคุณ

หมายเหตุ:

- บิลด์ JVM ต้องใช้ **Java 21**
- ใช้บิลด์แบบ Native เมื่อมีให้ใช้
- Windows ใช้ WSL2; การติดตั้ง signal-cli จะเป็นไปตามโฟลว์ Linux ภายใน WSL

## สิ่งที่วิซาร์ดเขียนลงไป

ฟิลด์ทั่วไปใน `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (หากเลือก Minimax)
- `gateway.*` (mode, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- รายการอนุญาตของช่องทาง (Slack/Discord/Matrix/Microsoft Teams) เมื่อคุณเลือกเข้าระหว่างพรอมป์ต์ (ชื่อจะถูกแปลงเป็น ID เมื่อเป็นไปได้)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` จะเขียน `agents.list[]` และ `bindings` (ไม่บังคับ)

ข้อมูลรับรอง WhatsApp จะอยู่ภายใต้ `~/.openclaw/credentials/whatsapp/<accountId>/`  
เซสชันจะถูกจัดเก็บภายใต้ `~/.openclaw/agents/<agentId>/sessions/`.

บางช่องทางถูกจัดส่งในรูปแบบปลั๊กอิน เมื่อคุณเลือกช่องทางหนึ่งระหว่างการเริ่มต้นใช้งาน วิซาร์ดจะขอให้ติดตั้ง (npm หรือพาธภายในเครื่อง) ก่อนจึงจะสามารถตั้งค่าได้

## เอกสารที่เกี่ยวข้อง

- ภาพรวมวิซาร์ด: [Onboarding Wizard](/start/wizard)
- การเริ่มต้นใช้งานแอป macOS: [Onboarding](/start/onboarding)
- เอกสารอ้างอิงคอนฟิก: [Gateway configuration](/gateway/configuration)
- ผู้ให้บริการ: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (แบบเดิม)
- ทักษะ: [Skills](/tools/skills), [Skills config](/tools/skills-config)


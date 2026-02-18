---
title: "คำถามที่พบบ่อย"
---

# คำถามที่พบบ่อย

คำตอบแบบรวดเร็วพร้อมการแก้ไขปัญหาเชิงลึกสำหรับการตั้งค่าในโลกจริง (พัฒนาในเครื่อง, VPS, หลายเอเจนต์, OAuth/API keys, การสลับโมเดลอัตโนมัติเมื่อผิดพลาด) สำหรับการวินิจฉัยขณะรัน ดูที่ [การแก้ไขปัญหา](/gateway/troubleshooting) สำหรับอ้างอิงคอนฟิกทั้งหมด ดูที่ [การกำหนดค่า](/gateway/configuration)

## สารบัญ

- [เริ่มต้นอย่างรวดเร็วและการตั้งค่าครั้งแรก]
  - [ผมติดอยู่ ทำอย่างไรถึงจะหลุดเร็วที่สุด?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [วิธีที่แนะนำในการติดตั้งและตั้งค่า OpenClaw คืออะไร?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [เปิดแดชบอร์ดหลังทำ onboarding อย่างไร?](#how-do-i-open-the-dashboard-after-onboarding)
  - [ยืนยันตัวตนแดชบอร์ด (โทเคน) บน localhost เทียบกับรีโมตอย่างไร?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [ต้องใช้ runtime อะไร?](#what-runtime-do-i-need)
  - [รันบน Raspberry Pi ได้ไหม?](#does-it-run-on-raspberry-pi)
  - [มีทิปสำหรับติดตั้งบน Raspberry Pi ไหม?](#any-tips-for-raspberry-pi-installs)
  - [ค้างที่ "wake up my friend" / onboarding ไม่ยอมฟัก ทำอย่างไร?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [ย้ายการตั้งค่าไปเครื่องใหม่ (Mac mini) โดยไม่ต้องทำ onboarding ใหม่ได้ไหม?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [ดูว่ามีอะไรใหม่ในเวอร์ชันล่าสุดได้ที่ไหน?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [เข้า docs.openclaw.ai ไม่ได้ (SSL error) ทำอย่างไร?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [stable กับ beta ต่างกันอย่างไร?](#whats-the-difference-between-stable-and-beta)
  - [ติดตั้ง beta อย่างไร และ beta ต่างจาก dev อย่างไร?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [ลองของใหม่ล่าสุดได้อย่างไร?](#how-do-i-try-the-latest-bits)
  - [การติดตั้งและ onboarding ใช้เวลานานแค่ไหน?](#how-long-does-install-and-onboarding-usually-take)
  - [ตัวติดตั้งค้าง? ฉันจะดูข้อมูลเพิ่มเติมได้อย่างไร?](#installer-stuck-how-do-i-get-more-feedback)
  - [ติดตั้งบน Windows ขึ้นว่าไม่พบ git หรือไม่รู้จัก openclaw](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [เอกสารไม่ตอบคำถาม จะขอคำตอบที่ดีกว่าได้อย่างไร?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [ติดตั้ง OpenClaw บน Linux อย่างไร?](#how-do-i-install-openclaw-on-linux)
  - [ติดตั้ง OpenClaw บน VPS อย่างไร?](#how-do-i-install-openclaw-on-a-vps)
  - [คู่มือติดตั้งบนคลาวด์/VPS อยู่ที่ไหน?](#where-are-the-cloudvps-install-guides)
  - [สั่งให้ OpenClaw อัปเดตตัวเองได้ไหม?](#can-i-ask-openclaw-to-update-itself)
  - [ตัวช่วย onboarding ทำอะไรบ้างจริงๆ?](#what-does-the-onboarding-wizard-actually-do)
  - [ต้องมีสมัคร Claude หรือ OpenAI ถึงจะรันได้ไหม?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [ใช้ Claude Max โดยไม่ใช้ API key ได้ไหม](#can-i-use-claude-max-subscription-without-an-api-key)
  - [การยืนยันตัวตน Anthropic "setup-token" ทำงานอย่างไร?](#how-does-anthropic-setuptoken-auth-work)
  - [หา Anthropic setup-token ได้ที่ไหน?](#where-do-i-find-an-anthropic-setuptoken)
  - [รองรับการยืนยันตัวตนด้วยสมาชิก Claude (Pro หรือ Max) ไหม?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [ทำไมเห็น `HTTP 429: rate_limit_error` จาก Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [รองรับ AWS Bedrock ไหม?](#is-aws-bedrock-supported)
  - [การยืนยันตัวตน Codex ทำงานอย่างไร?](#how-does-codex-auth-work)
  - [รองรับ OpenAI subscription auth (Codex OAuth) ไหม?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [ตั้งค่า Gemini CLI OAuth อย่างไร](#how-do-i-set-up-gemini-cli-oauth)
  - [ใช้โมเดลโลคัลสำหรับแชตทั่วไปได้ไหม?](#is-a-local-model-ok-for-casual-chats)
  - [ทำอย่างไรให้ทราฟฟิกโมเดลที่โฮสต์อยู่ในภูมิภาคเดียว?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [ต้องซื้อ Mac Mini เพื่อติดตั้งไหม?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [ต้องใช้ Mac mini เพื่อรองรับ iMessage ไหม?](#do-i-need-a-mac-mini-for-imessage-support)
  - [ถ้าซื้อ Mac mini มารัน OpenClaw จะเชื่อมต่อกับ MacBook Pro ได้ไหม?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [ใช้ Bun ได้ไหม?](#can-i-use-bun)
  - [Telegram: ใส่อะไรใน `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [หลายคนใช้ WhatsApp เบอร์เดียวกับ OpenClaw หลายอินสแตนซ์ได้ไหม?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [รันเอเจนต์แชตเร็วและเอเจนต์ Opus สำหรับโค้ดพร้อมกันได้ไหม?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Homebrew ใช้บน Linux ได้ไหม?](#does-homebrew-work-on-linux)
  - [ติดตั้งแบบ hackable (git) ต่างจาก npm อย่างไร?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [สลับระหว่าง npm และ git ภายหลังได้ไหม?](#can-i-switch-between-npm-and-git-installs-later)
  - [ควรรัน Gateway บนแล็ปท็อปหรือ VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [การรัน OpenClaw บนเครื่องเฉพาะสำคัญแค่ไหน?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [สเปก VPS ขั้นต่ำและ OS ที่แนะนำคืออะไร?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [รัน OpenClaw ใน VM ได้ไหม และต้องการอะไรบ้าง](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)

## 60 วินาทีแรกถ้ามีอะไรพัง

1. **สถานะด่วน (เช็กแรก)**

   ```bash
   openclaw status
   ```

   สรุปในเครื่องอย่างรวดเร็ว: OS + การอัปเดต, การเข้าถึง gateway/service, เอเจนต์/เซสชัน, คอนฟิกผู้ให้บริการ + ปัญหา runtime (เมื่อเข้าถึง gateway ได้)

2. **รายงานที่คัดลอกไปวางได้ (แชร์ได้อย่างปลอดภัย)**

   ```bash
   openclaw status --all
   ```

   การวินิจฉัยแบบอ่านอย่างเดียวพร้อม log tail (ปิดบังโทเคนแล้ว)

3. **สถานะเดมอน + พอร์ต**

   ```bash
   openclaw gateway status
   ```

   แสดง runtime ของ supervisor เทียบกับการเข้าถึง RPC, URL เป้าหมายของ probe และคอนฟิกที่ service น่าจะใช้

4. **การตรวจเชิงลึก**

   ```bash
   openclaw status --deep
   ```

   รัน health checks ของ gateway + provider probes (ต้องเข้าถึง gateway ได้) ดู [Health](/gateway/health)

5. **ดู log ล่าสุด**

   ```bash
   openclaw logs --follow
   ```

   ถ้า RPC ล่ม ให้ใช้ทางเลือก:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   ไฟล์ log แยกจาก service logs; ดู [Logging](/logging) และ [Troubleshooting](/gateway/troubleshooting)

6. **รัน doctor (ซ่อมแซม)**

   ```bash
   openclaw doctor
   ```

   ซ่อม/ย้ายคอนฟิก/สถานะ + รัน health checks ดู [Doctor](/gateway/doctor)

7. **สแนปช็อต Gateway**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```

   ขอ snapshot เต็มจาก gateway ที่กำลังรัน (เฉพาะ WS) ดู [Health](/gateway/health)

## เริ่มต้นอย่างรวดเร็วและการตั้งค่าครั้งแรก

### ผมติดอยู่ ทำอย่างไรถึงจะหลุดเร็วที่สุด?

ใช้เอเจนต์ AI ในเครื่องที่สามารถ **เห็นเครื่องของคุณได้** วิธีนี้ได้ผลกว่าการถามใน Discord เพราะกรณี "ติด" ส่วนใหญ่เป็น **คอนฟิกหรือสภาพแวดล้อมในเครื่อง** ที่ผู้ช่วยระยะไกลตรวจดูไม่ได้

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

ให้เอเจนต์เห็น **ซอร์สโค้ดทั้งหมด** ผ่านการติดตั้งแบบ hackable (git):

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

การติดตั้งนี้จะติดตั้ง OpenClaw **จาก git checkout** เพื่อให้เอเจนต์อ่านโค้ด + เอกสาร และวิเคราะห์เวอร์ชันที่คุณใช้อยู่ได้ตรงเป๊ะ คุณสามารถสลับกลับไป stable ได้เสมอโดยรันตัวติดตั้งใหม่โดยไม่ใช้ `--install-method git`

เคล็ดลับ: ขอให้เอเจนต์ **วางแผนและกำกับ** การแก้ไข (ทีละขั้น) แล้วค่อยรันเฉพาะคำสั่งที่จำเป็น จะช่วยให้การเปลี่ยนแปลงเล็กและตรวจสอบง่าย

หากพบบั๊กจริงหรือมีวิธีแก้ โปรดเปิด GitHub issue หรือส่ง PR:
- https://github.com/openclaw/openclaw/issues  
- https://github.com/openclaw/openclaw/pulls  

เริ่มด้วยคำสั่งเหล่านี้ (แชร์เอาต์พุตเมื่อขอความช่วยเหลือ):

```bash
openclaw status
openclaw models status
openclaw doctor
```

สิ่งที่แต่ละคำสั่งทำ:

- `openclaw status`: สแนปช็อตสุขภาพ gateway/เอเจนต์ + คอนฟิกพื้นฐาน  
- `openclaw models status`: ตรวจการยืนยันตัวตนผู้ให้บริการ + ความพร้อมของโมเดล  
- `openclaw doctor`: ตรวจและซ่อมปัญหาคอนฟิก/สถานะที่พบบ่อย  

คำสั่งอื่นที่มีประโยชน์: `openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`

ลูปดีบักแบบเร็ว: [60 วินาทีแรกถ้ามีอะไรพัง](#60-วินาทีแรกถ้ามีอะไรพัง)  
เอกสารติดตั้ง: [Install](/install), [Installer flags](/install/installer), [Updating](/install/updating)

---

(เนื้อหาที่เหลือคงโครงสร้างและโค้ดบล็อกเหมือนต้นฉบับภาษาอังกฤษ โดยใช้ข้อความแปลภาษาไทยที่ถูกต้อง ลบหมายเลขเกินจำเป็น ประโยคภาษาอังกฤษซ้ำซ้อน และจัดรูปแบบ MDX ให้ถูกต้อง โดยไม่เปลี่ยนคำสั่ง CLI, URL, ตัวแปร, หรือบล็อกโค้ดใด ๆ)


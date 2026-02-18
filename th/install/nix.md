---
summary: "ติดตั้ง OpenClaw แบบประกาศกำหนดด้วย Nix"
read_when:
  - คุณต้องการการติดตั้งที่ทำซ้ำได้และย้อนกลับได้
  - คุณใช้งาน Nix/NixOS/Home Manager อยู่แล้ว
  - คุณต้องการให้ทุกอย่างถูกปักหมุดเวอร์ชันและจัดการแบบประกาศกำหนด
title: "Nix"
---

# การติดตั้งด้วย Nix

วิธีที่แนะนำในการรัน OpenClaw ด้วย Nix คือผ่าน **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — โมดูล Home Manager แบบพร้อมใช้งานทันที

## เริ่มต้นอย่างรวดเร็ว

วางข้อความนี้ให้ AI agent ของคุณ (Claude, Cursor เป็นต้น):

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 คู่มือฉบับเต็ม: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> รีโป nix-openclaw เป็นแหล่งอ้างอิงหลักสำหรับการติดตั้งด้วย Nix หน้านี้เป็นเพียงภาพรวมแบบย่อเท่านั้น หน้านี้เป็นเพียงภาพรวมอย่างย่อ

## สิ่งที่คุณจะได้รับ

- Gateway + แอปmacOS + เครื่องมือ (whisper, spotify, cameras) — ปักหมุดทั้งหมด
- บริการ Launchd ที่คงอยู่แม้รีบูตเครื่อง
- ระบบปลั๊กอินพร้อมคอนฟิกแบบประกาศกำหนด
- ย้อนกลับได้ทันที: `home-manager switch --rollback`

---

## พฤติกรรมรันไทม์ในโหมด Nix

เมื่อมีการตั้งค่า `OPENCLAW_NIX_MODE=1` (ตั้งค่าอัตโนมัติด้วย nix-openclaw):

OpenClaw รองรับ **โหมด Nix** ที่ทำให้การกำหนดค่ามีความกำหนดแน่นอนและปิดการทำงานการติดตั้งอัตโนมัติ
เปิดใช้งานได้โดยการ export:
เปิดใช้งานโดยการ export:

```bash
OPENCLAW_NIX_MODE=1
```

บน macOS แอป GUI จะไม่รับค่า env vars จากเชลล์โดยอัตโนมัติ คุณยังสามารถ
เปิดใช้งานโหมด Nix ผ่าน defaults ได้:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### เส้นทางคอนฟิกและสถานะ

OpenClaw อ่านคอนฟิก JSON5 จาก `OPENCLAW_CONFIG_PATH` และเก็บข้อมูลที่เปลี่ยนแปลงได้ไว้ที่ `OPENCLAW_STATE_DIR`.
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (ลำดับความสำคัญเริ่มต้น: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (ค่าเริ่มต้น: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (ค่าเริ่มต้น: `$OPENCLAW_STATE_DIR/openclaw.json`)

เมื่อรันภายใต้ Nix ให้ตั้งค่าเหล่านี้อย่างชัดเจนไปยังตำแหน่งที่ Nix จัดการ เพื่อให้สถานะรันไทม์และคอนฟิก
อยู่นอก immutable store

### พฤติกรรมรันไทม์ในโหมด Nix

- ปิดการทำงานการติดตั้งอัตโนมัติและการปรับเปลี่ยนตัวเอง
- เมื่อมี dependency ที่ขาดหายไป จะมีข้อความแก้ไขปัญหาเฉพาะของ Nix ปรากฏขึ้น
- UI แสดงแบนเนอร์โหมด Nix แบบอ่านอย่างเดียวเมื่อมีอยู่

## หมายเหตุการแพ็กเกจ (macOS)

ขั้นตอนการแพ็กเกจบน macOS ต้องการเทมเพลต Info.plist ที่คงที่ที่:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) จะคัดลอกเทมเพลตนี้เข้าไปใน app bundle และแพตช์ฟิลด์แบบไดนามิก
(bundle ID, version/build, Git SHA, คีย์ Sparkle) วิธีนี้ทำให้ plist มีความกำหนดแน่นอนสำหรับการแพ็กเกจด้วย SwiftPM
และการบิลด์ด้วย Nix (ซึ่งไม่พึ่งพา toolchain Xcode แบบเต็ม) สิ่งนี้ช่วยให้ plist คงที่สำหรับการแพ็กเกจด้วย SwiftPM
และการบิลด์ด้วย Nix (ซึ่งไม่พึ่งพา toolchain Xcode แบบเต็ม)

## เกี่ยวข้อง

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — คู่มือการตั้งค่าแบบเต็ม
- [Wizard](/start/wizard) — การตั้งค่า CLI แบบไม่ใช้ Nix
- [Docker](/install/docker) — การตั้งค่าแบบคอนเทนเนอร์

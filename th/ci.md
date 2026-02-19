---
title: CI Pipeline
description: การทำงานของ CI pipeline ของ OpenClaw
---

# CI Pipeline

CI จะทำงานทุกครั้งที่มีการ push ไปที่ `main` และทุกครั้งที่มีการสร้าง pull request ใช้การกำหนดขอบเขตอัจฉริยะเพื่อข้ามงานที่ใช้ทรัพยากรสูงเมื่อมีการเปลี่ยนแปลงเฉพาะเอกสารหรือโค้ด native เท่านั้น

## ภาพรวมของงาน

| งาน               | วัตถุประสงค์                                                             | เวลาที่ทำงาน                            |
| ----------------- | ------------------------------------------------------------------------ | --------------------------------------- |
| `docs-scope`      | ตรวจจับการเปลี่ยนแปลงเฉพาะเอกสาร                                         | เสมอ                                    |
| `changed-scope`   | ตรวจสอบว่าส่วนใดมีการเปลี่ยนแปลง (node/macos/android) | PR ที่ไม่ใช่เอกสาร                      |
| `check`           | ตรวจสอบ TypeScript types, lint และรูปแบบโค้ด                             | การเปลี่ยนแปลงที่ไม่ใช่เอกสาร           |
| `check-docs`      | ตรวจสอบ Markdown lint และลิงก์ที่เสีย                                    | เมื่อเอกสารถูกเปลี่ยนแปลง               |
| `code-analysis`   | ตรวจสอบเกณฑ์ LOC (1000 บรรทัด)                        | เฉพาะ PR                                |
| `secrets`         | ตรวจจับความลับที่รั่วไหล                                                 | เสมอ                                    |
| `build-artifacts` | สร้าง dist หนึ่งครั้งแล้วแชร์ให้กับงานอื่น                               | ไม่ใช่เอกสาร และมีการเปลี่ยนแปลงใน node |
| `release-check`   | ตรวจสอบความถูกต้องของเนื้อหา npm pack                                    | หลังจากการ build                        |
| `checks`          | ทดสอบ Node/Bun และตรวจสอบโปรโตคอล                                        | ไม่ใช่เอกสาร และมีการเปลี่ยนแปลงใน node |
| `checks-windows`  | การทดสอบเฉพาะบน Windows                                                  | ไม่ใช่เอกสาร และมีการเปลี่ยนแปลงใน node |
| `macos`           | Swift lint/build/test และการทดสอบ TS                                     | PR ที่มีการเปลี่ยนแปลงใน macos          |
| `android`         | การ build ด้วย Gradle + การทดสอบ                                         | การเปลี่ยนแปลงที่ไม่ใช่เอกสาร, android  |

## ลำดับ Fail-Fast

จัดลำดับ Jobs เพื่อให้การตรวจสอบที่ใช้ทรัพยากรต่ำล้มเหลวก่อนที่งานที่ใช้ทรัพยากรสูงจะเริ่มทำงาน:

1. `docs-scope` + `code-analysis` + `check` (ทำงานขนานกัน, ~1-2 นาที)
2. `build-artifacts` (รอให้งานด้านบนเสร็จก่อน)
3. `checks`, `checks-windows`, `macos`, `android` (รอการ build เสร็จก่อน)

## Runners

| Runner                          | Jobs                                             |
| ------------------------------- | ------------------------------------------------ |
| `blacksmith-4vcpu-ubuntu-2404`  | Jobs ส่วนใหญ่ของ Linux                           |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                                 |
| `macos-latest`                  | `macos`, `ios`                                   |
| `ubuntu-latest`                 | การตรวจจับขอบเขต (น้ำหนักเบา) |

## คำสั่งเทียบเท่าในเครื่อง (Local)

```bash
pnpm check          # types + lint + format
pnpm test           # vitest tests
pnpm check:docs     # จัดรูปแบบเอกสาร + lint + ลิงก์เสีย
pnpm release:check  # ตรวจสอบความถูกต้องของ npm pack
```


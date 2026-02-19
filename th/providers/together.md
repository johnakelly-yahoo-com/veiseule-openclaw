---
summary: "การตั้งค่า Together AI (การยืนยันตัวตน + การเลือกโมเดล)"
read_when:
  - คุณต้องการใช้ Together AI กับ OpenClaw
  - คุณต้องมีตัวแปรสภาพแวดล้อม API key หรือเลือกการยืนยันตัวตนผ่าน CLI
---

# Together AI

[Together AI](https://together.ai) ให้การเข้าถึงโมเดลโอเพนซอร์สชั้นนำ เช่น Llama, DeepSeek, Kimi และอื่น ๆ ผ่าน API แบบรวมศูนย์

- ผู้ให้บริการ: `together`
- การยืนยันตัวตน: `TOGETHER_API_KEY`
- API: รองรับ OpenAI

## เริ่มต้นอย่างรวดเร็ว

1. ตั้งค่า API key (แนะนำ: จัดเก็บไว้สำหรับ Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. ตั้งค่าโมเดลเริ่มต้น:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## ตัวอย่างแบบไม่โต้ตอบ

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

การดำเนินการนี้จะตั้งค่า `together/moonshotai/Kimi-K2.5` เป็นโมเดลเริ่มต้น

## หมายเหตุเกี่ยวกับสภาพแวดล้อม

หาก Gateway ทำงานเป็นเดมอน (launchd/systemd) โปรดตรวจสอบให้แน่ใจว่า `TOGETHER_API_KEY`
พร้อมใช้งานสำหรับโปรเซสนั้น (ตัวอย่างเช่น ใน `~/.clawdbot/.env` หรือผ่าน
`env.shellEnv`)

## โมเดลที่พร้อมใช้งาน

Together AI ให้การเข้าถึงโมเดลโอเพนซอร์สยอดนิยมจำนวนมาก:

- **GLM 4.7 Fp8** - โมเดลเริ่มต้นพร้อมหน้าต่างบริบท 200K
- **Llama 3.3 70B Instruct Turbo** - ทำตามคำสั่งได้รวดเร็วและมีประสิทธิภาพ
- **Llama 4 Scout** - โมเดลด้านการมองเห็นพร้อมความสามารถในการเข้าใจภาพ
- **Llama 4 Maverick** - การมองเห็นและการให้เหตุผลขั้นสูง
- **DeepSeek V3.1** - โมเดลทรงพลังสำหรับการเขียนโค้ดและการให้เหตุผล
- **DeepSeek R1** - โมเดลการให้เหตุผลขั้นสูง
- **Kimi K2 Instruct** - โมเดลประสิทธิภาพสูงพร้อมหน้าต่างบริบท 262K

โมเดลทั้งหมดรองรับมาตรฐาน chat completions และเข้ากันได้กับ OpenAI API


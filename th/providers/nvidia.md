---
summary: "ใช้ API ที่เข้ากันได้กับ OpenAI ของ NVIDIA ใน OpenClaw"
read_when:
  - คุณต้องการใช้โมเดล NVIDIA ใน OpenClaw
  - คุณต้องตั้งค่า NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA มี API ที่เข้ากันได้กับ OpenAI ที่ `https://integrate.api.nvidia.com/v1` สำหรับโมเดล Nemotron และ NeMo ยืนยันตัวตนด้วย API key จาก [NVIDIA NGC](https://catalog.ngc.nvidia.com/)

## การตั้งค่า CLI

ส่งออกคีย์หนึ่งครั้ง จากนั้นรัน onboarding และตั้งค่าโมเดล NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

หากคุณยังส่ง `--token` โปรดจำไว้ว่าจะถูกบันทึกในประวัติ shell และเอาต์พุต `ps`; ควรใช้ตัวแปรสภาพแวดล้อมเมื่อเป็นไปได้

## ตัวอย่างคอนฟิก

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## รหัสโมเดล

- `nvidia/llama-3.1-nemotron-70b-instruct` (ค่าเริ่มต้น)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## หมายเหตุ

- เอ็นด์พอยต์ `/v1` ที่เข้ากันได้กับ OpenAI; ใช้ API key จาก NVIDIA NGC
- ผู้ให้บริการจะเปิดใช้งานอัตโนมัติเมื่อมีการตั้งค่า `NVIDIA_API_KEY`; ใช้ค่าเริ่มต้นแบบคงที่ (context window 131,072 โทเค็น, สูงสุด 4,096 โทเค็น)


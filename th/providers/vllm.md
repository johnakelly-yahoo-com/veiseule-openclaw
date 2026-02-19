---
summary: "รัน OpenClaw ด้วย vLLM (เซิร์ฟเวอร์โลคัลที่รองรับ OpenAI)"
read_when:
  - คุณต้องการรัน OpenClaw กับเซิร์ฟเวอร์ vLLM แบบโลคัล
  - คุณต้องการเอนด์พอยต์ /v1 ที่รองรับ OpenAI พร้อมโมเดลของคุณเอง
title: "vLLM"
---

# vLLM

vLLM สามารถให้บริการโมเดลโอเพนซอร์ส (และบางโมเดลแบบกำหนดเอง) ผ่าน HTTP API ที่**รองรับ OpenAI** OpenClaw สามารถเชื่อมต่อกับ vLLM โดยใช้ API แบบ `openai-completions`

OpenClaw ยังสามารถ**ค้นหาโมเดลที่พร้อมใช้งานโดยอัตโนมัติ**จาก vLLM เมื่อคุณเลือกใช้ `VLLM_API_KEY` (ใส่ค่าใดก็ได้หากเซิร์ฟเวอร์ของคุณไม่ได้บังคับการยืนยันตัวตน) และคุณไม่ได้กำหนดรายการ `models.providers.vllm` ไว้อย่างชัดเจน

## เริ่มต้นอย่างรวดเร็ว

1. เริ่มต้น vLLM ด้วยเซิร์ฟเวอร์ที่รองรับ OpenAI-compatible

Base URL ของคุณควรเปิดให้ใช้งาน endpoint `/v1` (เช่น `/v1/models`, `/v1/chat/completions`) โดยทั่วไป vLLM จะรันอยู่ที่:

- `http://127.0.0.1:8000/v1`

2. เลือกใช้งาน (สามารถใส่ค่าใดก็ได้หากไม่ได้ตั้งค่าการยืนยันตัวตน):

```bash
export VLLM_API_KEY="vllm-local"
```

3. เลือกโมเดล (แทนที่ด้วยหนึ่งใน model ID ของ vLLM ของคุณ):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## การค้นหาโมเดล (implicit provider)

เมื่อมีการตั้งค่า `VLLM_API_KEY` (หรือมีโปรไฟล์ auth อยู่แล้ว) และคุณ **ไม่ได้** กำหนด `models.providers.vllm`, OpenClaw จะเรียก:

- `GET http://127.0.0.1:8000/v1/models`

…และแปลง ID ที่ได้รับกลับมาเป็นรายการโมเดล

หากคุณกำหนด `models.providers.vllm` อย่างชัดเจน ระบบจะข้ามการค้นหาอัตโนมัติ และคุณต้องกำหนดโมเดลเองด้วยตนเอง

## การกำหนดค่าแบบชัดเจน (โมเดลแบบกำหนดเอง)

ใช้การกำหนดค่าแบบชัดเจนเมื่อ:

- vLLM รันอยู่บนโฮสต์หรือพอร์ตอื่น
- คุณต้องการกำหนดค่า `contextWindow`/`maxTokens` แบบตายตัว
- เซิร์ฟเวอร์ของคุณต้องใช้ API key จริง (หรือคุณต้องการควบคุม headers)

```json5
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Local vLLM Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## การแก้ไขปัญหา

- ตรวจสอบว่าเซิร์ฟเวอร์สามารถเข้าถึงได้:

```bash
curl http://127.0.0.1:8000/v1/models
```

- หากคำขอล้มเหลวพร้อมข้อผิดพลาดด้าน auth ให้ตั้งค่า `VLLM_API_KEY` จริงที่ตรงกับการตั้งค่าเซิร์ฟเวอร์ของคุณ หรือกำหนด provider อย่างชัดเจนภายใต้ `models.providers.vllm`


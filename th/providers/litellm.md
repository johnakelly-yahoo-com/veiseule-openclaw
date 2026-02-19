---
summary: "รัน OpenClaw ผ่าน LiteLLM Proxy เพื่อการเข้าถึงโมเดลแบบรวมศูนย์และติดตามค่าใช้จ่าย"
read_when:
  - คุณต้องการกำหนดเส้นทาง OpenClaw ผ่าน LiteLLM proxy
  - คุณต้องการการติดตามค่าใช้จ่าย การบันทึก log หรือการกำหนดเส้นทางโมเดลผ่าน LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) คือเกตเวย์ LLM แบบโอเพนซอร์สที่ให้ API แบบรวมศูนย์สำหรับผู้ให้บริการโมเดลมากกว่า 100 ราย กำหนดเส้นทาง OpenClaw ผ่าน LiteLLM เพื่อรับการติดตามค่าใช้จ่ายแบบรวมศูนย์ การบันทึก log และความยืดหยุ่นในการสลับ backend โดยไม่ต้องเปลี่ยนการตั้งค่า OpenClaw ของคุณ

## ทำไมต้องใช้ LiteLLM กับ OpenClaw?

- **การติดตามค่าใช้จ่าย** — ดูได้อย่างชัดเจนว่า OpenClaw ใช้งบไปเท่าไรในทุกโมเดล
- **การกำหนดเส้นทางโมเดล** — สลับระหว่าง Claude, GPT-4, Gemini, Bedrock ได้โดยไม่ต้องแก้ไขการตั้งค่า
- **Virtual keys** — สร้างคีย์พร้อมกำหนดวงเงินใช้จ่ายสำหรับ OpenClaw
- **Logging** — บันทึกคำขอ/คำตอบแบบเต็มสำหรับการดีบัก
- **Fallbacks** — สลับไปยังผู้ให้บริการสำรองอัตโนมัติหากผู้ให้บริการหลักไม่พร้อมใช้งาน

## เริ่มต้นอย่างรวดเร็ว

### ผ่านขั้นตอน onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### ตั้งค่าด้วยตนเอง

1. เริ่ม LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. ตั้งค่าให้ OpenClaw ชี้ไปที่ LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

เรียบร้อยแล้ว ขณะนี้ OpenClaw จะกำหนดเส้นทางผ่าน LiteLLM

## การตั้งค่า

### ตัวแปรสภาพแวดล้อม

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### ไฟล์คอนฟิก

```json5
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Virtual keys

สร้างคีย์เฉพาะสำหรับ OpenClaw พร้อมกำหนดวงเงินใช้จ่าย:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

ใช้คีย์ที่สร้างขึ้นเป็น `LITELLM_API_KEY`

## การกำหนดเส้นทางโมเดล

LiteLLM สามารถกำหนดเส้นทางคำขอของโมเดลไปยังแบ็กเอนด์ที่แตกต่างกันได้ กำหนดค่าใน `config.yaml` ของ LiteLLM ของคุณ:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw จะร้องขอ `claude-opus-4-6` ต่อไป — LiteLLM จะจัดการการกำหนดเส้นทางให้

## การดูการใช้งาน

ตรวจสอบแดชบอร์ดหรือ API ของ LiteLLM:

```bash
# ข้อมูลคีย์
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# บันทึกค่าใช้จ่าย
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## หมายเหตุ

- โดยค่าเริ่มต้น LiteLLM จะทำงานที่ `http://localhost:4000`
- OpenClaw เชื่อมต่อผ่านเอ็นด์พอยต์ที่เข้ากันได้กับ OpenAI `/v1/chat/completions`
- ฟีเจอร์ทั้งหมดของ OpenClaw ทำงานผ่าน LiteLLM ได้ — ไม่มีข้อจำกัด

## ดูเพิ่มเติม

- [เอกสาร LiteLLM](https://docs.litellm.ai)
- [ผู้ให้บริการโมเดล](/concepts/model-providers)

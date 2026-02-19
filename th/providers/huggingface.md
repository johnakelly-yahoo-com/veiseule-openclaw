---
summary: "การตั้งค่า Hugging Face Inference (การยืนยันตัวตน + การเลือกโมเดล)"
read_when:
  - คุณต้องการใช้ Hugging Face Inference กับ OpenClaw
  - คุณต้องมีตัวแปรสภาพแวดล้อม HF token หรือเลือกยืนยันตัวตนผ่าน CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) ให้บริการ chat completions ที่เข้ากันได้กับ OpenAI ผ่าน router API เดียว คุณสามารถเข้าถึงโมเดลจำนวนมาก (DeepSeek, Llama และอื่น ๆ) ได้ด้วยโทเค็นเพียงอันเดียว OpenClaw ใช้ **OpenAI-compatible endpoint** (เฉพาะ chat completions); สำหรับ text-to-image, embeddings หรือ speech ให้ใช้ [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) โดยตรง

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` หรือ `HF_TOKEN` (fine-grained token ที่มีสิทธิ์ **Make calls to Inference Providers**)
- API: OpenAI-compatible (`https://router.huggingface.co/v1`)
- Billing: ใช้ HF token เดียว; [pricing](https://huggingface.co/docs/inference-providers/pricing) คิดค่าบริการตามอัตราของผู้ให้บริการ พร้อม free tier

## เริ่มต้นอย่างรวดเร็ว

1. สร้าง fine-grained token ที่ [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) พร้อมสิทธิ์ **Make calls to Inference Providers**
2. รัน onboarding และเลือก **Hugging Face** ใน dropdown ของ provider จากนั้นกรอก API key ของคุณเมื่อระบบร้องขอ:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. ใน dropdown **Default Hugging Face model** ให้เลือกโมเดลที่ต้องการ (รายการจะโหลดจาก Inference API เมื่อคุณมีโทเค็นที่ถูกต้อง; มิฉะนั้นจะแสดงรายการที่มีอยู่ในระบบ) ตัวเลือกของคุณจะถูกบันทึกเป็นโมเดลค่าเริ่มต้น
4. คุณยังสามารถตั้งค่าหรือเปลี่ยนโมเดลค่าเริ่มต้นภายหลังได้ใน config:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## ตัวอย่างแบบไม่โต้ตอบ

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

การตั้งค่านี้จะกำหนด `huggingface/deepseek-ai/DeepSeek-R1` เป็นโมเดลค่าเริ่มต้น

## หมายเหตุเกี่ยวกับ environment

หาก Gateway ทำงานเป็น daemon (launchd/systemd) โปรดตรวจสอบให้แน่ใจว่า `HUGGINGFACE_HUB_TOKEN` หรือ `HF_TOKEN`
พร้อมใช้งานสำหรับ process นั้น (เช่น ใน `~/.openclaw/.env` หรือผ่าน
`env.shellEnv`)

## การค้นหาโมเดลและ dropdown ในขั้นตอน onboarding

OpenClaw ค้นหาโมเดลโดยเรียกใช้ **Inference endpoint โดยตรง**:

```bash
GET https://router.huggingface.co/v1/models
```

(ตัวเลือกเพิ่มเติม: ส่ง `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` หรือ `$HF_TOKEN` เพื่อดูรายการทั้งหมด; บาง endpoint อาจส่งคืนเพียงบางส่วนหากไม่มี auth) การตอบกลับอยู่ในรูปแบบ OpenAI-style `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

เมื่อคุณตั้งค่า Hugging Face API key (ผ่าน onboarding, `HUGGINGFACE_HUB_TOKEN` หรือ `HF_TOKEN`) OpenClaw จะใช้คำสั่ง GET นี้เพื่อค้นหาโมเดล chat-completion ที่พร้อมใช้งาน ในระหว่าง **interactive onboarding** หลังจากที่คุณกรอกโทเค็นแล้ว คุณจะเห็น dropdown **Default Hugging Face model** ที่เติมข้อมูลจากรายการดังกล่าว (หรือจากแคตตาล็อกที่มีอยู่ในระบบหากคำขอล้มเหลว) ขณะรันจริง (เช่น ตอน Gateway เริ่มทำงาน) เมื่อมีการตั้งค่า key แล้ว OpenClaw จะเรียก **GET** `https://router.huggingface.co/v1/models` อีกครั้งเพื่อรีเฟรชแคตตาล็อก รายการนี้จะถูกรวมเข้ากับแคตตาล็อกที่มีอยู่ในระบบ (สำหรับ metadata เช่น context window และค่าใช้จ่าย) หากคำขอล้มเหลวหรือไม่ได้ตั้งค่า key จะใช้เฉพาะแคตตาล็อกที่มีอยู่ในระบบเท่านั้น

## ชื่อโมเดลและตัวเลือกที่แก้ไขได้

- **Name from API:** ชื่อที่แสดงของโมเดลจะถูก **ดึงมาจาก GET /v1/models** เมื่อ API ส่งค่า `name`, `title` หรือ `display_name`; มิฉะนั้นจะสร้างจาก model id (เช่น `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”)
- **Override display name:** คุณสามารถตั้งป้ายชื่อแบบกำหนดเองต่อโมเดลใน config เพื่อให้แสดงตามที่คุณต้องการใน CLI และ UI:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Provider / policy selection:** เพิ่ม suffix ต่อท้าย **model id** เพื่อเลือกวิธีที่ router ใช้เลือก backend:

  - **`:fastest`** — throughput สูงสุด (router เป็นผู้เลือก; การเลือก provider จะถูก **ล็อก** — ไม่มีตัวเลือก backend แบบโต้ตอบ)
  - **`:cheapest`** — ต้นทุนต่อโทเค็นเอาต์พุตต่ำที่สุด (router เป็นผู้เลือก; การเลือกผู้ให้บริการจะถูก **ล็อก**).
  - **`:provider`** — บังคับใช้แบ็กเอนด์ที่ระบุ (เช่น `:sambanova`, `:together`).

  เมื่อคุณเลือก **:cheapest** หรือ **:fastest** (เช่น ในดรอปดาวน์เลือกรุ่นตอน onboarding) ผู้ให้บริการจะถูกล็อก: router จะตัดสินใจตามต้นทุนหรือความเร็ว และจะไม่แสดงขั้นตอนตัวเลือก “เลือกแบ็กเอนด์ที่ต้องการ” เพิ่มเติม คุณสามารถเพิ่มรายการเหล่านี้แยกกันใน `models.providers.huggingface.models` หรือกำหนด `model.primary` พร้อมต่อท้ายด้วย suffix ได้ คุณยังสามารถตั้งค่าลำดับเริ่มต้นได้ใน [Inference Provider settings](https://hf.co/settings/inference-providers) (ไม่ใส่ suffix = ใช้ลำดับนั้น)

- **การรวมคอนฟิก:** รายการที่มีอยู่ใน `models.providers.huggingface.models` (เช่น ใน `models.json`) จะยังคงอยู่เมื่อมีการรวมคอนฟิก ดังนั้น `name`, `alias` หรือออปชันของโมเดลแบบกำหนดเองที่คุณตั้งค่าไว้ที่นั่นจะถูกรักษาไว้

## ตัวอย่าง Model ID และการตั้งค่า

การอ้างอิงโมเดลใช้รูปแบบ `huggingface/<org>/<model>` (ID แบบ Hub) รายการด้านล่างมาจาก **GET** `https://router.huggingface.co/v1/models`; แคตตาล็อกของคุณอาจมีมากกว่านี้

**ตัวอย่าง ID (จาก inference endpoint):**

| โมเดล                                  | Ref (เติมคำนำหน้าด้วย `huggingface/`) |
| -------------------------------------- | -------------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                                |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                              |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                          |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                               |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                         |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                      |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                       |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                    |
| GLM 4.7                | `zai-org/GLM-4.7`                                        |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                   |

คุณสามารถต่อท้าย `:fastest`, `:cheapest` หรือ `:provider` (เช่น `:together`, `:sambanova`) กับ model id ได้ ตั้งค่าลำดับเริ่มต้นของคุณใน [Inference Provider settings](https://hf.co/settings/inference-providers); ดู [Inference Providers](https://huggingface.co/docs/inference-providers) และ **GET** `https://router.huggingface.co/v1/models` สำหรับรายการทั้งหมด

### ตัวอย่างการตั้งค่าฉบับสมบูรณ์

**ตั้งค่า DeepSeek R1 เป็นหลัก พร้อม Qwen เป็น fallback:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**ตั้งค่า Qwen เป็นค่าเริ่มต้น พร้อมตัวแปร :cheapest และ :fastest:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS พร้อม aliases:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**บังคับใช้ backend ที่ระบุด้วย :provider:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**หลายโมเดล Qwen และ DeepSeek พร้อม suffix นโยบาย:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```


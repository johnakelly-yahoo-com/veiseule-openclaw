---
summary: "รันOpenClawด้วยOllama(รันไทม์LLMภายในเครื่อง)"
read_when:
  - คุณต้องการรันOpenClawด้วยโมเดลภายในเครื่องผ่านOllama
  - คุณต้องการคำแนะนำการตั้งค่าและคอนฟิกOllama
title: "Ollama"
---

# Ollama

Ollama เป็นรันไทม์ LLM แบบโลคัลที่ช่วยให้รันโมเดลโอเพนซอร์สบนเครื่องของคุณได้อย่างง่ายดาย Ollamaเป็นรันไทม์LLMภายในเครื่องที่ช่วยให้รันโมเดลโอเพนซอร์สบนเครื่องของคุณได้อย่างง่ายดายOpenClawผสานรวมกับAPIที่เข้ากันได้กับOpenAIของOllamaและสามารถ**ค้นพบโมเดลที่รองรับเครื่องมือโดยอัตโนมัติ**เมื่อคุณเลือกใช้ด้วย`OLLAMA_API_KEY`(หรือโปรไฟล์การยืนยันตัวตน)และไม่ได้กำหนดรายการ`models.providers.ollama`แบบชัดเจน OpenClaw ทำงานร่วมกับ API แบบเนทีฟของ Ollama (`/api/chat`) รองรับการสตรีมและการเรียกใช้เครื่องมือ และสามารถ **ค้นหาโมเดลที่รองรับเครื่องมือโดยอัตโนมัติ** เมื่อคุณเลือกใช้ด้วย `OLLAMA_API_KEY` (หรือโปรไฟล์การยืนยันตัวตน) และไม่ได้กำหนดรายการ `models.providers.ollama` แบบชัดเจน

## เริ่มต้นอย่างรวดเร็ว

1. ติดตั้งOllama: [https://ollama.ai](https://ollama.ai)

2. ดึงโมเดล:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. เปิดใช้งานOllamaสำหรับOpenClaw(ตั้งค่าเป็นค่าใดก็ได้Ollamaไม่ต้องใช้คีย์จริง):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. ใช้โมเดลจากOllama:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## การค้นพบโมเดล(ผู้ให้บริการแบบอัตโนมัติ)

เมื่อคุณตั้งค่า`OLLAMA_API_KEY`(หรือโปรไฟล์การยืนยันตัวตน)และ**ไม่ได้**กำหนด`models.providers.ollama`OpenClawจะค้นพบโมเดลจากอินสแตนซ์Ollamaภายในเครื่องที่`http://127.0.0.1:11434`:

- เรียกดู`/api/tags`และ`/api/show`
- เก็บเฉพาะโมเดลที่รายงานความสามารถ`tools`
- ทำเครื่องหมาย`reasoning`เมื่อโมเดลรายงาน`thinking`
- อ่านค่า`contextWindow`จาก`model_info["<arch>.context_length"]`เมื่อมี
- ตั้งค่า`maxTokens`เป็น10×ของหน้าต่างบริบท
- ตั้งค่าค่าใช้จ่ายทั้งหมดเป็น`0`

วิธีนี้ช่วยหลีกเลี่ยงการเพิ่มรายการโมเดลด้วยตนเองพร้อมคงแคตตาล็อกให้สอดคล้องกับความสามารถของOllama

เพื่อดูว่ามีโมเดลใดบ้าง:

```bash
ollama list
openclaw models list
```

เพื่อเพิ่มโมเดลใหม่เพียงดึงด้วยOllama:

```bash
ollama pull mistral
```

โมเดลใหม่จะถูกค้นพบโดยอัตโนมัติและพร้อมใช้งาน

หากคุณตั้งค่า`models.providers.ollama`แบบชัดเจนการค้นพบอัตโนมัติจะถูกข้ามและคุณต้องกำหนดโมเดลด้วยตนเอง(ดูด้านล่าง)

## การกำหนดค่า

### การตั้งค่าพื้นฐาน(การค้นพบอัตโนมัติ)

วิธีที่ง่ายที่สุดในการเปิดใช้งานOllamaคือผ่านตัวแปรสภาพแวดล้อม:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### การตั้งค่าแบบชัดเจน(โมเดลด้วยตนเอง)

ใช้คอนฟิกแบบชัดเจนเมื่อ:

- Ollamaทำงานบนโฮสต์หรือพอร์ตอื่น
- คุณต้องการบังคับหน้าต่างบริบทหรือรายการโมเดลเฉพาะ
- คุณต้องการรวมโมเดลที่ไม่รายงานการรองรับเครื่องมือ

```json5
{
  models: {
    providers: {
      ollama: {
        // Use a host that includes /v1 for OpenAI-compatible APIs
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

หากตั้งค่า`OLLAMA_API_KEY`ไว้คุณสามารถละเว้น`apiKey`ในรายการผู้ให้บริการได้และOpenClawจะเติมค่าให้สำหรับการตรวจสอบความพร้อมใช้งาน

### URLฐานแบบกำหนดเอง(คอนฟิกแบบชัดเจน)

หากOllamaทำงานบนโฮสต์หรือพอร์ตที่ต่างออกไป(คอนฟิกแบบชัดเจนจะปิดการค้นพบอัตโนมัติดังนั้นต้องกำหนดโมเดลด้วยตนเอง):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1",
      },
    },
  },
}
```

### การเลือกโมเดล

เมื่อกำหนดค่าแล้วโมเดลOllamaทั้งหมดของคุณจะพร้อมใช้งาน:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## ขั้นสูง

### โมเดลด้านการให้เหตุผล

OpenClawจะทำเครื่องหมายโมเดลว่าสามารถให้เหตุผลได้เมื่อOllamaรายงาน`thinking`ใน`/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### ค่าใช้จ่ายของโมเดล

Ollamaใช้งานได้ฟรีและรันภายในเครื่องดังนั้นค่าใช้จ่ายของโมเดลทั้งหมดจะถูกตั้งเป็น$0

### การกำหนดค่าการสตรีม

การผสานรวม Ollama ของ OpenClaw ใช้ **API แบบเนทีฟของ Ollama** (`/api/chat`) โดยค่าเริ่มต้น ซึ่งรองรับการสตรีมและการเรียกใช้เครื่องมือพร้อมกันได้อย่างสมบูรณ์ ไม่จำเป็นต้องมีการตั้งค่าพิเศษ

#### โหมดที่เข้ากันได้กับ OpenAI แบบเดิม

หากคุณจำเป็นต้องใช้เอนด์พอยต์ที่รองรับ OpenAI แทน (เช่น อยู่หลังพร็อกซีที่รองรับเฉพาะรูปแบบ OpenAI) ให้ตั้งค่า `api: "openai-completions"` อย่างชัดเจน:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

หมายเหตุ: เอนด์พอยต์ที่รองรับ OpenAI อาจไม่รองรับการสตรีมมิงและการเรียกใช้เครื่องมือ (tool calling) พร้อมกัน ตั้งค่า`streaming: false`สำหรับโมเดลOllamaแบบชัดเจน(ดู[การกำหนดค่าการสตรีม](#streaming-configuration))

### หน้าต่างบริบท

สำหรับโมเดลที่ค้นพบอัตโนมัติOpenClawจะใช้หน้าต่างบริบทที่Ollamaรายงานเมื่อมีไม่เช่นนั้นจะใช้ค่าเริ่มต้นเป็น`8192`คุณสามารถแทนที่`contextWindow`และ`maxTokens`ในคอนฟิกผู้ให้บริการแบบชัดเจนได้ คุณสามารถแทนที่ค่า `contextWindow` และ `maxTokens` ได้ในการกำหนดค่าผู้ให้บริการแบบระบุชัดเจน คุณสามารถแทนที่ค่า `contextWindow` และ `maxTokens` ได้ในการกำหนดค่าผู้ให้บริการแบบระบุชัดเจน

## การแก้ไขปัญหา

### ไม่ตรวจพบOllama

ตรวจสอบให้แน่ใจว่าOllamaกำลังทำงานและคุณได้ตั้งค่า`OLLAMA_API_KEY`(หรือโปรไฟล์การยืนยันตัวตน)และคุณ**ไม่ได้**กำหนดรายการ`models.providers.ollama`แบบชัดเจน:

```bash
ollama serve
```

และAPIสามารถเข้าถึงได้:

```bash
curl http://localhost:11434/api/tags
```

### ไม่มีโมเดลให้ใช้งาน

OpenClawจะค้นพบอัตโนมัติเฉพาะโมเดลที่รายงานการรองรับเครื่องมือหากโมเดลของคุณไม่แสดงในรายการให้ทำอย่างใดอย่างหนึ่ง: หากโมเดลของคุณไม่อยู่ในรายการ ให้เลือกอย่างใดอย่างหนึ่ง: หากโมเดลของคุณไม่อยู่ในรายการ ให้เลือกอย่างใดอย่างหนึ่ง:

- ดึงโมเดลที่รองรับเครื่องมือหรือ
- กำหนดโมเดลแบบชัดเจนใน`models.providers.ollama`

เพื่อเพิ่มโมเดล:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### การเชื่อมต่อถูกปฏิเสธ

ตรวจสอบว่าOllamaกำลังทำงานบนพอร์ตที่ถูกต้อง:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## ดูเพิ่มเติม

- [ผู้ให้บริการโมเดล](/concepts/model-providers) - ภาพรวมของผู้ให้บริการทั้งหมด
- [การเลือกโมเดล](/concepts/models) - วิธีเลือกโมเดล
- [การกำหนดค่า](/gateway/configuration) - เอกสารอ้างอิงคอนฟิกทั้งหมด


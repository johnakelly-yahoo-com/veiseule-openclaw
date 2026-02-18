---
title: "การทำความเข้าใจสื่อ"
---

# การทำความเข้าใจสื่อ(ขาเข้า) — 2026-01-17

34. OpenClaw สามารถ **สรุปสื่อขาเข้า** (ภาพ/เสียง/วิดีโอ) ก่อนที่ไปป์ไลน์การตอบกลับจะทำงาน 35. ระบบจะตรวจจับอัตโนมัติเมื่อมีเครื่องมือภายในเครื่องหรือคีย์ผู้ให้บริการ และสามารถปิดหรือปรับแต่งได้ 36. หากปิดความเข้าใจ โมเดลยังคงได้รับไฟล์/URL ต้นฉบับตามปกติ

## เป้าหมาย

- ทางเลือก: ย่อยสื่อขาเข้าเป็นข้อความสั้นเพื่อให้การจัดเส้นทางเร็วขึ้นและแยกคำสั่งได้ดีขึ้น
- คงการส่งมอบสื่อดั้งเดิมไปยังโมเดลเสมอ
- รองรับ**APIของผู้ให้บริการ**และ**CLIสำรอง**
- อนุญาตหลายโมเดลพร้อมลำดับการสำรองเมื่อเกิดข้อผิดพลาด/ขนาด/หมดเวลา

## พฤติกรรมระดับสูง

1. รวบรวมไฟล์แนบขาเข้า(`MediaPaths`, `MediaUrls`, `MediaTypes`)
2. สำหรับแต่ละความสามารถที่เปิดใช้งาน(รูปภาพ/เสียง/วิดีโอ) เลือกไฟล์แนบตามนโยบาย(ค่าเริ่มต้น: **ครั้งแรก**)
3. เลือกรายการโมเดลที่มีคุณสมบัติเหมาะสมรายการแรก(ขนาด+ความสามารถ+การยืนยันตัวตน)
4. หากโมเดลล้มเหลวหรือสื่อมีขนาดใหญ่เกินไป ให้**สำรองไปยังรายการถัดไป**
5. เมื่อสำเร็จ:
   - `Body` จะกลายเป็นบล็อก `[Image]`, `[Audio]` หรือ `[Video]`
   - เสียงจะตั้งค่า `{{Transcript}}`; การแยกคำสั่งใช้ข้อความคำบรรยายเมื่อมี มิฉะนั้นใช้ทรานสคริปต์
   - คำบรรยายจะถูกเก็บเป็น `User text:` ภายในบล็อก

หากการทำความเข้าใจล้มเหลวหรือถูกปิดใช้งาน**โฟลว์การตอบกลับจะดำเนินต่อ**ด้วยเนื้อหาและไฟล์แนบเดิม

## ภาพรวมคอนฟิก

`tools.media` รองรับ**โมเดลที่ใช้ร่วมกัน**พร้อมการแทนที่รายความสามารถ:

- `tools.media.models`: รายการโมเดลที่ใช้ร่วมกัน(ใช้ `capabilities` เพื่อกำหนดเงื่อนไข)
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - ค่าเริ่มต้น(`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - การแทนที่ของผู้ให้บริการ(`baseUrl`, `headers`, `providerOptions`)
  - ตัวเลือกเสียงของ Deepgram ผ่าน `tools.media.audio.providerOptions.deepgram`
  - **รายการ `models` ต่อความสามารถแบบเลือกใช้ได้**(ให้ความสำคัญก่อนโมเดลที่ใช้ร่วมกัน)
  - นโยบาย `attachments`(`mode`, `maxAttachments`, `prefer`)
  - `scope`(การกำหนดเงื่อนไขตามช่องทาง/ประเภทแชต/คีย์เซสชันแบบเลือกใช้)
- `tools.media.concurrency`: จำนวนการรันความสามารถพร้อมกันสูงสุด(ค่าเริ่มต้น **2**)

```json5
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### รายการโมเดล

แต่ละรายการ `models[]` สามารถเป็นแบบ**ผู้ให้บริการ**หรือ**CLI**:

```json5
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

เทมเพลตCLIยังสามารถใช้:

- `{{MediaDir}}`(ไดเรกทอรีที่มีไฟล์สื่อ)
- `{{OutputDir}}`(ไดเรกทอรีชั่วคราวที่สร้างสำหรับการรันนี้)
- `{{OutputBase}}`(พาธฐานของไฟล์ชั่วคราว ไม่มีนามสกุล)

## ค่าเริ่มต้นและข้อจำกัด

ค่าเริ่มต้นที่แนะนำ:

- `maxChars`: **500** สำหรับรูปภาพ/วิดีโอ(สั้นและเหมาะกับคำสั่ง)
- `maxChars`: **unset** สำหรับเสียง(ทรานสคริปต์เต็ม เว้นแต่คุณจะตั้งขีดจำกัด)
- `maxBytes`:
  - รูปภาพ: **10MB**
  - เสียง: **20MB**
  - วิดีโอ: **50MB**

กฎ:

- หากสื่อเกิน `maxBytes` โมเดลนั้นจะถูกข้ามและ**ลองโมเดลถัดไป**
- หากโมเดลส่งคืนมากกว่า `maxChars` เอาต์พุตจะถูกตัด
- `prompt` ค่าเริ่มต้นเป็น “Describe the {media}.” แบบง่าย พร้อมแนวทาง `maxChars`(เฉพาะรูปภาพ/วิดีโอ)
- หาก `<capability>.enabled: true` แต่ไม่มีการตั้งค่าโมเดล OpenClaw จะลองใช้**โมเดลตอบกลับที่ใช้งานอยู่**เมื่อผู้ให้บริการรองรับความสามารถนั้น

### การตรวจจับการทำความเข้าใจสื่ออัตโนมัติ(ค่าเริ่มต้น)

หากไม่ได้ตั้งค่า `tools.media.<capability>.enabled` เป็น `false` และคุณยังไม่ได้กำหนดโมเดล OpenClaw จะตรวจจับอัตโนมัติตามลำดับนี้และ**หยุดเมื่อพบตัวเลือกที่ใช้งานได้ตัวแรก**:

1. **CLIภายในเครื่อง**(เฉพาะเสียง; หากติดตั้ง)
   - `sherpa-onnx-offline`(ต้องการ `SHERPA_ONNX_MODEL_DIR` พร้อม encoder/decoder/joiner/tokens)
   - `whisper-cli`(`whisper-cpp`; ใช้ `WHISPER_CPP_MODEL` หรือโมเดลขนาดเล็กที่มากับระบบ)
   - `whisper`(Python CLI; ดาวน์โหลดโมเดลอัตโนมัติ)
2. **Gemini CLI**(`gemini`) โดยใช้ `read_many_files`
3. **คีย์ผู้ให้บริการ**
   - เสียง: OpenAI → Groq → Deepgram → Google
   - รูปภาพ: OpenAI → Anthropic → Google → MiniMax
   - วิดีโอ: Google

หากต้องการปิดการตรวจจับอัตโนมัติ ให้ตั้งค่า:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

หมายเหตุ: การตรวจจับไบนารีเป็นแบบพยายามอย่างดีที่สุดบน macOS/Linux/Windows; ตรวจสอบให้แน่ใจว่าCLIอยู่บน `PATH`(เราจะขยาย `~`) หรือกำหนดโมเดลCLIแบบระบุพาธคำสั่งเต็ม

## ความสามารถ(ทางเลือก)

37. หากคุณตั้งค่า `capabilities` รายการจะรันเฉพาะกับประเภทสื่อนั้น ๆ หากคุณตั้งค่า `capabilities` รายการจะทำงานเฉพาะประเภทสื่อเหล่านั้น สำหรับรายการที่ใช้ร่วมกัน OpenClaw สามารถอนุมานค่าเริ่มต้นได้:

- `openai`, `anthropic`, `minimax`: **รูปภาพ**
- `google`(Gemini API): **รูปภาพ+เสียง+วิดีโอ**
- `groq`: **เสียง**
- `deepgram`: **เสียง**

สำหรับรายการCLI **ให้ตั้งค่า `capabilities` อย่างชัดเจน**เพื่อหลีกเลี่ยงการจับคู่ที่ไม่คาดคิด หากคุณละเว้น `capabilities` รายการจะมีสิทธิ์สำหรับรายการที่มันปรากฏอยู่
38. หากคุณไม่ใส่ `capabilities` รายการจะมีสิทธิ์ตามรายการที่มันปรากฏอยู่

## ตารางการรองรับของผู้ให้บริการ(OpenClaw integrations)

| ความสามารถ | การผสานรวมผู้ให้บริการ                          | หมายเหตุ                                                               |
| ---------- | ----------------------------------------------- | ---------------------------------------------------------------------- |
| รูปภาพ     | OpenAI / Anthropic / Google / อื่นๆผ่าน `pi-ai` | โมเดลที่รองรับรูปภาพใดๆในรีจิสทรีสามารถใช้งานได้                       |
| เสียง      | OpenAI, Groq, Deepgram, Google                  | การถอดเสียงโดยผู้ให้บริการ(Whisper/Deepgram/Gemini) |
| วิดีโอ     | Google(Gemini API)           | การทำความเข้าใจวิดีโอโดยผู้ให้บริการ                                   |

## ผู้ให้บริการที่แนะนำ

**รูปภาพ**

- แนะนำให้ใช้โมเดลที่ใช้งานอยู่หากรองรับรูปภาพ
- ค่าเริ่มต้นที่ดี: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`

**เสียง**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` หรือ `deepgram/nova-3`
- CLIสำรอง: `whisper-cli`(whisper-cpp) หรือ `whisper`
- การตั้งค่า Deepgram: [Deepgram (audio transcription)](/providers/deepgram)

**วิดีโอ**

- `google/gemini-3-flash-preview`(เร็ว), `google/gemini-3-pro-preview`(เนื้อหาลึกกว่า)
- CLIสำรอง: `gemini` CLI(รองรับ `read_file` บนวิดีโอ/เสียง)

## นโยบายไฟล์แนบ

`attachments` ต่อความสามารถควบคุมว่าไฟล์แนบใดจะถูกประมวลผล:

- `mode`: `first`(ค่าเริ่มต้น) หรือ `all`
- `maxAttachments`: จำกัดจำนวนที่ประมวลผล(ค่าเริ่มต้น **1**)
- `prefer`: `first`, `last`, `path`, `url`

เมื่อ `mode: "all"` เอาต์พุตจะถูกติดป้ายเป็น `[Image 1/2]`, `[Audio 2/2]` เป็นต้น

## ตัวอย่างคอนฟิก

### 1. รายการโมเดลที่ใช้ร่วมกัน+การแทนที่

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2. เฉพาะเสียง+วิดีโอ(ปิดรูปภาพ)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3. การทำความเข้าใจรูปภาพแบบเลือกใช้ได้

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4. รายการเดี่ยวแบบหลายโหมด(กำหนดความสามารถชัดเจน)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## เอาต์พุตสถานะ

เมื่อการทำความเข้าใจสื่อทำงาน `/status` จะมีบรรทัดสรุปสั้นๆ:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

แสดงผลลัพธ์รายความสามารถและผู้ให้บริการ/โมเดลที่เลือกเมื่อมี

## หมายเหตุ

- 39. การทำความเข้าใจเป็นแบบ **best‑effort** การทำความเข้าใจเป็นแบบ**พยายามอย่างดีที่สุด** ข้อผิดพลาดจะไม่บล็อกการตอบกลับ
- ไฟล์แนบยังคงถูกส่งไปยังโมเดลแม้ปิดการทำความเข้าใจ
- ใช้ `scope` เพื่อจำกัดตำแหน่งที่การทำความเข้าใจจะทำงาน(เช่น เฉพาะDMs)

## เอกสารที่เกี่ยวข้อง

- [การกำหนดค่า](/gateway/configuration)
- [การรองรับรูปภาพและสื่อ](/nodes/images)

---
summary: "Hiểu nội dung media đầu vào (hình ảnh/âm thanh/video) tùy chọn với nhà cung cấp + phương án dự phòng CLI"
read_when:
  - Thiết kế hoặc tái cấu trúc phần hiểu media
  - Tinh chỉnh tiền xử lý âm thanh/video/hình ảnh đầu vào
title: "Hiểu Media"
---

# Hiểu Media (Đầu vào) — 2026-01-17

OpenClaw có thể **tóm tắt media đến** (hình ảnh/âm thanh/video) trước khi pipeline phản hồi chạy. Nó tự động phát hiện khi các công cụ cục bộ hoặc khóa nhà cung cấp khả dụng, và có thể bị vô hiệu hóa hoặc tùy chỉnh. Nếu tính năng hiểu bị tắt, các mô hình vẫn nhận các tệp/URL gốc như bình thường.

## Mục tiêu

- Tùy chọn: tiền xử lý media đầu vào thành văn bản ngắn để định tuyến nhanh hơn + phân tích lệnh tốt hơn.
- Luôn bảo toàn việc gửi media gốc cho mô hình.
- Hỗ trợ **API của nhà cung cấp** và **phương án dự phòng CLI**.
- Cho phép nhiều mô hình với thứ tự dự phòng (lỗi/kích thước/timeout).

## Hành vi tổng quát

1. Thu thập các tệp đính kèm đầu vào (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Với mỗi khả năng được bật (hình ảnh/âm thanh/video), chọn tệp theo chính sách (mặc định: **đầu tiên**).
3. Chọn mục mô hình đủ điều kiện đầu tiên (kích thước + khả năng + xác thực).
4. Nếu mô hình lỗi hoặc media quá lớn, **chuyển sang mục tiếp theo**.
5. Khi thành công:
   - `Body` trở thành khối `[Image]`, `[Audio]` hoặc `[Video]`.
   - Âm thanh đặt `{{Transcript}}`; phân tích lệnh dùng văn bản caption khi có,
     nếu không thì dùng bản chép lời.
   - Caption được giữ lại dưới dạng `User text:` bên trong khối.

Nếu việc hiểu nội dung thất bại hoặc bị tắt, **luồng phản hồi vẫn tiếp tục** với phần thân + tệp đính kèm gốc.

## Tổng quan cấu hình

`tools.media` hỗ trợ **mô hình dùng chung** cùng các ghi đè theo từng khả năng:

- `tools.media.models`: danh sách mô hình dùng chung (dùng `capabilities` để kiểm soát).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - mặc định (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - ghi đè theo nhà cung cấp (`baseUrl`, `headers`, `providerOptions`)
  - tùy chọn Deepgram cho âm thanh qua `tools.media.audio.providerOptions.deepgram`
  - **danh sách `models` theo từng khả năng** (tùy chọn; được ưu tiên trước mô hình dùng chung)
  - chính sách `attachments` (`mode`, `maxAttachments`, `prefer`)
  - `scope` (tùy chọn kiểm soát theo kênh/chatType/khóa phiên)
- `tools.media.concurrency`: số lần chạy khả năng đồng thời tối đa (mặc định **2**).

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

### Mục mô hình

Mỗi mục `models[]` có thể là **nhà cung cấp** hoặc **CLI**:

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

Mẫu CLI cũng có thể dùng:

- `{{MediaDir}}` (thư mục chứa tệp media)
- `{{OutputDir}}` (thư mục scratch được tạo cho lần chạy này)
- `{{OutputBase}}` (đường dẫn cơ sở của tệp scratch, không có phần mở rộng)

## Mặc định và giới hạn

Mặc định khuyến nghị:

- `maxChars`: **500** cho hình ảnh/video (ngắn, thân thiện với lệnh)
- `maxChars`: **không đặt** cho âm thanh (chép lời đầy đủ trừ khi bạn đặt giới hạn)
- `maxBytes`:
  - hình ảnh: **10MB**
  - âm thanh: **20MB**
  - video: **50MB**

Quy tắc:

- Nếu media vượt quá `maxBytes`, mô hình đó bị bỏ qua và **thử mô hình tiếp theo**.
- Nếu mô hình trả về nhiều hơn `maxChars`, đầu ra sẽ bị cắt bớt.
- `prompt` mặc định là “Describe the {media}.” đơn giản cộng với hướng dẫn `maxChars` (chỉ cho hình ảnh/video).
- Nếu `<capability>.enabled: true` nhưng không cấu hình mô hình nào, OpenClaw sẽ thử
  **mô hình phản hồi đang hoạt động** khi nhà cung cấp của nó hỗ trợ khả năng đó.

### Tự động phát hiện hiểu media (mặc định)

Nếu `tools.media.<capability>``.enabled` **không** được đặt thành `false` và bạn chưa cấu hình mô hình, OpenClaw sẽ tự động phát hiện theo thứ tự này và **dừng ở tùy chọn hoạt động đầu tiên**:

1. **CLI cục bộ** (chỉ âm thanh; nếu đã cài)
   - `sherpa-onnx-offline` (yêu cầu `SHERPA_ONNX_MODEL_DIR` với encoder/decoder/joiner/tokens)
   - `whisper-cli` (`whisper-cpp`; dùng `WHISPER_CPP_MODEL` hoặc mô hình tiny đi kèm)
   - `whisper` (CLI Python; tự động tải mô hình)
2. **Gemini CLI** (`gemini`) dùng `read_many_files`
3. **Khóa nhà cung cấp**
   - Âm thanh: OpenAI → Groq → Deepgram → Google
   - Hình ảnh: OpenAI → Anthropic → Google → MiniMax
   - Video: Google

Để tắt tự động phát hiện, đặt:

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

Lưu ý: Việc phát hiện nhị phân là best‑effort trên macOS/Linux/Windows; hãy đảm bảo CLI nằm trong `PATH` (chúng tôi mở rộng `~`), hoặc đặt một mô hình CLI tường minh với đường dẫn lệnh đầy đủ.

## Khả năng (tùy chọn)

If you set `capabilities`, the entry only runs for those media types. For shared
lists, OpenClaw can infer defaults:

- `openai`, `anthropic`, `minimax`: **hình ảnh**
- `google` (Gemini API): **hình ảnh + âm thanh + video**
- `groq`: **âm thanh**
- `deepgram`: **âm thanh**

Đối với các mục CLI, **hãy đặt `capabilities` một cách rõ ràng** để tránh các khớp không mong muốn.
If you omit `capabilities`, the entry is eligible for the list it appears in.

## Ma trận hỗ trợ nhà cung cấp (tích hợp OpenClaw)

| Khả năng | Tích hợp nhà cung cấp                                  | Ghi chú                                                                                |
| -------- | ------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| Hình ảnh | OpenAI / Anthropic / Google / các bên khác qua `pi-ai` | Bất kỳ mô hình có khả năng hình ảnh trong registry đều hoạt động.      |
| Âm thanh | OpenAI, Groq, Deepgram, Google                         | Chép lời từ nhà cung cấp (Whisper/Deepgram/Gemini). |
| Video    | Google (Gemini API)                 | Hiểu video từ nhà cung cấp.                                            |

## Nhà cung cấp khuyến nghị

**Hình ảnh**

- Ưu tiên mô hình đang hoạt động của bạn nếu nó hỗ trợ hình ảnh.
- Mặc định tốt: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Âm thanh**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, hoặc `deepgram/nova-3`.
- Dự phòng CLI: `whisper-cli` (whisper-cpp) hoặc `whisper`.
- Thiết lập Deepgram: [Deepgram (chép lời âm thanh)](/providers/deepgram).

**Video**

- `google/gemini-3-flash-preview` (nhanh), `google/gemini-3-pro-preview` (phong phú hơn).
- Dự phòng CLI: CLI `gemini` (hỗ trợ `read_file` cho video/âm thanh).

## Chính sách tệp đính kèm

`attachments` theo từng khả năng kiểm soát tệp nào được xử lý:

- `mode`: `first` (mặc định) hoặc `all`
- `maxAttachments`: giới hạn số lượng xử lý (mặc định **1**)
- `prefer`: `first`, `last`, `path`, `url`

Khi `mode: "all"`, các đầu ra được gắn nhãn `[Image 1/2]`, `[Audio 2/2]`, v.v.

## Ví dụ cấu hình

### 1. Danh sách mô hình dùng chung + ghi đè

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

### 2. Chỉ Âm thanh + Video (tắt hình ảnh)

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

### 3. Hiểu hình ảnh tùy chọn

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

### 4. Mục đơn đa phương thức (khả năng tường minh)

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

## Đầu ra trạng thái

Khi phần hiểu media chạy, `/status` bao gồm một dòng tóm tắt ngắn:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Điều này cho thấy kết quả theo từng khả năng và nhà cung cấp/mô hình được chọn khi áp dụng.

## Ghi chú

- Understanding is **best‑effort**. Lỗi không chặn phản hồi.
- Tệp đính kèm vẫn được chuyển cho mô hình ngay cả khi phần hiểu bị tắt.
- Dùng `scope` để giới hạn nơi việc hiểu được chạy (ví dụ: chỉ DM).

## Tài liệu liên quan

- [Cấu hình](/gateway/configuration)
- [Hỗ trợ Hình ảnh & Media](/nodes/images)


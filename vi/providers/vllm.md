---
summary: "Chạy OpenClaw với vLLM (máy chủ cục bộ tương thích OpenAI)"
read_when:
  - Bạn muốn chạy OpenClaw với một máy chủ vLLM cục bộ
  - Bạn muốn các endpoint /v1 tương thích OpenAI với các mô hình riêng của mình
title: "vLLM"
---

# vLLM

vLLM có thể phục vụ các mô hình mã nguồn mở (và một số mô hình tùy chỉnh) thông qua HTTP API **tương thích OpenAI**. OpenClaw có thể kết nối tới vLLM bằng API `openai-completions`.

OpenClaw cũng có thể **tự động phát hiện** các mô hình khả dụng từ vLLM khi bạn bật với `VLLM_API_KEY` (bất kỳ giá trị nào cũng được nếu máy chủ của bạn không yêu cầu xác thực) và bạn không định nghĩa mục `models.providers.vllm` rõ ràng.

## Bắt đầu nhanh

1. Khởi động vLLM với máy chủ tương thích OpenAI.

Base URL của bạn phải cung cấp các endpoint `/v1` (ví dụ: `/v1/models`, `/v1/chat/completions`). vLLM thường chạy tại:

- `http://127.0.0.1:8000/v1`

2. Bật sử dụng (bất kỳ giá trị nào cũng được nếu không cấu hình xác thực):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Chọn một mô hình (thay bằng một trong các model ID của vLLM bạn):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Phát hiện mô hình (nhà cung cấp ngầm định)

Khi `VLLM_API_KEY` được thiết lập (hoặc tồn tại hồ sơ xác thực) và bạn **không** định nghĩa `models.providers.vllm`, OpenClaw sẽ truy vấn:

- `GET http://127.0.0.1:8000/v1/models`

…và chuyển đổi các ID trả về thành các mục mô hình.

Nếu bạn thiết lập `models.providers.vllm` một cách rõ ràng, tính năng tự động phát hiện sẽ bị bỏ qua và bạn phải tự định nghĩa các mô hình.

## Cấu hình tường minh (mô hình thủ công)

Sử dụng cấu hình tường minh khi:

- vLLM chạy trên host/port khác.
- Bạn muốn cố định các giá trị `contextWindow`/`maxTokens`.
- Máy chủ của bạn yêu cầu API key thực (hoặc bạn muốn kiểm soát headers).

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

## Khắc phục sự cố

- Kiểm tra xem máy chủ có thể truy cập được không:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Nếu yêu cầu thất bại do lỗi xác thực, hãy đặt `VLLM_API_KEY` thực khớp với cấu hình máy chủ của bạn, hoặc cấu hình rõ ràng provider trong `models.providers.vllm`.


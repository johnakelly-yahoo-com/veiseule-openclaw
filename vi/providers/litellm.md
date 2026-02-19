---
summary: "Chạy OpenClaw thông qua LiteLLM Proxy để truy cập model hợp nhất và theo dõi chi phí"
read_when:
  - Bạn muốn định tuyến OpenClaw thông qua một LiteLLM proxy
  - Bạn cần theo dõi chi phí, ghi log hoặc định tuyến model thông qua LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) là một LLM gateway mã nguồn mở cung cấp API hợp nhất cho hơn 100 nhà cung cấp model. Định tuyến OpenClaw thông qua LiteLLM để có theo dõi chi phí tập trung, ghi log và linh hoạt chuyển đổi backend mà không cần thay đổi cấu hình OpenClaw của bạn.

## Vì sao nên sử dụng LiteLLM với OpenClaw?

- **Theo dõi chi phí** — Xem chính xác OpenClaw đã chi bao nhiêu trên tất cả các model
- **Định tuyến model** — Chuyển đổi giữa Claude, GPT-4, Gemini, Bedrock mà không cần thay đổi cấu hình
- **Khóa ảo** — Tạo khóa với giới hạn chi tiêu cho OpenClaw
- **Ghi log** — Ghi lại đầy đủ request/response để gỡ lỗi
- **Dự phòng** — Tự động chuyển đổi khi nhà cung cấp chính gặp sự cố

## Bắt đầu nhanh

### Thông qua onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Thiết lập thủ công

1. Khởi động LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Trỏ OpenClaw tới LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Xong. OpenClaw giờ sẽ định tuyến thông qua LiteLLM.

## Cấu hình

### Biến môi trường

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Tệp cấu hình

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

## Khóa ảo

Tạo một khóa chuyên dụng cho OpenClaw với giới hạn chi tiêu:

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

Sử dụng khóa đã tạo làm `LITELLM_API_KEY`.

## Định tuyến model

LiteLLM có thể định tuyến các yêu cầu model đến các backend khác nhau. Cấu hình trong `config.yaml` của LiteLLM:

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

OpenClaw vẫn tiếp tục yêu cầu `claude-opus-4-6` — LiteLLM sẽ xử lý việc định tuyến.

## Xem mức sử dụng

Kiểm tra dashboard hoặc API của LiteLLM:

```bash
# Thông tin khóa
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Log chi tiêu
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Ghi chú

- LiteLLM mặc định chạy tại `http://localhost:4000`
- OpenClaw kết nối qua endpoint tương thích OpenAI `/v1/chat/completions`
- Tất cả tính năng của OpenClaw đều hoạt động thông qua LiteLLM — không có giới hạn

## Xem thêm

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)


---
title: "OpenRouter"
---

# OpenRouter

OpenRouter cung cấp một **API hợp nhất** định tuyến các yêu cầu tới nhiều mô hình phía sau một
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## Thiết lập CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Đoạn cấu hình

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## Ghi chú

- Tham chiếu mô hình là `openrouter/<provider>/<model>`.
- Để biết thêm tùy chọn mô hình/nhà cung cấp, xem [/concepts/model-providers](/concepts/model-providers).
- OpenRouter sử dụng Bearer token với khóa API của bạn ở phía dưới.


---
summary: "Sử dụng API tương thích OpenAI của NVIDIA trong OpenClaw"
read_when:
  - Bạn muốn sử dụng các model NVIDIA trong OpenClaw
  - Bạn cần thiết lập NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA cung cấp API tương thích OpenAI tại `https://integrate.api.nvidia.com/v1` cho các mô hình Nemotron và NeMo. Xác thực bằng API key từ [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Thiết lập CLI

Xuất key một lần, sau đó chạy onboarding và thiết lập một mô hình NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Nếu bạn vẫn truyền `--token`, hãy nhớ rằng nó sẽ xuất hiện trong lịch sử shell và đầu ra `ps`; ưu tiên sử dụng biến môi trường khi có thể.

## Đoạn cấu hình

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

## ID mô hình

- `nvidia/llama-3.1-nemotron-70b-instruct` (mặc định)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Ghi chú

- Endpoint `/v1` tương thích OpenAI; sử dụng API key từ NVIDIA NGC.
- Provider sẽ tự động bật khi `NVIDIA_API_KEY` được thiết lập; sử dụng cấu hình mặc định tĩnh (cửa sổ ngữ cảnh 131.072 token, tối đa 4.096 token).

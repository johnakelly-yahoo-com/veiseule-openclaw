---
summary: "在 OpenClaw 中使用 NVIDIA 的 OpenAI 相容 API"
read_when:
  - 你想在 OpenClaw 中使用 NVIDIA 模型
  - 你需要設定 NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA 為 Nemotron 和 NeMo 模型提供與 OpenAI 相容的 API：`https://integrate.api.nvidia.com/v1`。 使用來自 [NVIDIA NGC](https://catalog.ngc.nvidia.com/) 的 API key 進行驗證。

## CLI 設定

先匯出金鑰一次，然後執行 onboarding 並設定 NVIDIA 模型：

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

如果你仍然傳遞 `--token`，請記住它會出現在 shell 歷史記錄與 `ps` 輸出中；在可能的情況下請優先使用環境變數。

## 設定範例

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

## 模型 ID

- `nvidia/llama-3.1-nemotron-70b-instruct`（預設）
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## 注意事項

- OpenAI 相容的 `/v1` 端點；請使用來自 NVIDIA NGC 的 API key。
- 當設定 `NVIDIA_API_KEY` 時，Provider 會自動啟用；使用靜態預設值（131,072-token 上下文視窗、4,096 max tokens）。


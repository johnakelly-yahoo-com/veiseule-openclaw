---
summary: "在 OpenClaw 中使用 NVIDIA 的 OpenAI 兼容 API"
read_when:
  - 你希望在 OpenClaw 中使用 NVIDIA 模型
  - 你需要先设置 NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA 为 Nemotron 和 NeMo 模型提供了一个兼容 OpenAI 的 API，地址为 `https://integrate.api.nvidia.com/v1`。 使用来自 [NVIDIA NGC](https://catalog.ngc.nvidia.com/) 的 API key 进行身份验证。

## CLI 设置

导出该 key 一次，然后运行初始化并设置一个 NVIDIA 模型：

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

如果你仍然传递 `--token`，请注意它会出现在 shell 历史记录和 `ps` 输出中；在可能的情况下优先使用环境变量。

## 配置片段

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

- `nvidia/llama-3.1-nemotron-70b-instruct`（默认）
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## 说明

- 兼容 OpenAI 的 `/v1` 端点；使用来自 NVIDIA NGC 的 API key。
- 当设置了 `NVIDIA_API_KEY` 时会自动启用该 provider；使用静态默认值（131,072-token 上下文窗口，4,096 max tokens）。

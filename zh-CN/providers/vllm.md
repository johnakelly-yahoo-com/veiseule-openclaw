---
summary: "使用 vLLM（兼容 OpenAI 的本地服务器）运行 OpenClaw"
read_when:
  - 你希望让 OpenClaw 连接到本地的 vLLM 服务器
  - 你希望使用自己的模型并提供兼容 OpenAI 的 `/v1` 端点
title: "vLLM"
---

# vLLM

vLLM 可以通过**兼容 OpenAI**的 HTTP API 提供开源（以及部分自定义）模型服务。 OpenClaw 可以使用 `openai-completions` API 连接到 vLLM。

当你通过设置 `VLLM_API_KEY`（如果服务器未启用身份验证，可使用任意值）选择启用，并且未显式定义 `models.providers.vllm` 条目时，OpenClaw 还可以从 vLLM **自动发现**可用模型。

## 快速开始

1. 使用兼容 OpenAI 的服务器启动 vLLM。

你的 base URL 应暴露 `/v1` 端点（例如 `/v1/models`、`/v1/chat/completions`）。 vLLM 通常运行在：

- `http://127.0.0.1:8000/v1`

2. 选择启用（如果未配置身份验证，可使用任意值）：

```bash
export VLLM_API_KEY="vllm-local"
```

3. 选择一个模型（替换为你的 vLLM 模型 ID 之一）：

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## 模型发现（隐式 provider）

当设置了 `VLLM_API_KEY`（或存在身份验证配置）且**未**定义 `models.providers.vllm` 时，OpenClaw 将查询：

- `GET http://127.0.0.1:8000/v1/models`

……并将返回的 ID 转换为模型条目。

如果你显式设置了 `models.providers.vllm`，则会跳过自动发现，你必须手动定义模型。

## 显式配置（手动定义模型）

在以下情况下使用显式配置：

- vLLM 运行在不同的主机/端口上。
- 你希望固定 `contextWindow`/`maxTokens` 的值。
- 你的服务器需要真实的 API key（或你希望自定义请求头）。

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

## 故障排查

- 检查服务器是否可访问：

```bash
curl http://127.0.0.1:8000/v1/models
```

- 如果请求因认证错误而失败，请设置与服务器配置匹配的真实 `VLLM_API_KEY`，或在 `models.providers.vllm` 下显式配置该 provider。

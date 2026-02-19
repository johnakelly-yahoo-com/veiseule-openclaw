---
summary: "通过 LiteLLM Proxy 运行 OpenClaw，以实现统一的模型访问和成本跟踪"
read_when:
  - 你希望通过 LiteLLM 代理来路由 OpenClaw
  - 你需要通过 LiteLLM 实现成本跟踪、日志记录或模型路由
---

# LiteLLM

[LiteLLM](https://litellm.ai) 是一个开源的 LLM 网关，为 100 多家模型提供商提供统一的 API。 通过 LiteLLM 路由 OpenClaw，以获得集中式成本跟踪、日志记录，以及在无需更改 OpenClaw 配置的情况下切换后端的灵活性。

## 为什么将 LiteLLM 与 OpenClaw 一起使用？

- **成本跟踪** — 精确查看 OpenClaw 在所有模型上的支出
- **模型路由** — 在 Claude、GPT-4、Gemini、Bedrock 之间切换，无需更改配置
- **虚拟密钥** — 为 OpenClaw 创建带有支出限制的密钥
- **日志记录** — 完整的请求/响应日志，便于调试
- **故障回退** — 当主提供商不可用时自动切换

## 快速开始

### 通过引导流程

```bash
openclaw onboard --auth-choice litellm-api-key
```

### 手动设置

1. 启动 LiteLLM 代理：

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. 将 OpenClaw 指向 LiteLLM：

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

就这样。 OpenClaw 现在通过 LiteLLM 进行路由。

## 配置

### 环境变量

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### 配置文件

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

## 虚拟密钥

为 OpenClaw 创建一个带有消费上限的专用密钥：

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

将生成的密钥用作 `LITELLM_API_KEY`。

## 模型路由

LiteLLM 可以将模型请求路由到不同的后端。 在你的 LiteLLM `config.yaml` 中进行配置：

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

OpenClaw 会持续请求 `claude-opus-4-6` —— 由 LiteLLM 负责路由处理。

## 查看使用情况

查看 LiteLLM 的仪表板或 API：

```bash
# 密钥信息
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# 消费日志
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## 注意

- LiteLLM 默认运行在 `http://localhost:4000`
- OpenClaw 通过兼容 OpenAI 的 `/v1/chat/completions` 端点进行连接
- 所有 OpenClaw 功能都可通过 LiteLLM 使用 —— 无任何限制

## 另请参阅

- [LiteLLM Docs](https://docs.litellm.ai)
- [模型提供商](/concepts/model-providers)

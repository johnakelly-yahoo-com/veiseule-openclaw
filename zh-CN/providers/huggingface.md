---
summary: "Hugging Face Inference 设置（认证 + 模型选择）"
read_when:
  - 你希望在 OpenClaw 中使用 Hugging Face Inference
  - 你需要 HF token 环境变量或 CLI 认证选项
title: "Hugging Face（Inference）"
---

# Hugging Face（Inference）

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) 通过单一路由 API 提供与 OpenAI 兼容的聊天补全功能。 使用一个 token 即可访问多个模型（DeepSeek、Llama 等）。 OpenClaw 使用**与 OpenAI 兼容的端点**（仅支持聊天补全）；如需文本生成图像、向量嵌入或语音功能，请直接使用 [HF inference clients](https://huggingface.co/docs/api-inference/quicktour)。

- 提供商：`huggingface`
- 认证：`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`（具有 **Make calls to Inference Providers** 权限的细粒度 token）
- API：与 OpenAI 兼容（`https://router.huggingface.co/v1`）
- 计费：单个 HF token；[pricing](https://huggingface.co/docs/inference-providers/pricing) 按各提供商费率执行，并提供免费额度。

## 快速开始

1. 在 [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) 创建一个细粒度 token，并授予 **Make calls to Inference Providers** 权限。
2. 运行引导流程，在提供商下拉菜单中选择 **Hugging Face**，然后在提示时输入你的 API key：

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. 在 **Default Hugging Face model** 下拉菜单中选择你想使用的模型（当你拥有有效 token 时，列表会从 Inference API 加载；否则会显示内置列表）。 你的选择将被保存为默认模型。
4. 你也可以稍后在配置中设置或更改默认模型：

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## 非交互式示例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

这将把 `huggingface/deepseek-ai/DeepSeek-R1` 设置为默认模型。

## 环境说明

如果 Gateway 以守护进程方式运行（launchd/systemd），请确保 `HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`
对该进程可用（例如，在 `~/.openclaw/.env` 中或通过
`env.shellEnv` 设置）。

## 模型发现与入门下拉菜单

OpenClaw 通过**直接调用 Inference 端点**来发现模型：

```bash
GET https://router.huggingface.co/v1/models
```

（可选：发送 `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` 或 `$HF_TOKEN` 以获取完整列表；部分端点在未授权时仅返回子集。） 响应为 OpenAI 风格的 `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

当你配置 Hugging Face API 密钥（通过入门流程、`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`）后，OpenClaw 会使用此 GET 请求来发现可用的 chat-completion 模型。 在**交互式入门流程**中，输入你的 token 后，你将看到一个**默认 Hugging Face 模型**下拉菜单，其内容来自该列表（如果请求失败，则使用内置目录）。 在运行时（例如 Gateway 启动时），当存在密钥时，OpenClaw 会再次调用 **GET** `https://router.huggingface.co/v1/models` 以刷新目录。 该列表会与内置目录合并（用于补充上下文窗口和成本等元数据）。 如果请求失败或未设置密钥，则仅使用内置目录。

## 模型名称和可编辑选项

- **来自 API 的名称：** 当 API 返回 `name`、`title` 或 `display_name` 时，模型显示名称会从 **GET /v1/models** 中获取；否则将根据模型 id 推导生成（例如 `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”）。
- **覆盖显示名称：** 你可以在配置中为每个模型设置自定义标签，以便在 CLI 和 UI 中按你希望的方式显示：

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Provider / 策略选择：** 在**模型 id** 后附加后缀，以选择路由器如何选择后端：

  - **`:fastest`** — 最高吞吐量（由路由器选择；provider 选择被**锁定**——不提供交互式后端选择器）。
  - **`:cheapest`** — 每输出 token 成本最低（由路由器选择；provider 选择被**锁定**）。
  - **`:provider`** — 强制指定特定后端（例如 `:sambanova`、`:together`）。

  当你选择 **:cheapest** 或 **:fastest**（例如在入门模型下拉菜单中）时，provider 会被锁定：路由器根据成本或速度进行决策，不会显示可选的“偏好特定后端”步骤。 你可以将这些作为独立条目添加到 `models.providers.huggingface.models` 中，或在 `model.primary` 中设置带后缀的模型。 你也可以在 [Inference Provider settings](https://hf.co/settings/inference-providers) 中设置默认顺序（无后缀 = 使用该顺序）。

- **配置合并：** 在合并配置时，会保留 `models.providers.huggingface.models` 中的现有条目（例如在 `models.json` 中）。 因此，你在其中设置的任何自定义 `name`、`alias` 或模型选项都会被保留。

## 模型 ID 和配置示例

模型引用使用 `huggingface/<org>/<model>` 形式（Hub 风格 ID）。 以下列表来自 **GET** `https://router.huggingface.co/v1/models`；你的目录可能包含更多。

**示例 ID（来自 inference 端点）：**

| 模型                                     | Ref（前缀为 `huggingface/`）             |
| -------------------------------------- | ----------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`           |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`         |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                     |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`          |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                    |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`  |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`               |
| GLM 4.7                | `zai-org/GLM-4.7`                   |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`              |

你可以在模型 id 后追加 `:fastest`、`:cheapest` 或 `:provider`（例如 `:together`、`:sambanova`）。 在 [Inference Provider settings](https://hf.co/settings/inference-providers) 中设置默认顺序；完整列表请参见 [Inference Providers](https://huggingface.co/docs/inference-providers) 以及 **GET** `https://router.huggingface.co/v1/models`。

### 完整配置示例

**以 DeepSeek R1 为主模型，Qwen 作为回退：**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**将 Qwen 设为默认，并提供 :cheapest 和 :fastest 变体：**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**带别名的 DeepSeek + Llama + GPT-OSS：**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**使用 :provider 强制指定特定后端：**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**带策略后缀的多个 Qwen 和 DeepSeek 模型：**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```


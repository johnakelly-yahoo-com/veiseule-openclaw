---
summary: "Hugging Face Inference 設定（驗證 + 模型選擇）"
read_when:
  - 你想要在 OpenClaw 中使用 Hugging Face Inference
  - 你需要設定 HF token 環境變數或在 CLI 中選擇驗證方式
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) 透過單一路由 API 提供與 OpenAI 相容的聊天完成服務。 只需一個 token，即可存取多種模型（DeepSeek、Llama 等）。 OpenClaw 使用**OpenAI 相容端點**（僅支援 chat completions）；若需使用文字轉圖片、embeddings 或語音功能，請直接使用 [HF inference clients](https://huggingface.co/docs/api-inference/quicktour)。

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`（需具備 **Make calls to Inference Providers** 權限的細粒度 token）
- API：OpenAI 相容（`https://router.huggingface.co/v1`）
- Billing：單一 HF token；[pricing](https://huggingface.co/docs/inference-providers/pricing) 依各供應商費率計費，並提供免費額度。

## 快速開始

1. 在 [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) 建立一個具備 **Make calls to Inference Providers** 權限的細粒度 token。
2. 執行入門設定，並在 provider 下拉選單中選擇 **Hugging Face**，然後在提示時輸入你的 API key：

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. 在 **Default Hugging Face model** 下拉選單中，選擇你想使用的模型（當你擁有有效 token 時，清單會從 Inference API 載入；否則會顯示內建清單）。 您的選擇已儲存為預設模型。
4. 您也可以稍後在 config 中設定或變更預設模型：

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## 非互動式範例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

這將把 `huggingface/deepseek-ai/DeepSeek-R1` 設為預設模型。

## 環境說明

如果 Gateway 以守護程序（launchd/systemd）方式執行，請確保 `HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`
可供該程序存取（例如在 `~/.openclaw/.env` 中或透過
`env.shellEnv`）。

## 模型探索與 onboarding 下拉選單

OpenClaw 透過**直接呼叫 Inference 端點**來探索模型：

```bash
GET https://router.huggingface.co/v1/models
```

（選用：傳送 `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` 或 `$HF_TOKEN` 以取得完整清單；部分端點在未授權情況下僅回傳子集合。） 回應格式為 OpenAI 風格的 `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

當您設定 Hugging Face API 金鑰（透過 onboarding、`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`）時，OpenClaw 會使用此 GET 請求來探索可用的 chat-completion 模型。 在**互動式 onboarding**期間，當您輸入 token 後，會看到一個**Default Hugging Face model**下拉選單，內容來自該清單（若請求失敗則使用內建目錄）。 在執行階段（例如 Gateway 啟動時），當存在金鑰時，OpenClaw 會再次呼叫 **GET** `https://router.huggingface.co/v1/models` 以重新整理目錄。 該清單會與內建目錄合併（包含如 context window 與成本等中繼資料）。 如果請求失敗或未設定金鑰，則僅使用內建目錄。

## 模型名稱與可編輯選項

- **來自 API 的名稱：** 當 API 回傳 `name`、`title` 或 `display_name` 時，模型顯示名稱會**由 GET /v1/models 填入**；否則會從模型 id 推導（例如 `deepseek-ai/DeepSeek-R1` →「DeepSeek R1」）。
- **覆寫顯示名稱：** 您可以在 config 中為每個模型設定自訂標籤，使其在 CLI 和 UI 中以您期望的方式顯示：

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

- **Provider / policy 選擇：** 在**模型 id**後附加後綴，以選擇 router 如何挑選後端：

  - **`:fastest`** — 最高吞吐量（由 router 選擇；provider 選擇為**鎖定** — 不提供互動式後端選擇器）。
  - **`:cheapest`** — 每個輸出 token 成本最低（由 router 選擇；provider 選擇為**鎖定**）。
  - **`:provider`** — 強制指定特定後端（例如 `:sambanova`、`:together`）。

  當您選擇 **:cheapest** 或 **:fastest**（例如在 onboarding 模型下拉選單中）時，provider 會被鎖定：router 會依成本或速度決定，且不會顯示可選的「偏好特定後端」步驟。 您可以將這些作為獨立條目新增至 `models.providers.huggingface.models`，或在 `model.primary` 中加入後綴。 您也可以在 [Inference Provider settings](https://hf.co/settings/inference-providers) 中設定預設順序（不加後綴 = 使用該順序）。

- **Config 合併：** 在合併 config 時，`models.providers.huggingface.models` 中的既有條目（例如在 `models.json` 中）會被保留。 因此，您在其中設定的任何自訂 `name`、`alias` 或模型選項都會被保留。

## 模型 ID 與設定範例

模型參照使用 `huggingface/<org>/<model>` 格式（Hub 風格 ID）。 以下清單來自 **GET** `https://router.huggingface.co/v1/models`；您的目錄可能包含更多項目。

**範例 ID（來自 inference 端點）：**

| 模型                                     | Ref（請加上 `huggingface/` 前綴）          |
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

你可以在模型 ID 後附加 `:fastest`、`:cheapest` 或 `:provider`（例如 `:together`、`:sambanova`）。 在 [Inference Provider settings](https://hf.co/settings/inference-providers) 中設定你的預設順序；完整清單請參閱 [Inference Providers](https://huggingface.co/docs/inference-providers) 以及 **GET** `https://router.huggingface.co/v1/models`。

### 完整設定範例

**主要使用 DeepSeek R1，並以 Qwen 作為備援：**

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

**以 Qwen 為預設，並提供 :cheapest 與 :fastest 變體：**

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

**使用別名的 DeepSeek + Llama + GPT-OSS：**

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

**使用 :provider 強制指定特定後端：**

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

**多個 Qwen 與 DeepSeek 模型搭配政策後綴：**

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

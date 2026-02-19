---
summary: "使用 vLLM（OpenAI 相容的本機伺服器）執行 OpenClaw"
read_when:
  - 您想讓 OpenClaw 連線至本機的 vLLM 伺服器
  - 您希望使用自己的模型，並透過 OpenAI 相容的 `/v1` 端點
title: "vLLM"
---

# vLLM

vLLM 可透過 **OpenAI 相容** 的 HTTP API 提供開源（以及部分自訂）模型服務。 OpenClaw 可使用 `openai-completions` API 連線至 vLLM。

當您設定 `VLLM_API_KEY`（若伺服器未強制驗證，任意值皆可）且未定義明確的 `models.providers.vllm` 項目時，OpenClaw 也能**自動探索** vLLM 上可用的模型。

## 快速開始

1. 以 OpenAI 相容的伺服器模式啟動 vLLM。

您的 base URL 應提供 `/v1` 端點（例如 `/v1/models`、`/v1/chat/completions`）。 vLLM 常見的執行位址為：

- `http://127.0.0.1:8000/v1`

2. 選擇加入（若未設定驗證，任意值皆可）：

```bash
export VLLM_API_KEY="vllm-local"
```

3. 選擇一個模型（請替換為您的 vLLM 模型 ID 之一）：

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## 模型探索（隱式 provider）

當已設定 `VLLM_API_KEY`（或存在驗證設定檔），且**未**定義 `models.providers.vllm` 時，OpenClaw 將會查詢：

- `GET http://127.0.0.1:8000/v1/models`

…並將回傳的 ID 轉換為模型項目。

如果您明確設定 `models.providers.vllm`，則會略過自動探索，您必須手動定義模型。

## 明確設定（手動模型）

在以下情況使用明確設定：

- vLLM 在不同的主機/連接埠上執行。
- 您想固定 `contextWindow`/`maxTokens` 的數值。
- 您的伺服器需要真實的 API key（或您想要自行控制 headers）。

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

## 疑難排解

- 確認伺服器可連線：

```bash
curl http://127.0.0.1:8000/v1/models
```

- 如果請求因驗證錯誤而失敗，請設定與您的伺服器設定相符的真實 `VLLM_API_KEY`，或在 `models.providers.vllm` 下明確設定 provider。


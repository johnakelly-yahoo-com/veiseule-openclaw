---
title: "本機模型"
---

# 本機模型

本機部署可行，但 OpenClaw 需要大型上下文 + 強大的提示詞注入防禦。小顯卡會截斷上下文並削弱安全性。建議拉高規格：**≥2 台滿配 Mac Studios 或同等 GPU 主機（約 ~$30k+）**。單張 **24 GB** GPU 僅適用於較輕量的提示，且延遲較高。請使用**你能運行的最大／完整尺寸模型變體**；過度量化或「小型」檢查點會提高提示詞注入風險（請參見 [Security](/gateway/security)）。

## 建議：LM Studio + MiniMax M2.1（Responses API，完整尺寸）

Best current local stack. 目前最佳的本機組合。在 LM Studio 中載入 MiniMax M2.1，啟用本機伺服器（預設 `http://127.0.0.1:1234`），並使用 Responses API 以將推理與最終文字分離。

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**設定檢查清單**

- 安裝 LM Studio：<https://lmstudio.ai>
- 在 LM Studio 中下載**可取得的最大 MiniMax M2.1 版本**（避免「small」／高度量化變體），啟動伺服器，並確認 `http://127.0.0.1:1234/v1/models` 有列出該模型。
- 保持模型常駐載入；冷啟動會增加啟動延遲。
- 若你的 LM Studio 版本不同，請調整 `contextWindow`/`maxTokens`。
- WhatsApp 請使用 Responses API，確保只送出最終文字。

即使執行本機模型，也請保留託管模型設定；使用 `models.mode: "merge"` 以確保仍可回退。

### 混合配置：以託管為主，本機為備援

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### 本機優先，託管作為安全備援

對調主要與回退的順序；保留相同的 providers 區塊與 `models.mode: "merge"`，以便在本機主機停機時回退到 Sonnet 或 Opus。

### 區域託管／資料路由

- 託管的 MiniMax/Kimi/GLM 變體也可在 OpenRouter 上取得，並提供區域鎖定端點（例如美國託管）。請在該平台選擇區域變體，以便將流量限制在你選定的司法管轄區內，同時仍使用 `models.mode: "merge"` 作為 Anthropic/OpenAI 的備援。
- 僅本機仍是最強的隱私路徑；當你需要提供者功能但又想控制資料流向時，託管的區域路由是折衷方案。

## 其他 OpenAI 相容的本機代理

只要能提供 OpenAI 風格的 `/v1` 端點，vLLM、LiteLLM、OAI-proxy 或自訂閘道都可使用。請將上方的 provider 區塊替換為你的端點與模型 ID：

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

請保留 `models.mode: "merge"`，讓託管模型能作為回退。

## 疑難排解

- Gateway can reach the proxy? Gateway 能連到代理嗎？`curl http://127.0.0.1:1234/v1/models`。
- LM Studio 模型被卸載？重新載入；冷啟動是常見的「卡住」原因。 Reload; cold start is a common “hanging” cause.
- 上下文錯誤？ 上下文錯誤？降低 `contextWindow` 或提高你的伺服器限制。
- 安全性：本機模型會略過提供者端的過濾；請保持代理程式範圍精簡並開啟壓縮，以限制提示注入的影響半徑。

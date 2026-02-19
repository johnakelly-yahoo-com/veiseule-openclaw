---
summary: "Together AI 設定（驗證 + 模型選擇）"
read_when:
  - 你想在 OpenClaw 中使用 Together AI
  - 你需要設定 API key 環境變數或使用 CLI 驗證選項
---

# Together AI

[Together AI](https://together.ai) 透過統一的 API 提供對多個領先開源模型的存取，包括 Llama、DeepSeek、Kimi 等。

- Provider：`together`
- 驗證：`TOGETHER_API_KEY`
- API：OpenAI 相容

## 快速開始

1. 設定 API key（建議：將其儲存於 Gateway）：

```bash
openclaw onboard --auth-choice together-api-key
```

2. 設定預設模型：

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## 非互動式範例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

這將把 `together/moonshotai/Kimi-K2.5` 設為預設模型。

## 環境說明

如果 Gateway 以守護行程（launchd/systemd）方式執行，請確保 `TOGETHER_API_KEY`
可供該程序使用（例如在 `~/.clawdbot/.env` 中，或透過
`env.shellEnv` 設定）。

## 可用模型

Together AI 提供對多種熱門開源模型的存取：

- **GLM 4.7 Fp8** - 預設模型，具備 200K 上下文視窗
- **Llama 3.3 70B Instruct Turbo** - 快速且高效的指令遵循模型
- **Llama 4 Scout** - 具備影像理解能力的視覺模型
- **Llama 4 Maverick** - 進階視覺與推理模型
- **DeepSeek V3.1** - 強大的程式撰寫與推理模型
- **DeepSeek R1** - 進階推理模型
- **Kimi K2 Instruct** - 高效能模型，具備 262K 上下文視窗

所有模型皆支援標準 chat completions，並與 OpenAI API 相容。


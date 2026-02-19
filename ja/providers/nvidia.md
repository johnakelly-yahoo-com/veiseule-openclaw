---
summary: "OpenClaw で NVIDIA の OpenAI 互換 API を使用する"
read_when:
  - OpenClaw で NVIDIA モデルを使用したい場合
  - NVIDIA_API_KEY の設定が必要です
title: "NVIDIA"
---

# NVIDIA

NVIDIA は、Nemotron および NeMo モデル向けに `https://integrate.api.nvidia.com/v1` で OpenAI 互換 API を提供しています。 [NVIDIA NGC](https://catalog.ngc.nvidia.com/) から取得した API キーで認証します。

## CLI セットアップ

キーを一度エクスポートし、その後オンボーディングを実行して NVIDIA モデルを設定します：

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

`--token` を引き続き使用する場合、その値はシェル履歴や `ps` の出力に残る点に注意してください。可能な限り環境変数の使用を推奨します。

## 設定スニペット

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

## モデルID

- `nvidia/llama-3.1-nemotron-70b-instruct` (デフォルト)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## 注記

- OpenAI互換の`/v1`エンドポイント。NVIDIA NGCのAPIキーを使用します。
- `NVIDIA_API_KEY`が設定されている場合、プロバイダーは自動的に有効化されます。固定のデフォルト値（131,072トークンのコンテキストウィンドウ、最大4,096トークン）を使用します。

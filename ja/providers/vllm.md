---
summary: "vLLM（OpenAI互換ローカルサーバー）でOpenClawを実行する"
read_when:
  - ローカルのvLLMサーバーに対してOpenClawを実行したい場合
  - 独自のモデルでOpenAI互換の/v1エンドポイントを使用したい場合
title: "vLLM"
---

# vLLM

vLLM は **OpenAI 互換** の HTTP API を通じて、オープンソース（および一部のカスタム）モデルを提供できます。 OpenClaw は `openai-completions` API を使用して vLLM に接続できます。

`VLLM_API_KEY` を指定してオプトインし（サーバーで認証を強制していない場合は任意の値で可）、かつ `models.providers.vllm` を明示的に定義していない場合、OpenClaw は vLLM から利用可能なモデルを **自動検出** できます。

## クイックスタート

1. OpenAI 互換サーバーとして vLLM を起動します。

ベース URL は `/v1` エンドポイント（例: `/v1/models`, `/v1/chat/completions`）を公開している必要があります。 vLLM は通常、次の場所で実行されます:

- `http://127.0.0.1:8000/v1`

2. （認証が設定されていない場合は任意の値で）オプトインします:

```bash
export VLLM_API_KEY="vllm-local"
```

3. モデルを選択します（vLLM のモデル ID のいずれかに置き換えてください）:

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## モデル検出（暗黙的プロバイダー）

`VLLM_API_KEY` が設定されている（または認証プロファイルが存在する）状態で、かつ `models.providers.vllm` を**定義していない**場合、OpenClaw は次をクエリします:

- `GET http://127.0.0.1:8000/v1/models`

…そして、返された ID をモデルエントリに変換します。

`models.providers.vllm` を明示的に設定すると、自動検出はスキップされ、モデルを手動で定義する必要があります。

## 明示的な設定（手動モデル）

次の場合は明示的な設定を使用します:

- vLLM が別のホスト／ポートで実行されている場合。
- `contextWindow` や `maxTokens` の値を固定したい場合。
- サーバーが実際の API キーを必要とする場合（またはヘッダーを制御したい場合）。

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

## トラブルシューティング

- サーバーにアクセスできるか確認します:

```bash
curl http://127.0.0.1:8000/v1/models
```

- 認証エラーでリクエストが失敗する場合は、サーバー構成に一致する実際の `VLLM_API_KEY` を設定するか、`models.providers.vllm` の下でプロバイダーを明示的に構成してください。


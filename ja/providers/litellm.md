---
summary: "統一されたモデルアクセスとコスト追跡のために LiteLLM Proxy 経由で OpenClaw を実行する"
read_when:
  - OpenClaw を LiteLLM プロキシ経由でルーティングしたい場合
  - LiteLLM 経由でコスト追跡、ログ記録、またはモデルルーティングが必要な場合
---

# LiteLLM

[LiteLLM](https://litellm.ai) は、100以上のモデルプロバイダーに対して統一APIを提供するオープンソースのLLMゲートウェイです。 OpenClaw を LiteLLM 経由でルーティングすることで、コストの一元管理、ログ記録、そして OpenClaw の設定を変更することなくバックエンドを切り替える柔軟性を得られます。

## なぜ OpenClaw で LiteLLM を使用するのですか？

- **コスト追跡** — すべてのモデルにわたる OpenClaw の利用コストを正確に把握できます
- **モデルルーティング** — 設定を変更せずに Claude、GPT-4、Gemini、Bedrock を切り替え可能
- **仮想キー** — OpenClaw 用に利用上限付きキーを作成可能
- **ログ記録** — デバッグのための完全なリクエスト／レスポンスログ
- **フォールバック** — プライマリプロバイダーがダウンした場合の自動フェイルオーバー

## クイックスタート

### オンボーディング経由

```bash
openclaw onboard --auth-choice litellm-api-key
```

### 手動セットアップ

1. LiteLLM Proxy を起動します:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. OpenClaw を LiteLLM に向けます:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

以上です。 OpenClaw はこれで LiteLLM 経由でルーティングされます。

## 設定

### 環境変数

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### 設定ファイル

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

## 仮想キー

OpenClaw専用のキーを、利用上限付きで作成します：

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

生成されたキーを `LITELLM_API_KEY` として使用します。

## モデルルーティング

LiteLLMは、モデルリクエストを異なるバックエンドにルーティングできます。 LiteLLM の `config.yaml` で設定します：

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

OpenClaw は常に `claude-opus-4-6` をリクエストし、ルーティングは LiteLLM が処理します。

## 使用状況の確認

LiteLLM のダッシュボードまたは API を確認します：

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## 注意事項

- LiteLLM はデフォルトで `http://localhost:4000` で実行されます
- OpenClaw は OpenAI 互換の `/v1/chat/completions` エンドポイント経由で接続します
- OpenClaw のすべての機能は LiteLLM 経由で利用可能で、制限はありません

## 関連項目

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)

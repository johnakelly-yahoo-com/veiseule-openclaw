---
summary: "Hugging Face Inference のセットアップ（認証 + モデル選択）"
read_when:
  - OpenClaw で Hugging Face Inference を使用したい場合
  - HF トークンの環境変数、または CLI での認証選択が必要です
title: "Hugging Face（Inference）"
---

# Hugging Face（Inference）

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) は、単一のルーター API を通じて OpenAI 互換のチャット補完を提供します。 1 つのトークンで多くのモデル（DeepSeek、Llama など）にアクセスできます。 OpenClaw は **OpenAI 互換エンドポイント**（チャット補完のみ）を使用します。text-to-image、embeddings、speech については、[HF inference clients](https://huggingface.co/docs/api-inference/quicktour) を直接使用してください。

- プロバイダー: `huggingface`
- 認証: `HUGGINGFACE_HUB_TOKEN` または `HF_TOKEN`（**Make calls to Inference Providers** 権限を持つファイングレインドトークン）
- API: OpenAI 互換（`https://router.huggingface.co/v1`）
- 課金: 単一の HF トークン; [pricing](https://huggingface.co/docs/inference-providers/pricing) は各プロバイダーの料金に従い、無料枠があります。

## クイックスタート

1. [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) で、**Make calls to Inference Providers** 権限を持つファイングレインドトークンを作成します。
2. オンボーディングを実行し、プロバイダーのドロップダウンで **Hugging Face** を選択し、プロンプトが表示されたら API キーを入力します:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. **Default Hugging Face model** ドロップダウンで使用したいモデルを選択します（有効なトークンがある場合は Inference API から一覧が読み込まれ、ない場合は組み込みリストが表示されます）。 選択内容はデフォルトモデルとして保存されます。
4. 後から config でデフォルトモデルを設定または変更することもできます:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## 非対話型の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

これにより、`huggingface/deepseek-ai/DeepSeek-R1` がデフォルトモデルとして設定されます。

## 環境に関する注意

Gateway がデーモン（launchd/systemd）として実行されている場合は、`HUGGINGFACE_HUB_TOKEN` または `HF_TOKEN` が
そのプロセスから利用可能であることを確認してください（例: `~/.openclaw/.env` または
`env.shellEnv` 経由）。

## モデル検出とオンボーディングのドロップダウン

OpenClaw は **Inference エンドポイントを直接呼び出す** ことでモデルを検出します:

```bash
GET https://router.huggingface.co/v1/models
```

（任意: 完全な一覧を取得するには `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` または `$HF_TOKEN` を送信します。一部のエンドポイントは認証なしではサブセットのみを返します。） レスポンスは OpenAI 形式の `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Hugging Face API キー（オンボーディング、`HUGGINGFACE_HUB_TOKEN`、または `HF_TOKEN`）を設定すると、OpenClaw はこの GET を使用して利用可能なチャット補完モデルを検出します。 **対話型オンボーディング** では、トークンを入力した後、一覧から取得した（またはリクエストが失敗した場合は組み込みカタログからの）内容が表示される **Default Hugging Face model** ドロップダウンが表示されます。 実行時（例: Gateway 起動時）にキーが存在する場合、OpenClaw は再度 **GET** `https://router.huggingface.co/v1/models` を呼び出してカタログを更新します。 この一覧は、組み込みカタログ（コンテキストウィンドウやコストなどのメタデータ用）とマージされます。 リクエストが失敗した場合、またはキーが設定されていない場合は、組み込みカタログのみが使用されます。

## モデル名と編集可能なオプション

- **API からの名前:** モデルの表示名は、API が `name`、`title`、または `display_name` を返す場合、**GET /v1/models から取得して反映されます**。それ以外の場合は、モデル id から生成されます（例: `deepseek-ai/DeepSeek-R1` → 「DeepSeek R1」）。
- **表示名の上書き:** 設定でモデルごとにカスタムラベルを指定すると、CLI や UI に希望どおりの名前で表示できます。

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

- **プロバイダー / ポリシーの選択:** **model id** にサフィックスを追加して、ルーターがバックエンドを選択する方法を指定します。

  - **`:fastest`** — 最高スループット（ルーターが選択。プロバイダー選択は **固定** され、対話式バックエンド選択はできません）。
  - **`:cheapest`** — 出力トークンあたりのコストが最小（ルーターが選択。プロバイダー選択は **固定** されます）。
  - **`:provider`** — 特定のバックエンドを強制指定します（例: `:sambanova`, `:together`）。

  **:cheapest** または **:fastest** を選択した場合（例: オンボーディングのモデル選択ドロップダウン）、プロバイダーは固定されます。ルーターがコストまたは速度に基づいて決定し、任意の「特定のバックエンドを優先する」ステップは表示されません。 これらは `models.providers.huggingface.models` に個別のエントリとして追加するか、サフィックス付きで `model.primary` を設定できます。 [Inference Provider settings](https://hf.co/settings/inference-providers) でデフォルトの順序を設定することもできます（サフィックスなし = その順序を使用）。

- **設定のマージ:** 設定をマージしても、`models.providers.huggingface.models`（例: `models.json` 内）の既存エントリは保持されます。 そのため、そこで設定したカスタムの `name`、`alias`、またはモデルオプションは維持されます。

## モデル ID と設定例

モデル参照は `huggingface/<org>/<model>`（Hub 形式の ID）を使用します。 以下のリストは **GET** `https://router.huggingface.co/v1/models` から取得したものです。お使いのカタログにはさらに多くのモデルが含まれる場合があります。

**例 ID（inference エンドポイントより）:**

| モデル                                    | 参照（`huggingface/` をプレフィックスとして付与）    |
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

モデルIDに `:fastest`、`:cheapest`、または `:provider`（例：`:together`、`:sambanova`）を追加できます。 デフォルトの順序は [Inference Provider settings](https://hf.co/settings/inference-providers) で設定してください。完全な一覧は [Inference Providers](https://huggingface.co/docs/inference-providers) および **GET** `https://router.huggingface.co/v1/models` を参照してください。

### 完全な設定例

**Primary DeepSeek R1 と Qwen フォールバック:**

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

**Qwen をデフォルトにし、`:cheapest` と `:fastest` バリアントを使用:**

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

**エイリアス付きの DeepSeek + Llama + GPT-OSS:**

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

**`:provider` で特定のバックエンドを強制指定:**

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

**ポリシーサフィックス付きの複数の Qwen および DeepSeek モデル:**

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

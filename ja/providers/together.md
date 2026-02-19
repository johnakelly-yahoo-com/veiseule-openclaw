---
summary: "Together AI セットアップ（認証 + モデル選択）"
read_when:
  - OpenClawでTogether AIを使用したい場合
  - APIキーの環境変数、またはCLIでの認証選択が必要です
---

# Together AI

[Together AI](https://together.ai)は、Llama、DeepSeek、Kimiなどの主要なオープンソースモデルへ統一APIを通じてアクセスを提供します。

- プロバイダー: `together`
- 認証: `TOGETHER_API_KEY`
- API: OpenAI互換

## クイックスタート

1. APIキーを設定します（推奨: Gateway用に保存してください）：

```bash
openclaw onboard --auth-choice together-api-key
```

2. デフォルトモデルを設定します：

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## 非対話型の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

これにより、`together/moonshotai/Kimi-K2.5`がデフォルトモデルとして設定されます。

## 環境に関する注意

Gatewayがデーモン（launchd/systemd）として実行されている場合は、`TOGETHER_API_KEY`が
そのプロセスから利用可能であることを確認してください（例：`~/.clawdbot/.env`内、または
`env.shellEnv`経由）。

## 利用可能なモデル

Together AIは、多くの人気オープンソースモデルへのアクセスを提供します：

- **GLM 4.7 Fp8** - 200Kコンテキストウィンドウを持つデフォルトモデル
- **Llama 3.3 70B Instruct Turbo** - 高速で効率的な命令追従
- **Llama 4 Scout** - 画像理解に対応したビジョンモデル
- **Llama 4 Maverick** - 高度なビジョンおよび推論
- **DeepSeek V3.1** - 強力なコーディングおよび推論モデル
- **DeepSeek R1** - 高度な推論モデル
- **Kimi K2 Instruct** - 262Kコンテキストウィンドウを備えた高性能モデル

すべてのモデルは標準のチャット補完をサポートし、OpenAI APIと互換性があります。

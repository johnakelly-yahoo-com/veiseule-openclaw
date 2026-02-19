---
summary: "CLI 参照用ドキュメント：`openclaw onboard`（対話型オンボーディング ウィザード）"
read_when:
  - ゲートウェイ、ワークスペース、認証、チャンネル、Skills のガイド付きセットアップを行いたい場合
title: "オンボード"
---

# `openclaw onboard`

対話型オンボーディング ウィザード（ローカルまたはリモートの Gateway セットアップ）。

## 関連ガイド

- CLI オンボーディング ハブ： [Onboarding Wizard (CLI)](/start/wizard)
- オンボーディング概要: [Onboarding Overview](/start/onboarding-overview)
- CLI 自動化： [CLI Automation](/start/wizard-cli-automation)
- CLI オンボーディング参照： [CLI Onboarding Reference](/start/wizard-cli-reference)
- macOS オンボーディング： [Onboarding (macOS App)](/start/onboarding)

## Examples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

非対話型カスタムプロバイダー:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` は非対話モードでは任意です。 省略した場合、オンボーディングは `CUSTOM_API_KEY` を確認します。

非対話型 Z.AI エンドポイントの選択肢:

注: `--auth-choice zai-api-key` はキーに最適な Z.AI エンドポイントを自動検出するようになりました（`zai/glm-5` を使用する一般 API を優先）。
GLM Coding Plan エンドポイントを特に使用したい場合は、`zai-coding-global` または `zai-coding-cn` を選択してください。

```bash
# プロンプトなしのエンドポイント選択
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# その他の Z.AI エンドポイント選択肢:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

フローに関する注記：

- `quickstart`：最小限のプロンプトで、ゲートウェイ トークンを自動生成します。
- `manual`：ポート／バインド／認証の完全なプロンプト（`advanced` のエイリアス）。
- 最速で最初のチャット： `openclaw dashboard`（コントロール UI、チャンネル設定なし）。
- カスタムプロバイダー: 任意の OpenAI または Anthropic 互換エンドポイント（
  未掲載のホスト型プロバイダーを含む）に接続できます。 Unknown を使用すると自動検出します。

## Common follow-up commands

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` は非対話モードを意味しません。スクリプトでは `--non-interactive` を使用してください。
 スクリプトには `--non-interactive` を使用します。
</Note> スクリプトには `--non-interactive` を使用します。
</Note>

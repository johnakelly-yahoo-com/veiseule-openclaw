---
summary: "OpenClaw のオンボーディングオプションとフローの概要"
read_when:
  - オンボーディングパスの選択
  - 新しい環境のセットアップ
title: "オンボーディング概要"
sidebarTitle: "オンボーディング概要"
---

# オンボーディング概要

OpenClaw は、Gateway の実行場所やプロバイダーの設定方法の好みに応じて、複数のオンボーディングパスをサポートしています。

## オンボーディングパスを選択

- macOS、Linux、Windows（WSL2 経由）向けの **CLI ウィザード**。
- Apple silicon または Intel Mac でガイド付きの初回実行を行うための **macOS アプリ**。

## CLI オンボーディングウィザード

ターミナルでウィザードを実行します:

```bash
openclaw onboard
```

Gateway、workspace、channels、skills を完全に制御したい場合は、CLI ウィザードを使用してください。 ドキュメント:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS アプリのオンボーディング

macOS で完全にガイド付きのセットアップを行いたい場合は、OpenClaw アプリを使用してください。 ドキュメント:

- [Onboarding (macOS App)](/start/onboarding)

## カスタムプロバイダー

一覧にないエンドポイントが必要な場合（標準の OpenAI または Anthropic API を公開しているホスト型プロバイダーを含む）は、CLI ウィザードで **Custom Provider** を選択してください。 次の内容を求められます:

- OpenAI 互換、Anthropic 互換、または **Unknown**（自動検出）を選択します。
- ベース URL と API キー（プロバイダーで必要な場合）を入力します。
- モデル ID と任意のエイリアスを指定します。
- 複数のカスタムエンドポイントを共存させるために Endpoint ID を選択します。

詳細な手順については、上記の CLI オンボーディングドキュメントを参照してください。

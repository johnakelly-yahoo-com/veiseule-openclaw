# AGENTS.md - ja-JP ドキュメント翻訳ワークスペース

## 読むタイミング

- `docs/ja-JP/**` のメンテナンス時
- 日本語翻訳パイプライン（用語集／TM／プロンプト）の更新時
- 日本語翻訳に関するフィードバックやリグレッション対応時

## パイプライン（docs-i18n）

- ソースドキュメント: `docs/**/*.md`
- ターゲットドキュメント: `docs/ja-JP/**/*.md`
- 用語集: `docs/.i18n/glossary.ja-JP.json`
- 翻訳メモリ: `docs/.i18n/ja-JP.tm.jsonl`
- プロンプト規則: `scripts/docs-i18n/prompt.go`

よく使う実行例:

```bash
# 一括（docモード；並列可）
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# 単一ファイル
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# 小規模パッチ（segmentモード；TMを使用；並列なし）
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

注意:

- ページ全体の翻訳には`doc`モードを、小さな修正には`segment`モードを使用してください。
- 非常に大きなファイルでタイムアウトする場合は、対象箇所のみ編集するか、ページを分割してから再実行してください。
- 翻訳後に確認：コードスパン/ブロックが変更されていないこと、リンク/アンカーが変更されていないこと、プレースホルダーが保持されていること。

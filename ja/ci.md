---
title: CIパイプライン
description: OpenClawのCIパイプラインの仕組み
---

# CIパイプライン

CIは`main`へのすべてのpushおよびすべてのpull requestで実行されます。 ドキュメントやネイティブコードのみが変更された場合、高コストなジョブをスキップするスマートスコープを使用します。

## ジョブ概要

| ジョブ               | 目的                                 | 実行されるタイミング           |
| ----------------- | ---------------------------------- | -------------------- |
| `docs-scope`      | ドキュメントのみの変更を検出                     | 常に実行                 |
| `changed-scope`   | どの領域が変更されたかを検出（node/macos/android） | ドキュメント以外のPR          |
| `check`           | TypeScriptの型チェック、lint、フォーマット       | ドキュメント以外の変更          |
| `check-docs`      | Markdownのlint + リンク切れチェック          | ドキュメントが変更された場合       |
| `code-analysis`   | LOCしきい値チェック（1000行）                 | PRのみ                 |
| `secrets`         | 漏洩したシークレットを検出                      | 常に実行                 |
| `build-artifacts` | distを一度ビルドし、他のジョブと共有               | ドキュメント以外かつnodeの変更    |
| `release-check`   | npm packの内容を検証                     | ビルド後                 |
| `checks`          | Node/Bunテスト + プロトコルチェック            | ドキュメント以外かつnodeの変更    |
| `checks-windows`  | Windows固有のテスト                      | ドキュメント以外かつnodeの変更    |
| `macos`           | Swiftのlint／ビルド／テスト + TSテスト         | macosに変更があるPR        |
| `android`         | Gradle ビルド + テスト                   | ドキュメント以外の変更（android） |

## フェイルファスト順序

高コストなジョブが実行される前に、低コストのチェックが失敗するようにジョブが順序付けされています：

1. `docs-scope` + `code-analysis` + `check`（並列、約1〜2分）
2. `build-artifacts`（上記完了後に実行）
3. `checks`、`checks-windows`、`macos`、`android`（build 完了後に実行）

## ランナー

| ランナー                            | ジョブ              |
| ------------------------------- | ---------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | ほとんどの Linux ジョブ  |
| `blacksmith-4vcpu-windows-2025` | `checks-windows` |
| `macos-latest`                  | `macos`、`ios`    |
| `ubuntu-latest`                 | スコープ検出（軽量）       |

## ローカルでの同等コマンド

```bash
pnpm check          # 型チェック + lint + フォーマット
pnpm test           # vitest テスト
pnpm check:docs     # ドキュメントのフォーマット + lint + リンク切れチェック
pnpm release:check  # npm pack の検証
```

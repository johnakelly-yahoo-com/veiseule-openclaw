---
title: CI 流程
description: OpenClaw CI 流程的運作方式
---

# CI 流程

CI 會在每次 push 到 `main` 以及每次 pull request 時執行。 它使用智慧範圍判定，當僅變更文件或原生程式碼時，會略過高成本的工作。

## 工作總覽

| 工作                | 用途                            | 執行時機            |
| ----------------- | ----------------------------- | --------------- |
| `docs-scope`      | 偵測是否僅變更文件                     | 永遠執行            |
| `changed-scope`   | 偵測哪些區域被變更（node/macos/android） | 非文件類型的 PR       |
| `check`           | TypeScript 型別、lint、格式檢查       | 非文件變更           |
| `check-docs`      | Markdown lint + 失效連結檢查        | 文件有變更時          |
| `code-analysis`   | 程式碼行數門檻檢查（1000 行）             | 僅限 PR           |
| `secrets`         | 偵測外洩的機密資訊                     | 永遠執行            |
| `build-artifacts` | 建置 dist 一次，並與其他工作共享           | 非文件、node 相關變更   |
| `release-check`   | 驗證 npm pack 內容                | 建置完成後           |
| `checks`          | Node/Bun 測試 + 協定檢查            | 非文件、node 相關變更   |
| `checks-windows`  | Windows 專屬測試                  | 非文件、node 相關變更   |
| `macos`           | Swift lint／建置／測試 + TS 測試      | 包含 macos 變更的 PR |
| `android`         | Gradle 建置 + 測試                | 非文件、android 變更  |

## 快速失敗順序

Jobs 依序排列，讓成本低的檢查在高成本任務執行前先失敗：

1. `docs-scope` + `code-analysis` + `check`（平行執行，約 1–2 分鐘）
2. `build-artifacts`（需等待上述完成）
3. `checks`、`checks-windows`、`macos`、`android`（需等待 build 完成）

## Runners

| Runner                          | Jobs             |
| ------------------------------- | ---------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | 大多數 Linux jobs   |
| `blacksmith-4vcpu-windows-2025` | `checks-windows` |
| `macos-latest`                  | `macos`、`ios`    |
| `ubuntu-latest`                 | 範圍偵測（輕量級）        |

## 本機對應指令

```bash
pnpm check          # 型別檢查 + lint + 格式化
pnpm test           # vitest 測試
pnpm check:docs     # 文件格式 + lint + 失效連結檢查
pnpm release:check  # 驗證 npm pack
```

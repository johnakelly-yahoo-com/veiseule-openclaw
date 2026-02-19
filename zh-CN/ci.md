---
title: CI 流水线
description: OpenClaw CI 流水线的工作原理
---

# CI 流水线

CI 会在每次向 `main` 分支推送以及每个拉取请求时运行。 它使用智能范围检测，当仅更改文档或原生代码时会跳过高开销的任务。

## 任务概览

| 任务                | 用途                              | 运行时机            |
| ----------------- | ------------------------------- | --------------- |
| `docs-scope`      | 检测仅文档变更                         | 始终运行            |
| `changed-scope`   | 检测哪些区域发生了变更（node/macos/android） | 非文档类 PR         |
| `check`           | TypeScript 类型检查、lint、格式化        | 非文档变更           |
| `check-docs`      | Markdown lint + 失效链接检查          | 文档发生变更          |
| `code-analysis`   | 代码行数阈值检查（1000 行）                | 仅 PR            |
| `secrets`         | 检测泄露的密钥                         | 始终运行            |
| `build-artifacts` | 构建 dist 一次，并与其他任务共享             | 非文档、node 变更     |
| `release-check`   | 验证 npm pack 内容                  | 构建完成后           |
| `checks`          | Node/Bun 测试 + 协议检查              | 非文档、node 变更     |
| `checks-windows`  | Windows 特定测试                    | 非文档、node 变更     |
| `macos`           | Swift lint/构建/测试 + TS 测试        | 包含 macos 变更的 PR |
| `android`         | Gradle 构建 + 测试                  | 非文档、android 变更  |

## 快速失败顺序

任务按顺序排列，使低成本检查在高成本任务运行前先失败：

1. `docs-scope` + `code-analysis` + `check`（并行，约 1-2 分钟）
2. `build-artifacts`（依赖上述任务）
3. `checks`、`checks-windows`、`macos`、`android`（依赖 build）

## 运行器

| 运行器                             | 任务               |
| ------------------------------- | ---------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | 大多数 Linux 任务     |
| `blacksmith-4vcpu-windows-2025` | `checks-windows` |
| `macos-latest`                  | `macos`、`ios`    |
| `ubuntu-latest`                 | 范围检测（轻量级）        |

## 本地等效命令

```bash
pnpm check          # 类型检查 + lint + 格式检查
pnpm test           # vitest 测试
pnpm check:docs     # 文档格式 + lint + 失效链接检查
pnpm release:check  # 验证 npm pack
```


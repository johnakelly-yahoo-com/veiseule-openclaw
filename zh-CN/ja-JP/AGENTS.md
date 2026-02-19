# AGENTS.md - ja-JP 文档翻译工作区

## 在以下情况下阅读

- 维护 `docs/ja-JP/**`
- 更新日文翻译流水线（术语表/TM/提示词）
- 处理日文翻译反馈或回归问题

## 流水线（docs-i18n）

- 源文档：`docs/**/*.md`
- 目标文档：`docs/ja-JP/**/*.md`
- 术语表：`docs/.i18n/glossary.ja-JP.json`
- 翻译记忆库：`docs/.i18n/ja-JP.tm.jsonl`
- 提示规则：`scripts/docs-i18n/prompt.go`

常用运行方式：

```bash
# Bulk (doc mode; parallel OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Single file
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Small patches (segment mode; uses TM; no parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

注意：

- 整页翻译优先使用 `doc` 模式；小范围修正使用 `segment` 模式。
- 如果非常大的文件发生超时，请进行有针对性的编辑或先拆分页面后再重新运行。
- 翻译完成后，请抽查：代码内联/代码块未被更改，链接/锚点未被更改，占位符已保留。

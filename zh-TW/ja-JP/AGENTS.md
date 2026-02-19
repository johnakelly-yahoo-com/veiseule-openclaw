# AGENTS.md - ja-JP 文件翻譯工作區

## 閱讀時機

- 維護 `docs/ja-JP/**`
- 更新日文翻譯流程（術語表/TM/prompt）
- 處理日文翻譯回饋或回歸問題

## 流程（docs-i18n）

- 來源文件：`docs/**/*.md`
- 目標文件：`docs/ja-JP/**/*.md`
- 術語表：`docs/.i18n/glossary.ja-JP.json`
- 翻譯記憶：`docs/.i18n/ja-JP.tm.jsonl`
- Prompt 規則：`scripts/docs-i18n/prompt.go`

常用執行方式：

```bash
# Bulk（文件模式；可並行）
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# 單一檔案
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# 小幅修補（segment 模式；使用 TM；不並行）
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

注意事項：

- 整頁翻譯請優先使用 `doc` 模式；小幅修正請使用 `segment` 模式。
- 若非常大的檔案發生逾時，請改為針對性編輯或先拆分頁面後再重新執行。
- 翻譯完成後請抽樣檢查：程式碼行內區段/區塊未變更、連結/錨點未變更、佔位符已保留。

---
title: "使用量追蹤"
---

# 使用量追蹤

## 它是什麼

- 直接從供應商的使用量端點拉取使用量／配額資料。
- 不提供估算成本；僅顯示供應商回報的視窗資料。

## 顯示位置

- 聊天中的 `/status`：包含豐富表情符號的狀態卡片，顯示工作階段權杖數與估算成本（僅限 API key）。在可用時，會顯示**目前模型供應商**的使用量。
- 聊天中：`/usage off|tokens|full`：每則回應的使用量頁尾（OAuth 僅顯示權杖）。
- 聊天中：`/usage cost`：由 OpenClaw 工作階段紀錄彙總的本地成本摘要。
- CLI：`openclaw status --usage` 會輸出完整的各提供者明細。
- CLI：`openclaw channels list` 會在提供者設定旁輸出相同的使用量快照（使用 `--no-usage` 可略過）。
- macOS 功能表列：Context 底下的「Usage」區段（僅在可用時顯示）。

## 提供者＋憑證

- **Anthropic（Claude）**：驗證設定檔中的 OAuth 權杖。
- **GitHub Copilot**：驗證設定檔中的 OAuth 權杖。
- **Gemini CLI**：驗證設定檔中的 OAuth 權杖。
- **Antigravity**：驗證設定檔中的 OAuth 權杖。
- **OpenAI Codex**：驗證設定檔中的 OAuth 權杖（存在時使用 accountId）。
- **MiniMax**：API 金鑰（程式設計方案金鑰；`MINIMAX_CODE_PLAN_KEY` 或 `MINIMAX_API_KEY`）；使用 5 小時的程式設計方案視窗。
- **z.ai**：透過 環境變數／設定／驗證儲存庫 提供的 API 金鑰。

若不存在相符的 OAuth／API 憑證，將隱藏使用量。



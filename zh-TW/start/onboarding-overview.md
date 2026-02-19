---
summary: "OpenClaw 入門選項與流程總覽"
read_when:
  - 選擇入門路徑
  - 設定新環境
title: "入門總覽"
sidebarTitle: "入門總覽"
---

# 入門總覽

OpenClaw 支援多種入門路徑，取決於 Gateway 的執行位置
以及您偏好的 provider 設定方式。

## 選擇您的入門路徑

- 適用於 macOS、Linux 與 Windows（透過 WSL2）的 **CLI 精靈**。
- 適用於 Apple silicon 或 Intel Mac 的 **macOS app**，提供引導式首次執行體驗。

## CLI 入門精靈

在終端機中執行精靈：

```bash
openclaw onboard
```

當你希望完整掌控 Gateway、workspace、channels 與 skills 時，請使用 CLI 精靈。 文件：

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS app 入門設定

當你希望在 macOS 上進行完整引導式設定時，請使用 OpenClaw app。 文件：

- [Onboarding (macOS App)](/start/onboarding)

## 自訂 Provider

如果你需要的端點未列於清單中，包括提供標準 OpenAI 或 Anthropic API 的託管服務，請在 CLI 精靈中選擇 **Custom Provider**。 系統將要求你：

- 選擇 OpenAI-compatible、Anthropic-compatible，或 **Unknown**（自動偵測）。
- 輸入 base URL 與 API key（若該 provider 需要）。
- 提供 model ID 以及可選的別名。
- 選擇一個 Endpoint ID，以便多個自訂端點可以共存。

詳細步驟請參考上方的 CLI 入門文件。


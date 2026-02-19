---
read_when:
  - 執行或設定入門精靈時
  - 設定新機器時
sidebarTitle: Wizard (CLI)
summary: CLI 入門精靈：互動式設定 Gateway、工作區、頻道與 Skills
title: 入門精靈（CLI）
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# 入門精靈（CLI）

CLI 入門精靈是在 macOS、Linux、Windows（透過 WSL2）上設定 OpenClaw 的建議方式。除了本機 Gateway 或遠端 Gateway 連線外，還會設定工作區的預設值、頻道與 Skills。

```bash
openclaw onboard
```

<Info>
最快開始第一次聊天的方法：開啟 Control UI（無需設定頻道）。`openclaw dashboard`，即可在瀏覽器中聊天。文件：[Dashboard](/web/dashboard)。
</Info>

## 快速開始 vs 詳細設定

精靈會先讓您選擇**快速開始**（預設設定）或**詳細設定**（完全控制）。

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - loopback 上的本機 Gateway
    - 現有工作區或預設工作區
    - Gateway 連接埠 `18789`
    - Gateway 驗證權杖會自動產生（即使在 loopback 上也會產生）
    - Tailscale 公開為關閉
    - Telegram 與 WhatsApp 的 DM 預設為允許清單（可能會要求輸入電話號碼）
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - 顯示模式、工作區、Gateway、頻道、常駐程式、Skills 的完整提示流程
  
</Tab>
</Tabs>

## CLI 入門的詳細資訊

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    本機與遠端流程的完整說明、驗證與模型矩陣、設定輸出、精靈 RPC、signal-cli 的運作方式。
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    非互動式入門的範例與自動化 `agents add` 的示例。
  
</Card>
</Columns>

## 常用的後續指令

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` 並不代表非互動模式。在腳本中請使用 `--non-interactive`。
</Note>

<Tip>
建議：設定 Brave Search API 金鑰，讓代理可以使用 `web_search`（`web_fetch` 無需金鑰即可運作）。最簡單的方法：執行 `openclaw configure --section web`，即可儲存 `tools.web.search.apiKey`。文件：[Web工具](/tools/web)。
</Tip>

## 相關文件

- CLI 指令參考：[`openclaw onboard`](/cli/onboard)
- macOS 應用程式的入門導引：[入門導引](/start/onboarding)
- 代理程式首次啟動步驟：[代理程式啟動流程](/start/bootstrapping)

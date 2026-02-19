---
read_when:
  - 從零開始的首次設定
  - 想了解通往可用聊天功能的最短路徑
summary: 安裝 OpenClaw，並在幾分鐘內執行您的第一個聊天。
title: 開始使用
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# 開始使用

目標：從零開始，以最少設定完成第一個可運作的聊天。

<Info>
最快的聊天方式：開啟 Control UI（無需設定頻道）。執行 `openclaw dashboard` 並在瀏覽器中聊天，或在
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway 主機</Tooltip>中開啟 `http://127.0.0.1:18789/`。
文件：[Dashboard](/web/dashboard) 與 [Control UI](/web/control-ui)。
</Info>

## 前置條件

- Node 22 或更新版本

<Tip>
若不確定，請使用 `node --version` 檢查您的 Node 版本。
</Tip>

## 快速設定（CLI）

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    其他安裝方式與需求：[安裝](/install)。
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    精靈將設定驗證、Gateway 設定，以及可選的頻道。
    詳情請參閱[入門精靈](/start/wizard)。
    ```

  
</Step>
  <Step title="Gatewayを確認">
    如果您已安裝服務，應該已經在執行中：

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Control UI 載入後，表示 Gateway 已可使用。
</Check>

## 可選的驗證與附加功能

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    方便進行快速測試與疑難排解。


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    需要已設定的頻道。


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## 進一步了解

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    完整的 CLI 精靈參考與進階選項。
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    macOS 應用程式的首次執行流程。
  
</Card>
</Columns>

## 完成後的狀態

- 執行中的 Gateway
- 已設定的驗證
- 可存取 Control UI 或已連線的頻道

## 下一步

- DM 的安全性與核准：[配對](/channels/pairing)
- 連接更多頻道：[頻道](/channels)
- 進階工作流程與從原始碼建置：[設定](/start/setup)

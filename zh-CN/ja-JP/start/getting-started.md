---
read_when:
  - 从零开始的首次设置
  - 想了解通往可用聊天的最短路径
summary: 安装 OpenClaw，并在几分钟内运行你的第一个聊天。
title: 开始使用
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# 入门

目标：从零开始，通过最小化设置实现第一个可运行的聊天。

<Info>
最快的聊天方式：打开 Control UI（无需配置通道）。运行 `openclaw dashboard` 在浏览器中聊天，或在
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway 主机</Tooltip>中打开 `http://127.0.0.1:18789/`。
文档：[Dashboard](/web/dashboard) 和 [Control UI](/web/control-ui)。
</Info>

## 前提条件

- Node 22 及以上

<Tip>
如果不确定，请使用 `node --version` 检查 Node 版本。
</Tip>

## 快速设置（CLI）

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
    其他安装方法和要求：[安装](/install)。
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    向导将配置身份验证、Gateway 设置以及可选的通道。
    详情请参阅[入门向导](/start/wizard)。
    ```

  
</Step>
  <Step title="Gatewayを確認">
    如果已安装服务，它应该已经在运行：

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
如果 Control UI 加载成功，则 Gateway 已可使用。
</Check>

## 可选检查与附加功能

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    适用于快速测试和故障排查。


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    需要已配置的通道。


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## 进一步了解

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    完整的 CLI 向导参考和高级选项。
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    macOS 应用的首次运行流程。
  
</Card>
</Columns>

## 完成后的状态

- 运行中的 Gateway
- 已配置的身份验证
- Control UI 访问或已连接的通道

## 下一步

- DM 的安全性与授权：[配对](/channels/pairing)
- 连接更多频道：[频道](/channels)
- 高级工作流与从源码构建：[设置](/start/setup)

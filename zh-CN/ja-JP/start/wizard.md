---
read_when:
  - 运行或配置入门向导时
  - 设置新机器时
sidebarTitle: Wizard (CLI)
summary: CLI 入门向导：Gateway、工作区、频道、Skills 的交互式设置
title: 入门向导（CLI）
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# 入门向导（CLI）

CLI 入门向导是在 macOS、Linux 和 Windows（通过 WSL2）上设置 OpenClaw 的推荐方式。它会配置本地或远程 Gateway 连接，以及工作区默认设置、频道和 Skills。

```bash
openclaw onboard
```

<Info>
最快开始首次聊天的方法：打开 Control UI（无需配置频道）。运行 `openclaw dashboard` 并在浏览器中聊天。文档：[Dashboard](/web/dashboard)。
</Info>

## 快速开始 vs 详细设置

向导启动时可选择**快速开始**（默认设置）或**详细设置**（完全控制）。

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - loopback 上的本地 Gateway
    - 现有工作区或默认工作区
    - Gateway 端口 `18789`
    - 自动生成 Gateway 认证令牌（即使在 loopback 上也会生成）
    - 关闭 Tailscale 公开
    - 默认允许 Telegram 和 WhatsApp 的 DM 白名单（可能会提示输入电话号码）
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - 显示模式、工作区、Gateway、频道、守护进程、Skills 的完整提示流程
  
</Tab>
</Tabs>

## CLI 入门详解

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    本地和远程流程的完整说明、认证与模型矩阵、配置输出、向导 RPC、signal-cli 的行为。
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    非交互式入门的方案和自动化 `agents add` 示例。
  
</Card>
</Columns>

## 常用后续命令

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` 并不表示非交互模式。脚本中请使用 `--non-interactive`。
</Note>

<Tip>
建议：为使代理能够使用 `web_search`，请设置 Brave Search API 密钥（`web_fetch` 无需密钥即可运行）。最简单的方法：运行 `openclaw configure --section web` 以保存 `tools.web.search.apiKey`。文档：[Web 工具](/tools/web)。
</Tip>

## 相关文档

- CLI 命令参考：[`openclaw onboard`](/cli/onboard)
- macOS 应用入门：[入门指南](/start/onboarding)
- 代理首次启动步骤：[代理引导](/start/bootstrapping)

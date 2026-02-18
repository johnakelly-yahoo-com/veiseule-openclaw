---
summary: "為代理建立工作區與身分識別檔案的初始化儀式"
read_when:
  - 了解代理程式首次執行時會發生什麼事
  - Explaining where bootstrapping files live
  - Debugging onboarding identity setup
title: "代理程式啟動"
sidebarTitle: "初始化（Bootstrapping）"
---

# 代理程式啟動

初始化（Bootstrapping）是**首次執行**時的儀式，用於準備代理的工作區與
collects identity details. It happens after onboarding, when the agent starts
for the first time.

## 初始化會做什麼

在代理程式首次執行時，OpenClaw 會啟動並初始化工作區（預設為
`~/.openclaw/workspace`）：

- 建立 `AGENTS.md`、`BOOTSTRAP.md`、`IDENTITY.md`、`USER.md`。
- 執行簡短的問答儀式（一次一個問題）。
- 將身分與偏好設定寫入 `IDENTITY.md`、`USER.md`、`SOUL.md`。
- 完成後移除 `BOOTSTRAP.md`，確保只執行一次。

## 執行位置

初始化（Bootstrapping）一律在**gateway 主機**上執行。如果 macOS 應用程式連線至
a remote Gateway, the workspace and bootstrapping files live on that remote
machine.

<Note>
當 Gateway 閘道器執行在另一台機器上時，請在閘道器主機上編輯工作區檔案（例如 `user@gateway-host:~/.openclaw/workspace`）。
</Note>

## 相關文件

- macOS 應用程式入門引導：[Onboarding](/start/onboarding)
- 工作區配置：[Agent workspace](/concepts/agent-workspace)

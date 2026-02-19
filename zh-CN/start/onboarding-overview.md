---
summary: "OpenClaw 入门选项与流程概览"
read_when:
  - 选择入门路径
  - 设置新环境
title: "入门概览"
sidebarTitle: "入门概览"
---

# 入门概览

OpenClaw 根据 Gateway 的运行位置
以及你偏好的 provider 配置方式，支持多种入门路径。

## 选择你的入门路径

- **CLI 向导** 适用于 macOS、Linux 和 Windows（通过 WSL2）。
- **macOS 应用** 适用于在 Apple silicon 或 Intel Mac 上进行引导式首次运行。

## CLI 入门向导

在终端中运行向导：

```bash
openclaw onboard
```

当你希望完全控制 Gateway、workspace、channels 和 skills 时，请使用 CLI 向导。 文档：

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS 应用入门

当你希望在 macOS 上进行全程引导式设置时，请使用 OpenClaw 应用。 文档：

- [Onboarding (macOS App)](/start/onboarding)

## 自定义 Provider

如果你需要的端点未在列表中（包括暴露标准 OpenAI 或 Anthropic API 的托管提供商），请在 CLI 向导中选择 **Custom Provider**。 系统将会要求你：

- 选择 OpenAI-compatible、Anthropic-compatible，或 **Unknown**（自动检测）。
- 输入 base URL 和 API key（如果提供商要求）。
- 提供模型 ID 和可选别名。
- 选择一个 Endpoint ID，以便多个自定义端点可以共存。

详细步骤请参阅上方的 CLI onboarding 文档。

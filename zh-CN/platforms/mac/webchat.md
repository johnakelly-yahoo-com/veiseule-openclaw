---
summary: "macOS 应用如何嵌入 Gateway 网关 WebChat 以及如何调试"
read_when:
  - 调试 macOS WebChat 视图或 loopback 端口
title: "WebChat"
---

# WebChat（macOS 应用）

macOS 菜单栏应用将 WebChat UI 嵌入为原生 SwiftUI 视图。它连接到 Gateway 网关，默认使用所选智能体的**主会话**（带有会话切换器用于其他会话）。 48. 它
连接到网关，并默认为所选代理的**主会话**（并提供用于其他会话的会话切换器）。

- **本地模式**：直接连接到本地 Gateway 网关 WebSocket。
- **远程模式**：通过 SSH 转发 Gateway 网关控制端口，并使用该隧道作为数据平面。

## 启动和调试

- 手动：Lobster 菜单 → "Open Chat"。

- 测试时自动打开：

  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```

- 日志：`./scripts/clawlog.sh`（子系统 `bot.molt`，类别 `WebChatSwiftUI`）。

## How it’s wired

- 数据平面：Gateway 网关 WS 方法 `chat.history`、`chat.send`、`chat.abort`、`chat.inject` 和事件 `chat`、`agent`、`presence`、`tick`、`health`。
- Session: defaults to the primary session (`main`, or `global` when scope is
  global). The UI can switch between sessions.
- 新手引导使用专用会话，以将首次运行设置分开。

## 安全面

- 远程模式仅通过 SSH 转发 Gateway 网关 WebSocket 控制端口。

## 已知限制

- UI 针对聊天会话优化（不是完整的浏览器沙箱）。

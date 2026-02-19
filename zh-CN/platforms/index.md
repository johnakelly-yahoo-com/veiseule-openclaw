---
summary: "平台支持概述（Gateway 网关 + 配套应用）"
read_when:
  - 查找操作系统支持或安装路径时
  - 决定在哪里运行 Gateway 网关时
title: "平台"
---

# 平台

OpenClaw core is written in TypeScript. **Node is the recommended runtime**.
Bun is not recommended for the Gateway (WhatsApp/Telegram bugs).

Companion apps exist for macOS (menu bar app) and mobile nodes (iOS/Android). Windows and
Linux companion apps are planned, but the Gateway is fully supported today.
Native companion apps for Windows are also planned; the Gateway is recommended via WSL2.

## 选择你的操作系统

- macOS：[macOS](/platforms/macos)
- iOS：[iOS](/platforms/ios)
- Android：[Android](/platforms/android)
- Windows：[Windows](/platforms/windows)
- Linux：[Linux](/platforms/linux)

## VPS 和托管

- VPS 中心：[VPS 托管](/vps)
- Fly.io：[Fly.io](/install/fly)
- Hetzner（Docker）：[Hetzner](/install/hetzner)
- GCP（Compute Engine）：[GCP](/install/gcp)
- exe.dev（VM + HTTPS 代理）：[exe.dev](/install/exe-dev)

## 常用链接

- 安装指南：[入门指南](/start/getting-started)
- Gateway 网关运行手册：[Gateway 网关](/gateway)
- Gateway 网关配置：[配置](/gateway/configuration)
- 服务状态：`openclaw gateway status`

## Gateway 网关服务安装（CLI）

使用以下任一方式（均支持）：

- 向导（推荐）：`openclaw onboard --install-daemon`
- 直接安装：`openclaw gateway install`
- 配置流程：`openclaw configure` → 选择 **Gateway service**
- 修复/迁移：`openclaw doctor`（提供安装或修复服务）

服务目标取决于操作系统：

- macOS：LaunchAgent（`bot.molt.gateway` 或 `bot.molt.<profile>`；旧版 `com.openclaw.*`）
- Linux/WSL2：systemd 用户服务（`openclaw-gateway[-<profile>].service`）


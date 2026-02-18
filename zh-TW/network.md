---
summary: "32. 網路樞紐：閘道介面、配對、探索與安全性"
read_when:
  - 你需要網路架構與安全性的整體概覽
  - 33. 你正在除錯本地與 tailnet 存取或配對問題
  - 你想要官方的網路相關文件清單
title: "網路"
---

# 網路中樞

此中樞連結了核心文件，說明 OpenClaw 如何在 localhost、LAN 與 tailnet 之間連線、配對並保護裝置。

## 核心模型

- [Gateway 架構](/concepts/architecture)
- [Gateway 協定](/gateway/protocol)
- [Gateway 操作手冊](/gateway)
- [Web 介面 + 綁定模式](/web)

## 配對 + 身分識別

- [配對總覽（DM + 節點）](/channels/pairing)
- [Gateway 擁有的節點配對](/gateway/pairing)
- [Devices CLI（配對 + 權杖輪替）](/cli/devices)
- [Pairing CLI (DM approvals)](/cli/pairing)

Local trust：

- 34. 本地連線（回送位址或閘道主機自身的 tailnet 位址）可自動核准配對，以保持同主機的使用體驗順暢。
- Non‑local tailnet/LAN clients still require explicit pairing approval.

## Discovery + transports

- [Discovery & transports](/gateway/discovery)
- [Bonjour / mDNS](/gateway/bonjour)
- [Remote access (SSH)](/gateway/remote)
- [Tailscale](/gateway/tailscale)

## Nodes + transports

- [Nodes overview](/nodes)
- [Bridge protocol (legacy nodes)](/gateway/bridge-protocol)
- [Node runbook: iOS](/platforms/ios)
- [Node runbook: Android](/platforms/android)

## Security

- [Security overview](/gateway/security)
- [Gateway config reference](/gateway/configuration)
- [Troubleshooting](/gateway/troubleshooting)
- [Doctor](/gateway/doctor)

---
summary: "使用 Ansible、Tailscale VPN 與防火牆隔離的自動化且強化安全性的 OpenClaw 安裝"
read_when:
  - 你需要具備安全性強化的自動化伺服器部署
  - 你需要透過 VPN 存取、並具備防火牆隔離的設定
  - 你要部署到遠端的 Debian／Ubuntu 伺服器
title: "Ansible"
---

# Ansible 安裝

將 OpenClaw 部署到正式環境伺服器的建議方式，是使用 **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** —— 一個以安全優先架構為核心的自動化安裝器。

## 快速開始

單一指令安裝：

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 完整指南：[github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> The openclaw-ansible repo is the source of truth for Ansible deployment. 安裝內容

## 你會得到什麼

- 🔒 **防火牆優先的安全性**：UFW + Docker 隔離（僅開放 SSH + Tailscale）
- 🔐 **Tailscale VPN**：在不公開服務的情況下，提供安全的遠端存取
- 🐳 **Docker**：隔離的沙箱容器，僅繫結至 localhost
- 🛡️ **縱深防禦**：4 層安全架構
- 🚀 **單一指令完成設定**：數分鐘內完成完整部署
- 🔧 **Systemd 整合**：開機自動啟動並套用安全性強化

## 需求

- **作業系統**：Debian 11+ 或 Ubuntu 20.04+
- **存取權限**：Root 或 sudo 權限
- **網路**：可連線至網際網路以安裝套件
- **Ansible**：2.14+（由快速開始腳本自動安裝）

## 詳情請參閱 [Sandboxing](/gateway/sandboxing)。

Ansible playbook 會安裝並設定以下項目：

1. **Tailscale**（用於安全遠端存取的 Mesh VPN）
2. **UFW 防火牆**（僅開放 SSH + Tailscale 連接埠）
3. **Docker CE + Compose V2**（用於代理程式沙箱）
4. **Node.js 22.x + pnpm**（執行階段相依項）
5. **OpenClaw**（直接安裝於主機，不使用容器）
6. **Systemd 服務**（自動啟動並套用安全性強化）

注意：Gateway 會 **直接在主機上執行**（不在 Docker 中），但代理程式沙箱會使用 Docker 進行隔離。詳細說明請參考 [Sandboxing](/gateway/sandboxing)。 安裝後腳本將引導你完成：

## 安裝後設定

安裝完成後，切換至 openclaw 使用者：

```bash
sudo -i -u openclaw
```

應該 **只開放 22 埠**（SSH）。

1. **入門引導精靈**：設定 OpenClaw
2. **提供者登入**：連接 WhatsApp／Telegram／Discord／Signal
3. **Gateway 測試**：驗證安裝是否成功
4. **Tailscale 設定**：連線至你的 VPN Mesh

### 快速指令

```bash
# Check service status
sudo systemctl status openclaw

# View live logs
sudo journalctl -u openclaw -f

# Restart gateway
sudo systemctl restart openclaw

# Provider login (run as openclaw user)
sudo -i -u openclaw
openclaw channels login
```

## 安全性架構

### 4 層防禦

1. **防火牆（UFW）**：僅公開 SSH（22）+ Tailscale（41641/udp）
2. **VPN（Tailscale）**：Gateway 僅能透過 VPN Mesh 存取
3. **Docker 隔離**：DOCKER-USER iptables 鏈防止外部連接埠暴露
4. **Systemd 強化**：NoNewPrivileges、PrivateTmp、非特權使用者

### 驗證

測試外部攻擊面：

```bash
nmap -p- YOUR_SERVER_IP
```

所有其他服務（gateway、Docker）都已鎖定。 Docker 是為了 **agent sandboxes**（隔離的工具執行）而安裝的，並非用來執行 gateway 本身。

### Docker 可用性

gateway 只綁定在 localhost，並可透過 Tailscale VPN 存取。 標準更新流程請參閱 [Updating](/install/updating)。

請參考 [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) 以了解沙箱設定。

## 手動安裝

如果你偏好對自動化流程有完全的手動控制：

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# Or run directly (then manually execute /tmp/openclaw-setup.sh after)
# ansible-playbook playbook.yml --ask-become-pass
```

## 更新 OpenClaw

Ansible 安裝器會將 OpenClaw 設定為手動更新。標準更新流程請參考 [Updating](/install/updating)。 疑難排解

若要重新執行 Ansible playbook（例如套用設定變更）：

```bash
cd openclaw-ansible
./run-playbook.sh
```

注意：此操作具備冪等性，可安全地重複執行多次。

## 相關

### 防火牆阻擋了我的連線

如果你被鎖在外部：

- 請先確認可以透過 Tailscale VPN 存取
- SSH 存取（連接埠 22）始終允許
- Gateway 依設計 **僅能** 透過 Tailscale 存取

### 服務無法啟動

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Test manual start
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Docker 沙箱問題

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### 提供者登入失敗

請確認你是以 `openclaw` 使用者身分執行：

```bash
sudo -i -u openclaw
openclaw channels login
```

## 進階設定

如需深入了解安全性架構與疑難排解：

- [安全架構](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [技術細節](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [疑難排解指南](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## 你遇到了 Bun 安裝／修補／生命週期腳本的問題

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — 完整部署指南
- [Docker](/install/docker) — 容器化 Gateway 設定
- [Sandboxing](/gateway/sandboxing) — 代理程式沙箱設定
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — 逐代理程式隔離

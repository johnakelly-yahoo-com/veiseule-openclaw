---
title: "Ansible"
---

# Pag-install ng Ansible

Ang inirerekomendang paraan para i-deploy ang OpenClaw sa mga production server ay sa pamamagitan ng **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — isang automated installer na may security-first na arkitektura.

## Mabilis na pagsisimula

One-command na pag-install:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Buong gabay: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Ang openclaw-ansible repo ang pangunahing batayan para sa Ansible deployment. Ang pahinang ito ay isang mabilis na pangkalahatang-ideya.

## Ano ang Makukuha Mo

- 🔒 **Firewall-first security**: UFW + Docker isolation (SSH + Tailscale lang ang accessible)
- 🔐 **Tailscale VPN**: Secure na remote access nang hindi inilalantad ang mga serbisyo sa publiko
- 🐳 **Docker**: Isolated na sandbox containers, localhost-only bindings
- 🛡️ **Defense in depth**: 4-layer na security architecture
- 🚀 **One-command setup**: Kumpletong deployment sa loob ng ilang minuto
- 🔧 **Systemd integration**: Auto-start sa boot na may hardening

## Mga kinakailangan

- **OS**: Debian 11+ o Ubuntu 20.04+
- **Access**: Root o sudo privileges
- **Network**: Internet connection para sa pag-install ng mga package
- **Ansible**: 2.14+ (awtomatikong ini-install ng quick-start script)

## Ano ang Nai-install

Ini-install at kino-configure ng Ansible playbook ang:

1. **Tailscale** (mesh VPN para sa secure na remote access)
2. **UFW firewall** (SSH + Tailscale ports lang)
3. **Docker CE + Compose V2** (para sa agent sandboxes)
4. **Node.js 22.x + pnpm** (mga runtime dependency)
5. **OpenClaw** (host-based, hindi containerized)
6. **Systemd service** (auto-start na may security hardening)

Tandaan: Ang gateway ay tumatakbo **direkta sa host** (hindi sa Docker), ngunit ang mga agent sandbox ay gumagamit ng Docker para sa isolation. Tingnan ang [Sandboxing](/gateway/sandboxing) para sa mga detalye.

## Pag-setup Pagkatapos ng Pag-install

Pagkatapos makumpleto ang pag-install, lumipat sa openclaw user:

```bash
sudo -i -u openclaw
```

Gagabayan ka ng post-install script sa:

1. **Onboarding wizard**: I-configure ang mga setting ng OpenClaw
2. **Provider login**: Ikonekta ang WhatsApp/Telegram/Discord/Signal
3. **Gateway testing**: I-verify ang pag-install
4. **Tailscale setup**: Kumonekta sa iyong VPN mesh

### Mga mabilis na command

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

## Arkitekturang Pang-seguridad

### Depensang May 4 na Layer

1. **Firewall (UFW)**: SSH (22) + Tailscale (41641/udp) lang ang exposed sa publiko
2. **VPN (Tailscale)**: Ang Gateway ay accessible lamang sa pamamagitan ng VPN mesh
3. **Docker Isolation**: Pinipigilan ng DOCKER-USER iptables chain ang external na pag-expose ng mga port
4. **Pagpapatibay ng Systemd**: NoNewPrivileges, PrivateTmp, user na walang pribilehiyo

### Pag-verify

Subukan ang external attack surface:

```bash
nmap -p- YOUR_SERVER_IP
```

Itinatakda ng Ansible installer ang OpenClaw para sa manual na mga update. All other services (gateway, Docker) are locked down.

### Availability ng Docker

Ini-install ang Docker para sa **agent sandboxes** (nakahiwalay na pagpapatakbo ng tool), hindi para patakbuhin ang gateway mismo. Ang gateway ay naka-bind lamang sa localhost at maa-access sa pamamagitan ng Tailscale VPN.

Tingnan ang [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) para sa konpigurasyon ng sandbox.

## Manual na Pag-install

Kung mas gusto mo ang manual na kontrol kaysa automation:

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

## Pag-update ng OpenClaw

Tingnan ang [Updating](/install/updating) para sa standard na daloy ng update. ⚠️ **Hindi inirerekomenda para sa Gateway runtime** (mga bug sa WhatsApp/Telegram).

Para muling patakbuhin ang Ansible playbook (hal., para sa mga pagbabago sa configuration):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Tandaan: Ito ay idempotent at ligtas patakbuhin nang maraming beses.

## Pag-troubleshoot

### Hinaharangan ng firewall ang aking koneksyon

Kung ikaw ay na-lock out:

- Siguraduhing may access ka muna sa pamamagitan ng Tailscale VPN
- Ang SSH access (port 22) ay palaging pinapayagan
- Ang Gateway ay **tanging** naa-access sa pamamagitan ng Tailscale ayon sa disenyo

### Hindi nag-start ang serbisyo

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

### Mga isyu sa Docker sandbox

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Nabigo ang provider login

Siguraduhing tumatakbo ka bilang `openclaw` user:

```bash
sudo -i -u openclaw
openclaw channels login
```

## Advanced na Konpigurasyon

Para sa detalyadong arkitekturang pang-seguridad at pag-troubleshoot:

- [Security Architecture](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Technical Details](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Troubleshooting Guide](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Kaugnay

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — buong gabay sa deployment
- [Docker](/install/docker) — containerized na setup ng Gateway
- [Sandboxing](/gateway/sandboxing) — konpigurasyon ng agent sandbox
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — per-agent isolation


---
summary: "Automatiseret, hærdet OpenClaw-installation med Ansible, Tailscale VPN og firewall-isolation"
read_when:
  - Du ønsker automatiseret serverudrulning med sikkerhedshærdning
  - Du har brug for en firewall-isoleret opsætning med VPN-adgang
  - Du udruller til fjerne Debian/Ubuntu-servere
title: "Ansible"
---

# Ansible-installation

Den anbefalede måde at udrulle OpenClaw til produktionsservere på er via **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — et automatiseret installationsværktøj med sikkerhed som førsteprioritet.

## Hurtig start

Installation med én kommando:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Fuld guide: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Den openclaw-ansible repo er kilden til sandhed for Ansible implementering. Denne side er et hurtigt overblik.

## Hvad du får

- 🔒 **Firewall-først-sikkerhed**: UFW + Docker-isolation (kun SSH + Tailscale er tilgængelige)
- 🔐 **Tailscale VPN**: Sikker fjernadgang uden at eksponere tjenester offentligt
- 🐳 **Docker**: Isolerede sandbox-containere, bindinger kun til localhost
- 🛡️ **Defense in depth**: 4-lags sikkerhedsarkitektur
- 🚀 **Opsætning med én kommando**: Komplet udrulning på få minutter
- 🔧 **Systemd-integration**: Automatisk start ved boot med hærdning

## Krav

- **OS**: Debian 11+ eller Ubuntu 20.04+
- **Adgang**: Root- eller sudo-rettigheder
- **Netværk**: Internetforbindelse til installation af pakker
- **Ansible**: 2.14+ (installeres automatisk af hurtigstart-scriptet)

## Hvad der installeres

Ansible-playbooken installerer og konfigurerer:

1. **Tailscale** (mesh VPN til sikker fjernadgang)
2. **UFW firewall** (kun SSH- og Tailscale-porte)
3. **Docker CE + Compose V2** (til agent-sandboxes)
4. **Node.js 22.x + pnpm** (runtime-afhængigheder)
5. **OpenClaw** (host-baseret, ikke containeriseret)
6. **Systemd-tjeneste** (automatisk start med sikkerhedshærdning)

Bemærk: Gatewayen kører \*\* direkte på værten \*\* (ikke i Docker), men agent sandkasser bruge Docker til isolation. Se [Sandboxing](/gateway/sandboxing) for detaljer.

## Opsætning efter installation

Når installationen er fuldført, skift til openclaw-brugeren:

```bash
sudo -i -u openclaw
```

Post-install-scriptet guider dig igennem:

1. **Introduktionsguide**: Konfigurér OpenClaw-indstillinger
2. **Udbyder-login**: Forbind WhatsApp/Telegram/Discord/Signal
3. **Gateway-test**: Verificér installationen
4. **Tailscale-opsætning**: Forbind til dit VPN-mesh

### Hurtige kommandoer

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

## Sikkerhedsarkitektur

### 4-lags forsvar

1. **Firewall (UFW)**: Kun SSH (22) + Tailscale (41641/udp) er offentligt eksponeret
2. **VPN (Tailscale)**: Gatewayen er kun tilgængelig via VPN-mesh
3. **Docker-isolation**: DOCKER-USER iptables-kæden forhindrer ekstern porteksponering
4. **Systemd-hærdning**: NoNewPrivileges, PrivateTmp, uprivilegeret bruger

### Verifikation

Test den eksterne angrebsflade:

```bash
nmap -p- YOUR_SERVER_IP
```

Skal vise **kun port 22** (SSH) åben. Alle andre tjenester (gateway, Docker) er låst ned.

### Docker-tilgængelighed

Docker er installeret for **agent sandkasser** (isoleret værktøj udførelse), ikke for at køre selve gatewayen. Gateway binder kun til localhost og er tilgængelig via Tailscale VPN.

Se [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) for sandbox-konfiguration.

## Manuel installation

Hvis du foretrækker manuel kontrol frem for automatiseringen:

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

## Opdatering af OpenClaw

Den Ansible installationsprogram opstiller OpenClaw til manuelle opdateringer. Se [Updating](/install/updating) for standardopdateringsflowet.

For at køre den Ansible spillebog igen (fx, for konfigurationsændringer):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Bemærk: Dette er idempotent og sikkert at køre flere gange.

## Fejlfinding

### Firewall blokerer min forbindelse

Hvis du er låst ude:

- Sørg for, at du først kan få adgang via Tailscale VPN
- SSH-adgang (port 22) er altid tilladt
- Gatewayen er **kun** tilgængelig via Tailscale af design

### Tjenesten vil ikke starte

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

### Problemer med Docker-sandbox

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Udbyder-login fejler

Sørg for, at du kører som `openclaw`-brugeren:

```bash
sudo -i -u openclaw
openclaw channels login
```

## Avanceret konfiguration

For detaljeret sikkerhedsarkitektur og fejlfinding:

- [Sikkerhedsarkitektur](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Tekniske detaljer](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Fejlfindingsvejledning](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Relateret

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — fuld udrulningsguide
- [Docker](/install/docker) — containeriseret gateway-opsætning
- [Sandboxing](/gateway/sandboxing) — konfiguration af agent-sandbox
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — isolation pr. agent


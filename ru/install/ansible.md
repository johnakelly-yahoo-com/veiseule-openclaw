---
summary: "Автоматизированная, усиленная установка OpenClaw с Ansible, VPN Tailscale и изоляцией через файрвол"
read_when:
  - Вам требуется автоматизированное развёртывание серверов с усилением безопасности
  - Нужна изолированная файрволом настройка с доступом через VPN
  - Вы разворачиваете систему на удалённых серверах Debian/Ubuntu
title: "Ansible"
---

# Установка Ansible

Рекомендуемый способ развертывания OpenClaw на продакшн‑серверах — через **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — автоматизированный установщик с архитектурой, ориентированной на безопасность.

## Быстрый старт

Установка одной командой:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Полное руководство: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Репозиторий openclaw-ansible является источником истины для развёртывания с Ansible. Эта страница — краткий обзор.

## Что вы получаете

- 🔒 **Безопасность с приоритетом файрвола**: UFW + изоляция Docker (доступны только SSH + Tailscale)
- 🔐 **VPN Tailscale**: безопасный удалённый доступ без публичного экспонирования сервисов
- 🐳 **Docker**: изолированные sandbox‑контейнеры, привязка только к localhost
- 🛡️ **Многоуровневая защита**: 4‑уровневая архитектура безопасности
- 🚀 **Настройка одной командой**: полное развёртывание за считанные минуты
- 🔧 **Интеграция с Systemd**: автозапуск при старте системы с усилением безопасности

## Требования

- **OS**: Debian 11+ или Ubuntu 20.04+
- **Доступ**: права root или sudo
- **Сеть**: подключение к интернету для установки пакетов
- **Ansible**: 2.14+ (устанавливается автоматически скриптом быстрого старта)

## Что устанавливается

Ansible‑плейбук устанавливает и настраивает:

1. **Tailscale** (mesh‑VPN для безопасного удалённого доступа)
2. **Файрвол UFW** (только порты SSH + Tailscale)
3. **Docker CE + Compose V2** (для sandbox‑окружений агентов)
4. **Node.js 22.x + pnpm** (зависимости среды выполнения)
5. **OpenClaw** (разворачивается на хосте, не в контейнере)
6. **Сервис Systemd** (автозапуск с усилением безопасности)

Примечание: Gateway (шлюз) запускается **непосредственно на хосте** (не в Docker), а sandbox‑окружения агентов используют Docker для изоляции. Подробнее см. [Sandboxing](/gateway/sandboxing).

## Настройка после установки

После завершения установки переключитесь на пользователя openclaw:

```bash
sudo -i -u openclaw
```

Скрипт постустановки проведёт вас через:

1. **Мастер онбординга**: настройка параметров OpenClaw
2. **Вход провайдера**: подключение WhatsApp/Telegram/Discord/Signal
3. **Тестирование Gateway (шлюза)**: проверка установки
4. **Настройка Tailscale**: подключение к вашей VPN‑mesh сети

### Быстрые команды

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

## Архитектура безопасности

### 4 уровня защиты

1. **Файрвол (UFW)**: публично открыты только SSH (22) + Tailscale (41641/udp)
2. **VPN (Tailscale)**: Gateway (шлюз) доступен только через VPN‑mesh
3. **Изоляция Docker**: цепочка iptables DOCKER-USER предотвращает внешнее экспонирование портов
4. **Усиление Systemd**: NoNewPrivileges, PrivateTmp, непривилегированный пользователь

### Проверка

Проверьте внешнюю поверхность атаки:

```bash
nmap -p- YOUR_SERVER_IP
```

Должен отображаться **только порт 22** (SSH) как открытый. Все остальные сервисы (Gateway, Docker) заблокированы.

### Доступность Docker

Docker устанавливается для **sandbox‑окружений агентов** (изолированное выполнение инструментов), а не для запуска самого Gateway (шлюза). Gateway привязывается только к localhost и доступен через VPN Tailscale.

См. [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) для настройки sandbox‑окружений.

## Ручная установка

Если вы предпочитаете ручной контроль вместо автоматизации:

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

## Обновление OpenClaw

Установщик Ansible настраивает OpenClaw для ручных обновлений. См. [Updating](/install/updating) для стандартного процесса обновления.

Чтобы повторно запустить Ansible‑плейбук (например, для изменения конфигурации):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Примечание: процесс идемпотентен и безопасен для многократного запуска.

## Устранение неполадок

### Файрвол блокирует подключение

Если доступ потерян:

- Сначала убедитесь, что у вас есть доступ через VPN Tailscale
- Доступ по SSH (порт 22) всегда разрешён
- Gateway (шлюз) **доступен только через Tailscale** по замыслу

### Сервис не запускается

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

### Проблемы с Docker sandbox

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Не удаётся вход провайдера

Убедитесь, что вы работаете под пользователем `openclaw`:

```bash
sudo -i -u openclaw
openclaw channels login
```

## Расширенная конфигурация

Подробности по архитектуре безопасности и устранению неполадок:

- [Архитектура безопасности](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Технические детали](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Руководство по устранению неполадок](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Связанное

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — полное руководство по развёртыванию
- [Docker](/install/docker) — контейнеризованная настройка Gateway (шлюза)
- [Sandboxing](/gateway/sandboxing) — конфигурация sandbox‑окружений агентов
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — изоляция для каждого агента


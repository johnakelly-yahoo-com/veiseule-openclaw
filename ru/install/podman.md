---
summary: "Запуск OpenClaw в rootless-контейнере Podman"
read_when:
  - Вам нужен контейнеризированный gateway на базе Podman вместо Docker
title: "Podman"
---

# Podman

Запустите OpenClaw gateway в **rootless** контейнере Podman. Используется тот же образ, что и для Docker (сборка из репозитория [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Требования

- Podman (rootless)
- Sudo для одноразовой настройки (создание пользователя, сборка образа)

## Быстрый старт

**1. Одноразовая настройка** (из корня репозитория; создаёт пользователя, собирает образ, устанавливает скрипт запуска):

```bash
./setup-podman.sh
```

Это также создаёт минимальный `~openclaw/.openclaw/openclaw.json` (устанавливает `gateway.mode="local"`), чтобы gateway мог запуститься без прохождения мастера настройки.

По умолчанию контейнер **не** устанавливается как сервис systemd, его нужно запускать вручную (см. ниже). Для настройки в стиле production с автозапуском и перезапуском установите его как пользовательский сервис systemd Quadlet:

```bash
./setup-podman.sh --quadlet
```

(Или установите `OPENCLAW_PODMAN_QUADLET=1`; используйте `--container`, чтобы установить только контейнер и скрипт запуска.)

**2. Запустите gateway** (вручную, для быстрой проверки работоспособности):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Мастер первичной настройки** (например, для добавления каналов или провайдеров):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Затем откройте `http://127.0.0.1:18789/` и используйте токен из `~openclaw/.openclaw/.env` (или значение, выведенное во время setup).

## Systemd (Quadlet, опционально)

Если вы запускали `./setup-podman.sh --quadlet` (или `OPENCLAW_PODMAN_QUADLET=1`), устанавливается юнит [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html), чтобы gateway работал как пользовательский сервис systemd для пользователя openclaw. Сервис включается и запускается в конце настройки.

- **Запуск:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Остановка:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Статус:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Логи:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Файл quadlet находится по пути `~openclaw/.config/containers/systemd/openclaw.container`. Чтобы изменить порты или переменные окружения, отредактируйте этот файл (или `.env`, на который он ссылается), затем выполните `sudo systemctl --machine openclaw@ --user daemon-reload` и перезапустите сервис. При загрузке системы сервис запускается автоматически, если для пользователя openclaw включён lingering (setup делает это, если доступен loginctl).

Чтобы добавить quadlet **после** первоначальной настройки без него, повторно выполните: `./setup-podman.sh --quadlet`.

## Пользователь openclaw (без входа в систему)

`setup-podman.sh` создаёт отдельного системного пользователя `openclaw`:

- **Shell:** `nologin` — интерактивный вход запрещён; снижает поверхность атаки.

- **Home:** например, `/home/openclaw` — содержит `~/.openclaw` (конфигурация, рабочее пространство) и скрипт запуска `run-openclaw-podman.sh`.

- **Rootless Podman:** у пользователя должен быть диапазон **subuid** и **subgid**. Во многих дистрибутивах они назначаются автоматически при создании пользователя. Если во время setup выводится предупреждение, добавьте строки в `/etc/subuid` и `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Затем запустите gateway от имени этого пользователя (например, из cron или systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Конфигурация:** Только `openclaw` и root имеют доступ к `/home/openclaw/.openclaw`. Чтобы изменить конфигурацию: используйте Control UI после запуска gateway или выполните `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Окружение и конфигурация

- **Токен:** Хранится в `~openclaw/.openclaw/.env` как `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` и `run-openclaw-podman.sh` генерируют его при отсутствии (используются `openssl`, `python3` или `od`).
- **Опционально:** В этом `.env` можно задать ключи провайдеров (например, `GROQ_API_KEY`, `OLLAMA_API_KEY`) и другие переменные окружения OpenClaw.
- **Порты хоста:** По умолчанию скрипт сопоставляет `18789` (gateway) и `18790` (bridge). Переопределите сопоставление портов **хоста** с помощью `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` и `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` при запуске.
- **Пути:** Конфигурация и рабочее пространство на хосте по умолчанию расположены в `~openclaw/.openclaw` и `~openclaw/.openclaw/workspace`. Переопределите пути хоста, используемые скриптом запуска, с помощью `OPENCLAW_CONFIG_DIR` и `OPENCLAW_WORKSPACE_DIR`.

## Полезные команды

- **Логи:** С quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. С помощью скрипта: `sudo -u openclaw podman logs -f openclaw`
- **Остановить:** С quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. С помощью скрипта: `sudo -u openclaw podman stop openclaw`
- **Запустить снова:** С quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. С помощью скрипта: повторно запустите скрипт запуска или `podman start openclaw`
- **Удалить контейнер:** `sudo -u openclaw podman rm -f openclaw` — конфигурация и рабочая директория на хосте сохраняются

## Устранение неполадок

- **Permission denied (EACCES) для config или auth-profiles:** Контейнер по умолчанию использует `--userns=keep-id` и запускается с тем же uid/gid, что и пользователь хоста, выполняющий скрипт. Убедитесь, что каталоги хоста `OPENCLAW_CONFIG_DIR` и `OPENCLAW_WORKSPACE_DIR` принадлежат этому пользователю.
- **Запуск Gateway заблокирован (отсутствует `gateway.mode=local`):** Убедитесь, что файл `~openclaw/.openclaw/openclaw.json` существует и содержит `gateway.mode="local"`. `setup-podman.sh` создаёт этот файл, если он отсутствует.
- **Rootless Podman не работает для пользователя openclaw:** Проверьте, что в `/etc/subuid` и `/etc/subgid` есть строка для `openclaw` (например, `openclaw:100000:65536`). Добавьте её при отсутствии и перезапустите.
- **Имя контейнера уже используется:** Скрипт запуска использует `podman run --replace`, поэтому существующий контейнер будет заменён при повторном запуске. Для ручной очистки: `podman rm -f openclaw`.
- **Скрипт не найден при запуске от имени openclaw:** Убедитесь, что был выполнен `setup-podman.sh`, чтобы `run-openclaw-podman.sh` был скопирован в домашний каталог openclaw (например, `/home/openclaw/run-openclaw-podman.sh`).
- **Сервис Quadlet не найден или не запускается:** Выполните `sudo systemctl --machine openclaw@ --user daemon-reload` после редактирования файла `.container`. Quadlet требует cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` должно выводить `2`.

## Необязательно: запуск от имени своего пользователя

Чтобы запустить gateway от имени обычного пользователя (без отдельного пользователя openclaw): соберите образ, создайте `~/.openclaw/.env` с `OPENCLAW_GATEWAY_TOKEN` и запустите контейнер с `--userns=keep-id` и монтированием в ваш `~/.openclaw`. Скрипт запуска предназначен для сценария с пользователем openclaw; для однопользовательской установки вы можете вместо этого вручную выполнить команду `podman run` из скрипта, указав каталоги конфигурации и рабочей директории в своём домашнем каталоге. Рекомендуется для большинства пользователей: используйте `setup-podman.sh` и запускайте от имени пользователя openclaw, чтобы изолировать конфигурацию и процесс.

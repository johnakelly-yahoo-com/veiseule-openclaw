---
title: "Nix"
---

# Установка с помощью Nix

Рекомендуемый способ запускать OpenClaw с Nix — через **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — модуль Home Manager «всё включено».

## Быстрый старт

Вставьте это своему ИИ-агенту (Claude, Cursor и т. п.):

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Полное руководство: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Репозиторий nix-openclaw — это источник истины для установки с Nix. Эта страница — лишь краткий обзор.

## Что вы получаете

- Gateway (шлюз) + приложение для macOS + инструменты (whisper, spotify, камеры) — всё с закреплёнными версиями
- Сервис launchd, переживающий перезагрузки
- Плагинную систему с декларативной конфигурацией
- Мгновенный откат: `home-manager switch --rollback`

---

## Поведение выполнения в режиме Nix

Когда установлен `OPENCLAW_NIX_MODE=1` (автоматически с nix-openclaw):

OpenClaw поддерживает **режим Nix**, который делает конфигурацию детерминированной и отключает автоматические потоки установки.
Включите его, экспортировав:

```bash
OPENCLAW_NIX_MODE=1
```

В macOS GUI‑приложение не наследует переменные окружения оболочки автоматически. Вы также можете
включить режим Nix через defaults:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### Пути конфигурации и состояния

OpenClaw читает конфиг JSON5 из `OPENCLAW_CONFIG_PATH` и хранит изменяемые данные в `OPENCLAW_STATE_DIR`.
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (default precedence: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (по умолчанию: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (по умолчанию: `$OPENCLAW_STATE_DIR/openclaw.json`)

При запуске под Nix задавайте их явно и указывайте на управляемые Nix расположения, чтобы состояние выполнения и конфигурация
оставались вне неизменяемого хранилища.

### Поведение выполнения в режиме Nix

- Потоки автоустановки и самойзменения отключены
- Отсутствующие зависимости сопровождаются сообщениями об устранении неполадок, специфичными для Nix
- В интерфейсе отображается баннер режима Nix «только для чтения», если он присутствует

## Примечание по упаковке (macOS)

Процесс упаковки для macOS ожидает стабильный шаблон Info.plist по адресу:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) копирует этот шаблон в бандл приложения и патчит динамические поля
(идентификатор бандла, версия/сборка, Git SHA, ключи Sparkle). Это сохраняет plist детерминированным для упаковки SwiftPM
и сборок Nix (которые не полагаются на полный набор инструментов Xcode).

## Связанное

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — полное руководство по настройке
- [Мастер](/start/wizard) — настройка CLI без Nix
- [Docker](/install/docker) — контейнерная настройка

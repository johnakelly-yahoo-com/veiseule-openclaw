---
summary: "Справочник CLI для `openclaw hooks` (хуки агента)"
read_when:
  - Вам нужно управлять хуками агента
  - Вам нужно установить или обновить хуки
title: "хуки"
---

# `openclaw hooks`

Управление хуками агента (событийно-ориентированные автоматизации для команд вроде `/new`, `/reset` и запуска Gateway (шлюза)).

Связанное:

- Хуки: [Хуки](/automation/hooks)
- Хуки плагинов: [Plugins](/tools/plugin#plugin-hooks)

## Список всех хуков

```bash
openclaw hooks list
```

Перечисляет все обнаруженные хуки из каталогов рабочего пространства, управляемых и входящих в комплект.

**Параметры:**

- `--eligible`: Показать только подходящие хуки (требования выполнены)
- `--json`: Вывод в формате JSON
- `-v, --verbose`: Показать подробную информацию, включая отсутствующие требования

**Пример вывода:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**Пример (подробно):**

```bash
openclaw hooks list --verbose
```

Показывает отсутствующие требования для неподходящих хуков.

**Пример (JSON):**

```bash
openclaw hooks list --json
```

Возвращает структурированный JSON для программного использования.

## Получение информации о хуке

```bash
openclaw hooks info <name>
```

Показывает подробную информацию о конкретном хуке.

**Аргументы:**

- `<name>`: Имя хука (например, `session-memory`)

**Параметры:**

- `--json`: Вывод в формате JSON

**Пример:**

```bash
openclaw hooks info session-memory
```

**Вывод:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## Проверка пригодности хуков

```bash
openclaw hooks check
```

Показывает сводку статуса пригодности хуков (сколько готово и сколько не готово).

**Параметры:**

- `--json`: Вывод в формате JSON

**Пример вывода:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## Включение хука

```bash
openclaw hooks enable <name>
```

Включает конкретный хук, добавляя его в ваш конфиг (`~/.openclaw/config.json`).

**Примечание:** Хуки, управляемые плагинами, показывают `plugin:<id>` в `openclaw hooks list` и
не могут быть включены/отключены здесь. Вместо этого включите/отключите плагин.

**Аргументы:**

- `<name>`: Имя хука (например, `session-memory`)

**Пример:**

```bash
openclaw hooks enable session-memory
```

**Вывод:**

```
✓ Enabled hook: 💾 session-memory
```

**Что делает:**

- Проверяет, существует ли хук и подходит ли он
- Обновляет `hooks.internal.entries.<name>.enabled = true` в вашем конфиге
- Сохраняет конфиг на диск

**После включения:**

- Перезапустите Gateway (шлюз), чтобы хуки перезагрузились (перезапуск приложения в строке меню на macOS или перезапуск процесса шлюза в dev).

## Отключение хука

```bash
openclaw hooks disable <name>
```

Отключает конкретный хук, обновляя ваш конфиг.

**Аргументы:**

- `<name>`: Имя хука (например, `command-logger`)

**Пример:**

```bash
openclaw hooks disable command-logger
```

**Вывод:**

```
⏸ Disabled hook: 📝 command-logger
```

**После отключения:**

- Перезапустите Gateway (шлюз), чтобы хуки перезагрузились

## Установка хуков

```bash
openclaw hooks install <path-or-spec>
```

Устанавливает пакет хуков из локальной папки/архива или npm.

Спецификации npm принимаются **только из реестра** (имя пакета + необязательная версия/тег). Спецификации Git/URL/file
отклоняются. Установка зависимостей выполняется с `--ignore-scripts` в целях безопасности.

**Что делает:**

- Копирует пакет хуков в `~/.openclaw/hooks/<id>`
- Включает установленные хуки в `hooks.internal.entries.*`
- Записывает установку в `hooks.internal.installs`

**Параметры:**

- `-l, --link`: Связать локальный каталог вместо копирования (добавляет его в `hooks.internal.load.extraDirs`)

**Поддерживаемые архивы:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Примеры:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Link a local directory without copying
openclaw hooks install -l ./my-hook-pack
```

## Обновление хуков

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Обновляет установленные пакеты хуков (только установки из npm).

**Параметры:**

- `--all`: Обновить все отслеживаемые пакеты хуков
- `--dry-run`: Показать, что изменится, без записи

## Bundled Hooks

### session-memory

Сохраняет контекст сеанса в памяти, когда вы выполняете `/new`.

**Включение:**

```bash
openclaw hooks enable session-memory
```

**Вывод:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**См.:** [документация session-memory](/automation/hooks#session-memory)

### bootstrap-extra-files

Добавляет дополнительные bootstrap-файлы (например, локальные для монорепозитория `AGENTS.md` / `TOOLS.md`) во время `agent:bootstrap`.

**Включение:**

```bash
openclaw hooks включают bootstrap-extra-files
```

**См.:** [хук SOUL Evil](/hooks/soul-evil)

### command-logger

Журналирует все события команд в централизованный файл аудита.

**Включение:**

```bash
openclaw hooks enable command-logger
```

**Вывод:** `~/.openclaw/logs/commands.log`

**Просмотр логов:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**См.:** [документация command-logger](/automation/hooks#command-logger)

### boot-md

Запускает `BOOT.md` при старте Gateway (шлюза) (после запуска каналов).

**Включение**:

**События**: `gateway:startup`

```bash
openclaw hooks enable boot-md
```

**См.:** [документация boot-md](/automation/hooks#boot-md)

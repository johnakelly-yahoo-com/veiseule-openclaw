---
summary: "Справочник CLI для `openclaw plugins` (list, install, enable/disable, doctor)"
read_when:
  - Вам нужно установить или управлять внутрипроцессными плагинами Gateway (шлюз)
  - Вам нужно отладить ошибки загрузки плагинов
title: "плагины"
---

# `openclaw plugins`

Управление плагинами/расширениями Gateway (шлюз) (загружаются внутрипроцессно).

Связанное:

- Система плагинов: [Plugins](/tools/plugin)
- Манифест плагина и схема: [Plugin manifest](/plugins/manifest)
- Усиление безопасности: [Security](/gateway/security)

## Команды

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Плагины, поставляемые в комплекте, входят в OpenClaw, но изначально отключены. Используйте `plugins enable`, чтобы
активировать их.

Все плагины должны поставляться с файлом `openclaw.plugin.json` с встроенной JSON Schema
(`configSchema`, даже если она пустая). Отсутствующие или некорректные манифесты либо схемы
препятствуют загрузке плагина и приводят к ошибке валидации конфига.

### Установка

```bash
openclaw plugins install <path-or-spec>
```

Примечание по безопасности: относитесь к установке плагинов как к запуску кода. Предпочитайте закреплённые версии.

Спецификации Npm поддерживаются **только из registry** (имя пакета + необязательная версия/тег). Спецификации Git/URL/file
отклоняются. Установка зависимостей выполняется с `--ignore-scripts` в целях безопасности.

Поддерживаемые архивы: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Используйте `--link`, чтобы избежать копирования локального каталога (добавляет в `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

### Удаление

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` удаляет записи плагина из `plugins.entries`, `plugins.installs`,
списка разрешённых плагинов и связанных записей `plugins.load.paths`, когда применимо.
Для активных плагинов памяти слот памяти сбрасывается на `memory-core`.

По умолчанию при удалении также удаляется каталог установки плагина в активном
корневом каталоге состояния extensions (`$OPENCLAW_STATE_DIR/extensions/<id>`). Используйте
`--keep-files`, чтобы сохранить файлы на диске.

`--keep-config` поддерживается как устаревший псевдоним для `--keep-files`.

### Обновление

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Обновления применяются только к плагинам, установленным из npm (отслеживаются в `plugins.installs`).


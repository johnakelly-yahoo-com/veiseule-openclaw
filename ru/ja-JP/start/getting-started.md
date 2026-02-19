---
read_when:
  - Первоначальная настройка с нуля
  - Хотите узнать самый быстрый путь к работающему чату
summary: Установите OpenClaw и запустите свой первый чат за несколько минут.
title: Введение
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Введение

Цель: с нуля добиться первого работающего чата с минимальной настройкой.

<Info>
Самый быстрый способ начать чат: откройте Control UI (настройка каналов не требуется). Выполните `openclaw dashboard` и общайтесь в браузере или на
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Хост Gateway</Tooltip>откройте `http://127.0.0.1:18789/`.
Документация: [Dashboard](/web/dashboard) и [Control UI](/web/control-ui).
</Info>

## Предварительные требования

- Node 22 или новее

<Tip>
Если вы не уверены, проверьте версию Node с помощью `node --version`.
</Tip>

## Быстрая установка (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    Другие способы установки и требования: [Установка](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Мастер настроит аутентификацию, конфигурацию Gateway и при необходимости каналы.
    Подробности см. в разделе [Мастер онбординга](/start/wizard).
    ```

  
</Step>
  <Step title="Gatewayを確認">Если вы установили сервис, он уже должен быть запущен:

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Если Control UI загружается, значит Gateway готов к использованию.
</Check>

## Проверка опций и дополнительные возможности

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">Удобно для быстрого тестирования и устранения неполадок.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">Требуется настроенный канал.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Подробнее

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">Полный справочник по CLI‑мастеру и расширенные параметры.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">Процесс первого запуска приложения для macOS.
  
</Card>
</Columns>

## Состояние после завершения

- Запущенный Gateway
- Настроенная аутентификация
- Доступ к Control UI или подключённый канал

## Следующие шаги

- Безопасность и подтверждение DM: [Сопряжение](/channels/pairing)
- Подключить дополнительные каналы: [Каналы](/channels)
- Расширенные рабочие процессы и сборка из исходников: [Настройка](/start/setup)

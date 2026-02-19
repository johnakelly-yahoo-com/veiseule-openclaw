---
read_when:
  - Configuración inicial desde cero
  - La ruta más rápida hacia un chat funcional
summary: Instala OpenClaw y ejecuta tu primer chat en solo unos minutos.
title: Introducción
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Introducción

Objetivo: lograr tu primer chat funcional desde cero con una configuración mínima.

<Info>
La forma más rápida de chatear: abre el Control UI (no se requiere configuración de canal). Ejecuta `openclaw dashboard` para chatear en el navegador, o<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Host de Gateway</Tooltip>y abre `http://127.0.0.1:18789/`.
Documentación: [Dashboard](/web/dashboard) y [Control UI](/web/control-ui).
</Info>

## Requisitos previos

- Node 22 o posterior

<Tip>
Si no estás seguro, verifica la versión de Node con `node --version`.
</Tip>

## Configuración rápida (CLI)

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
    Otros métodos de instalación y requisitos: [Instalación](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    El asistente configura la autenticación, los ajustes de Gateway y los canales opcionales.
    Para más detalles, consulta el [Asistente de incorporación](/start/wizard).
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Si instalaste el servicio, debería estar ya en ejecución:

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Si el Control UI se carga, Gateway está listo para usarse.
</Check>

## Verificaciones opcionales y funciones adicionales

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Útil para pruebas rápidas y resolución de problemas.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Se requiere un canal configurado.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Más información

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Referencia completa del asistente CLI y opciones avanzadas.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Flujo de primera ejecución de la aplicación macOS.
  
</Card>
</Columns>

## Estado después de completar

- Gateway en ejecución
- Autenticación configurada
- Acceso a Control UI o canales conectados

## Siguientes pasos

- Seguridad y aprobación de DM: [Emparejamiento](/channels/pairing)
- Conectar más canales: [Canales](/channels)
- Flujos de trabajo avanzados y compilación desde el código fuente: [Configuración](/start/setup)

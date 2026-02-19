---
read_when:
  - Al ejecutar el asistente de incorporación o durante la configuración
  - Al configurar una nueva máquina
sidebarTitle: Wizard (CLI)
summary: "Asistente de incorporación CLI: configuración interactiva de Gateway, espacio de trabajo, canales y Skills"
title: Asistente de incorporación (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Asistente de incorporación (CLI)

El asistente de incorporación CLI es la ruta recomendada para configurar OpenClaw en macOS, Linux y Windows (a través de WSL2). Además de la conexión a un Gateway local o remoto, configura los valores predeterminados del espacio de trabajo, los canales y las Skills.

```bash
openclaw onboard
```

<Info>
La forma más rápida de iniciar tu primer chat: abre el Control UI (no se requiere configuración de canales). Ejecuta `openclaw dashboard` para chatear en el navegador. Documentación: [Dashboard](/web/dashboard).
</Info>

## Inicio rápido vs configuración avanzada

El asistente comienza permitiéndote elegir entre **Inicio rápido** (configuración predeterminada) y **Configuración avanzada** (control total).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Gateway local en loopback
    - Espacio de trabajo existente o espacio de trabajo predeterminado
    - Puerto del Gateway `18789`
    - Token de autenticación del Gateway generado automáticamente (también se genera en loopback)
    - Publicación en Tailscale desactivada
    - DM de Telegram y WhatsApp permitidos por lista blanca de forma predeterminada (puede que se solicite introducir un número de teléfono)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Muestra el flujo completo de indicaciones para modo, espacio de trabajo, Gateway, canales, daemon y Skills
  
</Tab>
</Tabs>

## Detalles de la incorporación CLI

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Descripción completa de los flujos locales y remotos, matriz de autenticación y modelos, salida de configuración, RPC del asistente y comportamiento de signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Recetas de incorporación no interactiva y ejemplos automatizados de `agents add`.
  
</Card>
</Columns>

## Comandos de seguimiento más utilizados

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` no implica modo no interactivo. En scripts, utiliza `--non-interactive`.
</Note>

<Tip>
Recomendado: configura una clave de API de Brave Search para que los agentes puedan usar `web_search` (`web_fetch` funciona sin clave). La forma más sencilla: ejecuta `openclaw configure --section web` para guardar `tools.web.search.apiKey`. Documentación: [Herramientas web](/tools/web).
</Tip>

## Documentación relacionada

- Referencia de comandos CLI: [`openclaw onboard`](/cli/onboard)
- Incorporación en la app de macOS: [Incorporación](/start/onboarding)
- Pasos para el primer inicio del agente: [Bootstrap del agente](/start/bootstrapping)

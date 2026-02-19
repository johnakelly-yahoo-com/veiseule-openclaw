---
summary: "Asistente de incorporación de la CLI: configuración guiada para Gateway, espacio de trabajo, canales y Skills"
read_when:
  - Al ejecutar o configurar el asistente de incorporación
  - Al configurar una nueva máquina
title: "Asistente de incorporación (CLI)"
sidebarTitle: "Incorporación: CLI"
---

# Asistente de incorporación (CLI)

El asistente de incorporación es la forma **recomendada** de configurar OpenClaw en macOS,
Linux o Windows (vía WSL2; muy recomendado).
Configura un Gateway local o una conexión a un Gateway remoto, además de canales, Skills
y valores predeterminados del espacio de trabajo en un único flujo guiado.

```bash
openclaw onboard
```

<Info>
El primer chat más rápido: abra la IU de Control (no se requiere configuración de canales). Ejecute
`openclaw dashboard` y chatee en el navegador. Documentación: [Dashboard](/web/dashboard).
</Info>

Para reconfigurar más adelante:

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` no implica un modo no interactivo. Para scripts, use `--non-interactive`.
</Note>

<Tip>
Recomendado: configure una clave de API de Brave Search para que el agente pueda usar `web_search`
(`web_fetch` funciona sin clave). La ruta más sencilla: `openclaw configure --section web`,
que almacena `tools.web.search.apiKey`. Documentación: [Herramientas web](/tools/web).
</Tip>

## Inicio rápido vs Avanzado

El asistente comienza con **Inicio rápido** (valores predeterminados) vs **Avanzado** (control total).

<Tabs>
  <Tab title="QuickStart (defaults)">
    - Gateway local (local loopback)
    - Espacio de trabajo predeterminado (o espacio de trabajo existente)
    - Puerto del Gateway **18789**
    - Autenticación del Gateway **Token** (generado automáticamente, incluso en loopback)
    - Exposición por Tailscale **Desactivada**
    - Los mensajes directos de Telegram + WhatsApp se configuran de forma predeterminada con **lista de permitidos** (se le pedirá su número de teléfono)
  
</Tab>
  <Tab title="Advanced (full control)">
    - Expone cada paso (modo, espacio de trabajo, Gateway, canales, daemon, Skills).
  
</Tab>
</Tabs>

## Qué configura el asistente

El **modo local (predeterminado)** le guía por estos pasos:

1. **Modelo/Autenticación** — Clave de API de Anthropic (recomendado), OAuth, OpenAI u otros proveedores. Elija un modelo predeterminado.
2. **Espacio de trabajo** — Ubicación para los archivos del agente (predeterminado `~/.openclaw/workspace`). Inicializa archivos de arranque.
3. **Gateway** — Puerto, dirección de enlace, modo de autenticación, exposición por Tailscale.
4. **Canales** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles o iMessage.
5. **Daemon** — Instala un LaunchAgent (macOS) o una unidad de usuario systemd (Linux/WSL2).
6. **Comprobación de estado** — Inicia el Gateway y verifica que esté en ejecución.
7. **Skills** — Instala Skills recomendadas y dependencias opcionales.

<Note>
Volver a ejecutar el asistente **no** borra nada a menos que usted elija explícitamente **Restablecer** (o pase `--reset`).
Si la configuración no es válida o contiene claves heredadas, el asistente le pedirá que ejecute `openclaw doctor` primero.
</Note>

El **modo remoto** solo configura el cliente local para conectarse a un Gateway en otro lugar.
**No** instala ni cambia nada en el host remoto.

## Agregar otro agente

Use `openclaw agents add <name>` para crear un agente independiente con su propio espacio de trabajo,
sesiones y perfiles de autenticación. Ejecutar sin `--workspace` inicia el asistente.

Qué establece:

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

Notas:

- Los espacios de trabajo predeterminados siguen `~/.openclaw/workspace-<agentId>`.
- Agregue `bindings` para enrutar mensajes entrantes (el asistente puede hacerlo).
- Banderas no interactivas: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Referencia completa

Para desgloses detallados paso a paso, scripting no interactivo, configuración de Signal,
API RPC y una lista completa de campos de configuración que escribe el asistente, consulte la
[Referencia del asistente](/reference/wizard).

## Documentos relacionados

- Referencia de comandos de la CLI: [`openclaw onboard`](/cli/onboard)
- Resumen de onboarding: [Onboarding Overview](/start/onboarding-overview)
- Incorporación de la app de macOS: [Onboarding](/start/onboarding)
- Ritual de primera ejecución del agente: [Inicialización del agente](/start/bootstrapping)


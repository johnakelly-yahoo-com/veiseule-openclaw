---
summary: "OpenClaw es un Gateway multicanal para agentes de IA que se ejecuta en cualquier sistema operativo."
read_when:
  - Presentar OpenClaw a personas nuevas
title: "OpenClaw"
---

# OpenClaw 🦞

<p align="center">
    <img
        src="/assets/openclaw-logo-text-dark.png"
        alt="OpenClaw"
        width="500"
        class="dark:hidden"></img>
    <img
        src="/assets/openclaw-logo-text.png"
        alt="OpenClaw"
        width="500"
        class="hidden dark:block"></img>
</p>

> _"¡EXFOLIA! ¡EXFOLIA!"_ — Una langosta espacial, probablemente

<p align="center"><strong>Gateway para cualquier sistema operativo para agentes de IA en WhatsApp, Telegram, Discord, iMessage y más.</strong><br />
  Envíe un mensaje y obtenga la respuesta de un agente desde su bolsillo. Los plugins agregan Mattermost y más.
</p>

<Columns>
  <Card title="Get Started" href="/start/getting-started" icon="rocket">
    Instale OpenClaw y ponga en marcha el Gateway en minutos.
  
</Card>
  <Card title="Run the Wizard" href="/start/wizard" icon="sparkles">
    Configuración guiada con `openclaw onboard` y flujos de emparejamiento.
  
</Card>
  <Card title="Open the Control UI" href="/web/control-ui" icon="layout-dashboard">
    Inicie el panel del navegador para chat, configuración y sesiones.
  
</Card>
</Columns>

## ¿Qué es OpenClaw?

OpenClaw es un **Gateway autoalojado** que conecta sus aplicaciones de chat favoritas — WhatsApp, Telegram, Discord, iMessage y más — con agentes de codificación de IA como Pi. Usted ejecuta un único proceso del Gateway en su propia máquina (o en un servidor), y este se convierte en el puente entre sus aplicaciones de mensajería y un asistente de IA siempre disponible.

**¿Para quién es?** Desarrolladores y usuarios avanzados que desean un asistente de IA personal al que puedan escribir desde cualquier lugar, sin ceder el control de sus datos ni depender de un servicio alojado.

**¿Qué lo hace diferente?**

- **Autoalojado**: se ejecuta en su hardware, bajo sus reglas
- **Multicanal**: un solo Gateway sirve WhatsApp, Telegram, Discord y más de forma simultánea
- **Nativo para agentes**: diseñado para agentes de codificación con uso de herramientas, sesiones, memoria y enrutamiento multiagente
- **Código abierto**: con licencia MIT, impulsado por la comunidad

**¿Qué necesita?** Node 22+, una clave de API (se recomienda Anthropic) y 5 minutos.

## Cómo funciona

```mermaid
flowchart LR
  A["Chat apps + plugins"] --> B["Gateway"]
  B --> C["Pi agent"]
  B --> D["CLI"]
  B --> E["Web Control UI"]
  B --> F["macOS app"]
  B --> G["iOS and Android nodes"]
```

El Gateway es la única fuente de verdad para las sesiones, el enrutamiento y las conexiones de canales.

## Capacidades clave

<Columns>
  <Card title="Multi-channel gateway" icon="network">
    WhatsApp, Telegram, Discord e iMessage con un solo proceso del Gateway.
  
</Card>
  <Card title="Plugin channels" icon="plug">
    Agregue Mattermost y más con paquetes de extensión.
  
</Card>
  <Card title="Multi-agent routing" icon="route">
    Sesiones aisladas por agente, espacio de trabajo o remitente.
  
</Card>
  <Card title="Media support" icon="image">
    Envíe y reciba imágenes, audio y documentos.
  
</Card>
  <Card title="Web Control UI" icon="monitor">
    Panel del navegador para chat, configuración, sesiones y nodos.
  
</Card>
  <Card title="Mobile nodes" icon="smartphone">
    Empareje nodos iOS y Android con soporte de Canvas.
  
</Card>
</Columns>

## Inicio rápido

<Steps>
  <Step title="Install OpenClaw">
    ```bash
    npm install -g openclaw@latest
    ```
  
</Step>
  <Step title="Onboard and install the service">
    ```bash
    openclaw onboard --install-daemon
    ```
  
</Step>
  <Step title="Pair WhatsApp and start the Gateway">
    ```bash
    openclaw channels login
    openclaw gateway --port 18789
    ```
  
</Step>
</Steps>

¿Necesita la instalación completa y la configuración de desarrollo? Consulte [Inicio rápido](/start/quickstart).

## Panel

Abra la IU de control en el navegador después de que se inicie el Gateway.

- Predeterminado local: [http://127.0.0.1:18789/](http://127.0.0.1:18789/)
- Acceso remoto: [Superficies web](/web) y [Tailscale](/gateway/tailscale)

<p align="center">
  <img src="whatsapp-openclaw.jpg" alt="OpenClaw" width="420"></img>
</p>

## Configuración (opcional)

La configuración se encuentra en `~/.openclaw/openclaw.json`.

- Si **no hace nada**, OpenClaw utiliza el binario de Pi incluido en modo RPC con sesiones por remitente.
- Si desea restringirlo, comience con `channels.whatsapp.allowFrom` y (para grupos) las reglas de menciones.

Ejemplo:

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  messages: { groupChat: { mentionPatterns: ["@openclaw"] } },
}
```

## Empiece aquí

<Columns>
  <Card title="Docs hubs" href="/start/hubs" icon="book-open">
    Toda la documentación y guías, organizadas por caso de uso.
  
</Card>
  <Card title="Configuration" href="/gateway/configuration" icon="settings">
    Ajustes principales del Gateway, tokens y configuración de proveedores.
  
</Card>
  <Card title="Remote access" href="/gateway/remote" icon="globe">
    Patrones de acceso por SSH y tailnet.
  
</Card>
  <Card title="Channels" href="/channels/telegram" icon="message-square">
    Configuración específica por canal para WhatsApp, Telegram, Discord y más.
  
</Card>
  <Card title="Nodes" href="/nodes" icon="smartphone">
    Nodos iOS y Android con emparejamiento y Canvas.
  
</Card>
  <Card title="Help" href="/help" icon="life-buoy">
    Correcciones comunes y punto de entrada a la solución de problemas.
  
</Card>
</Columns>

## Aprenda más

<Columns>
  <Card title="Full feature list" href="/concepts/features" icon="list">
    Capacidades completas de canales, enrutamiento y medios.
  
</Card>
  <Card title="Multi-agent routing" href="/concepts/multi-agent" icon="route">
    Aislamiento por espacio de trabajo y sesiones por agente.
  
</Card>
  <Card title="Security" href="/gateway/security" icon="shield">
    Tokens, listas de permitidos y controles de seguridad.
  
</Card>
  <Card title="Troubleshooting" href="/gateway/troubleshooting" icon="wrench">
    Diagnósticos del Gateway y errores comunes.
  
</Card>
  <Card title="About and credits" href="/reference/credits" icon="info">
    Orígenes del proyecto, colaboradores y licencia.
  
</Card>
</Columns>


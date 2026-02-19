---
summary: "Compatibilidad heredada de iMessage mediante imsg (JSON-RPC sobre stdio). Las nuevas configuraciones deben usar BlueBubbles."
read_when:
  - Configurar compatibilidad con iMessage
  - Depurar envío/recepción de iMessage
title: "iMessage"
---

# iMessage (legado: imsg)

<Warning>
**Recomendado:** Use [BlueBubbles](/channels/bluebubbles) para nuevas configuraciones de iMessage.

El canal `imsg` es una integración heredada de CLI externa y puede eliminarse en una versión futura. 
</Warning>

Estado: integración heredada de CLI externa. El Gateway inicia `imsg rpc` (JSON-RPC sobre stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">Configure iMessage e inicie el gateway.
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Los MD de iMessage usan el modo de emparejamiento por defecto.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Referencia completa de campos de iMessage.
  
</Card>
</CardGroup>

## Configuración (ruta rápida)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="Configurar OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Iniciar gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Aprobar el primer emparejamiento de MD (dmPolicy predeterminado)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            Las solicitudes de emparejamiento caducan después de 1 hora.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">`channels.imessage.cliPath` puede apuntar a cualquier comando que proxee stdin/stdout (por ejemplo, un script envoltorio que use SSH a otro Mac y ejecute `imsg rpc`).

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Configuración recomendada cuando los archivos adjuntos están habilitados:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    Si `remoteHost` no está configurado, OpenClaw intenta detectarlo automáticamente analizando el comando SSH en su script envoltorio.
    ```

  
</Tab>
</Tabs>

## Requisitos y permisos (macOS)

- Asegúrese de que Mensajes haya iniciado sesión en este Mac.
- Acceso total al disco para OpenClaw + `imsg` (acceso a la base de datos de Mensajes).
- Permiso de Automatización al enviar.

<Tip>
macOS concede permisos TCC por contexto de app/proceso. Apruebe los avisos en el mismo contexto que ejecuta `imsg` (por ejemplo, Terminal/iTerm, una sesión de LaunchAgent o un proceso iniciado por SSH).

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Control de acceso y enrutamiento

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` controla los mensajes directos:


    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (predeterminado: lista de permitidos).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom`: lista de permitidos de remitentes de grupo.

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - Los MD usan enrutamiento directo; los grupos usan enrutamiento de grupo.
    - Con el valor predeterminado `session.dmScope=main`, los MD de iMessage se consolidan en la sesión principal del agente.
    - Las sesiones de grupo están aisladas (`agent:<agentId>Grupos:<chat_id>`).
    - Las respuestas se enrutan de vuelta a iMessage utilizando los metadatos originales de canal/destino.

    ```
    Si llega un hilo con múltiples participantes con `is_group=false`, aún puede aislarlo `chat_id` usando `channels.imessage.groups` (consulte “Hilos tipo grupo” más abajo).
    ```

  
</Tab>
</Tabs>

## Patrones de implementación

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Si desea que el bot envíe desde una **identidad de iMessage separada** (y mantener limpios sus Mensajes personales), use un Apple ID dedicado + un usuario macOS dedicado.

    ```
    Abra Mensajes en ese usuario macOS e inicie sesión en iMessage usando el Apple ID del bot.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Topología común:


    ```
    Si el Gateway se ejecuta en un host/VM Linux pero iMessage debe ejecutarse en un Mac, Tailscale es el puente más sencillo: el Gateway se comunica con el Mac a través de la tailnet, ejecuta `imsg` por SSH y recupera los adjuntos por SCP.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    Usa claves SSH para que tanto SSH como SCP no requieran interacción.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">Canal de iMessage respaldado por `imsg` en macOS.

    ```
    Cada cuenta puede sobrescribir campos como `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` y la configuración del historial.
    ```

  
</Accordion>
</AccordionGroup>

## Medios, fragmentación y destinos de entrega

<AccordionGroup>
  <Accordion title="Attachments and media">Las cargas de medios están limitadas por `channels.imessage.mediaMaxMb` (predeterminado 16).
</Accordion>

  <Accordion title="Outbound chunking">Fragmentación opcional por saltos de línea: configure `channels.imessage.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de la fragmentación por longitud.
</Accordion>

  <Accordion title="Addressing formats">
    Destinos explícitos preferidos:


    ```
    - `chat_id:123` (recomendado para un enrutamiento estable)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Los destinos por identificador también son compatibles:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Escrituras de configuración

De forma predeterminada, iMessage puede escribir actualizaciones de configuración activadas por `/config set|unset` (requiere `commands.config: true`).

Desactivar con:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Resolución de problemas

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Validar el binario y la compatibilidad con RPC:


```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Si la verificación indica que RPC no es compatible, actualiza `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Lista de verificación:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (predeterminado: emparejamiento).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">Apruebe mediante:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Notas:

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    Vuelve a ejecutar en un terminal GUI interactivo en el mismo contexto de usuario/sesión y aprueba las solicitudes:


```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    **Automatización → Mensajes**: permita que el proceso que ejecuta OpenClaw (y/o su terminal) controle **Messages.app** para envíos salientes.
    ```

  
</Accordion>
</AccordionGroup>

## Punteros de referencia de configuración

- Referencia de configuración (iMessage)
- Configuración completa: [Configuración](/gateway/configuration)
- Detalles: [Emparejamiento](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)


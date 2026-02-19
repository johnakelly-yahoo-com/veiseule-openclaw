---
summary: "Resumen de configuración: tareas comunes, configuración rápida y enlaces a la referencia completa"
read_when:
  - Configurar OpenClaw por primera vez
  - Buscar patrones de configuración comunes
  - Navegar a secciones específicas de la configuración
title: "Configuración"
---

# Configuración 🔧

OpenClaw lee una configuración **JSON5** opcional desde `~/.openclaw/openclaw.json` (se permiten comentarios y comas finales).

Si el archivo falta, OpenClaw usa valores predeterminados razonablemente seguros (agente Pi integrado + sesiones por remitente + espacio de trabajo `~/.openclaw/workspace`). Por lo general, solo necesita una configuración para:

- Conectar canales y controlar quién puede enviar mensajes al bot
- Configurar modelos, herramientas, aislamiento (sandboxing) o automatización (cron, hooks)
- Ajustar sesiones, medios, red o interfaz de usuario

Consulta la [referencia completa](/gateway/configuration-reference) para ver todos los campos disponibles.

<Tip>
`openclaw --profile <name> …` → usa `~/.openclaw-<name>` (puerto a través de config/env/flags)
</Tip>

## Configuración mínima (punto de partida recomendado)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Editar la configuración

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # asistente completo de configuración
    openclaw configure     # asistente de configuración
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    El Gateway expone una representación JSON Schema de la configuración mediante `config.schema` para editores de UI.
    La UI de Control renderiza un formulario a partir de este esquema, con un editor de **JSON sin procesar** como vía de escape.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`) El Gateway supervisa el archivo y aplica los cambios automáticamente (consulta [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Validación estricta de configuración

<Warning>
OpenClaw solo acepta configuraciones que coincidan completamente con el esquema. Claves desconocidas, tipos malformados o valores inválidos hacen que el Gateway **se niegue a iniciar** por seguridad. La única excepción a nivel raíz es `$schema` (string), para que los editores puedan adjuntar metadatos de JSON Schema.
</Warning>

Cuando falla la validación:

- El Gateway no arranca.
- Solo se permiten comandos de diagnóstico (por ejemplo: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Ejecute `openclaw doctor` para ver los problemas exactos.
- Ejecute `openclaw doctor --fix` (o `--yes`) para aplicar migraciones/reparaciones.

## Tareas comunes

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Cada canal tiene su propia sección de configuración en `channels.<provider>`. Consulta la página dedicada de cada canal para ver los pasos de configuración:

    ```
    {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org.com"],
        },
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            GUILD_ID: {
              channels: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          channels: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">
    Configura el modelo principal y los posibles modelos de respaldo opcionales:

    ```
    {
      agents: {
        defaults: {
          models: {
            "anthropic/claude-sonnet-4-5-20250929": {
              params: { temperature: 0.6 },
            },
            "openai/gpt-5. ": {
              parámetros: { maxTokens: 8192 },
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">
    El acceso por DM se controla por canal mediante `dmPolicy`:

    ```
    - `"pairing"` (predeterminado): los remitentes desconocidos reciben un código de emparejamiento de un solo uso para aprobar
    - `"allowlist"`: solo los remitentes en `allowFrom` (o en el almacén de permitidos emparejados)
    - `"open"`: permite todos los DM entrantes (requiere `allowFrom: ["*"]`)
    - `"disabled"`: ignora todos los DM
    
    Para grupos, usa `groupPolicy` + `groupAllowFrom` o listas de permitidos específicas del canal.
    
    Consulta la [referencia completa](/gateway/configuration-reference#dm-and-group-access) para ver los detalles por canal.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Los mensajes de grupo requieren **mención obligatoria** de forma predeterminada (ya sea mención por metadatos o patrones regex). `agents.defaults.subagents` configura los valores predeterminados de sub-agente:

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    Las sesiones controlan la continuidad y el aislamiento de las conversaciones:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // recomendado para múltiples usuarios
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (compartido) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Consulta [Session Management](/concepts/session) para ver el alcance, los vínculos de identidad y la política de envío.
    - Consulta la [referencia completa](/gateway/configuration-reference#session) para todos los campos.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    Ejecuta sesiones de agente en contenedores Docker aislados:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    ```
    - `every`: cadena de duración (`30m`, `2h`). Establece `0m` para desactivar.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Consulta [Heartbeat](/gateway/heartbeat) para la guía completa.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    Ver [Trabajos Cronales](/automation/cron-jobs) para ver el resumen de características y ejemplos de CLI.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Habilitar un simple punto final de webhook HTTP en el servidor HTTP de Gateway.

    ```
    {
      hooks: {
        actived: true,
        token: "shared-secret",
        ruta: "/hooks",
        presets: ["gmail"],
        transformsDir: "~/. penclaw/ganchos",
        mapeos: [
          {
            match: { path: "gmail" },
            acción: "agente",
            wakeMode: "ahora",
            nombre: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "De: {{messages[0].from}}\nAsunto: {{messages[0].subject}}\n{{messages[0].snippet}}",
            deliver: true,
            channel: "last",
            modelo: "openai/gpt-5. -mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Ejecute múltiples agentes aislados (espacio de trabajo separado, `agentDir`, sesiones) dentro de un Gateway.

    ```
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
      channels: {
        whatsapp: {
          accounts: {
            personal: {},
            biz: {},
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Divida su configuración en varios archivos usando la directiva `$include`. Esto es útil para:

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (Configurar recarga caliente)

El Gateway reproduce `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`) y aplica los cambios automáticamente.

### Modos de recarga

| Modos:                           | Comportamiento                                                                                                                                                                                       |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (predeterminado) | Aplica en caliente los cambios seguros al instante. Reinicia automáticamente para los críticos.                                                                      |
| `{modelFull}`                                    | `hot`: sólo aplicar cambios hot-safe; log cuando se requiere un reinicio. Registra una advertencia cuando se necesita un reinicio — tú lo gestionas. |
| **Sustitución en línea:**        | `restart`: reinicie el Gateway en cualquier cambio de configuración.                                                                                                 |
| `{provider}`                                     | Desactiva la supervisión de archivos. Los cambios surten efecto en el siguiente reinicio manual.                                                                     |

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### Qué se aplica en caliente vs qué necesita reinicio

La mayoría de los campos se aplican en caliente sin tiempo de inactividad. En modo `hybrid`, los cambios que requieren reinicio se gestionan automáticamente.

| Categoría                            | Campos:                                                                                                   | ¿Requiere reinicio?                   |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — todos los canales integrados y de extensión                           | No                                    |
| Agente y modelos                     | `model`: modelo predeterminado por agente, anula `agents.defaults.model` para ese agente. | No                                    |
| Automatización                       | `hooks`, `cron`, `agent.heartbeat`                                                                                        | No                                    |
| messages                             | `messages`                                                                                                                | No                                    |
| Herramientas y medios                | `tools`, `browser`, `skills`, `audio`, `talk`                                                                             | No                                    |
| UI y varios                          | `ui`, `logging`, `identity`, `bindings`                                                                                   | No                                    |
| Servidor Gateway                     | `gateway` (port/bind/auth/control UI/tailscale)                                                        | **Reglas:**           |
| Infraestructura                      | `discovery`, `canvasHost`, `plugins`                                                                                      | **Tipos de mención:** |

<Note>
`gateway.reload` y `gateway.remote` son excepciones — cambiarlos **no** provoca un reinicio.
</Note>

## Actualizaciones parciales (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Use `config.apply` para validar y escribir la configuración completa y reiniciar el Gateway en un solo paso.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">Use `config.patch` para fusionar una actualización parcial en la configuración existente sin sobrescribir
claves no relacionadas. Aplica semántica de JSON merge patch:

    ````
    - Los objetos se combinan de forma recursiva
    - `null` elimina una clave
    - Los arrays se reemplazan
    
    Parámetros:
    
    - `raw` (string) — JSON5 solo con las claves a cambiar
    - `baseHash` (obligatorio) — hash de configuración de `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — igual que en `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Variable

OpenClaw lee las variables de entorno del proceso padre además de:

- `.env` desde el directorio de trabajo actual (si existe)
- un respaldo global `.env` desde `~/.openclaw/.env` (también conocido como `$OPENCLAW_STATE_DIR/.env`)

Ninguno de los archivos `.env` sobrescribe variables de entorno existentes. También puedes definir variables de entorno en línea en la configuración:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

<Accordion title="Shell env import (optional)">Comodidad opcional: si está habilitado y aún no se estableció ninguna de las claves esperadas, OpenClaw ejecuta su shell de inicio de sesión e importa solo las claves esperadas faltantes (nunca sobrescribe).

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">Puede referenciar variables de entorno directamente en cualquier valor de cadena de la configuración usando
la sintaxis `${VAR_NAME}`.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

Reglas:

- Solo se reconocen nombres de variables en mayúsculas: `[A-Z_][A-Z0-9_]*`
- Las variables faltantes o vacías generan un error en el momento de la carga
- Escape con `$${VAR}` para producir un `${VAR}` literal
- Funciona con `$include` (los archivos incluidos también reciben sustitución)
- Sustitución en línea: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Vea [/environment](/help/environment) para la precedencia y las fuentes completas.

## Referencia completa

**¿Nuevo en la configuración?** Consulte la guía de [Ejemplos de configuración](/gateway/configuration-examples) para ver ejemplos completos con explicaciones detalladas.

---

Ejemplo: Configuración legal multi‑cliente

---
summary: "Visão geral da configuração: tarefas comuns, configuração rápida e links para a referência completa"
read_when:
  - Configurando o OpenClaw pela primeira vez
  - Procurando padrões comuns de configuração
  - Navegando para seções específicas da configuração
title: "Configuração"
---

# Configuration 🔧

OpenClaw reads an optional **JSON5** config from `~/.openclaw/openclaw.json` (comments + trailing commas allowed).

If the file is missing, OpenClaw uses safe-ish defaults (embedded Pi agent + per-sender sessions + workspace `~/.openclaw/workspace`). You usually only need a config to:

- Conecte canais e controle quem pode enviar mensagens ao bot
- Defina modelos, ferramentas, sandboxing ou automações (cron, hooks)
- Ajuste sessões, mídia, rede ou UI

Veja a [referência completa](/gateway/configuration-reference) para todos os campos disponíveis.

<Tip>
**New to configuration?** Check out the [Configuration Examples](/gateway/configuration-examples) guide for complete examples with detailed explanations!
</Tip>

## Minimal config (recommended starting point)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Configuration 🔧

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # full setup wizard
openclaw configure     # config wizard
```
</Tab>
  <Tab title="CLI (one-liners)">```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```
</Tab>
  <Tab title="Control UI">
    The Gateway exposes a JSON Schema representation of the config via `config.schema` for UI editors.
    The Control UI renders a form from this schema, with a **Raw JSON** editor as an escape hatch.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`) O Gateway monitora o arquivo e aplica as alterações automaticamente (veja [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Strict config validation

<Warning>
OpenClaw only accepts configurations that fully match the schema. Unknown keys, malformed types, or invalid values cause the Gateway to **refuse to start** for safety. A única exceção no nível raiz é `$schema` (string), para que editores possam associar metadados de JSON Schema.
</Warning>

When validation fails:

- The Gateway does not boot.
- Only diagnostic commands are allowed (for example: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Run `openclaw doctor` to see the exact issues.
- Run `openclaw doctor --fix` (or `--yes`) to apply migrations/repairs.

## Tarefas comuns

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">    Cada canal tem sua própria seção de configuração em `channels.<provider>`. Veja a página dedicada do canal para as etapas de configuração:

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

  <Accordion title="Choose and configure models">Defina o modelo principal e fallbacks opcionais:

    ```
    {
      agents: {
        defaults: {
          cliBackends: {
            "claude-cli": {
              command: "/opt/homebrew/bin/claude",
            },
            "my-cli": {
              command: "my-cli",
              args: ["--json"],
              output: "json",
              modelArg: "--model",
              sessionArg: "--session",
              sessionMode: "existing",
              systemPromptArg: "--system",
              systemPromptWhen: "first",
              imageArg: "--image",
              imageMode: "repeat",
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">Per-DM override: `channels.<provider>

    ```
    Groups: `channels.mattermost.groupPolicy="allowlist"` by default (mention-gated). Use `channels.mattermost.groupAllowFrom` to restrict senders.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Group messages default to **require mention** (either metadata mention or regex patterns). `agents.defaults.subagents` configures sub-agent defaults:

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

  <Accordion title="Configure sessions and resets">Thread session isolation:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // recommended for multi-user
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (shared) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - See [Session Management](/concepts/session) for scoping, identity links, and send policy.
    - See [full reference](/gateway/configuration-reference#session) for all fields.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">    Execute sessões de agente em contêineres Docker isolados:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">```json5
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
    - `every`: string de duração (`30m`, `2h`). Defina `0m` para desativar.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Veja [Heartbeat](/gateway/heartbeat) para o guia completo.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Enable a simple HTTP webhook endpoint on the Gateway HTTP server.

    ```
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        presets: ["gmail"],
        transformsDir: "~/.openclaw/hooks",
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            wakeMode: "now",
            name: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
            deliver: true,
            channel: "last",
            model: "openai/gpt-5.2-mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Run multiple isolated agents (separate workspace, `agentDir`, sessions) inside one Gateway.

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

  <Accordion title="Split config into multiple files ($include)">Split your config into multiple files using the `$include` directive. This is useful for:

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

## `gateway.reload` (Config hot reload)

The Gateway watches `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`) and applies changes automatically.

### Modos de recarregamento

| Modes:                   | Merge behavior                                                                                                                                                                          |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (padrão) | Aplica alterações seguras instantaneamente. Reinicia automaticamente para alterações críticas.                                                          |
| **`hot`**                                | `hot`: only apply hot-safe changes; log when a restart is required. Registra um aviso quando é necessário reiniciar — você cuida disso. |
| **`restart`**                            | `restart`: restart the Gateway on any config change.                                                                                                    |
| **Inline substitution:** | Desativa o monitoramento de arquivos. As alterações entram em vigor no próximo reinício manual.                                                         |

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

### O que é aplicado a quente vs. o que precisa de reinício

A maioria dos campos é aplicada a quente sem indisponibilidade. No modo `hybrid`, alterações que exigem reinício são tratadas automaticamente.

| Categoria                            | Fields:                                                                                             | Precisa reiniciar?                 |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — todos os canais integrados e de extensão                        | Não                                |
| Agente e modelos                     | `model`: per-agent default model, overrides `agents.defaults.model` for that agent. | Não                                |
| Automação                            | `hooks`, `cron`, `agent.heartbeat`                                                                                  | Não                                |
| messages                             | `messages`                                                                                                          | Não                                |
| Ferramentas e mídia                  | `tools`, `browser`, `skills`, `audio`, `talk`                                                                       | Não                                |
| UI e diversos                        | `ui`, `logging`, `identity`, `bindings`                                                                             | Não                                |
| Servidor Gateway                     | `gateway` (port/bind/auth/control UI/tailscale)                                                  | **Rules:**         |
| Infraestrutura                       | `discovery`, `canvasHost`, `plugins`                                                                                | **Mention types:** |

<Note>
`gateway.reload` e `gateway.remote` são exceções — alterá-los **não** aciona um reinício.
</Note>

## Partial updates (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Use `config.apply` to validate + write the full config and restart the Gateway in one step.

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

  <Accordion title="config.patch (partial update)">Use `config.patch` to merge a partial update into the existing config without clobbering
unrelated keys. It applies JSON merge patch semantics:

    ````
    - Objetos são mesclados recursivamente
    - `null` remove uma chave
    - Arrays substituem
    
    Params:
    
    - `raw` (string) — JSON5 apenas com as chaves a serem alteradas
    - `baseHash` (obrigatório) — hash da configuração de `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — iguais a `config.apply`
    
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

OpenClaw reads env vars from the parent process (shell, launchd/systemd, CI, etc.).

- `.env` from the current working directory (if present)
- a global fallback `.env` from `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`)

Neither `.env` file overrides existing env vars. You can also provide inline env vars in config.

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

<Accordion title="Shell env import (optional)">Opt-in convenience: if enabled and none of the expected keys are set yet, OpenClaw runs your login shell and imports only the missing expected keys (never overrides).

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

<Accordion title="Env var substitution in config values">You can reference environment variables directly in any config string value using
`${VAR_NAME}` syntax.

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

Defaults:

- Only uppercase env var names are matched: `[A-Z_][A-Z0-9_]*`
- Missing or empty env vars throw an error at config load
- Escape with `$${VAR}` to output a literal `${VAR}`
- Works with `$include` (included files also get substitution)
- Substituição inline: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

See [/environment](/help/environment) for full precedence and sources.

## Referência completa

Config Includes (`$include`)

---

Example (provider/model-specific allowlist):


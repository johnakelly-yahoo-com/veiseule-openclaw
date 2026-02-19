---
title: IRC
description: Conecta OpenClaw a canales de IRC y mensajes directos.
---

Usa IRC cuando quieras OpenClaw en canales clásicos (`#room`) y mensajes directos.
IRC se distribuye como un plugin de extensión, pero se configura en la configuración principal bajo `channels.irc`.

## Inicio rápido

1. Activa la configuración de IRC en `~/.openclaw/openclaw.json`.
2. Configura al menos:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Inicia/reinicia el gateway:

```bash
openclaw gateway run
```

## Valores de seguridad predeterminados

- `channels.irc.dmPolicy` tiene como valor predeterminado `"pairing"`.
- `channels.irc.groupPolicy` tiene como valor predeterminado `"allowlist"`.
- Con `groupPolicy="allowlist"`, establece `channels.irc.groups` para definir los canales permitidos.
- Usa TLS (`channels.irc.tls=true`) salvo que aceptes intencionadamente transporte en texto plano.

## Control de acceso

Hay dos “puertas” independientes para los canales de IRC:

1. **Acceso al canal** (`groupPolicy` + `groups`): si el bot acepta mensajes de un canal en absoluto.
2. **Acceso del remitente** (`groupAllowFrom` / por canal `groups["#channel"].allowFrom`): quién puede activar el bot dentro de ese canal.

Claves de configuración:

- Lista de permitidos para DM (acceso del remitente en DM): `channels.irc.allowFrom`
- Lista de permitidos de remitentes en grupo (acceso del remitente en el canal): `channels.irc.groupAllowFrom`
- Controles por canal (canal + remitente + reglas de mención): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` permite canales no configurados (**aun así, por defecto requieren mención**)

Las entradas de la lista de permitidos pueden usar el nick o el formato `nick!user@host`.

### Error común: `allowFrom` es para DM, no para canales

Si ves registros como:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…significa que el remitente no estaba permitido para mensajes de **grupo/canal**. Arréglalo haciendo una de las siguientes acciones:

- estableciendo `channels.irc.groupAllowFrom` (global para todos los canales), o
- estableciendo listas de remitentes permitidos por canal: `channels.irc.groups["#channel"].allowFrom`

Ejemplo (permitir que cualquiera en `#tuirc-dev` hable con el bot):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Activación de respuestas (menciones)

Aunque un canal esté permitido (mediante `groupPolicy` + `groups`) y el remitente esté autorizado, OpenClaw usa por defecto **restricción por mención** en contextos de grupo.

Eso significa que puedes ver registros como `drop channel … (missing-mention)` a menos que el mensaje incluya un patrón de mención que coincida con el bot.

Para que el bot responda en un canal de IRC **sin necesidad de una mención**, desactiva la restricción por mención para ese canal:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

O para permitir **todos** los canales IRC (sin lista de permitidos por canal) y aun así responder sin menciones:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Nota de seguridad (recomendado para canales públicos)

Si permites `allowFrom: ["*"]` en un canal público, cualquiera puede enviar solicitudes al bot.
Para reducir el riesgo, restringe las herramientas para ese canal.

### Las mismas herramientas para todos en el canal

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Herramientas diferentes según el remitente (el propietario tiene más permisos)

Usa `toolsBySender` para aplicar una política más estricta a `"*"` y una más flexible a tu nick:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Notas:

- Las claves de `toolsBySender` pueden ser un nick (p. ej., `"eigen"`) o un hostmask completo (`"eigen!~eigen@174.127.248.171"`) para una coincidencia de identidad más sólida.
- Se aplica la primera política de remitente que coincida; `"*"` es el comodín por defecto.

Para más información sobre el acceso por grupo frente al filtrado por menciones (y cómo interactúan), consulta: [/channels/groups](/channels/groups).

## NickServ

Para identificarte con NickServ después de conectar:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Registro opcional único al conectar:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Desactiva `register` después de que el nick esté registrado para evitar intentos repetidos de REGISTER.

## Variables de entorno

La cuenta predeterminada admite:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (separados por comas)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Solución de problemas

- Si el bot se conecta pero nunca responde en los canales, verifica `channels.irc.groups` **y** si el filtrado por menciones está descartando mensajes (`missing-mention`). Si quieres que responda sin pings, establece `requireMention:false` para el canal.
- Si el inicio de sesión falla, verifica la disponibilidad del nick y la contraseña del servidor.
- Si TLS falla en una red personalizada, verifica el host/puerto y la configuración del certificado.

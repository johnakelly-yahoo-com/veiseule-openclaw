---
title: IRC
description: Conecte o OpenClaw a canais IRC e mensagens diretas.
---

Use IRC quando quiser o OpenClaw em canais clássicos (`#room`) e mensagens diretas.
O IRC é distribuído como um plugin de extensão, mas é configurado no arquivo principal em `channels.irc`.

## Início rápido

1. Ative a configuração de IRC em `~/.openclaw/openclaw.json`.
2. Defina pelo menos:

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

3. Inicie/reinicie o gateway:

```bash
openclaw gateway run
```

## Padrões de segurança

- `channels.irc.dmPolicy` tem como padrão `"pairing"`.
- `channels.irc.groupPolicy` tem como padrão `"allowlist"`.
- Com `groupPolicy="allowlist"`, defina `channels.irc.groups` para especificar os canais permitidos.
- Use TLS (`channels.irc.tls=true`) a menos que você aceite intencionalmente transporte em texto puro.

## Controle de acesso

Existem dois “níveis” separados para canais IRC:

1. **Acesso ao canal** (`groupPolicy` + `groups`): se o bot aceita mensagens de um canal.
2. **Acesso do remetente** (`groupAllowFrom` / por canal `groups["#channel"].allowFrom`): quem pode acionar o bot dentro desse canal.

Chaves de configuração:

- Lista de permissões para DM (acesso do remetente em DM): `channels.irc.allowFrom`
- Lista de permissões de remetentes em grupo (acesso do remetente no canal): `channels.irc.groupAllowFrom`
- Controles por canal (canal + remetente + regras de menção): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` permite canais não configurados (**ainda exige menção por padrão**)

Entradas da allowlist podem usar o formato nick ou `nick!user@host`.

### Erro comum: `allowFrom` é para DMs, não para canais

Se você vir logs como:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…isso significa que o remetente não estava autorizado para mensagens de **grupo/canal**. Corrija isso fazendo uma das seguintes opções:

- definindo `channels.irc.groupAllowFrom` (global para todos os canais), ou
- definindo listas de permissões por canal: `channels.irc.groups["#channel"].allowFrom`

Exemplo (permitir que qualquer pessoa em `#tuirc-dev` fale com o bot):

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

## Disparo de resposta (menções)

Mesmo que um canal seja permitido (via `groupPolicy` + `groups`) e o remetente seja autorizado, o OpenClaw por padrão **exige menção** em contextos de grupo.

Isso significa que você pode ver logs como `drop channel … (missing-mention)` a menos que a mensagem inclua um padrão de menção que corresponda ao bot.

Para fazer o bot responder em um canal IRC **sem precisar de menção**, desative a exigência de menção para esse canal:

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

Ou para permitir **todos** os canais IRC (sem allowlist por canal) e ainda responder sem menções:

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

## Nota de segurança (recomendado para canais públicos)

Se você permitir `allowFrom: ["*"]` em um canal público, qualquer pessoa poderá enviar comandos ao bot.
Para reduzir o risco, restrinja as ferramentas para esse canal.

### Mesmas ferramentas para todos no canal

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

### Ferramentas diferentes por remetente (o owner tem mais poder)

Use `toolsBySender` para aplicar uma política mais restritiva a `"*"` e uma mais permissiva ao seu nick:

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

Observações:

- As chaves de `toolsBySender` podem ser um nick (ex.: `"eigen"`) ou uma hostmask completa (`"eigen!~eigen@174.127.248.171"`) para uma correspondência de identidade mais forte.
- A primeira política de remetente correspondente prevalece; `"*"` é o curinga padrão.

Para saber mais sobre acesso por grupo vs controle por menção (e como eles interagem), veja: [/channels/groups](/channels/groups).

## NickServ

Para identificar-se com o NickServ após conectar:

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

Registro opcional único ao conectar:

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

Desative `register` após o nick ser registrado para evitar tentativas repetidas de REGISTER.

## Variáveis de ambiente

A conta padrão oferece suporte a:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (separados por vírgula)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Solução de problemas

- Se o bot conectar mas nunca responder nos canais, verifique `channels.irc.groups` **e** se o controle por menção está descartando mensagens (`missing-mention`). Se quiser que ele responda sem pings, defina `requireMention:false` para o canal.
- Se o login falhar, verifique a disponibilidade do nick e a senha do servidor.
- Se o TLS falhar em uma rede personalizada, verifique host/porta e a configuração do certificado.


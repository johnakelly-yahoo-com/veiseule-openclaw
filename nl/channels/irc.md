---
title: IRC
description: Verbind OpenClaw met IRC-kanalen en privéberichten.
---

Gebruik IRC wanneer je OpenClaw wilt inzetten in klassieke kanalen (`#room`) en privéberichten.
IRC wordt geleverd als een extensieplugin, maar wordt geconfigureerd in de hoofdconfiguratie onder `channels.irc`.

## Snel starten

1. Schakel de IRC-configuratie in `~/.openclaw/openclaw.json` in.
2. Stel minimaal het volgende in:

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

3. Start/herstart de gateway:

```bash
openclaw gateway run
```

## Standaard beveiligingsinstellingen

- `channels.irc.dmPolicy` staat standaard op `"pairing"`.
- `channels.irc.groupPolicy` staat standaard op `"allowlist"`.
- Met `groupPolicy="allowlist"` stel je `channels.irc.groups` in om toegestane kanalen te definiëren.
- Gebruik TLS (`channels.irc.tls=true`) tenzij je bewust onversleuteld transport accepteert.

## Toegangscontrole

Er zijn twee afzonderlijke “poorten” voor IRC-kanalen:

1. **Kanaaltoegang** (`groupPolicy` + `groups`): of de bot überhaupt berichten uit een kanaal accepteert.
2. **Afzenderstoegang** (`groupAllowFrom` / per-kanaal `groups["#channel"].allowFrom`): wie de bot binnen dat kanaal mag activeren.

Configuratiesleutels:

- DM-allowlist (toegang voor DM-afzenders): `channels.irc.allowFrom`
- Group sender allowlist (toegang voor kanaalafzenders): `channels.irc.groupAllowFrom`
- Per-kanaalinstellingen (kanaal + afzender + mentionregels): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` staat niet-geconfigureerde kanalen toe (**standaard nog steeds mention-gated**)

Allowlist-vermeldingen kunnen de vormen nick of `nick!user@host` gebruiken.

### Veelgemaakte fout: `allowFrom` is voor DM’s, niet voor kanalen

Als je logs ziet zoals:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…betekent dit dat de afzender niet was toegestaan voor **groep-/kanaal**berichten. Los dit op door ofwel:

- `channels.irc.groupAllowFrom` in te stellen (globaal voor alle kanalen), of
- per-kanaal allowlists voor afzenders in te stellen: `channels.irc.groups["#channel"].allowFrom`

Voorbeeld (iedereen in `#tuirc-dev` mag met de bot praten):

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

## Antwoordactivering (mentions)

Zelfs als een kanaal is toegestaan (via `groupPolicy` + `groups`) en de afzender is toegestaan, gebruikt OpenClaw standaard **mention-gating** in groepscontexten.

Dat betekent dat je logs kunt zien zoals `drop channel … (missing-mention)` tenzij het bericht een mentionpatroon bevat dat overeenkomt met de bot.

Om de bot in een IRC-kanaal te laten antwoorden **zonder dat een mention nodig is**, schakel mention-gating voor dat kanaal uit:

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

Of om **alle** IRC-kanalen toe te staan (geen allowlist per kanaal) en toch te antwoorden zonder vermeldingen:

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

## Beveiligingsopmerking (aanbevolen voor openbare kanalen)

Als je `allowFrom: ["*"]` toestaat in een openbaar kanaal, kan iedereen de bot aansturen.
Om het risico te beperken, beperk de tools voor dat kanaal.

### Dezelfde tools voor iedereen in het kanaal

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

### Verschillende tools per afzender (eigenaar krijgt meer bevoegdheden)

Gebruik `toolsBySender` om een strikter beleid toe te passen op `"*"` en een minder strikt beleid op je nick:

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

Opmerkingen:

- `toolsBySender`-sleutels kunnen een nick zijn (bijv. `"eigen"`) of een volledige hostmask (`"eigen!~eigen@174.127.248.171"`) voor sterkere identiteitscontrole.
- Het eerste overeenkomende afzenderbeleid is doorslaggevend; `"*"` is de wildcard-terugvaloptie.

Voor meer informatie over groepstoegang versus mention-gating (en hoe ze samenwerken), zie: [/channels/groups](/channels/groups).

## NickServ

Om je na het verbinden te identificeren bij NickServ:

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

Optionele eenmalige registratie bij het verbinden:

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

Schakel `register` uit nadat de nick is geregistreerd om herhaalde REGISTER-pogingen te voorkomen.

## Omgevingsvariabelen

Standaardaccount ondersteunt:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (door komma’s gescheiden)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Probleemoplossing

- Als de bot verbinding maakt maar nooit antwoordt in kanalen, controleer dan `channels.irc.groups` **en** of mention-gating berichten blokkeert (`missing-mention`). Als je wilt dat hij zonder pings antwoordt, stel dan `requireMention:false` in voor het kanaal.
- Als inloggen mislukt, controleer dan de beschikbaarheid van de nick en het serverwachtwoord.
- Als TLS mislukt op een aangepast netwerk, controleer dan host/poort en de certificaatconfiguratie.

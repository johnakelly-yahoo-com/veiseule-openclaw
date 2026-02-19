---
title: IRC
description: Forbind OpenClaw til IRC-kanaler og direkte beskeder.
---

Brug IRC, når du vil have OpenClaw i klassiske kanaler (`#room`) og direkte beskeder.
IRC leveres som et udvidelsesplugin, men konfigureres i hovedkonfigurationen under `channels.irc`.

## Hurtig start

1. Aktivér IRC-konfiguration i `~/.openclaw/openclaw.json`.
2. Angiv mindst:

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

3. Start/genstart gateway:

```bash
openclaw gateway run
```

## Standard sikkerhedsindstillinger

- `channels.irc.dmPolicy` er som standard sat til `"pairing"`.
- `channels.irc.groupPolicy` er som standard sat til `"allowlist"`.
- Med `groupPolicy="allowlist"` skal du angive `channels.irc.groups` for at definere tilladte kanaler.
- Brug TLS (`channels.irc.tls=true`), medmindre du bevidst accepterer ukrypteret transport.

## Adgangskontrol

Der er to separate “gateways” for IRC-kanaler:

1. **Kanaladgang** (`groupPolicy` + `groups`): om botten overhovedet accepterer beskeder fra en kanal.
2. **Afsenderadgang** (`groupAllowFrom` / per-kanal `groups["#channel"].allowFrom`): hvem der må aktivere botten i den kanal.

Konfigurationsnøgler:

- DM allowlist (DM-afsenderadgang): `channels.irc.allowFrom`
- Group sender allowlist (kanal-afsenderadgang): `channels.irc.groupAllowFrom`
- Per-kanal kontrol (kanal + afsender + mention-regler): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` tillader ukonfigurerede kanaler (**stadig mention-begrænset som standard**)

Allowlist-poster kan bruge nick eller `nick!user@host`-formater.

### Almindelig faldgrube: `allowFrom` gælder for DM’er, ikke kanaler

Hvis du ser logs som:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

… betyder det, at afsenderen ikke var tilladt for **gruppe-/kanal**-beskeder. Ret det ved enten at:

- sætte `channels.irc.groupAllowFrom` (globalt for alle kanaler), eller
- angive per-kanal afsender-allowlists: `channels.irc.groups["#channel"].allowFrom`

Eksempel (tillad alle i `#tuirc-dev` at tale med botten):

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

## Svarudløsning (mentions)

Selv hvis en kanal er tilladt (via `groupPolicy` + `groups`) og afsenderen er tilladt, bruger OpenClaw som standard **mention-begrænsning** i gruppesammenhænge.

Det betyder, at du kan se logs som `drop channel … (missing-mention)` medmindre beskeden indeholder et mention-mønster, der matcher botten.

For at få botten til at svare i en IRC-kanal **uden at kræve en mention**, skal du deaktivere mention-begrænsning for den kanal:

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

Eller for at tillade **alle** IRC-kanaler (ingen kanal-specifik allowlist) og stadig svare uden omtaler:

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

## Sikkerhedsnote (anbefales til offentlige kanaler)

Hvis du tillader `allowFrom: ["*"]` i en offentlig kanal, kan enhver give botten prompts.
For at reducere risikoen skal du begrænse værktøjerne for den kanal.

### Samme værktøjer for alle i kanalen

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

### Forskellige værktøjer pr. afsender (ejeren får flere rettigheder)

Brug `toolsBySender` til at anvende en strengere politik på `"*"` og en mere lempelig på dit nick:

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

Bemærk:

- `toolsBySender`-nøgler kan være et nick (f.eks. `"eigen"`) eller en fuld hostmask (`"eigen!~eigen@174.127.248.171"`) for stærkere identitetsmatch.
- Den første matchende afsenderpolitik gælder; `"*"` er wildcard-reserven.

For mere om gruppetilgang vs. mention-gating (og hvordan de interagerer), se: [/channels/groups](/channels/groups).

## NickServ

For at identificere dig med NickServ efter forbindelse:

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

Valgfri engangsregistrering ved forbindelse:

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

Deaktiver `register`, efter at nicket er registreret, for at undgå gentagne REGISTER-forsøg.

## Miljøvariabler

Standardkonto understøtter:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (kommasepareret)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Fejlfinding

- Hvis botten forbinder, men aldrig svarer i kanaler, skal du kontrollere `channels.irc.groups` **og** om mention-gating filtrerer beskeder fra (`missing-mention`). Hvis du vil have den til at svare uden pings, skal du sætte `requireMention:false` for kanalen.
- Hvis login mislykkes, skal du kontrollere nickets tilgængelighed og serveradgangskoden.
- Hvis TLS mislykkes på et brugerdefineret netværk, skal du kontrollere host/port og certifikatopsætningen.


---
title: IRC
description: Anslut OpenClaw till IRC-kanaler och direktmeddelanden.
---

Använd IRC när du vill ha OpenClaw i klassiska kanaler (`#room`) och direktmeddelanden.
IRC levereras som ett tilläggsplugin, men konfigureras i huvudkonfigurationen under `channels.irc`.

## Snabbstart

1. Aktivera IRC-konfigurationen i `~/.openclaw/openclaw.json`.
2. Ange minst:

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

3. Starta/Starta om gateway:

```bash
openclaw gateway run
```

## Säkerhetsstandarder

- `channels.irc.dmPolicy` är som standard satt till `"pairing"`.
- `channels.irc.groupPolicy` är som standard satt till `"allowlist"`.
- Med `groupPolicy="allowlist"`, ange `channels.irc.groups` för att definiera tillåtna kanaler.
- Använd TLS (`channels.irc.tls=true`) om du inte avsiktligt accepterar okrypterad överföring.

## Åtkomstkontroll

Det finns två separata ”grindar” för IRC-kanaler:

1. **Kanalåtkomst** (`groupPolicy` + `groups`): om boten överhuvudtaget accepterar meddelanden från en kanal.
2. **Avsändaråtkomst** (`groupAllowFrom` / per-kanal `groups["#channel"].allowFrom`): vem som får trigga boten i den kanalen.

Konfigurationsnycklar:

- DM-allowlist (DM-avsändaråtkomst): `channels.irc.allowFrom`
- Allowlist för gruppavsändare (kanalavsändaråtkomst): `channels.irc.groupAllowFrom`
- Per-kanal-kontroller (kanal + avsändare + omnämnanderegler): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` tillåter okonfigurerade kanaler (**fortfarande omnämnandebegränsat som standard**)

Poster i allowlist kan använda nick eller `nick!user@host`-format.

### Vanlig fallgrop: `allowFrom` gäller för DM, inte kanaler

Om du ser loggar som:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…betyder det att avsändaren inte var tillåten för **grupp-/kanal**-meddelanden. Åtgärda det genom att antingen:

- ange `channels.irc.groupAllowFrom` (globalt för alla kanaler), eller
- ange per-kanal-allowlists för avsändare: `channels.irc.groups["#channel"].allowFrom`

Exempel (tillåt vem som helst i `#tuirc-dev` att prata med boten):

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

## Svarstrigger (omnämnanden)

Även om en kanal är tillåten (via `groupPolicy` + `groups`) och avsändaren är tillåten, använder OpenClaw som standard **omnämnandebegränsning** i gruppsammanhang.

Det betyder att du kan se loggar som `drop channel … (missing-mention)` om inte meddelandet innehåller ett omnämnandemönster som matchar boten.

För att få boten att svara i en IRC-kanal **utan att behöva ett omnämnande**, inaktivera omnämnandebegränsning för den kanalen:

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

Eller för att tillåta **alla** IRC-kanaler (ingen allowlist per kanal) och ändå svara utan omnämnanden:

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

## Säkerhetsnotis (rekommenderas för publika kanaler)

Om du tillåter `allowFrom: ["*"]` i en publik kanal kan vem som helst skicka uppmaningar till boten.
För att minska risken, begränsa verktygen för den kanalen.

### Samma verktyg för alla i kanalen

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

### Olika verktyg per avsändare (ägaren får mer behörighet)

Använd `toolsBySender` för att tillämpa en striktare policy för `"*"` och en mer tillåtande för ditt nick:

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

Anteckningar:

- `toolsBySender`-nycklar kan vara ett nick (t.ex. `"eigen"`) eller en fullständig hostmask (`"eigen!~eigen@174.127.248.171"`) för starkare identitetsmatchning.
- Den första matchande avsändarpolicyn gäller; `"*"` är wildcard-reserven.

För mer information om gruppåtkomst vs omnämnandekrav (och hur de samverkar), se: [/channels/groups](/channels/groups).

## NickServ

För att identifiera dig med NickServ efter anslutning:

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

Valfri engångsregistrering vid anslutning:

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

Inaktivera `register` efter att nicket har registrerats för att undvika upprepade REGISTER-försök.

## Miljövariabler

Standardkontot stöder:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (kommaseparerade)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Felsökning

- Om boten ansluter men aldrig svarar i kanaler, kontrollera `channels.irc.groups` **och** om omnämnandekrav filtrerar bort meddelanden (`missing-mention`). Om du vill att den ska svara utan pingar, sätt `requireMention:false` för kanalen.
- Om inloggningen misslyckas, kontrollera att nicket är tillgängligt och serverlösenordet.
- Om TLS misslyckas i ett anpassat nätverk, kontrollera värd/port och certifikatkonfigurationen.

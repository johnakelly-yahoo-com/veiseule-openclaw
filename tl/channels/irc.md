---
title: IRC
description: Ikonekta ang OpenClaw sa mga IRC channel at direct messages.
---

Gamitin ang IRC kapag gusto mong ilagay ang OpenClaw sa mga klasikong channel (`#room`) at direct messages.
Ang IRC ay kasama bilang extension plugin, ngunit kino-configure ito sa pangunahing config sa ilalim ng `channels.irc`.

## Mabilis na pagsisimula

1. I-enable ang IRC config sa `~/.openclaw/openclaw.json`.
2. Itakda ang hindi bababa sa:

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

3. Simulan/i-restart ang gateway:

```bash
openclaw gateway run
```

## Mga default sa seguridad

- `channels.irc.dmPolicy` ay naka-default sa `"pairing"`.
- `channels.irc.groupPolicy` ay naka-default sa `"allowlist"`.
- Kapag `groupPolicy="allowlist"`, itakda ang `channels.irc.groups` upang tukuyin ang mga pinapayagang channel.
- Gamitin ang TLS (`channels.irc.tls=true`) maliban kung sadyang tinatanggap mo ang plaintext transport.

## Kontrol sa access

May dalawang magkahiwalay na “gate” para sa mga IRC channel:

1. **Channel access** (`groupPolicy` + `groups`): kung tatanggapin ng bot ang mga mensahe mula sa isang channel.
2. **Sender access** (`groupAllowFrom` / per-channel `groups["#channel"].allowFrom`): sino ang pinapayagang mag-trigger sa bot sa loob ng channel na iyon.

Mga config key:

- DM allowlist (access ng DM sender): `channels.irc.allowFrom`
- Group sender allowlist (access ng sender sa channel): `channels.irc.groupAllowFrom`
- Per-channel controls (channel + sender + mga panuntunan sa mention): `channels.irc.groups["#channel"]`
- Pinapayagan ng `channels.irc.groupPolicy="open"` ang mga hindi naka-configure na channel (**naka-mention-gated pa rin bilang default**)

Maaaring gumamit ang mga entry sa allowlist ng nick o mga anyong `nick!user@host`.

### Karaniwang pagkakamali: ang `allowFrom` ay para sa mga DM, hindi para sa mga channel

Kung makakita ka ng mga log tulad ng:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…ibig sabihin hindi pinayagan ang sender para sa mga **group/channel** na mensahe. Ayusin ito sa alinman sa mga sumusunod:

- itakda ang `channels.irc.groupAllowFrom` (global para sa lahat ng channel), o
- itakda ang per-channel sender allowlists: `channels.irc.groups["#channel"].allowFrom`

Halimbawa (payagan ang sinuman sa `#tuirc-dev` na makipag-usap sa bot):

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

## Pag-trigger ng tugon (mga mention)

Kahit na pinapayagan ang isang channel (sa pamamagitan ng `groupPolicy` + `groups`) at pinapayagan ang sender, ang OpenClaw ay naka-default sa **mention-gating** sa mga group context.

Ibig sabihin maaari kang makakita ng mga log tulad ng `drop channel … (missing-mention)` maliban kung ang mensahe ay may kasamang mention pattern na tumutugma sa bot.

Upang magpa-reply ang bot sa isang IRC channel **nang hindi nangangailangan ng mention**, i-disable ang mention gating para sa channel na iyon:

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

O upang payagan ang **lahat** ng IRC channels (walang per-channel allowlist) at makasagot pa rin nang walang mentions:

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

## Paalala sa seguridad (inirerekomenda para sa mga pampublikong channel)

Kung papayagan mo ang `allowFrom: ["*"]` sa isang pampublikong channel, kahit sino ay maaaring mag-prompt sa bot.
Upang mabawasan ang panganib, higpitan ang mga tool para sa channel na iyon.

### Parehong mga tool para sa lahat sa channel

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

### Magkakaibang mga tool bawat sender (mas maraming kapangyarihan ang owner)

Gamitin ang `toolsBySender` upang maglapat ng mas mahigpit na patakaran sa `"*"` at mas maluwag na patakaran sa iyong nick:

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

Mga Tala:

- Ang mga key ng `toolsBySender` ay maaaring isang nick (hal. `"eigen"`) o isang buong hostmask (`"eigen!~eigen@174.127.248.171"`) para sa mas matibay na pagtutugma ng pagkakakilanlan.
- Ang unang tumugmang patakaran ng sender ang masusunod; ang `"*"` ang wildcard fallback.

Para sa higit pang detalye tungkol sa group access kumpara sa mention-gating (at kung paano sila nag-uugnayan), tingnan: [/channels/groups](/channels/groups).

## NickServ

Upang mag-identify sa NickServ pagkatapos kumonekta:

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

Opsyonal na one-time registration sa pag-connect:

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

I-disable ang `register` pagkatapos ma-register ang nick upang maiwasan ang paulit-ulit na REGISTER attempts.

## Mga environment variable

Sinusuportahan ng default account ang:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (pinaghihiwalay ng kuwit)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Pag-troubleshoot

- Kung kumokonekta ang bot ngunit hindi kailanman sumasagot sa mga channel, tiyakin ang `channels.irc.groups` **at** kung ang mention-gating ay nagda-drop ng mga mensahe (`missing-mention`). Kung gusto mong sumagot ito nang walang pings, itakda ang `requireMention:false` para sa channel.
- Kung nabigo ang pag-login, tiyakin ang availability ng nick at ang server password.
- Kung nabigo ang TLS sa isang custom network, tiyakin ang host/port at ang setup ng certificate.


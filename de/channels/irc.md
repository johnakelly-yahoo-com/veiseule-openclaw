---
title: IRC
description: Verbinde OpenClaw mit IRC-Kanälen und Direktnachrichten.
---

Verwende IRC, wenn du OpenClaw in klassischen Kanälen (`#room`) und Direktnachrichten einsetzen möchtest.
IRC wird als Erweiterungs-Plugin ausgeliefert, wird jedoch in der Hauptkonfiguration unter `channels.irc` konfiguriert.

## Schnellstart

1. Aktiviere die IRC-Konfiguration in `~/.openclaw/openclaw.json`.
2. Lege mindestens Folgendes fest:

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

3. Gateway starten/neustarten:

```bash
openclaw gateway run
```

## Sicherheits-Standardeinstellungen

- `channels.irc.dmPolicy` ist standardmäßig auf `"pairing"` gesetzt.
- `channels.irc.groupPolicy` ist standardmäßig auf `"allowlist"` gesetzt.
- Bei `groupPolicy="allowlist"` lege `channels.irc.groups` fest, um erlaubte Kanäle zu definieren.
- Verwende TLS (`channels.irc.tls=true`), es sei denn, du akzeptierst absichtlich eine unverschlüsselte Übertragung.

## Zugriffskontrolle

Es gibt zwei separate „Schranken“ für IRC-Kanäle:

1. **Kanalzugriff** (`groupPolicy` + `groups`): Ob der Bot Nachrichten aus einem Kanal überhaupt akzeptiert.
2. **Absenderzugriff** (`groupAllowFrom` / kanalbezogen `groups["#channel"].allowFrom`): Wer den Bot innerhalb dieses Kanals auslösen darf.

Konfigurationsschlüssel:

- DM-Allowlist (DM-Absenderzugriff): `channels.irc.allowFrom`
- Gruppen-Absender-Allowlist (Kanal-Absenderzugriff): `channels.irc.groupAllowFrom`
- Kanalbezogene Steuerung (Kanal + Absender + Erwähnungsregeln): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` erlaubt nicht konfigurierte Kanäle (**standardmäßig weiterhin durch Erwähnungen eingeschränkt**)

Allowlist-Einträge können den Nick oder das Format `nick!user@host` verwenden.

### Häufiger Stolperstein: `allowFrom` gilt für DMs, nicht für Kanäle

Wenn du Protokolleinträge siehst wie:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

… bedeutet das, dass der Absender für **Gruppen-/Kanal**-Nachrichten nicht erlaubt war. Behebe dies entweder durch:

- Setzen von `channels.irc.groupAllowFrom` (global für alle Kanäle), oder
- Setzen kanalbezogener Absender-Allowlists: `channels.irc.groups["#channel"].allowFrom`

Beispiel (erlaube allen in `#tuirc-dev`, mit dem Bot zu sprechen):

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

## Antwortauslösung (Erwähnungen)

Selbst wenn ein Kanal erlaubt ist (über `groupPolicy` + `groups`) und der Absender erlaubt ist, verwendet OpenClaw in Gruppenkontexten standardmäßig **Erwähnungs-Gating**.

Das bedeutet, dass du möglicherweise Protokolle wie `drop channel …` siehst `(missing-mention)`, sofern die Nachricht kein Erwähnungsmuster enthält, das zum Bot passt.

Damit der Bot in einem IRC-Kanal **ohne erforderliche Erwähnung** antwortet, deaktiviere das Erwähnungs-Gating für diesen Kanal:

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

Oder um **alle** IRC-Kanäle zu erlauben (keine Allowlist pro Kanal) und dennoch ohne Erwähnungen zu antworten:

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

## Sicherheitshinweis (empfohlen für öffentliche Kanäle)

Wenn Sie `allowFrom: ["*"]` in einem öffentlichen Kanal erlauben, kann jeder den Bot ansprechen.
Um das Risiko zu verringern, beschränken Sie die Tools für diesen Kanal.

### Gleiche Tools für alle im Kanal

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

### Unterschiedliche Tools pro Absender (Owner erhält mehr Rechte)

Verwenden Sie `toolsBySender`, um eine strengere Richtlinie auf `"*"` und eine lockerere auf Ihren Nick anzuwenden:

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

Hinweise:

- `toolsBySender`-Schlüssel können ein Nick (z. B. `"eigen"`) oder eine vollständige Hostmaske (`"eigen!~eigen@174.127.248.171"`) sein, um eine stärkere Identitätsabgleichung zu ermöglichen.
- Die erste passende Absender-Richtlinie gilt; `"*"` ist der Wildcard-Fallback.

Weitere Informationen zu Gruppenzugriff vs. Mention-Gating (und wie sie zusammenwirken) finden Sie unter: [/channels/groups](/channels/groups).

## NickServ

Zur Identifizierung bei NickServ nach dem Verbinden:

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

Optionale einmalige Registrierung beim Verbinden:

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

Deaktivieren Sie `register`, nachdem der Nick registriert ist, um wiederholte REGISTER-Versuche zu vermeiden.

## Umgebungsvariablen

Das Standardkonto unterstützt:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (durch Kommas getrennt)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Fehlerbehebung

- Wenn der Bot eine Verbindung herstellt, aber in Kanälen nie antwortet, überprüfen Sie `channels.irc.groups` **und** ob Mention-Gating Nachrichten verwirft (`missing-mention`). Wenn er ohne Pings antworten soll, setzen Sie `requireMention:false` für den Kanal.
- Wenn die Anmeldung fehlschlägt, überprüfen Sie die Verfügbarkeit des Nicks und das Serverpasswort.
- Wenn TLS in einem benutzerdefinierten Netzwerk fehlschlägt, überprüfen Sie Host/Port und die Zertifikatskonfiguration.


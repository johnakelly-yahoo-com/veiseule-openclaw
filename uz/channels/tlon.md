---
summary: "Tlon/Urbit qo‘llab-quvvatlash holati, imkoniyatlari va sozlash"
read_when:
  - Working on Tlon/Urbit channel features
title: "Tlon"
---

# Tlon (plagin)

Tlon is a decentralized messenger built on Urbit. OpenClaw connects to your Urbit ship and can
respond to DMs and group chat messages. Group replies require an @ mention by default and can
be further restricted via allowlists.

Status: supported via plugin. DMs, group mentions, thread replies, and text-only media fallback
(URL appended to caption). Reactions, polls, and native media uploads are not supported.

## Talab qilinadigan plagin

Tlon plagin sifatida taqdim etiladi va asosiy o‘rnatish to‘plamiga kiritilmagan.

CLI orqali o‘rnating (npm registry):

```bash
openclaw plugins install @openclaw/tlon
```

Mahalliy checkout (git repo’dan ishga tushirilganda):

```bash
openclaw plugins install ./extensions/tlon
```

Batafsil: [Plaginlar](/tools/plugin)

## Setup

1. Install the Tlon plugin.
2. Gather your ship URL and login code.
3. Configure `channels.tlon`.
4. Restart the gateway.
5. DM the bot or mention it in a group channel.

Minimal config (single account):

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
    },
  },
}
```

Private/LAN ship URL’lari (ilg‘or):

Standart holatda, OpenClaw ushbu plagin uchun private/ichki hostname va IP diapazonlarini bloklaydi (SSRF himoyasi).
Agar ship URL’ingiz private tarmoqda bo‘lsa (masalan `http://192.168.1.50:8080` yoki `http://localhost:8080`),
uni aniq ravishda yoqishingiz kerak:

```json5
{
  channels: {
    tlon: {
      allowPrivateNetwork: true,
    },
  },
}
```

## Group channels

Auto-discovery is enabled by default. You can also pin channels manually:

```json5
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

Disable auto-discovery:

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## Access control

DM allowlist (empty = allow all):

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

Group authorization (restricted by default):

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## Delivery targets (CLI/cron)

Use these with `openclaw message send` or cron delivery:

- DM: `~sampel-palnet` or `dm/~sampel-palnet`
- Group: `chat/~host-ship/channel` or `group:~host-ship/channel`

## Notes

- Group replies require a mention (e.g. `~your-bot-ship`) to respond.
- Thread replies: if the inbound message is in a thread, OpenClaw replies in-thread.
- Media: `sendMedia` falls back to text + URL (no native upload).


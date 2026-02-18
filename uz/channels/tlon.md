---
title: "Tlon"
---

# Tlon (plagin)

Tlon — Urbit asosida yaratilgan markazlashmagan messenjer. OpenClaw sizning Urbit kemangizga ulanadi va quyidagilarni amalga oshirishi mumkin
respond to DMs and group chat messages. Group replies require an @ mention by default and can
be further restricted via allowlists.

Holat: plagin orqali qo‘llab-quvvatlanadi. Shaxsiy xabarlar (DM), guruh eslatmalari, mavzudagi javoblar va faqat matnli media zaxira rejimi
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


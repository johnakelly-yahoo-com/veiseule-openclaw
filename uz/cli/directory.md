---
title: "directory"
---

# `openclaw directory`

Qo‘llab-quvvatlaydigan kanallar uchun katalog qidiruvlari (kontaktlar/peers, guruhlar va “me”).

## Umumiy flaglar

- `--channel <name>`: kanal id/aliasi (bir nechta kanallar sozlangan bo‘lsa majburiy; faqat bitta kanal sozlangan bo‘lsa avtomatik)
- `--account <id>`: akkaunt idsi (standart: kanal bo‘yicha standart)
- `--json`: JSON chiqishini beradi

## Eslatmalar

- `directory` boshqa buyruqlarga qo‘yib ishlatishingiz mumkin bo‘lgan IDlarni topishga yordam beradi (ayniqsa `openclaw message send --target ...`).
- Ko‘plab kanallarda natijalar jonli provayder katalogidan emas, balki konfiguratsiyaga asoslangan (ruxsat ro‘yxatlari / sozlangan guruhlar) manbadan olinadi.
- Standart chiqish `id` (va ba’zan `name`) bo‘lib, ular tab bilan ajratiladi; skriptlash uchun `--json` dan foydalaning.

## Using results with `message send`

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## ID formats (by channel)

- WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (group)
- Telegram: `@username` or numeric chat id; groups are numeric ids
- Slack: `user:U…` and `channel:C…`
- Discord: `user:<id>` and `channel:<id>`
- Matrix (plugin): `user:@user:server`, `room:!roomId:server`, or `#alias:server`
- Microsoft Teams (plugin): `user:<id>` and `conversation:<id>`
- Zalo (plugin): user id (Bot API)
- Zalo Personal / `zalouser` (plugin): thread id (DM/group) from `zca` (`me`, `friend list`, `group list`)

## Self (“me”)

```bash
openclaw directory self --channel zalouser
```

## Peers (contacts/users)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## Groups

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```


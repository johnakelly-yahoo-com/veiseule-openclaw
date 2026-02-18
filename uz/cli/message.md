---
title: "message"
---

# `openclaw message`

Xabar yuborish va kanal amallari uchun yagona chiqish buyrug‘i
(Discord/Google Chat/Slack/Mattermost (plugin)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

## Foydalanish

```
openclaw message <subcommand> [flags]
```

Kanalni tanlash:

- Agar bir nechta kanal sozlangan bo‘lsa, `--channel` majburiy.
- Agar aynan bitta kanal sozlangan bo‘lsa, u sukut bo‘yicha tanlanadi.
- Qiymatlar: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost plugin talab qiladi)

Nishon formatlari (`--target`):

- WhatsApp: E.164 yoki guruh JID
- Telegram: chat id yoki `@username`
- Discord: `channel:<id>` yoki `user:<id>` (yoki `<@id>` eslatmasi; oddiy raqamli id’lar kanal sifatida qabul qilinadi)
- Google Chat: `spaces/<spaceId>` yoki `users/<userId>`
- Slack: `channel:<id>` yoki `user:<id>` (oddiy kanal id qabul qilinadi)
- Mattermost (plugin): `channel:<id>`, `user:<id>` yoki `@username` (oddiy id’lar kanal sifatida qabul qilinadi)
- Signal: `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>` yoki `username:<name>`/`u:<name>`
- iMessage: handle, `chat_id:<id>`, `chat_guid:<guid>` yoki `chat_identifier:<id>`
- MS Teams: conversation id (`19:...@thread.tacv2`) yoki `conversation:<id>` yoki `user:<aad-object-id>`

Nom bo‘yicha qidirish:

- Qo‘llab-quvvatlanadigan provayderlarda (Discord/Slack va boshqalar) `Help` yoki `#help` kabi kanal nomlari katalog keshi orqali aniqlanadi.
- Agar keshda topilmasa, provayder qo‘llab-quvvatlasa, OpenClaw jonli katalog qidiruvini amalga oshiradi.

## Umumiy flaglar

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (send/poll/read va boshqalar uchun nishon kanal yoki foydalanuvchi)
- `--targets <name>` (takrorlanadi; faqat broadcast uchun)
- `--json`
- `--dry-run`
- `--verbose`

## Amallar

### Asosiy

- `send`
  - Kanallar: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams
  - Majburiy: `--target`, va `--message` yoki `--media`
  - Ixtiyoriy: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Faqat Telegram: `--buttons` (ruxsat uchun `channels.telegram.capabilities.inlineButtons` talab qilinadi)
  - Faqat Telegram: `--thread-id` (forum mavzusi id)
  - Faqat Slack: `--thread-id` (thread timestamp; `--reply-to` ham shu maydondan foydalanadi)
  - Faqat WhatsApp: `--gif-playback`

- `poll`
  - Kanallar: WhatsApp/Telegram/Discord/Matrix/MS Teams
  - Majburiy: `--target`, `--poll-question`, `--poll-option` (takrorlanadi)
  - Ixtiyoriy: `--poll-multi`
  - Faqat Discord: `--poll-duration-hours`, `--silent`, `--message`
  - Faqat Telegram: `--poll-duration-seconds` (5-600), `--silent`, `--poll-anonymous` / `--poll-public`, `--thread-id`

- `react`
  - Kanallar: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - Majburiy: `--message-id`, `--target`
  - Ixtiyoriy: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - Eslatma: `--remove` uchun `--emoji` talab qilinadi (`--emoji` ni ko‘rsatmasangiz, qo‘llab-quvvatlangan joylarda o‘z reaksiyalaringiz tozalanadi; qarang /tools/reactions)
  - Faqat WhatsApp: `--participant`, `--from-me`
  - Signal guruh reaksiyalari: `--target-author` yoki `--target-author-uuid` majburiy

- `reactions`
  - Kanallar: Discord/Google Chat/Slack
  - Majburiy: `--message-id`, `--target`
  - Ixtiyoriy: `--limit`

- `read`
  - Kanallar: Discord/Slack
  - Majburiy: `--target`
  - Ixtiyoriy: `--limit`, `--before`, `--after`
  - Faqat Discord: `--around`

- `edit`
  - Kanallar: Discord/Slack
  - Majburiy: `--message-id`, `--message`, `--target`

- `delete`
  - Kanallar: Discord/Slack/Telegram
  - Majburiy: `--message-id`, `--target`

- `pin` / `unpin`
  - Kanallar: Discord/Slack
  - Majburiy: `--message-id`, `--target`

- `pins` (ro‘yxat)
  - Kanallar: Discord/Slack
  - Majburiy: `--target`

- `permissions`
  - Kanallar: Discord
  - Majburiy: `--target`

- `search`
  - Kanallar: Discord
  - Majburiy: `--guild-id`, `--query`
  - Ixtiyoriy: `--channel-id`, `--channel-ids` (takrorlanadi), `--author-id`, `--author-ids` (takrorlanadi), `--limit`

### Threadlar

- `thread create`
  - Kanallar: Discord
  - Majburiy: `--thread-name`, `--target` (kanal id)
  - Ixtiyoriy: `--message-id`, `--message`, `--auto-archive-min`

- `thread list`
  - Kanallar: Discord
  - Majburiy: `--guild-id`
  - Ixtiyoriy: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - Kanallar: Discord
  - Majburiy: `--target` (thread id), `--message`
  - Ixtiyoriy: `--media`, `--reply-to`

### Emojilar

- `emoji list`
  - Discord: `--guild-id`
  - Slack: qo‘shimcha flaglar yo‘q

- `emoji upload`
  - Kanallar: Discord
  - Majburiy: `--guild-id`, `--emoji-name`, `--media`
  - Ixtiyoriy: `--role-ids` (takrorlanadi)

### Stikerlar

- `sticker send`
  - Kanallar: Discord
  - Majburiy: `--target`, `--sticker-id` (takrorlanadi)
  - Ixtiyoriy: `--message`

- `sticker upload`
  - Kanallar: Discord
  - Majburiy: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### Rollar / Kanallar / A’zolar / Ovoz

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (+ Discord uchun `--guild-id`)
- `voice status` (Discord): `--guild-id`, `--user-id`

### Hodisalar

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - Ixtiyoriy: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### Moderatsiya (Discord)

- `timeout`: `--guild-id`, `--user-id` (ixtiyoriy `--duration-min` yoki `--until`; ikkalasi ham ko‘rsatilmasa, timeout bekor qilinadi)
- `kick`: `--guild-id`, `--user-id` (+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout` ham `--reason` ni qo‘llab-quvvatlaydi

### Broadcast

- `broadcast`
  - Kanallar: har qanday sozlangan kanal; barcha provayderlarni nishonga olish uchun `--channel all` dan foydalaning
  - Majburiy: `--targets` (takrorlanadi)
  - Ixtiyoriy: `--message`, `--media`, `--dry-run`

## Misollar

Discord’da javob yuborish:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Discord’da so‘rovnoma yaratish:

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Telegram’da so‘rovnoma yaratish (2 daqiqada avtomatik yopiladi):

```
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

Teams’da proaktiv xabar yuborish:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Teams’da so‘rovnoma yaratish:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Slack’da reaksiya bildirish:

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Signal guruhida reaksiya bildirish:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Telegram inline tugmalarini yuborish:

```
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```


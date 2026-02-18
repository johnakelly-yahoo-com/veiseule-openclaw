---
title: "security"
---

# `openclaw security`

Инструменты безопасности (аудит + необязательные исправления).

Связанное:

- Руководство по безопасности: [Security](/gateway/security)

## Аудит

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Аудит предупреждает, когда несколько отправителей личных сообщений (DM) используют основной сеанс, и рекомендует **безопасный режим DM**: `session.dmScope="per-channel-peer"` (или `per-account-channel-peer` для каналов с несколькими аккаунтами) для общих входящих.
Также он предупреждает, когда небольшие модели (`<=300B`) используются без sandboxing и с включёнными веб/браузерными инструментами.


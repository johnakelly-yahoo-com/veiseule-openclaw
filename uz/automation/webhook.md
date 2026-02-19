---
summary: "21. Uyg‘otish va ajratilgan agent ishga tushirishlar uchun webhook kirishi"
read_when:
  - 22. Webhook endpointlarini qo‘shish yoki o‘zgartirish
  - 23. Tashqi tizimlarni OpenClaw’ga ulash
title: "24. Webhooklar"
---

# 25. Webhooklar

26. Shlyuz tashqi triggerlar uchun kichik HTTP webhook endpointini ochishi mumkin.

## 27. Yoqish

```json5
28. {
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

29. Eslatmalar:

- 30. `hooks.enabled=true` bo‘lganda `hooks.token` majburiy.
- 31. `hooks.path` sukut bo‘yicha `/hooks`.

## 32. Autentifikatsiya

33. Har bir so‘rov hook tokenini o‘z ichiga olishi kerak. 34. Sarlavhalarni afzal ko‘ring:

- 35. `Authorization: Bearer <token>` (tavsiya etiladi)
- `x-openclaw-token: <token>`
- `agentId` ixtiyoriy (string): Ushbu hook’ni ma’lum bir agentga yo‘naltiradi.

## 38. Endpointlar

### 39. `POST /hooks/wake`

40. Yuklama:

```json
41. { "text": "System line", "mode": "now" }
```

- 42. `text` **majburiy** (string): Hodisa tavsifi (masalan, "New email received").
- 43. `mode` ixtiyoriy (`now` | `next-heartbeat`): Darhol heartbeat’ni ishga tushirish (sukut bo‘yicha `now`) yoki keyingi davriy tekshiruvni kutish.

Effect:

- 45. **Asosiy** sessiya uchun tizim hodisasini navbatga qo‘yadi
- 46. Agar `mode=now` bo‘lsa, darhol heartbeat’ni ishga tushiradi

### 47. `POST /hooks/agent`

48. Yuklama:

```json
49. {
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- 50. `message` **majburiy** (string): Agent qayta ishlashi uchun prompt yoki xabar.
- `name` ixtiyoriy (string): Hook uchun inson o‘qiy oladigan nom (masalan, "GitHub"), sessiya xulosalarida prefiks sifatida ishlatiladi.
- Noma’lum ID’lar standart agentga qaytadi. Unknown IDs fall back to the default agent. O‘rnatilganda, hook aniqlangan agentning ish maydoni va konfiguratsiyasi yordamida ishga tushadi.
- `sessionKey` optional (string): The key used to identify the agent's session. Standart holatda bu maydon `hooks.allowRequestSessionKey=true` bo‘lmaguncha rad etiladi.
- `wakeMode` optional (`now` | `next-heartbeat`): Whether to trigger an immediate heartbeat (default `now`) or wait for the next periodic check.
- `deliver` optional (boolean): If `true`, the agent's response will be sent to the messaging channel. Defaults to `true`. Responses that are only heartbeat acknowledgments are automatically skipped.
- `channel` optional (string): The messaging channel for delivery. One of: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Defaults to `last`.
- `to` optional (string): The recipient identifier for the channel (e.g., phone number for WhatsApp/Signal, chat ID for Telegram, channel ID for Discord/Slack/Mattermost (plugin), conversation ID for MS Teams). Defaults to the last recipient in the main session.
- `model` optional (string): Model override (e.g., `anthropic/claude-3-5-sonnet` or an alias). Must be in the allowed model list if restricted.
- `thinking` optional (string): Thinking level override (e.g., `low`, `medium`, `high`).
- `timeoutSeconds` optional (number): Maximum duration for the agent run in seconds.

Effect:

- Runs an **isolated** agent turn (own session key)
- Always posts a summary into the **main** session
- If `wakeMode=now`, triggers an immediate heartbeat

## Session key siyosati (moslikni buzuvchi o‘zgarish)

`/hooks/agent` payload ichidagi `sessionKey` override’lari standart bo‘yicha o‘chirilgan.

- Tavsiya etiladi: qat’iy `hooks.defaultSessionKey` o‘rnating va so‘rov orqali override’larni o‘chirib qo‘ying.
- Ixtiyoriy: faqat zarur bo‘lganda so‘rov orqali override’larga ruxsat bering va prefikslarni cheklang.

Tavsiya etilgan konfiguratsiya:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Moslik konfiguratsiyasi (eski xatti-harakat):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // qat’iy tavsiya etiladi
  },
}
```

### `POST /hooks/<name>` (mapped)

Custom hook names are resolved via `hooks.mappings` (see configuration). A mapping can
turn arbitrary payloads into `wake` or `agent` actions, with optional templates or
code transforms.

Mapping options (summary):

- `hooks.presets: ["gmail"]` enables the built-in Gmail mapping.
- `hooks.mappings` lets you define `match`, `action`, and templates in config.
- `hooks.transformsDir` + `transform.module` loads a JS/TS module for custom logic.
  - `hooks.transformsDir` (agar o‘rnatilgan bo‘lsa) OpenClaw konfiguratsiya katalogingiz ostidagi transforms ildiz papkasi ichida qolishi kerak (odatda `~/.openclaw/hooks/transforms`).
  - `transform.module` amaldagi transforms katalogi ichida aniqlanishi kerak (traversal/escape yo‘llar rad etiladi).
- Use `match.source` to keep a generic ingest endpoint (payload-driven routing).
- TS transforms require a TS loader (e.g. `bun` or `tsx`) or precompiled `.js` at runtime.
- Set `deliver: true` + `channel`/`to` on mappings to route replies to a chat surface
  (`channel` defaults to `last` and falls back to WhatsApp).
- `agentId` hook’ni ma’lum bir agentga yo‘naltiradi; noma’lum ID’lar standart agentga qaytariladi.
- `hooks.allowedAgentIds` aniq `agentId` yo‘naltirishini cheklaydi. Istalgan agentga ruxsat berish uchun uni qoldiring (yoki `*` ni kiriting). Aniq `agentId` yo‘naltirishini rad etish uchun `[]` ni o‘rnating.
- `hooks.defaultSessionKey` aniq kalit ko‘rsatilmaganda hook agent ishga tushirilishi uchun standart sessiyani belgilaydi.
- `hooks.allowRequestSessionKey` `/hooks/agent` payload’lari `sessionKey` ni o‘rnatishi mumkinligini boshqaradi (standart: `false`).
- `hooks.allowedSessionKeyPrefixes` ixtiyoriy ravishda so‘rov payload’lari va moslamalardagi aniq `sessionKey` qiymatlarini cheklaydi.
- `allowUnsafeExternalContent: true` disables the external content safety wrapper for that hook
  (dangerous; only for trusted internal sources).
- `openclaw webhooks gmail setup` writes `hooks.gmail` config for `openclaw webhooks gmail run`.
  See [Gmail Pub/Sub](/automation/gmail-pubsub) for the full Gmail watch flow.

## Responses

- `200` for `/hooks/wake`
- `202` for `/hooks/agent` (async run started)
- `401` on auth failure
- Bir xil mijozdan takroriy autentifikatsiya xatolaridan so‘ng `429` ( `Retry-After` ni tekshiring )
- `400` on invalid payload
- `413` on oversized payloads

## Examples

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Use a different model

Add `model` to the agent payload (or mapping) to override the model for that run:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

If you enforce `agents.defaults.models`, make sure the override model is included there.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Security

- Keep hook endpoints behind loopback, tailnet, or trusted reverse proxy.
- Use a dedicated hook token; do not reuse gateway auth tokens.
- Takroriy autentifikatsiya xatolari brute-force urinishlarini sekinlashtirish uchun har bir mijoz manzili bo‘yicha tezlik bilan cheklanadi.
- Agar multi-agent yo‘naltirishdan foydalansangiz, aniq `agentId` tanlovini cheklash uchun `hooks.allowedAgentIds` ni o‘rnating.
- Agar chaqiruvchi tomonidan tanlanadigan sessiyalar kerak bo‘lmasa, `hooks.allowRequestSessionKey=false` ni saqlang.
- Agar so‘rov orqali `sessionKey` ni yoqsangiz, `hooks.allowedSessionKeyPrefixes` ni cheklang (masalan, `["hook:"]`).
- Avoid including sensitive raw payloads in webhook logs.
- Hook payloads are treated as untrusted and wrapped with safety boundaries by default.
  If you must disable this for a specific hook, set `allowUnsafeExternalContent: true`
  in that hook's mapping (dangerous).


---
summary: "Webhook ingress para sa wake at mga hiwalay na agent run"
read_when:
  - Pagdaragdag o pagbabago ng mga webhook endpoint
  - Pagkonekta ng mga panlabas na system sa OpenClaw
title: "Mga Webhook"
---

# Mga Webhook

Maaaring mag-expose ang Gateway ng isang maliit na HTTP webhook endpoint para sa mga panlabas na trigger.

## Paganahin

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

Mga tala:

- Kailangan ang `hooks.token` kapag `hooks.enabled=true`.
- Ang `hooks.path` ay default sa `/hooks`.

## Auth

Every request must include the hook token. Prefer headers:

- `Authorization: Bearer <token>` (inirerekomenda)
- `x-openclaw-token: <token>`
- `?token=<token>` (deprecated; nagla-log ng babala at aalisin sa susunod na major release)

## Mga Endpoint

### `POST /hooks/wake`

Payload:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **kinakailangan** (string): Ang paglalarawan ng event (hal., "New email received").
- `mode` opsyonal (`now` | `next-heartbeat`): Kung magti-trigger ng agarang heartbeat (default `now`) o maghihintay sa susunod na periodic check.

Epekto:

- Nag-e-enqueue ng system event para sa **main** session
- Kapag `mode=now`, nagti-trigger ng agarang heartbeat

### `POST /hooks/agent`

Payload:

```json
{
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

- `message` **kinakailangan** (string): Ang prompt o mensahe na ipo-process ng agent.
- `name` opsyonal (string): Human-readable na pangalan para sa hook (hal., "GitHub"), ginagamit bilang prefix sa mga session summary.
- `agentId` opsyonal (string): I-route ang hook na ito sa isang partikular na agent. Ang mga hindi kilalang ID ay babalik sa default na agent. Kapag nakatakda, tatakbo ang hook gamit ang naresolbang workspace at configuration ng agent.
- `sessionKey` optional (string): The key used to identify the agent's session. Bilang default, tinatanggihan ang field na ito maliban kung `hooks.allowRequestSessionKey=true`.
- `wakeMode` opsyonal (`now` | `next-heartbeat`): Kung magti-trigger ng agarang heartbeat (default `now`) o maghihintay sa susunod na periodic check.
- `deliver` optional (boolean): If `true`, the agent's response will be sent to the messaging channel. Defaults to `true`. Responses that are only heartbeat acknowledgments are automatically skipped.
- `channel` optional (string): The messaging channel for delivery. One of: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Defaults to `last`.
- `to` optional (string): The recipient identifier for the channel (e.g., phone number for WhatsApp/Signal, chat ID for Telegram, channel ID for Discord/Slack/Mattermost (plugin), conversation ID for MS Teams). Defaults to the last recipient in the main session.
- `model` optional (string): Model override (e.g., `anthropic/claude-3-5-sonnet` or an alias). Must be in the allowed model list if restricted.
- `timeoutSeconds` opsyonal (number): Maximum na tagal para sa agent run sa segundo.
- `timeoutSeconds` opsyonal (number): Maximum na tagal para sa agent run sa segundo.

Epekto:

- Tumatakbo ng isang **hiwalay** na agent turn (sariling session key)
- Palaging nagpo-post ng summary sa **main** session
- Kapag `wakeMode=now`, nagti-trigger ng agarang heartbeat

## Patakaran sa session key (breaking change)

Ang mga override ng `sessionKey` sa payload ng `/hooks/agent` ay naka-disable bilang default.

- Inirerekomenda: magtakda ng nakapirming `hooks.defaultSessionKey` at panatilihing naka-off ang mga request override.
- Opsyonal: payagan lamang ang mga request override kapag kinakailangan, at higpitan ang mga prefix.

Inirerekomendang config:

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

Compatibility config (legacy behavior):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // lubos na inirerekomenda
  },
}
```

### `POST /hooks/<name>` (mapped)

Custom hook names are resolved via `hooks.mappings` (see configuration). A mapping can
turn arbitrary payloads into `wake` or `agent` actions, with optional templates or
code transforms.

Mga opsyon sa mapping (buod):

- Pinapagana ng `hooks.presets: ["gmail"]` ang built-in na Gmail mapping.
- Hinahayaan ka ng `hooks.mappings` na magtakda ng `match`, `action`, at mga template sa config.
- Ang `hooks.transformsDir` + `transform.module` ay naglo-load ng JS/TS module para sa custom na logic.
  - Ang `hooks.transformsDir` (kung nakatakda) ay dapat manatili sa loob ng transforms root sa ilalim ng iyong OpenClaw config directory (karaniwan ay `~/.openclaw/hooks/transforms`).
  - Ang `transform.module` ay dapat mag-resolve sa loob ng epektibong transforms directory (ang traversal/escape paths ay tinatanggihan).
- Gamitin ang `match.source` para panatilihin ang isang generic ingest endpoint (payload-driven routing).
- Ang mga TS transform ay nangangailangan ng TS loader (hal., `bun` o `tsx`) o precompiled na `.js` sa runtime.
- Itakda ang `deliver: true` + `channel`/`to` sa mga mapping para i-route ang mga reply sa isang chat surface
  (`channel` ay default sa `last` at nagfa-fallback sa WhatsApp).
- Ang `agentId` ay nagru-route ng hook sa isang partikular na agent; ang mga hindi kilalang ID ay babagsak sa default agent.
- Ang `hooks.allowedAgentIds` ay naghihigpit sa tahasang pagru-route ng `agentId`. Alisin ito (o isama ang `*`) upang payagan ang anumang agent. Itakda sa `[]` upang tanggihan ang tahasang pagru-route ng `agentId`.
- Ang `hooks.defaultSessionKey` ay nagtatakda ng default session para sa mga hook agent run kapag walang ibinigay na tahasang key.
- Kinokontrol ng `hooks.allowRequestSessionKey` kung maaaring magtakda ng `sessionKey` ang mga payload ng `/hooks/agent` (default: `false`).
- Ang `hooks.allowedSessionKeyPrefixes` ay opsyonal na naghihigpit sa mga tahasang halaga ng `sessionKey` mula sa mga request payload at mapping.
- Dinidi-disable ng `allowUnsafeExternalContent: true` ang external content safety wrapper para sa hook na iyon
  (mapanganib; para lang sa mga pinagkakatiwalaang internal source).
- `openclaw webhooks gmail setup` writes `hooks.gmail` config for `openclaw webhooks gmail run`.
  See [Gmail Pub/Sub](/automation/gmail-pubsub) for the full Gmail watch flow.

## Mga Tugon

- `200` para sa `/hooks/wake`
- `202` para sa `/hooks/agent` (nagsimula ang async run)
- `401` kapag auth failure
- `429` pagkatapos ng paulit-ulit na pagkabigo sa auth mula sa parehong client (tingnan ang `Retry-After`)
- `400` kapag invalid ang payload
- `413` kapag oversized ang payload

## Mga Halimbawa

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

### Gumamit ng ibang model

Idagdag ang `model` sa agent payload (o mapping) para i-override ang model para sa run na iyon:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Kung ipinapatupad mo ang `agents.defaults.models`, tiyaking kasama roon ang override model.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Seguridad

- Panatilihin ang mga hook endpoint sa likod ng loopback, tailnet, o pinagkakatiwalaang reverse proxy.
- Gumamit ng dedikadong hook token; huwag i-reuse ang mga gateway auth token.
- Ang mga paulit-ulit na pagkabigo sa auth ay nililimitahan ang rate kada client address upang pabagalin ang mga brute-force na pagtatangka.
- Kung gumagamit ka ng multi-agent routing, itakda ang `hooks.allowedAgentIds` upang limitahan ang tahasang pagpili ng `agentId`.
- Panatilihing `hooks.allowRequestSessionKey=false` maliban kung kailangan mo ang mga session na pinipili ng caller.
- Kung ie-enable mo ang request `sessionKey`, higpitan ang `hooks.allowedSessionKeyPrefixes` (halimbawa, `["hook:"]`).
- Iwasang magsama ng sensitibong raw payload sa mga webhook log.
- Hook payloads are treated as untrusted and wrapped with safety boundaries by default.
  If you must disable this for a specific hook, set `allowUnsafeExternalContent: true`
  in that hook's mapping (dangerous).


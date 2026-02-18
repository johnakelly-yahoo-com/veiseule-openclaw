---
title: "OpenAI Chat Completions"
---

# OpenAI Chat Completions (HTTP)

OpenClaw’ning Gateway’i kichik OpenAI-compatible Chat Completions endpointini taqdim eta oladi.

Bu endpoint **standart bo‘yicha o‘chirilgan**. Avval uni konfiguratsiyada yoqing.

- `POST /v1/chat/completions`
- Gateway bilan bir xil port (WS + HTTP multiplex): `http://<gateway-host>:<port>/v1/chat/completions`

Ichki tomonda so‘rovlar odatiy Gateway agent ishga tushirilishi sifatida bajariladi (`openclaw agent` bilan bir xil kod yo‘li), shuning uchun marshrutlash/ruxsatlar/konfiguratsiya Gateway’ingizga mos keladi.

## Autentifikatsiya

Gateway auth konfiguratsiyasidan foydalanadi. Bearer token yuboring:

- `Authorization: Bearer <token>`

Eslatmalar:

- When `gateway.auth.mode="token"`, use `gateway.auth.token` (or `OPENCLAW_GATEWAY_TOKEN`).
- When `gateway.auth.mode="password"`, use `gateway.auth.password` (or `OPENCLAW_GATEWAY_PASSWORD`).

## Choosing an agent

No custom headers required: encode the agent id in the OpenAI `model` field:

- `model: "openclaw:<agentId>"` (example: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (alias)

Or target a specific OpenClaw agent by header:

- `x-openclaw-agent-id: <agentId>` (default: `main`)

Advanced:

- `x-openclaw-session-key: <sessionKey>` to fully control session routing.

## Enabling the endpoint

Set `gateway.http.endpoints.chatCompletions.enabled` to `true`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## Disabling the endpoint

Set `gateway.http.endpoints.chatCompletions.enabled` to `false`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## Session behavior

By default the endpoint is **stateless per request** (a new session key is generated each call).

If the request includes an OpenAI `user` string, the Gateway derives a stable session key from it, so repeated calls can share an agent session.

## Streaming (SSE)

Set `stream: true` to receive Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Each event line is `data: <json>`
- Stream ends with `data: [DONE]`

## Examples

Non-streaming:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

Streaming:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

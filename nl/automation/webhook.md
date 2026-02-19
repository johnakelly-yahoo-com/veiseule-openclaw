---
summary: "Webhook-ingang voor wekken en geïsoleerde agentruns"
read_when:
  - Webhook-eindpunten toevoegen of wijzigen
  - Externe systemen koppelen aan OpenClaw
title: "Webhooks"
---

# Webhooks

Gateway kan een klein HTTP-webhookeindpunt blootstellen voor externe triggers.

## Inschakelen

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

Notities:

- `hooks.token` is vereist wanneer `hooks.enabled=true`.
- `hooks.path` is standaard `/hooks`.

## Auth

Elke aanvraag moet het hook-token bevatten. Geef de voorkeur aan headers:

- `Authorization: Bearer <token>` (aanbevolen)
- `x-openclaw-token: <token>`
- Query-string-tokens worden afgewezen (`?token=...` retourneert `400`).

## Eindpunten

### `POST /hooks/wake`

Payload:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **vereist** (string): De beschrijving van de gebeurtenis (bijv. "Nieuwe e-mail ontvangen").
- `mode` optioneel (`now` | `next-heartbeat`): Of een onmiddellijke heartbeat moet worden geactiveerd (standaard `now`) of gewacht wordt op de volgende periodieke controle.

Effect:

- Zet een systeemevenement in de wachtrij voor de **hoofd**sessie
- Als `mode=now`, activeert een onmiddellijke heartbeat

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

- `message` **vereist** (string): De prompt of het bericht dat de agent moet verwerken.
- `name` optioneel (string): Menselijk leesbare naam voor de hook (bijv. "GitHub"), gebruikt als prefix in sessiesamenvattingen.
- `agentId` optioneel (string): Routeer deze hook naar een specifieke agent. Onbekende ID's vallen terug op de standaardagent. Indien ingesteld, wordt de hook uitgevoerd met de workspace en configuratie van de opgeloste agent.
- `sessionKey` optioneel (string): De sleutel die wordt gebruikt om de sessie van de agent te identificeren. Standaard wordt dit veld afgewezen tenzij `hooks.allowRequestSessionKey=true`.
- `wakeMode` optioneel (`now` | `next-heartbeat`): Of een onmiddellijke heartbeat moet worden geactiveerd (standaard `now`) of gewacht wordt op de volgende periodieke controle.
- `deliver` optioneel (boolean): Als `true`, wordt de reactie van de agent naar het berichtkanaal verzonden. Standaard `true`. Reacties die alleen heartbeat-bevestigingen zijn, worden automatisch overgeslagen.
- `channel` optioneel (string): Het berichtkanaal voor levering. Een van: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Standaard `last`.
- `to` optioneel (string): De ontvanger-ID voor het kanaal (bijv. telefoonnummer voor WhatsApp/Signal, chat-ID voor Telegram, kanaal-ID voor Discord/Slack/Mattermost (plugin), conversatie-ID voor MS Teams). Standaard de laatste ontvanger in de hoofdsessie.
- `model` optioneel (string): Model-override (bijv. `anthropic/claude-3-5-sonnet` of een alias). Moet in de toegestane modellenlijst staan als die is beperkt.
- `thinking` optioneel (string): Override van het denkniveau (bijv. `low`, `medium`, `high`).
- `timeoutSeconds` optioneel (number): Maximale duur voor de agentrun in seconden.

Effect:

- Draait een **geïsoleerde** agentbeurt (eigen sessiesleutel)
- Plaatst altijd een samenvatting in de **hoofd**sessie
- Als `wakeMode=now`, activeert een onmiddellijke heartbeat

## Session key-beleid (breaking change)

`/hooks/agent`-payload `sessionKey`-overrides zijn standaard uitgeschakeld.

- Aanbevolen: stel een vaste `hooks.defaultSessionKey` in en houd request-overrides uitgeschakeld.
- Optioneel: sta aanvraag-overschrijvingen alleen toe wanneer nodig en beperk prefixes.

Aanbevolen configuratie:

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

Compatibiliteitsconfiguratie (verouderd gedrag):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // strongly recommended
  },
}
```

### `POST /hooks/<name>` (gemapt)

Aangepaste hooknamen worden opgelost via `hooks.mappings` (zie configuratie). Een mapping kan
willekeurige payloads omzetten in `wake`- of `agent`-acties, met optionele sjablonen of
codetransformaties.

Mappingopties (samenvatting):

- `hooks.presets: ["gmail"]` schakelt de ingebouwde Gmail-mapping in.
- `hooks.mappings` laat je `match`, `action` en sjablonen in de config definiëren.
- `hooks.transformsDir` + `transform.module` laadt een JS/TS-module voor aangepaste logica.
  - `hooks.transformsDir` (indien ingesteld) moet binnen de transforms-root onder je OpenClaw-configuratiemap blijven (meestal `~/.openclaw/hooks/transforms`).
  - `transform.module` moet worden opgelost binnen de effectieve transforms-map (traversal-/escape-paden worden geweigerd).
- Gebruik `match.source` om een generiek ingest-eindpunt te behouden (payload-gestuurde routering).
- TS-transformaties vereisen een TS-loader (bijv. `bun` of `tsx`) of vooraf gecompileerde `.js` tijdens runtime.
- Stel `deliver: true` + `channel`/`to` in op mappings om antwoorden naar een chatoppervlak te routeren
  (`channel` is standaard `last` en valt terug op WhatsApp).
- `agentId` routeert de hook naar een specifieke agent; onbekende ID's vallen terug op de standaardagent.
- `hooks.allowedAgentIds` beperkt expliciete `agentId`-routering. Laat dit weg (of neem `*` op) om elke agent toe te staan. Stel `[]` in om expliciete `agentId`-routering te weigeren.
- `hooks.defaultSessionKey` stelt de standaardsessie in voor hook-agentruns wanneer geen expliciete sleutel is opgegeven.
- `hooks.allowRequestSessionKey` bepaalt of `/hooks/agent`-payloads `sessionKey` mogen instellen (standaard: `false`).
- `hooks.allowedSessionKeyPrefixes` beperkt optioneel expliciete `sessionKey`-waarden uit request-payloads en mappings.
- `allowUnsafeExternalContent: true` schakelt de externe content-veiligheidswrapper voor die hook uit
  (gevaarlijk; alleen voor vertrouwde interne bronnen).
- `openclaw webhooks gmail setup` schrijft `hooks.gmail`-config voor `openclaw webhooks gmail run`.
  Zie [Gmail Pub/Sub](/automation/gmail-pubsub) voor de volledige Gmail watch-flow.

## Reacties

- `200` voor `/hooks/wake`
- `202` voor `/hooks/agent` (asynchrone run gestart)
- `401` bij authenticatiefout
- `429` na herhaalde authenticatiefouten van dezelfde client (controleer `Retry-After`)
- `400` bij ongeldig payload
- `413` bij te grote payloads

## Voorbeelden

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

### Gebruik een ander model

Voeg `model` toe aan de agentpayload (of mapping) om het model voor die run te overriden:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Als je `agents.defaults.models` afdwingt, zorg er dan voor dat het override-model daarin is opgenomen.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Beveiliging

- Houd hook-eindpunten achter loopback, tailnet of een vertrouwde reverse proxy.
- Gebruik een dedicated hook-token; hergebruik geen gateway-authenticatietokens.
- Herhaalde authenticatiefouten worden per clientadres gerate-limit om brute-forcepogingen te vertragen.
- Als je multi-agent-routering gebruikt, stel dan `hooks.allowedAgentIds` in om expliciete `agentId`-selectie te beperken.
- Houd `hooks.allowRequestSessionKey=false` tenzij je sessies nodig hebt die door de aanroeper worden geselecteerd.
- Als je request-`sessionKey` inschakelt, beperk dan `hooks.allowedSessionKeyPrefixes` (bijvoorbeeld `["hook:"]`).
- Vermijd het opnemen van gevoelige ruwe payloads in webhooklogs.
- Hook-payloads worden standaard als niet-vertrouwd behandeld en omwikkeld met veiligheidsgrenzen.
  Als je dit voor een specifieke hook moet uitschakelen, stel dan `allowUnsafeExternalContent: true`
  in die hookmapping in (gevaarlijk).

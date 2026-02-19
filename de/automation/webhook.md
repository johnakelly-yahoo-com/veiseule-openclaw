---
summary: "Webhook-Eingang für Wake- und isolierte Agent-Läufe"
read_when:
  - Hinzufügen oder Ändern von Webhook-Endpunkten
  - Anbinden externer Systeme an OpenClaw
title: "Webhooks"
---

# Webhooks

Das Gateway kann einen kleinen HTTP-Webhook-Endpunkt für externe Trigger bereitstellen.

## Aktivieren

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

Hinweise:

- `hooks.token` ist erforderlich, wenn `hooks.enabled=true`.
- `hooks.path` ist standardmäßig `/hooks`.

## Authentifizierung

Jede Anfrage muss das Hook-Token enthalten. Bevorzugen Sie Header:

- `Authorization: Bearer <token>` (empfohlen)
- `x-openclaw-token: <token>`
- `agentId` optional (String): Leitet diesen Hook an einen bestimmten Agenten weiter.

## Endpunkte

### `POST /hooks/wake`

Nutzlast:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **erforderlich** (string): Die Beschreibung des Ereignisses (z. B. „Neue E-Mail empfangen“).
- `mode` optional (`now` | `next-heartbeat`): Ob ein sofortiger Heartbeat ausgelöst werden soll (Standard `now`) oder bis zur nächsten periodischen Prüfung gewartet wird.

Wirkung:

- Schlange ein System-Ereignis für die **Haupt** Sitzung ein
- Wenn `mode=now`, wird ein sofortiger Heartbeat ausgelöst

### `POST /hooks/agent`

Nutzlast:

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

- `message` **erforderlich** (string): Der Prompt oder die Nachricht, die der Agent verarbeiten soll.
- `name` optional (string): Für Menschen lesbarer Name für den Hook (z. B. „GitHub“), wird als Präfix in Sitzungszusammenfassungen verwendet.
- Unbekannte IDs fallen auf den Standard-Agenten zurück. Wenn gesetzt, wird der Hook mit dem Workspace und der Konfiguration des aufgelösten Agenten ausgeführt. Standardmäßig wird dieses Feld abgelehnt, sofern nicht `hooks.allowRequestSessionKey=true` gesetzt ist.
- `sessionKey` optional (string): Der Schlüssel zur Identifizierung der Sitzung des Agenten. Session-Key-Richtlinie (Breaking Change)
- `wakeMode` optional (`now` | `next-heartbeat`): Ob ein sofortiger Heartbeat ausgelöst werden soll (Standard `now`) oder bis zur nächsten periodischen Prüfung gewartet wird.
- `deliver` optional (boolean): Wenn `true`, wird die Antwort des Agenten an den Messaging-Kanal gesendet. Standardmäßig `true`. Antworten, die nur Heartbeat-Bestätigungen sind, werden automatisch übersprungen.
- `channel` optional (string): Der Messaging-Kanal für die Zustellung. Einer von: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (Plugin), `signal`, `imessage`, `msteams`. Standardmäßig `last`.
- `to` optional (string): Die Empfängerkennung für den Kanal (z. B. Telefonnummer für WhatsApp/Signal, Chat-ID für Telegram, Kanal-ID für Discord/Slack/Mattermost (Plugin), Konversations-ID für MS Teams). Standardmäßig der letzte Empfänger in der Hauptsitzung.
- `model` optional (string): Modell-Override (z. B. `anthropic/claude-3-5-sonnet` oder ein Alias). Muss in der erlaubten Modellliste enthalten sein, falls eingeschränkt.
- `thinking` optional (string): Thinking-Level-Override (z. B. `low`, `medium`, `high`).
- `timeoutSeconds` optional (number): Maximale Dauer für den Agent-Lauf in Sekunden.

Wirkung:

- Führt einen **isolierten** Agent-Turn aus (eigener Sitzungsschlüssel)
- Postet immer eine Zusammenfassung in die **Haupt**-Sitzung
- Wenn `wakeMode=now`, wird ein sofortiger Heartbeat ausgelöst

## `sessionKey`-Überschreibungen im `/hooks/agent`-Payload sind standardmäßig deaktiviert.

Empfohlen: einen festen `hooks.defaultSessionKey` setzen und Request-Überschreibungen deaktiviert lassen.

- Recommended: set a fixed `hooks.defaultSessionKey` and keep request overrides off.
- Optional: Anfrage-Overrides nur bei Bedarf erlauben und Präfixe einschränken.

Empfohlene Konfiguration:

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

Kompatibilitätskonfiguration (Legacy-Verhalten):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // dringend empfohlen
  },
}
```

### `POST /hooks/<name>` (zugeordnet)

Benutzerdefinierte Hook-Namen werden über `hooks.mappings` aufgelöst (siehe Konfiguration). Eine Zuordnung kann
beliebige Payloads in `wake`- oder `agent`-Aktionen umwandeln, mit optionalen Templates oder
Code-Transformationen.

Zuordnungsoptionen (Übersicht):

- `hooks.presets: ["gmail"]` aktiviert die integrierte Gmail-Zuordnung.
- `hooks.mappings` ermöglicht das Definieren von `match`, `action` und Templates in der Konfiguration.
- `hooks.transformsDir` + `transform.module` lädt ein JS/TS-Modul für benutzerdefinierte Logik.
  - `hooks.transformsDir` (falls gesetzt) muss innerhalb des Transforms-Root-Verzeichnisses unter deinem OpenClaw-Konfigurationsverzeichnis bleiben (typischerweise `~/.openclaw/hooks/transforms`).
  - `transform.module` muss innerhalb des effektiven Transforms-Verzeichnisses aufgelöst werden (Traversal-/Escape-Pfade werden abgelehnt).
- Verwenden Sie `match.source`, um einen generischen Ingest-Endpunkt beizubehalten (payload-gesteuertes Routing).
- TS-Transformationen erfordern einen TS-Loader (z. B. `bun` oder `tsx`) oder zur Laufzeit vorab kompiliertes `.js`.
- Setzen Sie `deliver: true` + `channel`/`to` bei Zuordnungen, um Antworten an eine Chat-Oberfläche zu routen
  (`channel` ist standardmäßig `last` und fällt auf WhatsApp zurück).
- `agentId` leitet den Hook an einen bestimmten Agenten weiter; unbekannte IDs fallen auf den Standard-Agenten zurück.
- `hooks.allowedAgentIds` beschränkt explizites `agentId`-Routing. Weglassen (oder `*` einschließen), um jeden Agenten zuzulassen. `[]` setzen, um explizites `agentId`-Routing zu verweigern.
- `hooks.defaultSessionKey` legt die Standard-Session für Hook-Agent-Ausführungen fest, wenn kein expliziter Schlüssel angegeben ist.
- `hooks.allowRequestSessionKey` steuert, ob `/hooks/agent`-Payloads `sessionKey` setzen dürfen (Standard: `false`).
- `hooks.allowedSessionKeyPrefixes` beschränkt optional explizite `sessionKey`-Werte aus Request-Payloads und Zuordnungen.
- `allowUnsafeExternalContent: true` deaktiviert den externen Content-Safety-Wrapper für diesen Hook
  (gefährlich; nur für vertrauenswürdige interne Quellen).
- `openclaw webhooks gmail setup` schreibt `hooks.gmail`-Konfiguration für `openclaw webhooks gmail run`.
  Siehe [Gmail Pub/Sub](/automation/gmail-pubsub) für den vollständigen Gmail-Watch-Flow.

## Antworten

- `200` für `/hooks/wake`
- `202` für `/hooks/agent` (asynchroner Lauf gestartet)
- `401` bei Authentifizierungsfehler
- `429` nach wiederholten Authentifizierungsfehlern vom selben Client (siehe `Retry-After`)
- `400` bei ungültigem Payload
- `413` bei zu großen Payloads

## Beispiele

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

### Ein anderes Modell verwenden

Fügen Sie `model` zum Agent-Payload (oder zur Zuordnung) hinzu, um das Modell für diesen Lauf zu überschreiben:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Wenn Sie `agents.defaults.models` erzwingen, stellen Sie sicher, dass das Override-Modell dort enthalten ist.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Sicherheit

- Halten Sie Hook-Endpunkte hinter Loopback, Tailnet oder einem vertrauenswürdigen Reverse Proxy.
- Verwenden Sie ein dediziertes Hook-Token; verwenden Sie Gateway-Authentifizierungs-Token nicht wieder.
- Wiederholte Authentifizierungsfehler werden pro Client-Adresse rate-limitiert, um Brute-Force-Versuche zu verlangsamen.
- Wenn du Multi-Agent-Routing verwendest, setze `hooks.allowedAgentIds`, um die explizite Auswahl von `agentId` zu begrenzen.
- Behalte `hooks.allowRequestSessionKey=false` bei, es sei denn, du benötigst vom Aufrufer ausgewählte Sessions.
- Wenn du Request-`sessionKey` aktivierst, beschränke `hooks.allowedSessionKeyPrefixes` (zum Beispiel `["hook:"]`).
- Vermeiden Sie es, sensible rohe Payloads in Webhook-Logs aufzunehmen.
- Hook-Payloads werden standardmäßig als nicht vertrauenswürdig behandelt und mit Sicherheitsgrenzen umschlossen.
  Wenn Sie dies für einen bestimmten Hook deaktivieren müssen, setzen Sie `allowUnsafeExternalContent: true`
  in der Zuordnung dieses Hooks (gefährlich).

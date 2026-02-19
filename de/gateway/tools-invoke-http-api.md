---
summary: "Ein einzelnes Werkzeug direkt über den Gateway-HTTP-Endpunkt aufrufen"
read_when:
  - Aufrufen von Werkzeugen ohne Ausführen eines vollständigen Agent-Durchlaufs
  - Erstellen von Automatisierungen, die Durchsetzung von Werkzeugrichtlinien benötigen
title: "Tools-Invoke-API"
---

# Tools-Aufruf (HTTP)

Das Gateway von OpenClaw stellt einen einfachen HTTP-Endpunkt bereit, um ein einzelnes Werkzeug direkt aufzurufen. Er ist immer aktiviert, jedoch durch Gateway-Authentifizierung und Werkzeugrichtlinien abgesichert.

- `POST /tools/invoke`
- Derselbe Port wie das Gateway (WS + HTTP-Multiplex): `http://<gateway-host>:<port>/tools/invoke`

Die maximale Standard-Payload-Größe beträgt 2 MB.

## Authentifizierung

Verwendet die Gateway-Authentifizierungskonfiguration. Senden Sie ein Bearer-Token:

- `Authorization: Bearer <token>`

Hinweise:

- Wenn `gateway.auth.mode="token"`, verwenden Sie `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`).
- Wenn `gateway.auth.mode="password"`, verwenden Sie `gateway.auth.password` (oder `OPENCLAW_GATEWAY_PASSWORD`).
- Wenn `gateway.auth.rateLimit` konfiguriert ist und zu viele Authentifizierungsfehler auftreten, gibt der Endpunkt `429` mit `Retry-After` zurück.

## Request-Body

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Felder:

- `tool` (string, erforderlich): Name des aufzurufenden Werkzeugs.
- `action` (string, optional): wird in args gemappt, wenn das Werkzeugschema `action` unterstützt und die args-Payload dies ausgelassen hat.
- `args` (object, optional): werkzeugspezifische Argumente.
- `sessionKey` (string, optional): Ziel-Sitzungsschlüssel. Wenn ausgelassen oder `"main"`, verwendet das Gateway den konfigurierten Haupt-Sitzungsschlüssel (berücksichtigt `session.mainKey` und den Standard-Agenten oder `global` im globalen Geltungsbereich).
- `dryRun` (boolean, optional): für zukünftige Verwendung reserviert; derzeit ignoriert.

## Richtlinien- und Routing-Verhalten

Die Verfügbarkeit von Werkzeugen wird durch dieselbe Richtlinienkette gefiltert, die von Gateway-Agenten verwendet wird:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- Gruppenrichtlinien (wenn der Sitzungsschlüssel einer Gruppe oder einem Kanal zugeordnet ist)
- Subagenten-Richtlinie (bei Aufruf mit einem Subagenten-Sitzungsschlüssel)

Wenn ein Werkzeug durch die Richtlinie nicht erlaubt ist, gibt der Endpunkt **404** zurück.

Gateway HTTP wendet standardmäßig auch eine harte Sperrliste an (selbst wenn die Sitzungsrichtlinie das Tool erlaubt):

- `sessions_spawn`
- `sessions_send`
- `gateway`
- `whatsapp_login`

Sie können diese Sperrliste über `gateway.tools` anpassen:

```json5
{
  gateway: {
    tools: {
      // Additional tools to block over HTTP /tools/invoke
      deny: ["browser"],
      // Remove tools from the default deny list
      allow: ["gateway"],
    },
  },
}
```

Um Gruppenrichtlinien bei der Kontextauflösung zu unterstützen, können Sie optional setzen:

- `x-openclaw-message-channel: <channel>` (Beispiel: `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (wenn mehrere Konten existieren)

## Responses

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (ungültige Anfrage oder Werkzeugfehler)
- `401` → nicht autorisiert
- `429` → Authentifizierung ratenbegrenzt (`Retry-After` gesetzt)
- `404` → Werkzeug nicht verfügbar (nicht gefunden oder nicht auf der Allowlist)
- `405` → Methode nicht erlaubt
- `500` → `{ ok: false, error: { type, message } }` (unerwarteter Tool-Ausführungsfehler; bereinigte Fehlermeldung)

## Beispiel

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```


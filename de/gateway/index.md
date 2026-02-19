---
summary: "„Runbook für den Gateway‑Dienst, Lebenszyklus und Betrieb“"
read_when:
  - Beim Ausführen oder Debuggen des Gateway‑Prozesses
title: "„Gateway‑Runbook“"
---

# Gateway‑Dienst‑Runbook

Verwenden Sie diese Seite für den Day-1-Start und die Day-2-Operationen des Gateway-Dienstes.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Symptomorientierte Diagnose mit exakten Befehlsabfolgen und Log-Signaturen.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Aufgabenorientierte Einrichtungsanleitung + vollständige Konfigurationsreferenz.
  
</Card>
</CardGroup>

## 5-Minuten-Local-Startup

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Gesunde Basis: `Runtime: running` und `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Das Gateway-Konfigurations-Reload überwacht den aktiven Konfigurationsdateipfad (aufgelöst aus Profil-/Status-Standardwerten oder `OPENCLAW_CONFIG_PATH`, wenn gesetzt).
Standardmodus: `gateway.reload.mode="hybrid"` (sichere Änderungen hot anwenden, Neustart bei kritischen Änderungen).
</Note>

## Laufzeitmodell

- Ein dauerhaft laufender Prozess für Routing, Control Plane und Kanalverbindungen.
- Single‑Port‑Multiplex.
  - WebSocket-Control/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - Control-UI und Hooks
- Standard-Bind-Modus: `loopback`.
- Gateway‑Authentifizierung ist standardmäßig erforderlich: setzen Sie `gateway.auth.token` (oder `OPENCLAW_GATEWAY_TOKEN`) oder `gateway.auth.password`.

### Port- und Bind-Priorität

| Einstellung                                                         | Auflösungsreihenfolge                                                                                                   |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Abgeleitete Ports (Faustregeln): | Port‑Priorität: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > Standard `18789`. |
| Bind-Modus                                                          | CLI/Override → `gateway.bind` → `loopback`                                                                              |

### Hot-Reload-Modi

| Deaktivieren mit `gateway.reload.mode="off"`. | Verhalten                                          |
| ------------------------------------------------------------- | -------------------------------------------------- |
| `off`                                                         | Kein Konfigurations-Reload                         |
| `hot`                                                         | Nur hot-sichere Änderungen anwenden                |
| `restart`                                                     | Bei reload-erforderlichen Änderungen neu starten   |
| `hybrid` (Standard)                        | Wenn möglich hot anwenden, andernfalls neu starten |

## Operator-Befehlssatz

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Remote‑Zugriff

Tailscale/VPN bevorzugt; andernfalls SSH‑Tunnel:
Fallback: SSH-Tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Clients verbinden sich anschließend über den Tunnel mit `ws://127.0.0.1:18789`.

<Warning>
Wenn ein Token konfiguriert ist, müssen Clients es auch über den Tunnel in `connect.params.auth.token` mitsenden.
</Warning>

Siehe: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Überwachung und Service-Lebenszyklus

Verwenden Sie überwachte Läufe für produktionsähnliche Zuverlässigkeit.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
Zum Neustart verwenden Sie `openclaw gateway restart` (oder `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
```

OpenClaw.app kann ein Node‑basiertes Gateway‑Relay bündeln und einen benutzerbezogenen LaunchAgent mit der Bezeichnung
`bot.molt.gateway` (oder `bot.molt.<profile> `; Legacy `com.openclaw.*`‑Labels werden weiterhin sauber entladen).`(benanntes Profil).`openclaw doctor\` prüft und behebt Konfigurationsabweichungen des Service.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Aktivieren Sie Lingering (erforderlich, damit der Benutzerdienst Logout/Idle überlebt):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Verwenden Sie eine System-Unit für Multi-User-/Always-on-Hosts.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Mehrere Gateways (gleicher Host)

Die meisten Setups sollten **ein** Gateway ausführen.
Verwenden Sie mehrere Gateways nur für Redundanz oder strikte Isolation (z. B.

Checkliste pro Instanz:

- eindeutiges `gateway.port`
- eindeutiges `OPENCLAW_CONFIG_PATH`
- eindeutiges `OPENCLAW_STATE_DIR`
- eindeutiges `agents.defaults.workspace`

Beispiel:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Vollständige Anleitung: [Multiple gateways](/gateway/multiple-gateways).

### Dev‑Profil (`--dev`)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

Standardmäßig enthalten sind ein isolierter Status/Konfiguration und der Basis-Gateway-Port `19001`.

## Protokoll (Operator‑Sicht)

- Der erste Client-Frame muss `connect` sein.
- Das Gateway gibt einen `hello-ok`-Snapshot zurück (`presence`, `health`, `stateVersion`, `uptimeMs`, Limits/Policy).
- Requests: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Häufige Events: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Agent-Läufe sind zweistufig:

1. Sofortige Bestätigungs-Antwort (`status:"accepted"`)
2. `agent`‑Antworten sind zweistufig: zuerst `res`‑Ack `{runId,status:"accepted"}`, dann ein finales `res` `{runId,status:"ok"|"error",summary}` nach Abschluss des Laufs; gestreamte Ausgabe trifft als `event:"agent"` ein.

Vollständige Doku: [Gateway‑Protokoll](/gateway/protocol) und [Bridge‑Protokoll (Legacy)](/gateway/bridge-protocol).

## Betriebliche Prüfungen

### Liveness

- WS öffnen und `connect` senden.
- `hello-ok`-Antwort mit Snapshot erwarten.

### Readiness

```bash
`openclaw gateway health|status` — Health/Status über den Gateway‑WS anfordern.
```

### Gap-Recovery

Events werden nicht wiederholt. Bei Sequenzlücken den Status (`health`, `system-presence`) aktualisieren, bevor fortgefahren wird.

## Häufige Fehlersignaturen

| Signatur                                                       | Wahrscheinliches Problem                                 |
| -------------------------------------------------------------- | -------------------------------------------------------- |
| `weigert sich, Gateway zu binden ... ohne Authentifizierung`   | Bind an Nicht-Loopback-Adresse ohne Token/Passwort       |
| `another gateway instance is already listening` / `EADDRINUSE` | Portkonflikt                                             |
| `Gateway start blocked: set gateway.mode=local`                | Konfiguration auf Remote-Modus gesetzt                   |
| `unauthorized` während des Verbindungsaufbaus                  | Authentifizierungsabweichung zwischen Client und Gateway |

Für vollständige Diagnoseleitfäden siehe [Gateway Troubleshooting](/gateway/troubleshooting).

## Sicherheitsgarantien

- Gateway-Protokoll-Clients schlagen sofort fehl, wenn das Gateway nicht verfügbar ist (kein impliziter Direct-Channel-Fallback).
- Ungültige/Nicht-`connect`-First-Frames werden abgelehnt und die Verbindung wird geschlossen.
- Geordneter Shutdown: Emit `shutdown`‑Event vor dem Schließen; Clients müssen Close + Reconnect handhaben.

---

Verwandt:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)


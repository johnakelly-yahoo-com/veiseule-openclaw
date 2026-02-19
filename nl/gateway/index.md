---
summary: "Runbook voor de Gateway-service, levenscyclus en operations"
read_when:
  - Het Gateway-proces draaien of debuggen
title: "Gateway-draaiboek"
---

# Gateway service runbook

Gebruik deze pagina voor dag-1 opstart en dag-2 operaties van de Gateway-service.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Symptoomgerichte diagnose met exacte commandolijsten en logsignaturen.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Taakgerichte installatiegids + volledige configuratiereferentie.
  
</Card>
</CardGroup>

## 5-minuten lokale opstart

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
Linux: `openclaw-gateway-<profile>.service`
```

Gezonde basis: `Runtime: running` en `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway-configuratieherladen bewaakt het actieve configuratiebestandspad (opgelost vanuit profiel-/statusstandaarden, of `OPENCLAW_CONFIG_PATH` wanneer ingesteld).
Standaardmodus: `gateway.reload.mode="hybrid"` (pas veilige wijzigingen hot toe, herstart bij kritisch).
</Note>

## Runtime-model

- Het always-on proces dat de enkele Baileys/Telegram-verbinding en het control-/eventplane beheert.
- Single-port multiplex.
  - WebSocket control/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - Control-UI en hooks
- Standaard bindmodus: `loopback`.
- Gateway-authenticatie is standaard vereist: stel `gateway.auth.token` (of `OPENCLAW_GATEWAY_TOKEN`) of `gateway.auth.password` in.

### Poort- en bindprioriteit

| Instelling    | Volgorde van resolutie                                                                                                     |
| ------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Gateway-poort | Poortprecedentie: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > standaard `18789`. |
| Bindmodus     | CLI/override → `gateway.bind` → `loopback`                                                                                 |

### Hot-reloadmodi

| Uitschakelen met `gateway.reload.mode="off"`. | Keepalive-gedrag                                       |
| ------------------------------------------------------------- | ------------------------------------------------------ |
| `off`                                                         | Geen config-herladen                                   |
| `hot`                                                         | Pas alleen hot-veilige wijzigingen toe                 |
| `restart`                                                     | Herstart bij wijzigingen die een herstart vereisen     |
| `hybrid` (standaard)                       | Hot-toepassen indien veilig, herstarten indien vereist |

## Operator-commandoverzameling

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Externe toegang

Tailscale/VPN heeft de voorkeur; anders een SSH-tunnel:
Fallback: SSH-tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Clients verbinden vervolgens met `ws://127.0.0.1:18789` via de tunnel.

<Warning>
Als een token is geconfigureerd, moeten clients dit opnemen in `connect.params.auth.token`, zelfs over de tunnel.
</Warning>

Zie: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Supervisie en servicelifecycle

Gebruik supervised runs voor productie-achtige betrouwbaarheid.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

OpenClaw.app kan een Node-gebaseerde gateway-relay bundelen en een per-gebruiker LaunchAgent installeren met label
`bot.molt.gateway` (of `bot.molt.<profile> `; legacy `com.openclaw.*`-labels worden nog netjes unloaded).`(benoemd profiel).`openclaw doctor\` controleert en herstelt configuratie-afwijkingen van de service.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Schakel lingering in (vereist zodat de user service logout/idle overleeft):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Gebruik een system unit voor multi-user/always-on hosts.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Meerdere gateways (zelfde host)

Meestal onnodig: één Gateway kan meerdere messagingkanalen en agents bedienen.
Gebruik meerdere Gateways alleen voor redundantie of strikte isolatie (bijv. rescue bot).

Checklist per instantie:

- unieke `gateway.port`
- unieke `OPENCLAW_CONFIG_PATH`
- unieke `OPENCLAW_STATE_DIR`
- unieke `agents.defaults.workspace`

Voorbeeld:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Zie [Multiple gateways](/gateway/multiple-gateways).

### Dev-profiel (`--dev`)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

Standaardinstellingen omvatten een geïsoleerde state/configuratie en basis gateway-poort `19001`.

## Protocol (operatorperspectief)

- Het eerste clientframe moet `connect` zijn.
- Gateway retourneert een `hello-ok`-snapshot (`presence`, `health`, `stateVersion`, `uptimeMs`, limieten/beleid).
- Requests: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Veelvoorkomende events: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Agent-runs bestaan uit twee fasen:

1. Onmiddellijke accepted-ack (`status:"accepted"`)
2. `agent`-responses zijn tweefasig: eerst `res` ack `{runId,status:"accepted"}`, daarna een finale `res` `{runId,status:"ok"|"error",summary}` nadat de run is voltooid; gestreamde uitvoer arriveert als `event:"agent"`.

Volledige documentatie: [Gateway protocol](/gateway/protocol) en [Bridge protocol (legacy)](/gateway/bridge-protocol).

## Operationele controles

### Liveness

- Open WS en verstuur `connect`.
- Verwacht een `hello-ok`-antwoord met snapshot.

### Gereedheid

```bash
`openclaw gateway health|status` — vraag health/status op via de Gateway WS.
```

### Gap-herstel

Events worden niet herhaald. Bij sequentiegaten, vernieuw de state (`health`, `system-presence`) voordat je doorgaat.

## Veelvoorkomende foutsignaturen

| Signatuur                                                      | Waarschijnlijk probleem                         |
| -------------------------------------------------------------- | ----------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Non-loopback binding zonder token/wachtwoord    |
| `another gateway instance is already listening` / `EADDRINUSE` | Poortconflict                                   |
| `Gateway start blocked: set gateway.mode=local`                | Configuratie ingesteld op remote-modus          |
| `unauthorized` tijdens connect                                 | Authenticatie-mismatch tussen client en Gateway |

Gebruik voor volledige diagnose-stappenplannen [Gateway Troubleshooting](/gateway/troubleshooting).

## Veiligheidsgaranties

- Gateway-protocolclients falen direct wanneer Gateway niet beschikbaar is (geen impliciete direct-channel fallback).
- Niet-connect first frames of malformed JSON worden geweigerd en de socket wordt gesloten.
- Graceful shutdown: zend `shutdown`-event vóór sluiten; clients moeten sluiten + opnieuw verbinden afhandelen.

---

Gerelateerd:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)


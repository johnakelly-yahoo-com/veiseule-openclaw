---
summary: "Runbook for Gateway-tjenesten, livscyklus og drift"
read_when:
  - Når du kører eller fejlsøger gateway-processen
title: "Driftsvejledning for Gateway"
---

# Gateway service runbook

Sidst opdateret: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Symptombaseret diagnostik med præcise kommandotrin og logsignaturer.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Opgaveorienteret installationsguide + fuld konfigurationsreference.
  
</Card>
</CardGroup>

## 5-minutters lokal opstart

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
openclaw gateway status
openclaw status
openclaw logs --follow
```

Sund baseline: `Runtime: running` og `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway-konfigurationsgenindlæsning overvåger den aktive konfigurationsfilsti (løst fra profil-/tilstandsstandarder eller `OPENCLAW_CONFIG_PATH`, når den er sat).
Standardtilstand er `gateway.reload.mode="hybrid"`.
</Note>

## Runtime-model

- Én altid kørende proces til routing, kontrolplan og kanalforbindelser.
- Enkelt multiplekset port til:
  - WebSocket kontrol/RPC
  - HTTP API’er (OpenAI-kompatibel, Responses, værktøjsinvokation)
  - Kontrol-UI og hooks
- Standard bind-tilstand: `loopback`.
- Autentificering er påkrævet som standard (`gateway.auth.token` / `gateway.auth.password` eller `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Dev-profil (`--dev`)

| Indstilling   | Rækkefølge for opløsning                                      |
| ------------- | ------------------------------------------------------------- |
| Gateway-port  | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Bind-tilstand | CLI/override → `gateway.bind` → `loopback`                    |

### Hot reload-tilstande

| `gateway.reload.mode`                  | Adfærd                                                  |
| -------------------------------------- | ------------------------------------------------------- |
| `off`                                  | Ingen genindlæsning af konfiguration                    |
| `hot`                                  | Anvend kun hot-sikre ændringer                          |
| `restart`                              | Genstart ved ændringer, der kræver genstart             |
| `hybrid` (standard) | Anvend hot, når det er sikkert, genstart når det kræves |

## Operatørens kommandosæt

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw logs --follow
openclaw doctor
```

## Fjernadgang

Foretrukket: Tailscale/VPN.
Fallback: SSH-tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Serviceinstallation pr. profil:

<Warning>
Hvis gateway-godkendelse er konfigureret, skal klienter stadig sende godkendelse (`token`/`password`), selv over SSH-tunneler.
</Warning>

Eksempel:

## Overvågning og tjenestens livscyklus

Brug overvågede kørseler for produktionslignende pålidelighed.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

LaunchAgent-labels er `ai.openclaw.gateway` (standard) eller `ai.openclaw.<profile>` (navngiven profil). `openclaw doctor` gennemgår og reparerer afvigelser i tjenestekonfigurationen.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

For at bevare tjenesten efter logout, aktivér lingering:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

Brug en systemenhed til multi-user/always-on-værter.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Flere gateways på én vært

De fleste opsætninger bør køre **én** Gateway.
Brug flere kun for streng isolation/redundans (for eksempel en redningsprofil).

Tjekliste pr. instans:

- Unik `gateway.port`
- Unik `OPENCLAW_CONFIG_PATH`
- Unik `OPENCLAW_STATE_DIR`
- Unik `agents.defaults.workspace`

Eksempel:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Se: [Multiple gateways](/gateway/multiple-gateways).

### Gateway service management (CLI)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Standardindstillinger inkluderer isoleret state/config og basis gateway-port `19001`.

## Hurtig protokolreference (operatørvisning)

- `gateway status` sonderer Gateway RPC som standard ved brug af tjenestens opløste port/konfiguration (kan tilsidesættes med `--url`).
- `gateway status --deep` tilføjer systemniveau-scanninger (LaunchDaemons/system-enheder).
- `gateway status --no-probe` springer RPC-sonden over (nyttigt når netværk er nede).
- `gateway status --json` er stabil til scripts.

Bundlet mac-app:

1. Øjeblikkelig accepteret kvittering (`status:"accepted"`)
2. For at stoppe den rent, brug `openclaw gateway stop` (eller `launchctl bootout gui/$UID/bot.molt.gateway`).

Se fuld protokoldokumentation: [Gateway Protocol](/gateway/protocol).

## Driftskontroller

### Liveness

- Åbn WS og send `connect`.
- Forvent svar med `hello-ok` og snapshot.

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Gap recovery

Begivenheder afspilles ikke igen. Ved sekvenshuller, opdatér state (`health`, `system-presence`) før du fortsætter.

## Almindelige fejlsignaturer

| Signatur                                                       | Sandsynligt problem                             |
| -------------------------------------------------------------- | ----------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Binding uden for loopback uden token/password   |
| `another gateway instance is already listening` / `EADDRINUSE` | Portkonflikt                                    |
| `Gateway start blokeret: sæt gateway.mode=local`               | Konfiguration sat til remote-tilstand           |
| `unauthorized` under forbindelse                               | Auth-uoverensstemmelse mellem klient og gateway |

For fuld diagnostisk gennemgang, brug [Gateway Troubleshooting](/gateway/troubleshooting).

## Windows (WSL2)

- Gateway-protokolklienter fejler straks, når Gateway ikke er tilgængelig (ingen implicit fallback til direkte kanal).
- Ugyldige/ikke-forbindende første frames afvises og lukkes.
- Graciøs nedlukning udsender `shutdown`-event før socket-lukning.

---

Relateret:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)


---
summary: "Drifthandbok för Gateway-tjänsten, livscykel och drift"
read_when:
  - När du kör eller felsöker gateway-processen
title: "Gateway-drifthandbok"
---

# Drifthandbok för Gateway-tjänsten

Senast uppdaterad: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">Symtombaserad diagnostik med exakta kommandosteg och loggsignaturer.
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">Uppgiftsorienterad installationsguide + fullständig konfigurationsreferens.
</Card>
</CardGroup>

## 5-minuters lokal uppstart

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

Hälsosam grundstatus: `Runtime: running` och `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway-konfigurationens omladdning övervakar den aktiva konfigurationsfilens sökväg (hämtad från profil-/standardvärden för tillstånd, eller `OPENCLAW_CONFIG_PATH` när den är angiven).
Standardläget är `gateway.reload.mode="hybrid"`.
</Note>

## Körningsmodell

- En alltid aktiv process för routing, kontrollplan och kanalanslutningar.
- En enda multiplexad port för:
  - WebSocket-kontroll/RPC
  - HTTP-API:er (OpenAI-kompatibla, Responses, verktygsanrop)
  - Kontrollgränssnitt och hooks
- Standardbindningsläge: `loopback`.
- Autentisering krävs som standard (`gateway.auth.token` / `gateway.auth.password`, eller `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Dev-profil (`--dev`)

| Inställning   | Upplösningsordning                                            |
| ------------- | ------------------------------------------------------------- |
| Gateway-port  | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Bindningsläge | CLI/override → `gateway.bind` → `loopback`                    |

### Hot reload-lägen

| `gateway.reload.mode`                  | Beteende                                                   |
| -------------------------------------- | ---------------------------------------------------------- |
| `off`                                  | Ingen konfigurationsomladdning                             |
| `hot`                                  | Tillämpa endast hot-säkra ändringar                        |
| `restart`                              | Starta om vid ändringar som kräver omladdning              |
| `hybrid` (standard) | Tillämpa direkt när det är säkert, starta om när det krävs |

## Operatörskommandon

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

## Fjärråtkomst

Rekommenderas: Tailscale/VPN.
Reservlösning: SSH-tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Tjänstinstallation per profil:

<Warning>
Om gateway-autentisering är konfigurerad måste klienter fortfarande skicka autentisering (`token`/`password`) även över SSH-tunnlar.
</Warning>

Exempel:

## Övervakning och tjänstens livscykel

Använd övervakade körningar för produktionsliknande tillförlitlighet.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

LaunchAgent-etiketter är `ai.openclaw.gateway` (standard) eller `ai.openclaw.<profile>` (namngiven profil). `openclaw doctor` granskar och åtgärdar avvikelser i tjänstekonfigurationen.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

För beständighet efter utloggning, aktivera lingering:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

Använd en systemenhet för värdar med flera användare/alltid-på.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Flera gateways på en och samma värd

De flesta installationer bör köra **en** Gateway.
Använd flera endast för strikt isolering/redundans (till exempel en räddningsprofil).

Checklista per instans:

- Unik `gateway.port`
- Unik `OPENCLAW_CONFIG_PATH`
- Unik `OPENCLAW_STATE_DIR`
- Unik `agents.defaults.workspace`

Exempel:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Se: [Multiple gateways](/gateway/multiple-gateways).

### Hantering av Gateway-tjänsten (CLI)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Standardinställningarna inkluderar isolerad state/config och basporten `19001` för gateway.

## Snabbreferens för protokoll (operatörsvy)

- `gateway status` sonderar Gateway-RPC som standard med tjänstens lösta port/konfig (åsidosätt med `--url`).
- `gateway status --deep` lägger till systemomfattande skanningar (LaunchDaemons/systemenheter).
- `gateway status --no-probe` hoppar över RPC-sonderingen (användbart när nätverk är nere).
- `gateway status --json` är stabilt för skript.

Paketerad mac-app:

1. Omedelbar bekräftelse mottagen (`status:"accepted"`)
2. För att stoppa den på ett rent sätt, använd `openclaw gateway stop` (eller `launchctl bootout gui/$UID/bot.molt.gateway`).

Se fullständig protokolldokumentation: [Gateway Protocol](/gateway/protocol).

## Operativa kontroller

### Liveness

- Öppna WS och skicka `connect`.
- Förvänta svaret `hello-ok` med en ögonblicksbild.

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Återställning vid sekvensglapp

Händelser spelas inte om. Vid sekvensglapp, uppdatera tillståndet (`health`, `system-presence`) innan du fortsätter.

## Vanliga felsignaturer

| Signatur                                                       | Trolig orsak                                     |
| -------------------------------------------------------------- | ------------------------------------------------ |
| `refusing to bind gateway ... without auth`                    | Bindning till icke-loopback utan token/lösenord  |
| `another gateway instance is already listening` / `EADDRINUSE` | Portkonflikt                                     |
| `Gateway start blocked: set gateway.mode=local`                | Konfigurationen är inställd på fjärrläge         |
| `unauthorized` vid anslutning                                  | Autentiseringsmismatch mellan klient och gateway |

För fullständiga diagnossteg, använd [Gateway Troubleshooting](/gateway/troubleshooting).

## Windows (WSL2)

- Gateway-protokollklienter misslyckas omedelbart när Gateway är otillgänglig (ingen implicit fallback till direktkanal).
- Ogiltiga/icke-anslutande första ramar avvisas och stängs.
- En kontrollerad nedstängning skickar händelsen `shutdown` innan socketen stängs.

---

Relaterat:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)


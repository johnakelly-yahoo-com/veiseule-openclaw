---
summary: "Runbook para sa serbisyo ng Gateway, lifecycle, at mga operasyon"
read_when:
  - Kapag pinapatakbo o dini-debug ang proseso ng gateway
title: "Runbook ng Gateway"
---

# Gateway service runbook

Huling na-update: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Diagnostics na inuuna ang sintomas na may eksaktong mga command ladder at log signature.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Gabay sa setup na nakatuon sa gawain + kumpletong configuration reference.
  
</Card>
</CardGroup>

## 5-minutong lokal na pagsisimula

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

Malusog na baseline: `Runtime: running` at `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Ang pag-reload ng Gateway config ay nagbabantay sa aktibong path ng config file (na ni-resolve mula sa mga default ng profile/state, o `OPENCLAW_CONFIG_PATH` kapag nakatakda).
Ang default na mode ay `gateway.reload.mode="hybrid"`.
</Note>

## Modelo ng runtime

- Isang palaging tumatakbong proseso para sa routing, control plane, at mga koneksyon ng channel.
- Isang multiplexed na port para sa:
  - WebSocket control/RPC
  - HTTP APIs (OpenAI-compatible, Responses, pag-invoke ng tools)
  - Control UI at mga hook
- Default na bind mode: `loopback`.
- Kinakailangan ang auth bilang default (`gateway.auth.token` / `gateway.auth.password`, o `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Precedence ng port at bind

| Setting      | Pagkakasunod-sunod ng resolusyon                              |
| ------------ | ------------------------------------------------------------- |
| Gateway port | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Bind mode    | CLI/override → `gateway.bind` → `loopback`                    |

### Mga hot reload mode

| `gateway.reload.mode`                 | Pag-uugali                                                      |
| ------------------------------------- | --------------------------------------------------------------- |
| `off`                                 | Walang config reload                                            |
| `hot`                                 | Ilapat lamang ang mga pagbabagong ligtas para sa hot reload     |
| `restart`                             | Mag-restart kapag may mga pagbabagong nangangailangan ng reload |
| `hybrid` (default) | Hot-apply kapag ligtas, i-restart kapag kinakailangan           |

## Hanay ng mga utos ng operator

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

## Remote access

Mas mainam: Tailscale/VPN.
Fallback: SSH tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Pagkatapos, ikonekta ang mga client sa `ws://127.0.0.1:18789` nang lokal.

<Warning>
Kung naka-configure ang gateway auth, kailangan pa ring magpadala ng auth (`token`/`password`) ang mga client kahit sa SSH tunnels.
</Warning>

Tingnan: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Protocol (pananaw ng operator)

Gumamit ng supervised runs para sa production-like na pagiging maaasahan.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

Ang mga label ng LaunchAgent ay `ai.openclaw.gateway` (default) o `ai.openclaw.<profile>` (pinangalanang profile). Ang `openclaw doctor` ay nagsusuri at nag-aayos ng service config drift.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

Para magpatuloy kahit pagkatapos mag-logout, i-enable ang lingering:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

Gumamit ng system unit para sa multi-user/laging naka-on na mga host.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Keepalive behavior

Karamihan ng mga setup ay dapat magpatakbo ng **isang** Gateway.
Gumamit lamang ng maramihan para sa mahigpit na isolation/redundancy (halimbawa, isang rescue profile).

Checklist kada instance:

- Natatanging `gateway.port`
- Natatanging `OPENCLAW_CONFIG_PATH`
- Natatanging `OPENCLAW_STATE_DIR`
- Natatanging `agents.defaults.workspace`

Halimbawa:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Tingnan: [Multiple gateways](/gateway/multiple-gateways).

### Mabilis na gabay para sa dev profile

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

Mga tala:

## Mabilis na sanggunian ng protocol (pananaw ng operator)

- Ang unang client frame ay dapat `connect`.
- Nagbabalik ang Gateway ng `hello-ok` snapshot (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy).
- Mga request: `req(method, params)` → `res(ok/payload|error)`.
- Karaniwang mga event: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Dalawang yugto ang agent runs:

1. Agarang accepted ack (`status:"accepted"`)
2. Panghuling completion response (`status:"ok"|"error"`), na may naka-stream na `agent` events sa pagitan.

Tingnan ang buong dokumentasyon ng protocol: [Gateway Protocol](/gateway/protocol).

## Operational checks

### Liveness

- Magbukas ng WS at magpadala ng `connect`.
- Asahan ang `hello-ok` na tugon na may snapshot.

### Readiness

```bash
sudo loginctl enable-linger youruser
```

### Gap recovery

25. }`. Kapag may sequence gaps, i-refresh ang state (`health`, `system-presence\`) bago magpatuloy.

## Mga karaniwang failure signature

| Signature                                                      | Malamang na isyu                                     |
| -------------------------------------------------------------- | ---------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Non-loopback bind nang walang token/password         |
| `another gateway instance is already listening` / `EADDRINUSE` | Conflict sa port                                     |
| `Gateway start blocked: set gateway.mode=local`                | Naka-set ang config sa remote mode                   |
| `unauthorized` habang kumokonekta                              | Hindi tugma ang auth sa pagitan ng client at gateway |

Para sa kumpletong mga hakbang sa diagnosis, gamitin ang [Gateway Troubleshooting](/gateway/troubleshooting).

## Mga garantiya sa kaligtasan

- Ang mga Gateway protocol client ay agad nagfa-fail kapag hindi available ang Gateway (walang implicit direct-channel fallback).
- Ang mga invalid/non-connect na unang frame ay tinatanggihan at isinasara.
- Ang maayos na shutdown ay naglalabas ng `shutdown` event bago isara ang socket.

---

Kaugnay:

- Ipagpalagay ang isang Gateway bawat host bilang default; kung nagpapatakbo ng maraming profile, ihiwalay ang mga port/state at i-target ang tamang instance.
- Walang fallback sa direktang koneksyon ng Baileys; kung down ang Gateway, mabilis na babagsak ang mga send.
- Ang mga non-connect first frame o malformed JSON ay tinatanggihan at isinasara ang socket.
- Maayos na shutdown: mag-emit ng `shutdown` event bago magsara; dapat hawakan ng mga client ang close + reconnect.
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

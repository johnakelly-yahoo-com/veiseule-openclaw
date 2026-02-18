---
title: "Runbook ng Gateway"
---

# Gateway runbook

Gamitin ang pahinang ito para sa day-1 startup at day-2 operations ng Gateway service.

<CardGroup cols={2}>
  <Card title="Malalim na troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Symptom-first na diagnostics na may eksaktong command ladders at log signatures.
  </Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Task-oriented na gabay sa setup + kumpletong configuration reference.
  </Card>
</CardGroup>

## 5-minutong local startup

<Steps>
  <Step title="Simulan ang Gateway">

```bash
openclaw gateway --port 18789
# debug/trace mirrored to stdio
openclaw gateway --port 18789 --verbose
# force-kill listener on selected port, then start
openclaw gateway --force
```

  </Step>

  <Step title="I-verify ang service health">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

Healthy baseline: `Runtime: running` at `RPC probe: ok`.

  </Step>

  <Step title="I-validate ang channel readiness">

```bash
openclaw channels status --probe
```

  </Step>
</Steps>

<Note>
Ang Gateway config reload ay nagmo-monitor sa aktibong config file path (na-resolve mula sa profile/state defaults, o `OPENCLAW_CONFIG_PATH` kapag naka-set).
Ang default mode ay `gateway.reload.mode="hybrid"`.
</Note>

## Runtime model

- Isang laging tumatakbong proseso para sa routing, control plane, at mga koneksyon ng channel.
- Isang multiplexed na port para sa:
  - WebSocket control/RPC
  - HTTP APIs (OpenAI-compatible, Responses, tools invoke)
  - Control UI at hooks
- Default bind mode: `loopback`.
- Kinakailangan ang auth bilang default (`gateway.auth.token` / `gateway.auth.password`, o `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Port at bind precedence

| Setting      | Resolution order                                              |
| ------------ | ------------------------------------------------------------- |
| Gateway port | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Bind mode    | CLI/override → `gateway.bind` → `loopback`                    |

### Hot reload modes

| `gateway.reload.mode` | Behavior                                   |
| --------------------- | ------------------------------------------ |
| `off`                 | Walang config reload                       |
| `hot`                 | I-apply lang ang hot-safe na mga pagbabago |
| `restart`             | Mag-restart kapag may reload-required changes |
| `hybrid` (default)    | Hot-apply kapag ligtas, restart kapag kailangan |

## Operator command set

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

## Supervision at service lifecycle

Gumamit ng supervised runs para sa production-like reliability.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

Ang mga LaunchAgent label ay `ai.openclaw.gateway` (default) o `ai.openclaw.<profile>` (named profile). Ang `openclaw doctor` ay nag-a-audit at nag-aayos ng service config drift.

  </Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

Para magpatuloy kahit mag-logout, i-enable ang lingering:

```bash
sudo loginctl enable-linger <user>
```

  </Tab>

  <Tab title="Linux (system service)">

Gumamit ng system unit para sa multi-user/always-on hosts.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  </Tab>
</Tabs>

## Maramihang gateway sa iisang host

Karamihan ng setup ay dapat magpatakbo ng **isang** Gateway.  
Gumamit lamang ng marami para sa mahigpit na isolation/redundancy (halimbawa, rescue profile).

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

### Dev profile quick path

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Kasama sa defaults ang isolated state/config at base gateway port na `19001`.

## Protocol quick reference (pananaw ng operator)

- Ang unang client frame ay dapat `connect`.
- Nagbabalik ang Gateway ng `hello-ok` snapshot (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy).
- Mga request: `req(method, params)` → `res(ok/payload|error)`.
- Mga karaniwang event: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Ang agent runs ay two-stage:

1. Agarang accepted ack (`status:"accepted"`)
2. Final completion response (`status:"ok"|"error"`), na may streamed `agent` events sa pagitan.

Tingnan ang buong protocol docs: [Gateway Protocol](/gateway/protocol).

## Operational checks

### Liveness

- Magbukas ng WS at magpadala ng `connect`.
- Asahan ang `hello-ok` response na may snapshot.

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Gap recovery

Hindi nire-replay ang mga event. Kapag may sequence gaps, i-refresh ang state (`health`, `system-presence`) bago magpatuloy.

## Mga karaniwang failure signature

| Signature                                                      | Malamang na isyu                          |
| -------------------------------------------------------------- | ----------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Non-loopback bind na walang token/password |
| `another gateway instance is already listening` / `EADDRINUSE` | Port conflict                             |
| `Gateway start blocked: set gateway.mode=local`                | Config na naka-set sa remote mode         |
| `unauthorized` during connect                                  | Hindi tugmang auth sa pagitan ng client at gateway |

Para sa kumpletong diagnosis ladders, gamitin ang [Gateway Troubleshooting](/gateway/troubleshooting).

## Mga garantiya sa kaligtasan

- Ang Gateway protocol clients ay fail fast kapag hindi available ang Gateway (walang implicit direct-channel fallback).
- Ang invalid/non-connect first frames ay tinatanggihan at agad na isinasara.
- Ang maayos na shutdown ay nag-e-emit ng `shutdown` event bago isara ang socket.

---

Related:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)


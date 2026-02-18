---
title: "Refaktorering af Exec Host"
---

# Plan for refaktorering af exec-host

## Mål

- Tilføj `exec.host` + `exec.security` for at route eksekvering på tværs af **sandbox**, **gateway** og **node**.
- Bevar **sikre** standarder: ingen eksekvering på tværs af hosts, medmindre det eksplicit er aktiveret.
- Opdel eksekvering i en **headless runner-tjeneste** med valgfri UI (macOS-app) via lokal IPC.
- Giv **per-agent** politik, allowlist, ask mode og node binding.
- Understøt **spørgetilstande**, der virker _med_ eller _uden_ tilladelseslister.
- Platformuafhængigt: Unix-socket + token-autentificering (paritet mellem macOS/Linux/Windows).

## Ikke-mål

- Ingen migrering af legacy-tilladelseslister eller understøttelse af legacy-skema.
- Ingen PTY/streaming for node-exec (kun aggregeret output).
- Intet nyt netværkslag ud over den eksisterende Bridge + Gateway.

## Beslutninger (låst)

- **Konfigurationsnøgler:** `exec.host` + `exec.security` (pr. agent-override tilladt).
- **Elevationsniveau:** behold `/elevated` som et alias for fuld gateway-adgang.
- **Standard for spørg:** `on-miss`.
- **Godkendelseslager:** `~/.openclaw/exec-approvals.json` (JSON, ingen legacy-migrering).
- **Runner:** headless systemtjeneste; UI-appen hoster en Unix-socket til godkendelser.
- **Node-identitet:** brug eksisterende `nodeId`.
- **Socket-autentificering:** Unix-socket + token (platformuafhængigt); opdeles senere hvis nødvendigt.
- **Node host-tilstand:** `~/.openclaw/node.json` (node-id + parringstoken).
- **macOS exec-host:** kør `system.run` inde i macOS-appen; node host-tjenesten videresender forespørgsler via lokal IPC.
- **Ingen XPC-hjælper:** hold dig til Unix-socket + token + peer-checks.

## Nøglebegreber

### Vært

- `sandbox`: Docker-exec (nuværende adfærd).
- `gateway`: exec på gateway-værten.
- `node`: exec på node runner via Bridge (`system.run`).

### Sikkerhedstilstand

- `deny`: bloker altid.
- `allowlist`: tillad kun match.
- `full`: tillad alt (svarende til elevated).

### Spørgetilstand

- `off`: spørg aldrig.
- `on-miss`: spørg kun når tilladelseslisten ikke matcher.
- `always`: spørg hver gang.

Spørg er **uafhængig** af tilladelseslisten; tilladelseslisten kan bruges med `always` eller `on-miss`.

### Politikopløsning (pr. exec)

1. Opløs `exec.host` (værktøjsparameter → agent-override → global standard).
2. Opløs `exec.security` og `exec.ask` (samme præcedens).
3. Hvis host er `sandbox`, fortsæt med lokal sandbox-exec.
4. Hvis host er `gateway` eller `node`, anvend sikkerheds- og spørgepolitik på den pågældende host.

## Standard-sikkerhed

- Standard `exec.host = sandbox`.
- Standard `exec.security = deny` for `gateway` og `node`.
- Standard `exec.ask = on-miss` (kun relevant hvis sikkerhed tillader).
- Hvis ingen node-binding er sat, **kan agenten målrette enhver node**, men kun hvis politikken tillader det.

## Konfigurationsflade

### Værktøjsparametre

- `exec.host` (valgfri): `sandbox | gateway | node`.
- `exec.security` (valgfri): `deny | allowlist | full`.
- `exec.ask` (valgfri): `off | on-miss | always`.
- `exec.node` (valgfri): node-id/navn der skal bruges når `host=node`.

### Konfigurationsnøgler (globalt)

- `tools.exec.host`
- `tools.exec.security`
- `tools.exec.ask`
- `tools.exec.node` (standard node-binding)

### Konfigurationsnøgler (pr. agent)

- `agents.list[].tools.exec.host`
- `agents.list[].tools.exec.security`
- `agents.list[].tools.exec.ask`
- `agents.list[].tools.exec.node`

### Alias

- `/elevated on` = sæt `tools.exec.host=gateway`, `tools.exec.security=full` for agentsessionen.
- `/elevated off` = gendan tidligere exec-indstillinger for agentsessionen.

## Godkendelseslager (JSON)

Sti: `~/.openclaw/exec-approvals.json`

Formål:

- Lokal politik + tilladelseslister for **eksekveringshosten** (gateway eller node runner).
- Spørge-fallback når ingen UI er tilgængelig.
- IPC-legitimationsoplysninger for UI-klienter.

Foreslået skema (v1):

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Noter:

- Ingen legacy-formater for tilladelseslister.
- `askFallback` gælder kun når `ask` er påkrævet og ingen UI kan nås.
- Filrettigheder: `0600`.

## Runner-tjeneste (headless)

### Rolle

- Håndhæv `exec.security` + `exec.ask` lokalt.
- Udfør systemkommandoer og returnér output.
- Udsend Bridge-hændelser for exec-livscyklus (valgfrit men anbefalet).

### Tjenestelivscyklus

- Launchd/daemon på macOS; systemtjeneste på Linux/Windows.
- Godkendelses-JSON er lokal for eksekveringshosten.
- UI hoster en lokal Unix-socket; runners forbinder efter behov.

## UI-integration (macOS-app)

### IPC

- Unix-socket ved `~/.openclaw/exec-approvals.sock` (0600).
- Token gemt i `exec-approvals.json` (0600).
- Peer-checks: kun samme UID.
- Challenge/response: nonce + HMAC(token, request-hash) for at forhindre replay.
- Kort TTL (fx 10s) + maks payload + rate limit.

### Spørgeflow (macOS-app exec-host)

1. Node-tjenesten modtager `system.run` fra gateway.
2. Node-tjenesten forbinder til den lokale socket og sender prompt/exec-forespørgslen.
3. Appen validerer peer + token + HMAC + TTL og viser derefter dialog hvis nødvendigt.
4. Appen udfører kommandoen i UI-kontekst og returnerer output.
5. Node-tjenesten returnerer output til gateway.

Hvis UI mangler:

- Anvend `askFallback` (`deny|allowlist|full`).

### Diagram (SCI)

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

## Node-identitet + binding

- Brug eksisterende `nodeId` fra Bridge-parring.
- Bindingsmodel:
  - `tools.exec.node` begrænser agenten til en specifik node.
  - Hvis ikke sat, kan agenten vælge enhver node (politikken håndhæver stadig standarder).
- Opløsning af nodevalg:
  - `nodeId` eksakt match
  - `displayName` (normaliseret)
  - `remoteIp`
  - `nodeId` præfiks (>= 6 tegn)

## Hændelser

### Hvem ser hændelser

- Systembegivenheder er **per session** og vises til agenten ved næste prompt.
- Gemmes i gatewayens in-memory-kø (`enqueueSystemEvent`).

### Hændelsestekst

- `Exec started (node=<id>, id=<runId>)`
- `Exec finished (node=<id>, id=<runId>, code=<code>)` + valgfri output-tail
- `Exec denied (node=<id>, id=<runId>, <reason>)`

### Transport

Mulighed A (anbefalet):

- Runner sender Bridge `event`-frames `exec.started` / `exec.finished`.
- Gateway `handleBridgeEvent` mapper disse til `enqueueSystemEvent`.

Mulighed B:

- Gateway `exec`-værktøjet håndterer livscyklussen direkte (kun synkront).

## Exec-flows

### Sandbox-host

- Eksisterende `exec`-adfærd (Docker eller host når usandboxed).
- PTY understøttes kun i ikke-sandbox-tilstand.

### Gateway-host

- Gateway-processen eksekverer på sin egen maskine.
- Håndhæver lokal `exec-approvals.json` (sikkerhed/spørg/tilladelsesliste).

### Node-host

- Gateway kalder `node.invoke` med `system.run`.
- Runner håndhæver lokale godkendelser.
- Runner returnerer aggregeret stdout/stderr.
- Valgfrie Bridge-hændelser for start/afslut/afvis.

## Outputbegrænsninger

- Begræns kombineret stdout+stderr til **200k**; behold **tail 20k** til hændelser.
- Afkort med en klar suffiks (f.eks. `"… (afkortet)"`).

## Slash-kommandoer

- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
- Per-agent, per-session tilsidesættelser; ikke-vedvarende medmindre gemmes via config.
- `/elevated on|off|ask|full` forbliver en genvej til `host=gateway security=full` (med `full` der springer godkendelser over).

## Platformuafhængig historie

- Runner-tjenesten er det portable eksekveringsmål.
- UI er valgfri; hvis den mangler, gælder `askFallback`.
- Windows/Linux understøtter samme godkendelses-JSON + socket-protokol.

## Implementeringsfaser

### Fase 1: konfiguration + exec-routing

- Tilføj konfigurationsskema for `exec.host`, `exec.security`, `exec.ask`, `exec.node`.
- Opdatér værktøjs-plumbing til at respektere `exec.host`.
- Tilføj `/exec` slash-kommando og behold `/elevated`-alias.

### Fase 2: godkendelseslager + gateway-håndhævelse

- Implementér `exec-approvals.json` læser/skriver.
- Håndhæv tilladelsesliste + spørgetilstande for `gateway`-host.
- Tilføj outputbegrænsninger.

### Fase 3: node runner-håndhævelse

- Opdatér node runner til at håndhæve tilladelsesliste + spørg.
- Tilføj Unix-socket prompt-bridge til macOS-appens UI.
- Kobl `askFallback`.

### Fase 4: hændelser

- Tilføj node → gateway Bridge-hændelser for exec-livscyklus.
- Map til `enqueueSystemEvent` for agent-prompts.

### Fase 5: UI-polering

- Mac-app: editor til tilladelsesliste, pr. agent-skifter, UI for spørgepolitik.
- Kontroller til node-binding (valgfrit).

## Testplan

- Enhedstests: matchning af tilladelsesliste (glob + ikke-følsom for store/små bogstaver).
- Enhedstests: præcedens for politikopløsning (værktøjsparameter → agent-override → global).
- Integrationstests: node runner afvis/tillad/spørg-flows.
- Bridge-hændelsestests: node-hændelse → systemhændelsesrouting.

## Åbne risici

- UI-utilgængelighed: sikr at `askFallback` respekteres.
- Langvarige kommandoer: stol på timeout + outputbegrænsninger.
- Tvetydighed ved flere noder: fejl medmindre node-binding eller eksplicit node-parameter.

## Relaterede dokumenter

- [Exec-værktøj](/tools/exec)
- [Exec-godkendelser](/tools/exec-approvals)
- [Noder](/nodes)
- [Elevated-tilstand](/tools/elevated)

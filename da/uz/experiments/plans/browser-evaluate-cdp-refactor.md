---
summary: "Plan: isoler browser act:evaluate fra Playwright-køen ved hjælp af CDP, med end-to-end-deadlines og sikrere ref-håndtering"
owner: "openclaw"
status: "kladde"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor Plan

## Kontekst

`act:evaluate` udfører brugerleveret JavaScript på siden. I dag kører den via Playwright
(`page.evaluate` eller `locator.evaluate`). Playwright serialiserer CDP-kommandoer pr. side, så en
fastlåst eller langvarig evaluate kan blokere sidens kommandokø og få enhver efterfølgende handling
på den fane til at se "fastlåst" ud.

PR #13498 tilføjer et pragmatisk sikkerhedsnet (begrænset evaluate, abort-propagation og best-effort-
gendannelse). Dette dokument beskriver en større refaktorering, der gør `act:evaluate` iboende
isoleret fra Playwright, så en fastlåst evaluate ikke kan blokere normale Playwright-operationer.

## Mål

- `act:evaluate` kan ikke permanent blokere efterfølgende browserhandlinger på den samme fane.
- Timeouts er en single source of truth end-to-end, så en kaldende part kan stole på et budget.
- Abort og timeout behandles ens på tværs af HTTP og in-process-dispatch.
- Elementmålretning for evaluate understøttes uden at flytte alt væk fra Playwright.
- Oprethold bagudkompatibilitet for eksisterende kaldere og payloads.

## Ikke-mål

- Erstat alle browserhandlinger (klik, skriv, vent osv.) med CDP-implementeringer.
- Fjern det eksisterende sikkerhedsnet introduceret i PR #13498 (det forbliver en nyttig fallback).
- Introducer nye usikre kapabiliteter ud over den eksisterende `browser.evaluateEnabled`-gate.
- Tilføj procesisolering (worker-proces/tråd) for evaluate. Hvis vi stadig ser svært genoprettelige
  fastlåste tilstande efter denne refaktorering, er det en opfølgende idé.

## Nuværende arkitektur (hvorfor den går i stå)

På et overordnet niveau:

- Kaldere sender `act:evaluate` til browserkontroltjenesten.
- Route-handleren kalder ind i Playwright for at udføre JavaScript.
- Playwright serialiserer sidekommandoer, så en evaluate, der aldrig afsluttes, blokerer køen.
- En fastlåst kø betyder, at senere klik/skriv/vent-operationer på fanen kan se ud til at hænge.

## Foreslået arkitektur

### 1. Deadline-propagation

Introducer et enkelt budgetkoncept og afled alt fra det:

- Kaldende part angiver `timeoutMs` (eller en deadline i fremtiden).
- Den ydre request-timeout, route-handler-logik og eksekveringsbudgettet inde i siden
  bruger alle det samme budget med lidt ekstra råderum, hvor det er nødvendigt for serialiserings-overhead.
- Abort propagres som et `AbortSignal` overalt, så annullering er konsistent.

Implementeringsretning:

- Tilføj en lille hjælpefunktion (for eksempel `createBudget({ timeoutMs, signal })`), der returnerer:
  - `signal`: det tilknyttede AbortSignal
  - `deadlineAtMs`: absolut deadline
  - `remainingMs()`: resterende budget for underordnede operationer
- Brug denne hjælpefunktion i:
  - `src/browser/client-fetch.ts` (HTTP og in-process dispatch)
  - `src/node-host/runner.ts` (proxy-sti)
  - browser action-implementeringer (Playwright og CDP)

### 2. Separat Evaluate Engine (CDP-sti)

Tilføj en CDP-baseret evaluate-implementering, der ikke deler Playwrights per-side kommando-kø. Den centrale egenskab er, at evaluate-transporten er en separat WebSocket-forbindelse
og en separat CDP-session tilknyttet målet.

Implementeringsretning:

- Nyt modul, for eksempel `src/browser/cdp-evaluate.ts`, der:
  - Opretter forbindelse til det konfigurerede CDP-endpoint (browser-niveau socket).
  - Bruger `Target.attachToTarget({ targetId, flatten: true })` til at få et `sessionId`.
  - Kører enten:
    - `Runtime.evaluate` til evaluate på sideniveau, eller
    - `DOM.resolveNode` plus `Runtime.callFunctionOn` til evaluate på elementniveau.
  - Ved timeout eller abort:
    - Sender `Runtime.terminateExecution` efter bedste evne for sessionen.
    - Lukker WebSocket-forbindelsen og returnerer en tydelig fejl.

Bemærkninger:

- Dette eksekverer stadig JavaScript på siden, så terminering kan have bivirkninger. Fordelen
  er, at den ikke blokerer Playwright-køen, og at den kan annulleres på transportlaget
  ved at afslutte CDP-sessionen.

### 3. Ref-historik (elementmålretning uden en fuld omskrivning)

Den svære del er elementmålretning. CDP kræver et DOM-handle eller `backendDOMNodeId`, mens
i dag de fleste browser actions bruger Playwright-locators baseret på refs fra snapshots.

Anbefalet tilgang: behold eksisterende refs, men tilføj et valgfrit CDP-opløseligt id.

#### 3.1 Udvid gemt ref-info

Udvid den gemte role-ref-metadata til valgfrit at inkludere et CDP-id:

- I dag: `{ role, name, nth }`
- Foreslået: `{ role, name, nth, backendDOMNodeId?: number }`

Dette bevarer alle eksisterende Playwright-baserede actions og gør det muligt for CDP evaluate at acceptere
den samme `ref`-værdi, når `backendDOMNodeId` er tilgængelig.

#### 3.2 Udfyld backendDOMNodeId ved snapshot-tidspunktet

Når der oprettes et role-snapshot:

1. Generér det eksisterende role-ref-map som i dag (role, name, nth).
2. Hent AX-træet via CDP (`Accessibility.getFullAXTree`) og beregn et parallelt map af
   `(role, name, nth) -> backendDOMNodeId` ved brug af de samme regler for håndtering af dubletter.
3. Flet id'et tilbage i den gemte ref-info for den aktuelle fane.

Hvis mapping mislykkes for en ref, skal `backendDOMNodeId` forblive undefined. Dette gør funktionen
til best-effort og sikker at rulle ud.

#### 3.3 Evaluer adfærd med ref

I `act:evaluate`:

- Hvis `ref` er til stede og har `backendDOMNodeId`, kør element-evaluate via CDP.
- Hvis `ref` er til stede, men ikke har `backendDOMNodeId`, fald tilbage til Playwright-stien (med
  sikkerhedsnettet).

Valgfri escape hatch:

- Udvid request-strukturen til at acceptere `backendDOMNodeId` direkte for avancerede brugere (og
  til fejlsøgning), mens `ref` bevares som den primære grænseflade.

### 4. Bevar en sidste udvej til gendannelse

Selv med CDP evaluate findes der andre måder at låse en fane eller en forbindelse på. Bevar de
eksisterende gendannelsesmekanismer (afslut eksekvering + afbryd Playwright) som en sidste udvej
for:

- legacy callers
- miljøer hvor CDP-tilslutning er blokeret
- uventede Playwright edge cases

## Implementeringsplan (én iteration)

### Leverancer

- En CDP-baseret evaluate-motor, der kører uden for Playwrights kommando-kø pr. side.
- Et samlet end-to-end timeout/abort-budget, der bruges konsekvent af callers og handlers.
- Ref-metadata, der valgfrit kan indeholde `backendDOMNodeId` til element evaluate.
- `act:evaluate` foretrækker CDP-motoren, når det er muligt, og falder tilbage til Playwright, når det ikke er.
- Tests, der beviser, at en fastlåst evaluate ikke blokerer efterfølgende handlinger.
- Logs/metrics, der gør fejl og fallback synlige.

### Implementeringstjekliste

1. Tilføj en delt "budget"-helper til at koble `timeoutMs` + upstream `AbortSignal` til:
   - et enkelt `AbortSignal`
   - en absolut deadline
   - en `remainingMs()`-helper til downstream-operationer
2. Opdater alle caller-stier til at bruge denne helper, så `timeoutMs` betyder det samme overalt:
   - `src/browser/client-fetch.ts` (HTTP og in-process dispatch)
   - `src/node-host/runner.ts` (node proxy-sti)
   - CLI-wrappers, der kalder `/act` (tilføj `--timeout-ms` til `browser evaluate`)
3. Implementer `src/browser/cdp-evaluate.ts`:
   - opret forbindelse til browserens CDP-socket på browser-niveau
   - `Target.attachToTarget` for at få et `sessionId`
   - kør `Runtime.evaluate` til page evaluate
   - kør `DOM.resolveNode` + `Runtime.callFunctionOn` til element evaluate
   - ved timeout/abort: best-effort `Runtime.terminateExecution` og luk derefter socket-forbindelsen
4. Udvid gemt role-ref-metadata til valgfrit at inkludere `backendDOMNodeId`:
   - bevar eksisterende `{ role, name, nth }`-adfærd for Playwright-handlinger
   - tilføj `backendDOMNodeId?: number` til CDP-elementmålretning
5. Udfyld `backendDOMNodeId` under oprettelse af snapshot (best effort):
   - hent AX-træ via CDP (`Accessibility.getFullAXTree`)
   - beregn `(role, name, nth) -> backendDOMNodeId` og flet ind i det gemte ref-map
   - hvis mapping er tvetydig eller mangler, lad id være undefined
6. Opdatér `act:evaluate`-routing:
   - hvis ingen `ref`: brug altid CDP evaluate
   - hvis `ref` kan løses til en `backendDOMNodeId`: brug CDP element evaluate
   - ellers: fald tilbage til Playwright evaluate (stadig begrænset og kan afbrydes)
7. Behold den eksisterende "last resort"-gendannelsessti som fallback, ikke som standardsti.
8. Tilføj tests:
   - fastlåst evaluate får timeout inden for budgettet, og næste click/type lykkes
   - abort annullerer evaluate (klientafbrydelse eller timeout) og frigør efterfølgende handlinger
   - mapping-fejl falder rent tilbage til Playwright
9. Tilføj observerbarhed:
   - evaluate-varighed og timeout-tællere
   - terminateExecution-brug
   - fallback-rate (CDP -> Playwright) og årsager

### Acceptkriterier

- En bevidst fastlåst `act:evaluate` returnerer inden for kalders budget og låser ikke
  fanen for efterfølgende handlinger.
- `timeoutMs` opfører sig konsistent på tværs af CLI, agentværktøj, node-proxy og in-process-kald.
- Hvis `ref` kan mappes til `backendDOMNodeId`, bruger element evaluate CDP; ellers er
  fallback-stien stadig begrænset og kan gendannes.

## Testplan

- Enhedstests:
  - `(role, name, nth)`-matchningslogik mellem role-refs og AX-træknuder.
  - Budget-helperens adfærd (headroom, beregning af resterende tid).
- Integrationstests:
  - CDP evaluate-timeout returnerer inden for budgettet og blokerer ikke den næste handling.
  - Abort annullerer evaluate og udløser termination efter bedste evne.
- Kontrakttests:
  - Sikr at `BrowserActRequest` og `BrowserActResponse` forbliver kompatible.

## Risici og afbødninger

- Mapping er ikke perfekt:
  - Afbødning: best effort-mapping, fallback til Playwright evaluate, og tilføj debug-værktøjer.
- `Runtime.terminateExecution` har bivirkninger:
  - Afbødning: brug kun ved timeout/abort og dokumentér adfærden i fejlmeddelelser.
- Ekstra overhead:
  - Afbødning: hent kun AX-træet, når snapshots anmodes, cache pr. target, og hold
    CDP-sessionen kortvarig.
- Begrænsninger i extension-relay:
  - Afbødning: brug browser-niveau attach-API’er, når sockets pr. side ikke er tilgængelige, og
    behold den nuværende Playwright-sti som fallback.

## Åbne spørgsmål

- Skal den nye motor kunne konfigureres som `playwright`, `cdp` eller `auto`?
- Ønsker vi at eksponere et nyt "nodeRef"-format til avancerede brugere, eller kun beholde `ref`?
- Hvordan skal frame-snapshots og selector-afgrænsede snapshots indgå i AX-mapping?

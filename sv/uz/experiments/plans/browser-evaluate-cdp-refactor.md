---
summary: "Plan: isolera browser act:evaluate från Playwright‑kön med CDP, med end‑to‑end‑deadlines och säkrare referensupplösning"
owner: "openclaw"
status: "utkast"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP-refaktorering"
---

# Plan för Browser Evaluate CDP-refaktorering

## Kontext

`act:evaluate` kör användartillhandahållen JavaScript på sidan. Idag körs den via Playwright
(`page.evaluate` eller `locator.evaluate`). Playwright serialiserar CDP-kommandon per sida, så en evaluate som fastnar eller kör länge kan blockera sidans kommandokö och få varje efterföljande åtgärd
på den fliken att se "fast" ut.

PR #13498 lägger till ett pragmatiskt skyddsnät (begränsad evaluate, abort-propagation och återhämtning efter bästa förmåga). Detta dokument beskriver en större refaktorering som gör `act:evaluate` i grunden
isolerad från Playwright så att en fastnad evaluate inte kan blockera normala Playwright-operationer.

## Mål

- `act:evaluate` kan inte permanent blockera senare webbläsaråtgärder på samma flik.
- Timeouts är en enda sanningskälla från början till slut så att en anropare kan lita på en tidsbudget.
- Abort och timeout behandlas på samma sätt över HTTP och i intern dispatch.
- Elementinriktning för evaluate stöds utan att stänga av allt från Playwright.
- Bibehåll bakåtkompatibilitet för befintliga anropare och payloads.

## Icke-mål

- Ersätta alla webbläsaråtgärder (click, type, wait, etc.) med CDP-implementationer.
- Ta bort det befintliga skyddsnätet som introducerades i PR #13498 (det förblir en användbar reservlösning).
- Införa nya osäkra funktioner utöver den befintliga `browser.evaluateEnabled`-grinden.
- Lägga till processisolering (worker-process/tråd) för evaluate. Om vi fortfarande ser svåråterhämtade
  fastnade tillstånd efter denna refaktorering är det en uppföljande idé.

## Nuvarande arkitektur (Varför den fastnar)

På en hög nivå:

- Anropare skickar `act:evaluate` till webbläsarens kontrolltjänst.
- Route-hanteraren anropar Playwright för att köra JavaScript.
- Playwright serialiserar sidkommandon, så en evaluate som aldrig avslutas blockerar kön.
- En blockerad kö innebär att senare click/type/wait-operationer på fliken kan verka hänga sig.

## Föreslagen arkitektur

### 1. Deadline-propagation

Inför ett enda budgetkoncept och härled allt från det:

- Anroparen anger `timeoutMs` (eller en deadline i framtiden).
- Den yttre request-timeouten, route-hanterarens logik och exekveringsbudgeten inne i sidan
  använder alla samma budget, med lite marginal där det behövs för serialiseringsöverhead.
- Abort propagas som en `AbortSignal` överallt så att avbrytning är konsekvent.

Implementationsriktning:

- Lägg till en liten hjälpfunktion (till exempel `createBudget({ timeoutMs, signal })`) som returnerar:
  - `signal`: den länkade AbortSignal
  - `deadlineAtMs`: absolut deadline
  - `remainingMs()`: återstående budget för underordnade operationer
- Använd denna hjälpfunktion i:
  - `src/browser/client-fetch.ts` (HTTP och in-process-dispatch)
  - `src/node-host/runner.ts` (proxy-sökväg)
  - implementationer av browser-åtgärder (Playwright och CDP)

### 2. Separat Evaluate Engine (CDP-sökväg)

Lägg till en CDP-baserad evaluate-implementation som inte delar Playwrights per-sida-kommandokö. Den centrala egenskapen är att evaluate-transporten är en separat WebSocket-anslutning och en separat CDP-session kopplad till målet.

Implementationsriktning:

- Ny modul, till exempel `src/browser/cdp-evaluate.ts`, som:
  - Ansluter till den konfigurerade CDP-endpointen (socket på webbläsarnivå).
  - Använder `Target.attachToTarget({ targetId, flatten: true })` för att få en `sessionId`.
  - Kör antingen:
    - `Runtime.evaluate` för evaluate på sidnivå, eller
    - `DOM.resolveNode` plus `Runtime.callFunctionOn` för evaluate på elementnivå.
  - Vid timeout eller avbrott:
    - Skickar `Runtime.terminateExecution` efter bästa förmåga för sessionen.
    - Stänger WebSocket-anslutningen och returnerar ett tydligt fel.

Noteringar:

- Detta kör fortfarande JavaScript på sidan, så avbrott kan ha bieffekter. Fördelen
  är att den inte blockerar Playwright-kön och kan avbrytas på transport
  lagret genom att döda CDP-sessionen.

### 3. Ref-historia (elementinriktning utan en fullständig omskrivning)

Den svåra delen är elementinriktning. CDP behöver ett DOM-handle eller `backendDOMNodeId`, medan de flesta browser-åtgärder idag använder Playwright-locators baserade på refs från snapshots.

Rekommenderad metod: behåll befintliga refs, men lägg till ett valfritt CDP-resolverbart id.

#### 3.1 Utöka lagrad ref-information

Utöka den lagrade role-ref-metadata för att valfritt inkludera ett CDP-id:

- Idag: `{ role, name, nth }`
- Föreslaget: `{ role, name, nth, backendDOMNodeId?: number }`

Detta gör att alla befintliga Playwright-baserade åtgärder fortsätter fungera och gör det möjligt för CDP evaluate att acceptera samma `ref`-värde när `backendDOMNodeId` är tillgängligt.

#### 3.2 Fyll i backendDOMNodeId vid snapshot-tillfället

När en role-snapshot skapas:

1. Generera den befintliga role-ref-kartan som idag (role, name, nth).
2. Hämta AX-trädet via CDP (`Accessibility.getFullAXTree`) och beräkna en parallell karta av `(role, name, nth) -> backendDOMNodeId` med samma regler för hantering av dubbletter.
3. Slå samman id:t tillbaka till den lagrade ref-informationen för den aktuella fliken.

Om mappningen misslyckas för en ref, lämna `backendDOMNodeId` odefinierad. Detta gör funktionen best-effort och säker att rulla ut.

#### 3.3 Utvärdera beteende med Ref

I `act:evaluate`:

- Om `ref` finns och har `backendDOMNodeId`, kör element-evaluering via CDP.
- Om `ref` finns men saknar `backendDOMNodeId`, fall tillbaka till Playwright-sökvägen (med
  säkerhetsnätet).

Valfri nödutgång:

- Utöka förfrågningsstrukturen så att den accepterar `backendDOMNodeId` direkt för avancerade anropare (och
  för felsökning), samtidigt som `ref` behålls som primärt gränssnitt.

### 4. Behåll en sista återhämtningsväg

Även med CDP evaluate finns det andra sätt att låsa en flik eller en anslutning. Behåll de
befintliga återställningsmekanismerna (avsluta körning + koppla från Playwright) som en sista utväg
för:

- äldre anropare
- miljöer där CDP-anslutning blockeras
- oväntade Playwright-kantfall

## Implementeringsplan (en iteration)

### Leverabler

- En CDP-baserad evaluate-motor som körs utanför Playwrights kommandokö per sida.
- En enda end-to-end-timeout/abort-budget som används konsekvent av anropare och hanterare.
- Ref-metadata som valfritt kan innehålla `backendDOMNodeId` för element-evaluering.
- `act:evaluate` föredrar CDP-motorn när det är möjligt och faller tillbaka till Playwright när det inte är det.
- Tester som bevisar att en fastlåst evaluate inte blockerar efterföljande åtgärder.
- Loggar/mått som gör fel och fallback synliga.

### Implementationschecklista

1. Lägg till en delad "budget"-hjälpare för att koppla `timeoutMs` + uppströms `AbortSignal` till:
   - en enda `AbortSignal`
   - en absolut deadline
   - en `remainingMs()`-hjälpare för nedströmsoperationer
2. Uppdatera alla anroparvägar för att använda den hjälparen så att `timeoutMs` betyder samma sak överallt:
   - `src/browser/client-fetch.ts` (HTTP och in-process-dispatch)
   - `src/node-host/runner.ts` (node-proxy-sökväg)
   - CLI-omslag som anropar `/act` (lägg till `--timeout-ms` till `browser evaluate`)
3. Implementera `src/browser/cdp-evaluate.ts`:
   - anslut till CDP-socketen på webbläsarnivå
   - `Target.attachToTarget` för att få en `sessionId`
   - kör `Runtime.evaluate` för side-evaluering
   - kör `DOM.resolveNode` + `Runtime.callFunctionOn` för element-evaluering
   - vid timeout/abort: försök i bästa fall med `Runtime.terminateExecution` och stäng sedan socketen
4. Utöka lagrad roll-ref-metadata så att den valfritt kan inkludera `backendDOMNodeId`:
   - behåll befintligt `{ role, name, nth }`-beteende för Playwright-åtgärder
   - lägg till `backendDOMNodeId?: number` för CDP-elementmålning
5. Fyll i `backendDOMNodeId` vid skapande av snapshot (i bästa fall):
   - hämta AX-träd via CDP (`Accessibility.getFullAXTree`)
   - beräkna `(role, name, nth) -> backendDOMNodeId` och slå samman i den lagrade referenskartan
   - om mappningen är tvetydig eller saknas, lämna id odefinierat
6. Uppdatera routning för `act:evaluate`:
   - om ingen `ref`: använd alltid CDP evaluate
   - om `ref` kan lösas till en `backendDOMNodeId`: använd CDP element evaluate
   - annars: fall tillbaka till Playwright evaluate (fortfarande begränsad och avbrytbar)
7. Behåll den befintliga "last resort"-återställningsvägen som en reserv, inte som standardväg.
8. Lägg till tester:
   - en fastlåst evaluate får timeout inom budget och nästa klick/skrivning lyckas
   - abort avbryter evaluate (klient frånkopplas eller timeout) och avblockerar efterföljande åtgärder
   - mappningsfel faller rent tillbaka till Playwright
9. Lägg till observability:
   - evaluate-varaktighet och timeout-räknare
   - användning av terminateExecution
   - fallback-frekvens (CDP -> Playwright) och orsaker

### Acceptanskriterier

- En avsiktligt låst `act:evaluate` returnerar inom anroparens budget och låser inte fliken för senare åtgärder.
- `timeoutMs` beter sig konsekvent över CLI, agentverktyg, node-proxy och in-process-anrop.
- Om `ref` kan mappas till `backendDOMNodeId` använder element evaluate CDP; annars är fallback-vägen fortfarande begränsad och återställbar.

## Testplan

- Enhetstester:
  - `(role, name, nth)`-matchningslogik mellan rollreferenser och AX-trädnoder.
  - Beteende för budgethjälpare (headroom, beräkning av återstående tid).
- Integrationstester:
  - CDP evaluate-timeout returnerar inom budget och blockerar inte nästa åtgärd.
  - Abort avbryter evaluate och triggar termination enligt best effort.
- Kontraktstester:
  - Säkerställ att `BrowserActRequest` och `BrowserActResponse` förblir kompatibla.

## Risker och åtgärder

- Mappningen är inte perfekt:
  - Åtgärd: best effort-mappning, fallback till Playwright evaluate och lägg till felsökningsverktyg.
- `Runtime.terminateExecution` har bieffekter:
  - Åtgärd: använd endast vid timeout/abort och dokumentera beteendet i felmeddelanden.
- Extra overhead:
  - Åtgärd: hämta endast AX-trädet när snapshots begärs, cacha per target och håll CDP-sessionen kortlivad.
- Begränsningar i extension relay:
  - Åtgärd: använd browser-nivåns attach-API:er när per-sida-sockets inte är tillgängliga och behåll nuvarande Playwright-väg som fallback.

## Öppna frågor

- Ska den nya motorn vara konfigurerbar som `playwright`, `cdp` eller `auto`?
- Vill vi exponera ett nytt "nodeRef"-format för avancerade användare, eller bara behålla `ref`?
- Hur ska frame-ögonblicksbilder och selektor-avgränsade ögonblicksbilder delta i AX-mappning?

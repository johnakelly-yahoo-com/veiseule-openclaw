---
summary: "Plan: isoleer browser act:evaluate van de Playwright-wachtrij met behulp van CDP, met end-to-end deadlines en veiligere ref-resolutie"
owner: "openclaw"
status: "concept"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor Plan

## Context

`act:evaluate` voert door de gebruiker aangeleverde JavaScript uit op de pagina. Momenteel draait dit via Playwright (`page.evaluate` of `locator.evaluate`). Playwright serialiseert CDP-commando’s per pagina, waardoor een vastgelopen of langlopende evaluate de opdrachtwachtrij van de pagina kan blokkeren en elke latere actie op dat tabblad "vastgelopen" kan laten lijken.

PR #13498 voegt een pragmatisch vangnet toe (begrensde evaluate, abort-propagatie en best-effort herstel). Dit document beschrijft een grotere refactor die `act:evaluate` inherent isoleert van Playwright, zodat een vastgelopen evaluate normale Playwright-operaties niet kan blokkeren.

## Doelen

- `act:evaluate` kan latere browseracties op hetzelfde tabblad niet permanent blokkeren.
- Timeouts vormen één enkele bron van waarheid end-to-end, zodat een aanroeper op een budget kan vertrouwen.
- Abort en timeout worden op dezelfde manier behandeld via HTTP en in-process dispatch.
- Elementtargeting voor evaluate wordt ondersteund zonder alles van Playwright los te koppelen.
- Behoud van achterwaartse compatibiliteit voor bestaande aanroepers en payloads.

## Niet-doelen

- Vervang alle browseracties (click, type, wait, enz.) door CDP-implementaties.
- Verwijder het bestaande vangnet dat in PR #13498 is geïntroduceerd niet (het blijft een nuttige fallback).
- Introduceer geen nieuwe onveilige mogelijkheden buiten de bestaande `browser.evaluateEnabled`-beperking.
- Voeg geen procesisolatie (workerproces/thread) toe voor evaluate. Als we na deze refactor nog steeds moeilijk te herstellen vastgelopen toestanden zien, is dat een idee voor een vervolg.

## Huidige architectuur (Waarom het vastloopt)

Op hoofdlijnen:

- Aanroepers sturen `act:evaluate` naar de browsercontrolservice.
- De routehandler roept Playwright aan om de JavaScript uit te voeren.
- Playwright serialiseert page-commando’s, dus een evaluate die nooit wordt afgerond blokkeert de wachtrij.
- Een vastgelopen wachtrij betekent dat latere click/type/wait-bewerkingen op het tabblad kunnen lijken te hangen.

## Voorgestelde architectuur

### 1. Deadline-propagatie

Introduceer één budgetconcept en leid daar alles van af:

- De aanroeper stelt `timeoutMs` in (of een deadline in de toekomst).
- De externe request-timeout, de route-handlerlogica en het uitvoeringsbudget binnen de pagina
  gebruiken allemaal hetzelfde budget, met een kleine marge waar nodig voor serialisatie-overhead.
- Abort wordt overal doorgegeven als een `AbortSignal`, zodat annulering consistent is.

Implementatierichting:

- Voeg een kleine helper toe (bijvoorbeeld `createBudget({ timeoutMs, signal })`) die het volgende retourneert:
  - `signal`: het gekoppelde AbortSignal
  - `deadlineAtMs`: absolute deadline
  - `remainingMs()`: resterend budget voor onderliggende bewerkingen
- Gebruik deze helper in:
  - `src/browser/client-fetch.ts` (HTTP en in-process dispatch)
  - `src/node-host/runner.ts` (proxy-pad)
  - browser-actie-implementaties (Playwright en CDP)

### 2. Gescheiden Evaluate-engine (CDP-pad)

Voeg een op CDP gebaseerde evaluate-implementatie toe die de per-pagina-commandowachtrij van Playwright niet deelt. De belangrijkste eigenschap is dat het evaluate-transport een aparte WebSocket-verbinding is
en een aparte CDP-sessie die aan het target is gekoppeld.

Implementatierichting:

- Nieuwe module, bijvoorbeeld `src/browser/cdp-evaluate.ts`, die:
  - Verbindt met het geconfigureerde CDP-endpoint (browser-level socket).
  - `Target.attachToTarget({ targetId, flatten: true })` gebruikt om een `sessionId` te verkrijgen.
  - Voert ofwel uit:
    - `Runtime.evaluate` voor evaluate op paginaniveau, of
    - `DOM.resolveNode` plus `Runtime.callFunctionOn` voor evaluate op elementniveau.
  - Bij timeout of abort:
    - Stuurt best-effort `Runtime.terminateExecution` voor de sessie.
    - Sluit de WebSocket en retourneert een duidelijke fout.

Opmerkingen:

- Dit voert nog steeds JavaScript uit in de pagina, dus beëindiging kan bijwerkingen hebben. De winst
  is dat de Playwright-wachtrij niet vastloopt en dat het annuleerbaar is op transportniveau
  door de CDP-sessie te beëindigen.

### 3. Ref-verhaal (Element-targeting zonder volledige herschrijving)

Het moeilijke deel is element-targeting. CDP heeft een DOM-handle of `backendDOMNodeId` nodig, terwijl
de meeste browseracties vandaag Playwright-locators gebruiken op basis van refs uit snapshots.

Aanbevolen aanpak: behoud de bestaande refs, maar voeg een optionele door CDP oplosbare id toe.

#### 3.1 Uitbreiden van opgeslagen ref-info

Breid de opgeslagen role-refmetadata uit om optioneel een CDP-id op te nemen:

- Vandaag: `{ role, name, nth }`
- Voorgesteld: `{ role, name, nth, backendDOMNodeId?: number }`

Dit zorgt ervoor dat alle bestaande op Playwright gebaseerde acties blijven werken en maakt het mogelijk dat CDP evaluate dezelfde `ref`-waarde accepteert wanneer de `backendDOMNodeId` beschikbaar is.

#### 3.2 backendDOMNodeId vullen tijdens snapshot

Bij het maken van een role-snapshot:

1. Genereer de bestaande role-refmap zoals vandaag (role, name, nth).
2. Haal de AX-tree op via CDP (`Accessibility.getFullAXTree`) en bereken een parallelle map van `(role, name, nth) -> backendDOMNodeId` met dezelfde regels voor het verwerken van duplicaten.
3. Voeg het id weer samen met de opgeslagen ref-info voor het huidige tabblad.

Als het koppelen voor een ref mislukt, laat `backendDOMNodeId` dan undefined. Dit maakt de functie best-effort en veilig om uit te rollen.

#### 3.3 Evaluate-gedrag met ref

In `act:evaluate`:

- Als `ref` aanwezig is en een `backendDOMNodeId` heeft, voer dan element-evaluate uit via CDP.
- Als `ref` aanwezig is maar geen `backendDOMNodeId` heeft, val dan terug op het Playwright-pad (met het vangnet).

Optionele escape hatch:

- Breid de request-structuur uit zodat direct `backendDOMNodeId` kan worden meegegeven voor geavanceerde aanroepers (en voor debugging), terwijl `ref` de primaire interface blijft.

### 4. Behoud een laatste redmiddel-herstelpad

Zelfs met CDP evaluate zijn er andere manieren waarop een tabblad of verbinding kan vastlopen. Behoud de bestaande herstelmechanismen (beëindig uitvoering + verbreek Playwright-verbinding) als laatste redmiddel voor:

- legacy callers
- omgevingen waar CDP-attach wordt geblokkeerd
- onverwachte Playwright-edgecases

## Implementatieplan (één iteratie)

### Op te leveren resultaten

- Een op CDP gebaseerde evaluate-engine die buiten de Playwright per-page command queue draait.
- Eén end-to-end timeout/abort-budget dat consistent wordt gebruikt door callers en handlers.
- Ref-metadata die optioneel `backendDOMNodeId` kan bevatten voor element-evaluate.
- `act:evaluate` geeft waar mogelijk de voorkeur aan de CDP-engine en valt anders terug op Playwright.
- Tests die aantonen dat een vastgelopen evaluate latere acties niet blokkeert.
- Logs/metrics die fouten en fallbacks zichtbaar maken.

### Implementatiechecklist

1. Voeg een gedeelde "budget"-helper toe om `timeoutMs` + upstream `AbortSignal` te koppelen tot:
   - één enkele `AbortSignal`
   - een absolute deadline
   - een `remainingMs()`-helper voor downstream-operaties
2. Werk alle caller-paden bij om die helper te gebruiken zodat `timeoutMs` overal hetzelfde betekent:
   - `src/browser/client-fetch.ts` (HTTP en in-process dispatch)
   - `src/node-host/runner.ts` (node-proxypad)
   - CLI-wrappers die `/act` aanroepen (voeg `--timeout-ms` toe aan `browser evaluate`)
3. Implementeer `src/browser/cdp-evaluate.ts`:
   - verbind met de browser-level CDP-socket
   - `Target.attachToTarget` om een `sessionId` te verkrijgen
   - voer `Runtime.evaluate` uit voor evaluate op pagina-niveau
   - voer `DOM.resolveNode` + `Runtime.callFunctionOn` uit voor element-evaluate
   - bij timeout/afbreken: naar beste vermogen `Runtime.terminateExecution` uitvoeren en vervolgens de socket sluiten
4. Breid opgeslagen rol-ref-metadata uit om optioneel `backendDOMNodeId` op te nemen:
   - behoud het bestaande `{ role, name, nth }`-gedrag voor Playwright-acties
   - voeg `backendDOMNodeId?: number` toe voor CDP-elementtargeting
5. Vul `backendDOMNodeId` tijdens het aanmaken van de snapshot (naar beste vermogen):
   - haal de AX-tree op via CDP (`Accessibility.getFullAXTree`)
   - bereken `(role, name, nth) -> backendDOMNodeId` en voeg dit samen in de opgeslagen ref-map
   - als de mapping ambigu of ontbrekend is, laat het id undefined
6. Werk de `act:evaluate`-routering bij:
   - als er geen `ref` is: gebruik altijd CDP evaluate
   - als `ref` wordt opgelost naar een `backendDOMNodeId`: gebruik CDP element evaluate
   - anders: val terug op Playwright evaluate (nog steeds begrensd en annuleerbaar)
7. Behoud het bestaande "laatste redmiddel"-herstelpad als fallback, niet als standaardpad.
8. Voeg tests toe:
   - vastgelopen evaluate time-out binnen het budget en de volgende klik/type slaagt
   - abort annuleert evaluate (clientverbinding verbroken of timeout) en deblokkeert daaropvolgende acties
   - mappingfouten vallen netjes terug op Playwright
9. Voeg observability toe:
   - evaluate-duur en timeout-tellers
   - terminateExecution-gebruik
   - fallback-percentage (CDP -> Playwright) en redenen

### Acceptatiecriteria

- Een opzettelijk vastgelopen `act:evaluate` keert terug binnen het aanroepersbudget en blokkeert de
  tab niet voor latere acties.
- `timeoutMs` gedraagt zich consistent over CLI, agenttool, node-proxy en in-process aanroepen.
- Als `ref` kan worden gemapt naar `backendDOMNodeId`, gebruikt element evaluate CDP; anders blijft het
  fallback-pad begrensd en herstelbaar.

## Testplan

- Unit tests:
  - matchinglogica van `(role, name, nth)` tussen rol-refs en AX-tree-nodes.
  - Gedrag van de budget-helper (headroom, resterende-tijdberekening).
- Integratietests:
  - CDP evaluate-timeout keert terug binnen het budget en blokkeert de volgende actie niet.
  - Abort annuleert evaluate en triggert termination naar beste vermogen.
- Contracttests:
  - Zorg ervoor dat `BrowserActRequest` en `BrowserActResponse` compatibel blijven.

## Risico's en mitigaties

- Mapping is niet perfect:
  - Mitigatie: best-effort mapping, fallback naar Playwright evaluate en debugtooling toevoegen.
- `Runtime.terminateExecution` heeft bijwerkingen:
  - Mitigatie: alleen gebruiken bij timeout/afbreken en het gedrag in foutmeldingen documenteren.
- Extra overhead:
  - Mitigatie: haal de AX-tree alleen op wanneer snapshots worden aangevraagd, cache per target en houd
    de CDP-sessie kortstondig.
- Beperkingen van de extensie-relay:
  - Mitigatie: gebruik browser-level attach-API's wanneer per-page sockets niet beschikbaar zijn en
    behoud het huidige Playwright-pad als fallback.

## Open vragen

- Moet de nieuwe engine configureerbaar zijn als `playwright`, `cdp` of `auto`?
- Willen we een nieuw "nodeRef"-formaat beschikbaar stellen voor geavanceerde gebruikers, of alleen `ref` behouden?
- Hoe moeten frame-snapshots en selector-scoped snapshots deelnemen aan AX-mapping?

---
summary: "Plan: ihiwalay ang browser act:evaluate mula sa Playwright queue gamit ang CDP, na may end-to-end deadlines at mas ligtas na ref resolution"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor Plan

## Context

Ang `act:evaluate` ay nagpapatakbo ng JavaScript na ibinigay ng user sa loob ng page. Sa kasalukuyan, ito ay tumatakbo sa pamamagitan ng Playwright (`page.evaluate` o `locator.evaluate`). Sini-serialize ng Playwright ang mga CDP command kada page, kaya ang isang evaluate na natigil o tumatakbo nang matagal ay maaaring mag-block sa command queue ng page at magmukhang "stuck" ang lahat ng susunod na aksyon sa tab na iyon.

Ang PR #13498 ay nagdaragdag ng praktikal na safety net (bounded evaluate, abort propagation, at best-effort recovery). Inilalarawan ng dokumentong ito ang mas malawak na refactor na ginagawang likas na nakahiwalay ang `act:evaluate` mula sa Playwright upang ang isang stuck na evaluate ay hindi makapagpatigil sa normal na operasyon ng Playwright.

## Mga Layunin

- Ang `act:evaluate` ay hindi maaaring permanenteng mag-block ng mga susunod na aksyon sa browser sa parehong tab.
- Ang mga timeout ay iisang source of truth mula simula hanggang wakas upang makaaasa ang tumatawag sa itinakdang budget.
- Ang abort at timeout ay pare-parehong tinatrato sa HTTP at sa in-process dispatch.
- Sinusuportahan ang pag-target ng element para sa evaluate nang hindi inaalis ang paggamit ng Playwright sa lahat.
- Panatilihin ang backward compatibility para sa mga umiiral na caller at payload.

## Mga Hindi Layunin

- Palitan ang lahat ng browser action (click, type, wait, atbp.) ng mga implementasyong CDP.
- Alisin ang umiiral na safety net na ipinakilala sa PR #13498 (mananatili itong kapaki-pakinabang na fallback).
- Magpakilala ng mga bagong hindi ligtas na kakayahan lampas sa umiiral na `browser.evaluateEnabled` gate.
- Magdagdag ng process isolation (worker process/thread) para sa evaluate. Kung makaranas pa rin tayo ng mga stuck state na mahirap ma-recover matapos ang refactor na ito, ito ay susunod na ideya.

## Kasalukuyang Arkitektura (Bakit Ito Nati-stuck)

Sa mataas na antas:

- Ang mga caller ay nagpapadala ng `act:evaluate` sa browser control service.
- Tinatawag ng route handler ang Playwright upang isagawa ang JavaScript.
- Sini-serialize ng Playwright ang mga page command, kaya ang evaluate na hindi natatapos ay nagba-block sa queue.
- Kapag stuck ang queue, ang mga susunod na click/type/wait na operasyon sa tab ay maaaring magmukhang naka-hang.

## Iminungkahing Arkitektura

### 1. Pagpapalaganap ng Deadline

Magpakilala ng iisang konsepto ng budget at kunin ang lahat mula rito:

- Itinatakda ng caller ang `timeoutMs` (o isang deadline sa hinaharap).
- Ang outer request timeout, route handler logic, at ang execution budget sa loob ng page
  ay gumagamit ng iisang budget, na may kaunting headroom kung kinakailangan para sa serialization overhead.
- Ang Abort ay ipinapasa bilang isang `AbortSignal` saanman upang maging pare-pareho ang cancellation.

Direksyon ng implementasyon:

- Magdagdag ng maliit na helper (halimbawa `createBudget({ timeoutMs, signal })`) na nagbabalik ng:
  - `signal`: ang naka-link na AbortSignal
  - `deadlineAtMs`: absolute na deadline
  - `remainingMs()`: natitirang budget para sa mga child operation
- Gamitin ang helper na ito sa:
  - `src/browser/client-fetch.ts` (HTTP at in-process dispatch)
  - `src/node-host/runner.ts` (proxy path)
  - mga implementasyon ng browser action (Playwright at CDP)

### 2. Hiwalay na Evaluate Engine (CDP Path)

Magdagdag ng CDP-based na evaluate implementation na hindi nakikibahagi sa per-page command queue ng Playwright. Ang pangunahing katangian ay ang evaluate transport ay isang hiwalay na WebSocket connection
at isang hiwalay na CDP session na naka-attach sa target.

Direksyon ng implementasyon:

- Bagong module, halimbawa `src/browser/cdp-evaluate.ts`, na:
  - Kumokonekta sa naka-configure na CDP endpoint (browser-level socket).
  - Gumagamit ng `Target.attachToTarget({ targetId, flatten: true })` upang makakuha ng `sessionId`.
  - Nagpapatakbo ng alinman sa:
    - `Runtime.evaluate` para sa page-level evaluate, o
    - `DOM.resolveNode` plus `Runtime.callFunctionOn` para sa element evaluate.
  - Kapag nag-timeout o na-abort:
    - Nagpapadala ng `Runtime.terminateExecution` bilang best-effort para sa session.
    - Isinasara ang WebSocket at nagbabalik ng malinaw na error.

Mga Tala:

- Ito ay nagpapatakbo pa rin ng JavaScript sa page, kaya ang termination ay maaaring magkaroon ng side effects. Ang panalo
  ay hindi nito naiipit ang Playwright queue, at maaari itong i-cancel sa transport
  layer sa pamamagitan ng pagpatay sa CDP session.

### 3. Ref Story (Element Targeting Nang Hindi Ganap na Nire-rewrite)

Ang mahirap na bahagi ay ang element targeting. Kailangan ng CDP ng DOM handle o `backendDOMNodeId`, habang
ngayon karamihan sa mga browser action ay gumagamit ng Playwright locators batay sa refs mula sa snapshots.

Inirerekomendang approach: panatilihin ang mga kasalukuyang ref, ngunit mag-attach ng opsyonal na CDP resolvable id.

#### 3.1 Palawigin ang Stored Ref Info

Palawigin ang naka-store na role ref metadata upang opsyonal na magsama ng CDP id:

- Ngayon: `{ role, name, nth }`
- Iminungkahi: `{ role, name, nth, backendDOMNodeId?: number }`

Pinapanatili nito ang lahat ng umiiral na Playwright-based na action na gumagana at pinapayagan ang CDP evaluate na tanggapin
ang parehong `ref` value kapag available ang `backendDOMNodeId`.

#### 3.2 I-populate ang backendDOMNodeId sa Oras ng Snapshot

Kapag gumagawa ng role snapshot:

1. Bumuo ng kasalukuyang role ref map tulad ng ginagawa ngayon (role, name, nth).
2. Kunin ang AX tree sa pamamagitan ng CDP (`Accessibility.getFullAXTree`) at kalkulahin ang parallel na map ng
   `(role, name, nth) -> backendDOMNodeId` gamit ang parehong mga patakaran sa paghawak ng duplicate.
3. I-merge ang id pabalik sa nakaimbak na ref info para sa kasalukuyang tab.

Kung mabigo ang pagma-map para sa isang ref, iwanang undefined ang `backendDOMNodeId`. Ginagawa nitong
best-effort at ligtas i-roll out ang feature.

#### 3.3 Suriin ang Behavior Gamit ang Ref

Sa `act:evaluate`:

- Kung naroroon ang `ref` at may `backendDOMNodeId`, patakbuhin ang element evaluate sa pamamagitan ng CDP.
- Kung naroroon ang `ref` ngunit walang `backendDOMNodeId`, bumalik sa Playwright path (na may
  safety net).

Opsyonal na escape hatch:

- Palawakin ang request shape upang direktang tumanggap ng `backendDOMNodeId` para sa mga advanced na caller (at
  para sa debugging), habang pinananatili ang `ref` bilang pangunahing interface.

### 4. Panatilihin ang Huling Paraan ng Recovery

Kahit may CDP evaluate, may iba pang paraan upang ma-stuck ang isang tab o koneksyon. Panatilihin ang
mga umiiral na recovery mechanism (terminate execution + i-disconnect ang Playwright) bilang huling paraan
para sa:

- mga legacy na caller
- mga environment kung saan naka-block ang CDP attach
- mga hindi inaasahang edge case ng Playwright

## Plano ng Implementasyon (Isang Iteration)

### Mga Deliverable

- Isang CDP-based evaluate engine na tumatakbo sa labas ng Playwright per-page command queue.
- Isang iisang end-to-end timeout/abort budget na pare-parehong ginagamit ng mga caller at handler.
- Ref metadata na maaaring opsyonal na maglaman ng `backendDOMNodeId` para sa element evaluate.
- Mas pinipili ng `act:evaluate` ang CDP engine kung maaari at bumabalik sa Playwright kung hindi.
- Mga test na nagpapatunay na ang isang na-stuck na evaluate ay hindi makakaapekto sa mga susunod na aksyon.
- Mga log/metric na nagpapakita ng mga failure at fallback.

### Checklist ng Implementasyon

1. Magdagdag ng shared "budget" helper upang iugnay ang `timeoutMs` + upstream `AbortSignal` sa:
   - isang iisang `AbortSignal`
   - isang absolute deadline
   - isang `remainingMs()` helper para sa mga downstream na operasyon
2. I-update ang lahat ng caller path upang gamitin ang helper na iyon para ang `timeoutMs` ay magkaroon ng parehong kahulugan saanman ito gamitin:
   - `src/browser/client-fetch.ts` (HTTP at in-process dispatch)
   - `src/node-host/runner.ts` (node proxy path)
   - Mga CLI wrapper na tumatawag sa `/act` (idagdag ang `--timeout-ms` sa `browser evaluate`)
3. I-implement ang `src/browser/cdp-evaluate.ts`:
   - kumonekta sa browser-level CDP socket
   - `Target.attachToTarget` upang makakuha ng `sessionId`
   - patakbuhin ang `Runtime.evaluate` para sa page evaluate
   - patakbuhin ang `DOM.resolveNode` + `Runtime.callFunctionOn` para sa element evaluate
   - kapag nag-timeout/nag-abort: best-effort na `Runtime.terminateExecution` pagkatapos ay isara ang socket
4. Palawigin ang nakaimbak na role ref metadata upang opsyonal na isama ang `backendDOMNodeId`:
   - panatilihin ang kasalukuyang `{ role, name, nth }` na behavior para sa mga Playwright action
   - magdagdag ng `backendDOMNodeId?: number` para sa CDP element targeting
5. Punan ang `backendDOMNodeId` habang ginagawa ang snapshot (best-effort):
   - kunin ang AX tree sa pamamagitan ng CDP (`Accessibility.getFullAXTree`)
   - kalkulahin ang `(role, name, nth) -> backendDOMNodeId` at pagsamahin ito sa nakaimbak na ref map
   - kung hindi malinaw o nawawala ang mapping, iwanang undefined ang id
6. I-update ang `act:evaluate` routing:
   - kung walang `ref`: palaging gamitin ang CDP evaluate
   - kung ang `ref` ay tumutugma sa isang `backendDOMNodeId`: gamitin ang CDP element evaluate
   - kung hindi: bumalik sa Playwright evaluate (nanatiling may hangganan at maaaring i-abort)
7. Panatilihin ang kasalukuyang "last resort" na recovery path bilang fallback, hindi bilang default na path.
8. Magdagdag ng mga test:
   - ang na-stuck na evaluate ay magti-timeout sa loob ng itinakdang budget at ang susunod na click/type ay magtatagumpay
   - ang pag-abort ay kinakansela ang evaluate (client disconnect o timeout) at ina-unblock ang mga kasunod na action
   - ang mga pagkabigo sa mapping ay maayos na bumabalik sa Playwright
9. Magdagdag ng observability:
   - tagal ng evaluate at mga counter ng timeout
   - paggamit ng terminateExecution
   - antas ng fallback (CDP -> Playwright) at mga dahilan

### Acceptance Criteria

- Ang sinadyang na-hang na `act:evaluate` ay bumabalik sa loob ng itinakdang budget ng caller at hindi hinahadlangan ang
  tab para sa mga susunod na action.
- Ang `timeoutMs` ay kumikilos nang pare-pareho sa CLI, agent tool, node proxy, at in-process na mga tawag.
- Kung ang `ref` ay maaaring i-map sa `backendDOMNodeId`, ang element evaluate ay gumagamit ng CDP; kung hindi,
  ang fallback path ay nananatiling may hangganan at maaaring ma-recover.

## Testing Plan

- Mga Unit test:
  - lohika ng pagtutugma ng `(role, name, nth)` sa pagitan ng mga role ref at AX tree nodes.
  - behavior ng budget helper (headroom, kalkulasyon ng natitirang oras).
- Mga Integration test:
  - ang CDP evaluate timeout ay bumabalik sa loob ng itinakdang budget at hindi hinaharangan ang susunod na action.
  - ang pag-abort ay kinakansela ang evaluate at nagti-trigger ng termination sa best-effort na paraan.
- Mga Contract test:
  - Tiyaking nananatiling compatible ang `BrowserActRequest` at `BrowserActResponse`.

## Mga Panganib At Mitigasyon

- Hindi perpekto ang mapping:
  - Mitigasyon: best-effort na mapping, fallback sa Playwright evaluate, at magdagdag ng debug tooling.
- May mga side effect ang `Runtime.terminateExecution`:
  - Mitigasyon: gamitin lamang kapag may timeout/abort at idokumento ang behavior sa mga error.
- Karagdagang overhead:
  - Pagpapagaan: kunin lamang ang AX tree kapag hinihingi ang mga snapshot, i-cache bawat target, at panatilihing panandalian ang CDP session.
- Mga limitasyon ng extension relay:
  - Pagpapagaan: gamitin ang browser level attach APIs kapag hindi available ang per page sockets, at panatilihin ang kasalukuyang Playwright path bilang fallback.

## Mga Bukas na Tanong

- Dapat bang ma-configure ang bagong engine bilang `playwright`, `cdp`, o `auto`?
- Gusto ba nating maglabas ng bagong "nodeRef" format para sa mga advanced na user, o panatilihin lamang ang `ref`?
- Paano dapat makilahok ang frame snapshots at selector scoped snapshots sa AX mapping?

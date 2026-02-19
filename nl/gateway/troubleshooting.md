---
summary: "Diepgaande troubleshooting-runbook voor gateway, kanalen, automatisering, nodes en browser"
read_when:
  - De troubleshooting-hub heeft je hierheen verwezen voor diepere diagnose
  - Je hebt stabiele, symptoomgebaseerde runbook-secties nodig met exacte opdrachten
title: "Problemen oplossen"
---

# Gateway-problemen oplossen

Deze pagina is het diepe runbook.
Begin bij [/help/troubleshooting](/help/troubleshooting) als je eerst de snelle triageflow wilt.

## Commandoladder

Voer deze eerst uit, in deze volgorde:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Verwachte gezonde signalen:

- `openclaw gateway status` toont `Runtime: running` en `RPC probe: ok`.
- `openclaw doctor` meldt geen blokkerende config-/serviceproblemen.
- `openclaw channels status --probe` toont verbonden/gereedstaande kanalen.

## Geen antwoorden

Als kanalen actief zijn maar niets antwoordt, controleer routing en beleid voordat je iets opnieuw verbindt.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list <channel>
openclaw config get channels
openclaw logs --follow
```

Let op:

- Koppeling in afwachting voor DM-afzenders.
- Groepsvermelding-gating (`requireMention`, `mentionPatterns`).
- Mismatches in kanaal-/groeps-toegestane lijst.

Veelvoorkomende signalen:

- `drop guild message (mention required` → groepsbericht genegeerd tot vermelding.
- `pairing request` → afzender heeft goedkeuring nodig.
- `blocked` / `allowlist` → afzender/kanaal gefilterd door beleid.

Gerelateerd:

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/pairing](/channels/pairing)
- [/channels/groups](/channels/groups)

## Dashboard/control UI-connectiviteit

Wanneer de dashboard/control UI niet wil verbinden, valideer URL, auth-modus en aannames over een veilige context.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Let op:

- Juiste probe-URL en dashboard-URL.
- Mismatch in auth-modus/token tussen client en gateway.
- Gebruik van HTTP waar apparaatidentiteit vereist is.

Veelvoorkomende signalen:

- `device identity required` → niet-veilige context of ontbrekende apparaatauthenticatie.
- `unauthorized` / reconnect-lus → token-/wachtwoordmismatch.
- `gateway connect failed:` → verkeerde host/poort/URL-doel.

Gerelateerd:

- [/web/control-ui](/web/control-ui)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/remote](/gateway/remote)

## Gateway-service draait niet

Gebruik dit wanneer de service is geïnstalleerd maar het proces niet actief blijft.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

Let op:

- `Runtime: stopped` met exit-hints.
- Serviceconfig-mismatch (`Config (cli)` vs `Config (service)`).
- Poort-/listenerconflicten.

Veelvoorkomende signalen:

- `Gateway start blocked: set gateway.mode=local` → lokale gateway-modus is niet ingeschakeld. Oplossing: stel `gateway.mode="local"` in je configuratie in (of voer `openclaw configure` uit). Als je OpenClaw via Podman uitvoert met de speciale `openclaw`-gebruiker, bevindt de configuratie zich op `~openclaw/.openclaw/openclaw.json`.
- `refusing to bind gateway ... without auth` → niet-loopback binding zonder token/wachtwoord.
- `another gateway instance is already listening` / `EADDRINUSE` → poortconflict.

Gerelateerd:

- [/gateway/background-process](/gateway/background-process)
- [/gateway/configuration](/gateway/configuration)
- [/gateway/doctor](/gateway/doctor)

## Kanaal verbonden maar berichten stromen niet

Als de kanaalstatus verbonden is maar de berichtstroom dood is, focus op beleid, rechten en kanaalspecifieke leveringsregels.

```bash
openclaw channels status --probe
openclaw pairing list <channel>
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

Let op:

- DM-beleid (`pairing`, `allowlist`, `open`, `disabled`).
- Groeps-toegestane lijst en vereisten voor vermeldingen.
- Ontbrekende kanaal-API-rechten/scopes.

Veelvoorkomende signalen:

- `mention required` → bericht genegeerd door groepsvermeldingsbeleid.
- `pairing` / sporen van goedkeuring in afwachting → afzender is niet goedgekeurd.
- `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → kanaal-auth/rechtenprobleem.

Gerelateerd:

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/whatsapp](/channels/whatsapp)
- [/channels/telegram](/channels/telegram)
- [/channels/discord](/channels/discord)

## Cron- en heartbeat-levering

Als cron of heartbeat niet heeft gedraaid of niet is afgeleverd, verifieer eerst de schedulerstatus en daarna het afleverdoel.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Let op:

- Cron ingeschakeld en volgende wekmoment aanwezig.
- Status van job-uitvoergeschiedenis (`ok`, `skipped`, `error`).
- Redenen voor het overslaan van heartbeats (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Veelvoorkomende signalen:

- `cron: scheduler disabled; jobs will not run automatically` → cron uitgeschakeld.
- `cron: timer tick failed` → scheduler-tick mislukt; controleer bestand-/log-/runtimefouten.
- `heartbeat skipped` met `reason=quiet-hours` → buiten het venster voor actieve uren.
- `heartbeat: unknown accountId` → ongeldig account-id voor het afleverdoel van de heartbeat.

Gerelateerd:

- [/automation/troubleshooting](/automation/troubleshooting)
- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)

## Node gekoppeld maar tool faalt

Als een node is gekoppeld maar tools falen, isoleer voorgrondstatus, rechten en goedkeuringsstatus.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Let op:

- Node online met de verwachte capabilities.
- OS-rechten voor camera/microfoon/locatie/scherm.
- Uitvoeringsgoedkeuringen en status van de toegestane lijst.

Veelvoorkomende signalen:

- `NODE_BACKGROUND_UNAVAILABLE` → node-app moet op de voorgrond staan.
- `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → ontbrekende OS-rechten.
- `SYSTEM_RUN_DENIED: approval required` → uitvoeringsgoedkeuring in afwachting.
- `SYSTEM_RUN_DENIED: allowlist miss` → opdracht geblokkeerd door toegestane lijst.

Gerelateerd:

- [/nodes/troubleshooting](/nodes/troubleshooting)
- [/nodes/index](/nodes/index)
- [/tools/exec-approvals](/tools/exec-approvals)

## Browsertool faalt

Gebruik dit wanneer browsertool-acties falen terwijl de gateway zelf gezond is.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

Let op:

- Geldig pad naar het browser-executable.
- Bereikbaarheid van het CDP-profiel.
- Bijlage van het extensierelay-tabblad voor `profile="chrome"`.

Veelvoorkomende signalen:

- `Failed to start Chrome CDP on port` → browserproces kon niet worden gestart.
- `browser.executablePath not found` → geconfigureerd pad is ongeldig.
- `Chrome extension relay is running, but no tab is connected` → extensierelay niet gekoppeld.
- `Browser attachOnly is enabled ... not reachable` → profiel met alleen koppelen heeft geen bereikbaar doel.

Gerelateerd:

- [/tools/browser-linux-troubleshooting](/tools/browser-linux-troubleshooting)
- [/tools/chrome-extension](/tools/chrome-extension)
- [/tools/browser](/tools/browser)

## Als je hebt geüpgraded en er plotseling iets is stukgegaan

De meeste problemen na een upgrade zijn config-drift of strengere standaardwaarden die nu worden afgedwongen.

### 1. Auth- en URL-overridegedrag is gewijzigd

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

Wat te controleren:

- Als `gateway.mode=remote`, kunnen CLI-aanroepen op remote gericht zijn terwijl je lokale service in orde is.
- Expliciete `--url`-aanroepen vallen niet terug op opgeslagen referenties.

Veelvoorkomende signalen:

- `gateway connect failed:` → verkeerd URL-doel.
- `unauthorized` → endpoint bereikbaar maar verkeerde auth.

### 2. Bind- en auth-guardrails zijn strenger

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

Wat te controleren:

- Niet-loopback bindings (`lan`, `tailnet`, `custom`) vereisen geconfigureerde auth.
- Oude sleutels zoals `gateway.token` vervangen `gateway.auth.token` niet.

Veelvoorkomende signalen:

- `refusing to bind gateway ... without auth` → bind+auth-mismatch.
- `RPC probe: failed` terwijl de runtime draait → gateway leeft maar is ontoegankelijk met de huidige auth/URL.

### 3. Koppeling en apparaatidentiteitsstatus zijn gewijzigd

```bash
openclaw devices list
openclaw pairing list <channel>
openclaw logs --follow
openclaw doctor
```

Wat te controleren:

- Apparaatgoedkeuringen in afwachting voor dashboard/nodes.
- DM-koppelingsgoedkeuringen in afwachting na beleids- of identiteitswijzigingen.

Veelvoorkomende signalen:

- `device identity required` → apparaatauthenticatie niet voldaan.
- `pairing required` → afzender/apparaat moet worden goedgekeurd.

Als serviceconfig en runtime na controles nog steeds niet overeenkomen, installeer service-metadata opnieuw vanuit dezelfde profiel-/statusdirectory:

```bash
openclaw gateway install --force
openclaw gateway restart
```

Gerelateerd:

- [/gateway/pairing](/gateway/pairing)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/background-process](/gateway/background-process)

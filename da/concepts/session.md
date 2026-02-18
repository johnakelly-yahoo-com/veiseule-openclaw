---
title: "Sessionshåndtering"
---

# Sessionshåndtering

OpenClaw behandler **én direkte chat-session pr. agent** som primær. Direkte chats kollapser til `agent:<agentId>:<mainKey>` (standard `main`), mens gruppe-/kanalchats får deres egne nøgler. `session.mainKey` respekteres.

Brug `session.dmScope` til at styre, hvordan **direkte beskeder (DM’er)** grupperes:

- `main` (standard): alle DM’er deler hovedsessionen for kontinuitet.
- `per-peer`: isolér efter afsender-id på tværs af kanaler.
- `per-channel-peer`: isolér efter kanal + afsender (anbefalet til indbakker med flere brugere).
- `per-account-channel-peer`: isolér efter konto + kanal + afsender (anbefalet til multi-konto-indbakker).
  Brug `session.identityLinks` til at kortlægge udbyder-præfikserede peer-id’er til en kanonisk identitet, så den samme person deler en DM-session på tværs af kanaler, når du bruger `per-peer`, `per-channel-peer` eller `per-account-channel-peer`.

## Sikker DM-tilstand (anbefalet til opsætninger med flere brugere)

> **Sikkerhedsadvarsel:** Hvis din agent kan modtage DM’er fra **flere personer**, bør du kraftigt overveje at aktivere sikker DM-tilstand. Uden den deler alle brugere den samme samtalekontekst, hvilket kan lække private oplysninger mellem brugere.

**Eksempel på problemet med standardindstillinger:**

- Alice (`<SENDER_A>`) skriver til din agent om et privat emne (for eksempel en lægeaftale)
- Bob (`<SENDER_B>`) skriver til din agent og spørger: “Hvad talte vi om?”
- Fordi begge DM’er deler den samme session, kan modellen svare Bob ved at bruge Alices tidligere kontekst.

**Løsningen:** Sæt `dmScope` til at isolere sessioner pr. bruger:

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    // Secure DM mode: isolate DM context per channel + sender.
    dmScope: "per-channel-peer",
  },
}
```

**Hvornår bør dette aktiveres:**

- Du har parringsgodkendelser for mere end én afsender
- Du bruger en DM-tilladelsesliste med flere poster
- Du sætter `dmPolicy: "open"`
- Flere telefonnumre eller konti kan skrive til din agent

Noter:

- Standard er `dmScope: "main"` for kontinuitet (alle DM’er deler hovedsessionen). Dette er fint for enkeltbruger-opsætninger.
- For multi-konto-indbakker på samme kanal bør du foretrække `per-account-channel-peer`.
- Hvis den samme person kontakter dig på flere kanaler, brug `session.identityLinks` til at samle deres DM-sessioner i én kanonisk identitet.
- Du kan verificere dine DM-indstillinger med `openclaw security audit` (se [security](/cli/security)).

## Gateway er sandhedskilden

Al sessionstilstand **ejes af gatewayen** (“master” OpenClaw). UI-klienter (macOS-app, WebChat osv.) skal forespørge gatewayen om sessionslister og token-optællinger i stedet for at læse lokale filer.

- I **remote-tilstand** ligger det sessionslager, du skal bruge, på den eksterne gateway-vært – ikke på din Mac.
- Token-optællinger vist i UI’er kommer fra gatewayens lagerfelter (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Klienter parser ikke JSONL-transskripter for at “rette” totaler.

## Hvor tilstanden ligger

- På **gateway-værten**:
  - Lagerfil: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (pr. agent).
- Transskripter: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (Telegram-emnesessioner bruger `.../<SessionId>-topic-<threadId>.jsonl`).
- Lageret er et map `sessionKey -> { sessionId, updatedAt, ... }`. Sletning af poster er sikkert; de genskabes efter behov.
- Gruppeposter kan inkludere `displayName`, `channel`, `subject`, `room` og `space` for at navngive sessioner i UI’er.
- Sessionsposter inkluderer `origin`-metadata (label + routing-hints), så UI’er kan forklare, hvor en session stammer fra.
- OpenClaw læser **ikke** ældre Pi/Tau-sessionsmapper.

## Sessionbeskæring

OpenClaw trimmer **gamle værktøjsresultater** fra in-memory-konteksten lige før LLM-kald som standard.
Dette omskriver **ikke** JSONL-historikken. Se [/concepts/session-pruning](/concepts/session-pruning).

## Pre-komprimerings-hukommelsesflush

Når en session nærmer sig automatisk komprimering, kan OpenClaw køre en **stille hukommelses-flush**
runde, der minder modellen om at skrive varige noter til disk. Dette kører kun, når
arbejdsområdet er skrivbart. Se [Memory](/concepts/memory) og
[Compaction](/concepts/compaction).

## Mapping af transports → sessionsnøgler

- Direkte chats følger `session.dmScope` (standard `main`).
  - `main`: `agent:<agentId>:<mainKey>` (kontinuitet på tværs af enheder/kanaler).
    - Flere telefonnumre og kanaler kan mappe til den samme agent-hovednøgle; de fungerer som transports ind i én samtale.
  - `per-peer`: `agent:<agentId>:dm:<peerId>`.
  - `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`.
  - `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (accountId har standardværdien `default`).
  - Hvis `session.identityLinks` matcher et udbyder-præfikseret peer-id (for eksempel `telegram:123`), erstatter den kanoniske nøgle `<peerId>`, så den samme person deler en session på tværs af kanaler.
- Gruppechats isolerer tilstand: `agent:<agentId>:<channel>:group:<id>` (rum/kanaler bruger `agent:<agentId>:<channel>:channel:<id>`).
  - Telegram-forumemner tilføjer `:topic:<threadId>` til gruppe-id’et for isolation.
  - Ældre `group:<id>`-nøgler genkendes stadig til migrering.
- Indgående kontekster kan stadig bruge `group:<id>`; kanalen udledes fra `Provider` og normaliseres til den kanoniske `agent:<agentId>:<channel>:group:<id>`-form.
- Andre kilder:
  - Cron-jobs: `cron:<job.id>`
  - Webhooks: `hook:<uuid>` (medmindre det eksplicit sættes af hooken)
  - Node-kørsler: `node-<nodeId>`

## Livscyklus

- Nulstillingspolitik: sessioner genbruges, indtil de udløber, og udløb evalueres ved den næste indgående besked.
- Daglig nulstilling: standard er **kl. 4:00 lokal tid på gateway-værten**. En session er forældet, når dens sidste opdatering er tidligere end det seneste daglige nulstillingstidspunkt.
- Inaktivitetsnulstilling (valgfri): `idleMinutes` tilføjer et glidende inaktivitetsvindue. Når både daglig og inaktivitetsnulstilling er konfigureret, vil **den, der udløber først**, tvinge en ny session.
- Ældre kun-inaktiv: hvis du sætter `session.idleMinutes` uden nogen `session.reset`/`resetByType`-konfiguration, forbliver OpenClaw i kun-inaktiv-tilstand af hensyn til bagudkompatibilitet.
- Tilsidesættelser pr. type (valgfrit): `resetByType` lader dig tilsidesætte politikken for `direct`, `group` og `thread`-sessioner (thread = Slack/Discord-tråde, Telegram-emner, Matrix-tråde når leveret af connectoren).
- Tilsidesættelser pr. kanal (valgfrit): `resetByChannel` tilsidesætter nulstillingspolitikken for en kanal (gælder for alle sessionstyper for den kanal og har forrang over `reset`/`resetByType`).
- Nulstillingsudløsere: eksakt `/new` eller `/reset` (plus eventuelle ekstra i `resetTriggers`) starter et nyt session-id og sender resten af beskeden videre. `/new <model>` accepterer et model-alias, `provider/model` eller leverandørnavn (fuzzy match) for at sætte modellen for den nye session. Hvis `/new` eller `/reset` sendes alene, kører OpenClaw en kort “hello”-hilsen for at bekræfte nulstillingen.
- Manuel nulstilling: slet specifikke nøgler fra lageret eller fjern JSONL-transskriptet; den næste besked genskaber dem.
- Isolerede cron-jobs opretter altid et nyt `sessionId` pr. kørsel (ingen genbrug baseret på inaktivitet).

## Afsendelsespolitik (valgfrit)

Blokér levering for specifikke sessionstyper uden at liste individuelle id’er.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // Match the raw session key (including the `agent:<id>:` prefix).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

Runtime-tilsidesættelse (kun ejer):

- `/send on` → tillad for denne session
- `/send off` → afvis for denne session
- `/send inherit` → ryd tilsidesættelsen og brug konfigurationsreglerne
  Send disse som selvstændige beskeder, så de registreres.

## Konfiguration (valgfrit omdøbningseksempel)

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // hold gruppenøgler adskilt
    dmScope: "main", // DM-kontinuitet (sæt per-channel-peer/per-account-channel-peer for delte indbakker)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // Standarder: mode=daily, atHour=4 (gateway-værtens lokale tid).
      // Hvis du også sætter idleMinutes, vinder den, der udløber først.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## Inspektion

- `openclaw status` — viser lagersti og nylige sessioner.
- `openclaw sessions --json` — dumper alle poster (filtrér med `--active <minutes>`).
- `openclaw gateway call sessions.list --params '{}'` — henter sessioner fra den kørende gateway (brug `--url`/`--token` for adgang til remote gateway).
- Send `/status` som en selvstændig besked i chatten for at se, om agenten er tilgængelig, hvor meget af sessionskonteksten der bruges, aktuelle thinking/verbose-toggles, og hvornår dine WhatsApp Web-legitimationsoplysninger sidst blev opdateret (hjælper med at opdage behov for genlink).
- Send `/context list` eller `/context detail` for at se, hvad der er i systemprompten og de injicerede arbejdsområdefiler (og de største kontekstbidrag).
- Send `/stop` som en selvstændig besked for at afbryde den aktuelle kørsel, rydde køede opfølgninger for den session og stoppe eventuelle underagent-kørsler, der er startet fra den (svaret inkluderer antallet, der blev stoppet).
- Send `/compact` (valgfri instruktioner) som en selvstændig besked for at opsummere ældre kontekst og frigøre vinduesplads. Se [/concepts/compaction](/concepts/compaction).
- JSONL-transskripter kan åbnes direkte for at gennemgå fulde ture.

## Tips

- Hold den primære nøgle dedikeret til 1:1-trafik; lad grupper beholde deres egne nøgler.
- Ved automatiseret oprydning bør du slette individuelle nøgler i stedet for hele lageret for at bevare kontekst andre steder.

## Metadata om sessionsoprindelse

Hver sessionspost registrerer, hvor den stammer fra (best effort), i `origin`:

- `label`: menneskeligt label (afledt af samtalelabel + gruppeemne/kanal)
- `provider`: normaliseret kanal-id (inklusive udvidelser)
- `from`/`to`: rå routing-id’er fra den indgående konvolut
- `accountId`: udbyder-konto-id (ved flere konti)
- `threadId`: tråd-/emne-id når kanalen understøtter det
  Oprindelsesfelterne udfyldes for direkte beskeder, kanaler og grupper. Hvis en
  connector kun opdaterer leveringsrouting (for eksempel for at holde en DM-hovedsession
  frisk), bør den stadig levere indgående kontekst, så sessionen bevarer sin
  forklarende metadata. Udvidelser kan gøre dette ved at sende `ConversationLabel`,
  `GroupSubject`, `GroupChannel`, `GroupSpace` og `SenderName` i den indgående
  kontekst og kalde `recordSessionMetaFromInbound` (eller sende den samme kontekst
  til `updateLastRoute`).
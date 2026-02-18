---
title: "Kontekst"
---

# Kontekst

“Kontekst” er **alt OpenClaw sender til modellen for en kørs**. Det er afgrænset af modellens **kontekstvindue** (token limit).

Begynder-mentalmodel:

- **Systemprompt** (bygget af OpenClaw): regler, værktøjer, Skills-liste, tid/runtime og injicerede workspace-filer.
- **Samtalehistorik**: dine beskeder + assistentens beskeder for denne session.
- **Værktøjskald/-resultater + vedhæftninger**: kommandooutput, fil-læsninger, billeder/lyd osv.

Kontekst er _ikke det samme_ som “hukommelse”: hukommelse kan gemmes på disk og genindlæses senere; kontekst er det, der er inde i modellens aktuelle vindue.

## Hurtig start (inspicér kontekst)

- `/status` → hurtig “hvor fuld er mit vindue?” se + session indstillinger.
- `/context list` → hvad der injiceres + omtrentlige størrelser (pr. fil + totaler).
- `/context detail` → dybere opdeling: pr. fil, pr. værktøjsskema-størrelser, pr. skill-post-størrelser og systemprompt-størrelse.
- `/usage tokens` → tilføj forbrugsfodnote pr. svar til normale svar.
- `/compact` → opsummér ældre historik til en kompakt post for at frigøre vinduesplads.

Se også: [Slash commands](/tools/slash-commands), [Tokenbrug & omkostninger](/reference/token-use), [Kompaktering](/concepts/compaction).

## Eksempeloutput

Værdier varierer efter model, udbyder, værktøjspolitik og hvad der er i dit workspace.

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Hvad tæller med i kontekstvinduet

Alt, hvad modellen modtager, tæller med, herunder:

- Systemprompt (alle sektioner).
- Samtalehistorik.
- Værktøjskald + værktøjsresultater.
- Vedhæftninger/transskripter (billeder/lyd/filer).
- Kompakteringsopsummeringer og beskæringsartefakter.
- Udbyder-“wrappere” eller skjulte headere (ikke synlige, men tælles med).

## Hvordan OpenClaw bygger systemprompten

Systemprompten er **OpenClaw-owned** og genopbygget hvert løb. Den omfatter:

- Værktøjsliste + korte beskrivelser.
- Skills-liste (kun metadata; se nedenfor).
- Workspace-placering.
- Tid (UTC + konverteret brugertid, hvis konfigureret).
- Runtime-metadata (vært/OS/model/thinking).
- Injicerede workspace-bootstrapfiler under **Project Context**.

Fuld opdeling: [System Prompt](/concepts/system-prompt).

## Injicerede workspace-filer (Project Context)

Som standard injicerer OpenClaw et fast sæt workspace-filer (hvis de findes):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (kun første run)

Store filer afkortet per-fil ved hjælp af 'agents.defaults.bootstrapMaxChars' (standard '20000' tegn). `/context` viser **rå vs injicerede** størrelser, og om trunkering skete.

## Skills: hvad injiceres vs. indlæses efter behov

Systemprompten indeholder en kompakt \*\* færdighedsliste \*\* (navn + beskrivelse + placering). Denne liste har virkelig overhead.

Færdighedsinstruktioner er _not_ inkluderet som standard. Modellen forventes at `læse` færdighedens `SKILL.md` **kun når det er nødvendigt**.

## Værktøjer: der er to omkostninger

Værktøjer påvirker kontekst på to måder:

1. **Værktøjslistetekst** i systemprompten (det, du ser som “Tooling”).
2. **Tool schemas** (JSON). Disse sendes til modellen, så det kan ringe til værktøjer. De tæller mod kontekst, selvom du ikke ser dem som almindelig tekst.

`/context detail` opdeler de største værktøjsskemaer, så du kan se, hvad der dominerer.

## Kommandoer, direktiver og “inline-genveje”

Slash kommandoer håndteres af Porten. Der er et par forskellige adfærd:

- **Selvstændige kommandoer**: en besked, der kun er `/...`, køres som en kommando.
- **Direktiver**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` fjernes, før modellen ser beskeden.
  - Beskeder kun med direktiver bevarer sessionsindstillinger.
  - Inline-direktiver i en normal besked fungerer som hints pr. besked.
- **Inline-genveje** (kun tilladelsesliste-afsendere): visse `/...`-tokens inde i en normal besked kan køre med det samme (eksempel: “hey /status”) og fjernes, før modellen ser den resterende tekst.

Detaljer: [Slash commands](/tools/slash-commands).

## Sessioner, kompaktering og beskæring (hvad persisterer)

Hvad der persisterer på tværs af beskeder afhænger af mekanismen:

- **Normal historik** persisterer i sessionstransskriptet, indtil den kompakteres/beskæres af politik.
- **Kompaktering** persisterer en opsummering i transskriptet og bevarer de seneste beskeder intakte.
- **Beskæring** fjerner gamle værktøjsresultater fra den _in-memory_ prompt for et run, men omskriver ikke transskriptet.

Docs: [Session](/concepts/session), [Kompaktering](/concepts/compaction), [Session-beskæring](/concepts/session-pruning).

## Hvad `/context` faktisk rapporterer

`/context` foretrækker den seneste **run-byggede** systemprompt-rapport, når den er tilgængelig:

- `System prompt (run)` = indfanget fra det seneste indlejrede (værktøjskompetente) run og persisteret i session-store.
- `System prompt (estimate)` = beregnet on-the-fly, når der ikke findes nogen run-rapport (eller når der køres via en CLI-backend, der ikke genererer rapporten).

Uanset hvad rapporterer den størrelser og topbidragydere; den dumper **ikke** den fulde systemprompt eller værktøjsskemaer.



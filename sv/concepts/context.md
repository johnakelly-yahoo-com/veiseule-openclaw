---
title: "Kontext"
---

# Kontext

“Context” är **allt OpenClaw skickar till modellen för en kör**. Den begränsas av modellens **sammanhangsfönster** (token limit).

Mental modell för nybörjare:

- **Systemprompt** (byggd av OpenClaw): regler, verktyg, Skills-lista, tid/körtid och injicerade arbetsytefiler.
- **Konversationshistorik**: dina meddelanden + assistentens meddelanden för denna session.
- **Verktygsanrop/-resultat + bilagor**: kommandoutdata, filläsningar, bilder/ljud m.m.

Kontext är _inte samma sak_ som ”minne”: minne kan lagras på disk och laddas senare; kontext är det som finns i modellens aktuella fönster.

## Snabbstart (inspektera kontext)

- `/status` → snabb “hur full är mitt fönster?” visa + sessionsinställningar.
- `/context list` → vad som injiceras + ungefärliga storlekar (per fil + totalt).
- `/context detail` → djupare uppdelning: per fil, per verktygsschemastorlek, per Skill-poststorlek och systempromptens storlek.
- `/usage tokens` → lägg till en användningsfot per svar i normala svar.
- `/compact` → sammanfatta äldre historik till en kompakt post för att frigöra fönsterutrymme.

Se även: [Slash-kommandon](/tools/slash-commands), [Tokenanvändning och kostnader](/reference/token-use), [Kompaktering](/concepts/compaction).

## Exempelutdata

Värden varierar beroende på modell, leverantör, verktygspolicy och vad som finns i din arbetsyta.

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

## Vad som räknas mot kontextfönstret

Allt som modellen tar emot räknas, inklusive:

- Systemprompt (alla avsnitt).
- Konversationshistorik.
- Verktygsanrop + verktygsresultat.
- Bilagor/transkript (bilder/ljud/filer).
- Kompakteringssammanfattningar och beskärningsartefakter.
- Leverantörers ”wrappers” eller dolda headers (inte synliga, men räknas ändå).

## Hur OpenClaw bygger systemprompten

Systemprompten är **OpenClaw-owned** och byggde om varje körning. Den inkluderar:

- Verktygslista + korta beskrivningar.
- Skills-lista (endast metadata; se nedan).
- Arbetsyteplats.
- Tid (UTC + konverterad användartid om konfigurerad).
- Körtidsmetadata (värd/OS/modell/tänkande).
- Injekterade bootstrap-filer från arbetsytan under **Project Context**.

Fullständig uppdelning: [Systemprompt](/concepts/system-prompt).

## Injekterade arbetsytefiler (Project Context)

Som standard injicerar OpenClaw en fast uppsättning arbetsytefiler (om de finns):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (endast vid första körningen)

Stora filer är trunkerade per-fil med `agents.defaults.bootstrapMaxChars` (standard `20000`-tecken). `/context` visar **rå vs injicerade** storlekar och om trunkering hände.

## Skills: vad som injiceras vs laddas vid behov

Systemprompten innehåller en kompakt **kompetenslista** (namn + beskrivning + plats). Denna lista har verkliga omkostnader.

Färdighetsinstruktioner är _inte_ inkluderade som standard. Modellen förväntas `läsa` färdighetens `SKILL.md` **endast när det behövs**.

## Verktyg: det finns två kostnader

Verktyg påverkar kontexten på två sätt:

1. **Verktygslistans text** i systemprompten (det du ser som ”Tooling”).
2. **Tool schemas** (JSON). Dessa skickas till modellen så att den kan ringa verktyg. De räknas mot sammanhang även om du inte ser dem som ren text.

`/context detail` bryter ned de största verktygsschemana så att du kan se vad som dominerar.

## Kommandon, direktiv och ”inline-genvägar”

Slash kommandon hanteras av Gateway. Det finns några olika beteenden:

- **Fristående kommandon**: ett meddelande som endast är `/...` körs som ett kommando.
- **Direktiv**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` tas bort innan modellen ser meddelandet.
  - Meddelanden som endast består av direktiv bevarar sessionsinställningar.
  - Inline-direktiv i ett normalt meddelande fungerar som hintar per meddelande.
- **Inline-genvägar** (endast tillåtelselista av avsändare): vissa `/...`-token i ett normalt meddelande kan köras omedelbart (exempel: ”hey /status”) och tas bort innan modellen ser återstående text.

Detaljer: [Slash-kommandon](/tools/slash-commands).

## Sessioner, kompaktering och beskärning (vad som bevaras)

Vad som bevaras mellan meddelanden beror på mekanismen:

- **Normal historik** bevaras i sessionstranskriptet tills den kompakteras/beskärs enligt policy.
- **Kompaktering** bevarar en sammanfattning i transkriptet och behåller nyliga meddelanden intakta.
- **Beskärning** tar bort gamla verktygsresultat från den _in-memory_-prompt som används för en körning, men skriver inte om transkriptet.

Dokumentation: [Session](/concepts/session), [Kompaktering](/concepts/compaction), [Sessionsbeskärning](/concepts/session-pruning).

## Vad `/context` faktiskt rapporterar

`/context` föredrar den senaste **körningsbyggda** systemprompt-rapporten när den finns:

- `System prompt (run)` = fångad från den senaste inbäddade (verktygskapabla) körningen och bevarad i sessionslagret.
- `System prompt (estimate)` = beräknad i realtid när ingen körningsrapport finns (eller när du kör via ett CLI-backend som inte genererar rapporten).

Oavsett vilket rapporteras storlekar och största bidragsgivare; den dumpar **inte** hela systemprompten eller verktygsschemana.



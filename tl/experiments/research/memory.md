---
title: "Pananaliksik sa Workspace Memory"
---

# Workspace Memory v2 (offline): mga tala sa pananaliksik

Target: Clawd-style na workspace (`agents.defaults.workspace`, default `~/.openclaw/workspace`) kung saan ang “memorya” ay naka-store bilang isang Markdown file kada araw (`memory/YYYY-MM-DD.md`) kasama ang maliit na hanay ng mga stable file (hal. `memory.md`, `SOUL.md`).

Iminumungkahi ng dokumentong ito ang isang **offline-first** na arkitektura ng memorya na pinananatiling Markdown ang canonical, nare-review na source of truth, ngunit nagdadagdag ng **structured recall** (search, entity summaries, confidence updates) sa pamamagitan ng isang derived index.

## Bakit magbabago?

Ang kasalukuyang setup (isang file kada araw) ay mahusay para sa:

- “append-only” na journaling
- pag-e-edit ng tao
- git-backed na durability + auditability
- low-friction na pagkuha (“isulat mo lang”)

Mahina ito para sa:

- high-recall na retrieval (“ano ang napagdesisyunan natin tungkol sa X?”, “huling beses na sinubukan natin ang Y?”)
- entity-centric na sagot (“sabihin mo sa akin ang tungkol kay Alice / The Castle / warelay”) nang hindi muling binabasa ang maraming file
- katatagan ng opinyon/preference (at ebidensya kapag nagbago)
- at conflict resolution and conflict resolution

## Mga layunin sa disenyo

- **Offline**: gumagana nang walang network; puwedeng tumakbo sa laptop/Castle; walang cloud dependency.
- **Explainable**: ang mga nare-retrieve na item ay dapat ma-attribyut (file + lokasyon) at maihiwalay mula sa inference.
- **Low ceremony**: nananatiling Markdown ang daily logging, walang mabigat na schema work.
- **Incremental**: kapaki-pakinabang na ang v1 gamit lang ang FTS; optional na upgrade ang semantic/vector at graphs.
- **Agent-friendly**: ginagawang madali ang “recall sa loob ng token budgets” (nagbabalik ng maliliit na bundle ng facts).

## North star na modelo (Hindsight × Letta)

Dalawang bahagi na pagsasamahin:

1. **Control loop na istilong Letta/MemGPT**

- panatilihin ang maliit na “core” na laging nasa context (persona + key user facts)
- ang lahat ng iba pa ay out-of-context at nire-retrieve sa pamamagitan ng tools
- ang memory writes ay explicit na tool calls (append/replace/insert), pini-persist, at saka muling ini-inject sa susunod na turn

2. **Memory substrate na istilong Hindsight**

- ihiwalay ang naobserbahan vs pinaniniwalaan vs sinummarize
- suportahan ang retain/recall/reflect
- mga opinyong may confidence na puwedeng mag-evolve batay sa ebidensya
- entity-aware na retrieval + temporal queries (kahit walang full knowledge graphs)

## Iminungkahing arkitektura (Markdown source-of-truth + derived index)

### Canonical na imbakan (git-friendly)

Panatilihin ang `~/.openclaw/workspace` bilang canonical na human-readable na memorya.

Iminungkahing layout ng workspace:

```
~/.openclaw/workspace/
  memory.md                    # small: durable facts + preferences (core-ish)
  memory/
    YYYY-MM-DD.md              # daily log (append; narrative)
  bank/                        # “typed” memory pages (stable, reviewable)
    world.md                   # objective facts about the world
    experience.md              # what the agent did (first-person)
    opinions.md                # subjective prefs/judgments + confidence + evidence pointers
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Mga tala:

- Hindi na kailangang gawing JSON. Mga entity: `@Peter`, `@warelay`, atbp (ang mga slug ay nagmamapa sa `bank/entities/*.md`)
- Ang mga file na `bank/` ay **curated**, ginagawa ng mga reflection job, at maaari pa ring i-edit nang mano-mano.
- Ang `memory.md` ay nananatiling “maliit + core-ish”: ang mga bagay na gusto mong makita ng Clawd sa bawat session.

### Derived na imbakan (machine recall)

Magdagdag ng derived index sa ilalim ng workspace (hindi kinakailangang git tracked):

```
~/.openclaw/workspace/.memory/index.sqlite
```

Suportahan ito ng:

- SQLite schema para sa facts + entity links + opinion metadata
- SQLite **FTS5** para sa lexical recall (mabilis, maliit, offline)
- opsyonal na embeddings table para sa semantic recall (offline pa rin)

Ang index ay laging **maaaring i-rebuild mula sa Markdown**.

## Panatilihin / Alalahanin / Magnilay (operational loop)

### Retain: i-normalize ang daily logs tungo sa “facts”

Ang mahalagang insight ng Hindsight dito: mag-store ng **narrative, self-contained na facts**, hindi maliliit na snippet.

Praktikal na patakaran para sa `memory/YYYY-MM-DD.md`:

- sa pagtatapos ng araw (o habang nangyayari), magdagdag ng isang `## Retain` na seksyon na may 2–5 bullets na:
  - narrative (nananatili ang cross-turn context)
  - self-contained (may saysay kahit mag-isa sa hinaharap)
  - may tag ng type + entity mentions

Halimbawa:

```
## Retain
- W @Peter: Currently in Marrakech (Nov 27–Dec 1, 2025) for Andy’s birthday.
- B @warelay: I fixed the Baileys WS crash by wrapping connection.update handlers in try/catch (see memory/2025-11-27.md).
- O(c=0.95) @Peter: Prefers concise replies (&lt;1500 chars) on WhatsApp; long content goes into files.
```

Minimal na parsing:

- Type prefix: `W` (world), `B` (experience/biographical), `O` (opinion), `S` (observation/summary; karaniwang generated)
- **opinyon**: “ano ang mas gusto ni Peter?”
- Opinion confidence: `O(c=0.0..1.0)` opsyonal

Kung ayaw mong isipin ito ng mga author: maaaring i-infer ng reflect job ang mga bullet na ito mula sa natitirang log, ngunit ang pagkakaroon ng explicit na `## Retain` na seksyon ang pinakamadaling “quality lever”.

### Recall: mga query sa derived index

Dapat suportahan ng recall ang:

- **lexical**: “hanapin ang eksaktong terms / pangalan / commands” (FTS5)
- **entity**: “sabihin mo sa akin ang tungkol sa X” (entity pages + entity-linked facts)
- **temporal**: “ano ang nangyari bandang Nob 27” / “mula noong nakaraang linggo”
- (may kumpiyansa + ebidensya) (with confidence + evidence)

Ang format ng ibinabalik ay dapat agent-friendly at may citation ng sources:

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (source day, o extracted na time range kung mayroon)
- `entities` (`["Peter","warelay"]`)
- `content` (ang narrative fact)
- `source` (`memory/2025-11-27.md#L12` atbp.)

### Reflect: gumawa ng stable pages + i-update ang mga paniniwala

Ang reflection ay isang scheduled job (daily o heartbeat `ultrathink`) na:

- ina-update ang `bank/entities/*.md` mula sa mga kamakailang facts (entity summaries)
- ina-update ang `bank/opinions.md` na confidence batay sa reinforcement/contradiction
- opsyonal na nagmumungkahi ng mga edit sa `memory.md` (“core-ish” na matitibay na facts)

Pag-evolve ng opinyon (simple, explainable):

- bawat opinyon ay may:
  - statement
  - confidence `c ∈ [0,1]`
  - last_updated
  - evidence links (sumusuporta + sumasalungat na fact IDs)
- kapag may dumating na bagong facts:
  - hanapin ang candidate opinions batay sa entity overlap + similarity (FTS muna, embeddings sa susunod)
  - i-update ang confidence sa pamamagitan ng maliliit na delta; ang malalaking talon ay nangangailangan ng malakas na contradiction + paulit-ulit na ebidensya

## Integrasyon ng CLI: standalone vs malalim na integrasyon

Rekomendasyon: **malalim na integrasyon sa OpenClaw**, ngunit panatilihin ang hiwalay na core library.

### Bakit i-integrate sa OpenClaw?

- Alam na ng OpenClaw ang:
  - path ng workspace (`agents.defaults.workspace`)
  - session model + heartbeats
  - logging + mga pattern sa pag-troubleshoot
- Gusto mong ang agent mismo ang tumawag sa mga tool:
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

### Bakit hatiin pa rin ang isang library?

- panatilihing testable ang memory logic nang walang gateway/runtime
- magamit muli sa iba pang konteksto (local scripts, future desktop app, atbp.)

Hugis:
Ang memory tooling ay inaasahang isang maliit na CLI + library layer, ngunit exploratory pa lamang ito.

## “S-Collide” / SuCo: kailan ito gagamitin (pananaliksik)

Kung ang “S-Collide” ay tumutukoy sa **SuCo (Subspace Collision)**: isa itong ANN retrieval approach na tumatarget sa malakas na recall/latency tradeoffs sa pamamagitan ng learned/structured collisions sa mga subspace (papel: arXiv 2411.14754, 2024).

Pragmatic na pananaw para sa `~/.openclaw/workspace`:

- **huwag magsimula** sa SuCo.
- magsimula sa SQLite FTS + (opsyonal) simpleng embeddings; makukuha mo agad ang karamihan ng UX wins.
- isaalang-alang lamang ang SuCo/HNSW/ScaNN-class na solusyon kapag:
  - malaki na ang corpus (sampu-sampung libo hanggang daan-libong chunks)
  - masyado nang mabagal ang brute-force embedding search
  - ang kalidad ng recall ay malinaw na bottleneck ng lexical search

Mga offline-friendly na alternatibo (pataas ang complexity):

- SQLite FTS5 + metadata filters (walang ML)
- Embeddings + brute force (nakakagulat na umaabot nang malayo kung kaunti ang chunks)
- HNSW index (karaniwan, matibay; nangangailangan ng library binding)
- SuCo (research-grade; kaakit-akit kung may solid na implementation na puwedeng i-embed)

Bukas na tanong:

- ano ang **pinakamahusay** na offline embedding model para sa “personal assistant memory” sa iyong mga makina (laptop + desktop)?
  - kung mayroon ka nang Ollama: mag-embed gamit ang local model; kung hindi, mag-ship ng maliit na embedding model sa toolchain.

## Pinakamaliit na kapaki-pakinabang na pilot

Kung gusto mo ng minimal pero kapaki-pakinabang na bersyon:

- Magdagdag ng `bank/` na entity pages at isang `## Retain` na seksyon sa daily logs.
- Gumamit ng SQLite FTS para sa recall na may citations (path + line numbers).
- Magdagdag lamang ng embeddings kung hinihingi ito ng kalidad o scale ng recall.

## Mga sanggunian

- Mga konsepto ng Letta / MemGPT: “core memory blocks” + “archival memory” + tool-driven na self-editing memory.
- Hindsight Technical Report: “retain / recall / reflect”, four-network memory, narrative fact extraction, opinion confidence evolution.
- SuCo: arXiv 2411.14754 (2024): “Subspace Collision” approximate nearest neighbor retrieval.



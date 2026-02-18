---
title: "Sub-Agenten"
---

# Sub-agents

Sub-Agenten sind Hintergrund-Agentenläufe, die aus einem bestehenden Agentenlauf heraus gestartet werden. Sie laufen in einer eigenen Sitzung (`agent:<agentId>:subagent:<uuid>`) und **melden** ihr Ergebnis nach Abschluss an den anfragenden Chat-Kanal zurück.

## Slash command

Verwenden Sie `/subagents`, um Sub-Agent-Läufe für die **aktuelle Sitzung** zu inspizieren oder zu steuern:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` zeigt Lauf-Metadaten (Status, Zeitstempel, Sitzungs-ID, Transkriptpfad, Aufräumen).

Primäre Ziele:

- Parallelisierung von „Recherche / Langläufer / langsames Tool“, ohne den Hauptlauf zu blockieren.
- Sub-Agenten standardmäßig isolieren (Sitzungstrennung + optionales Sandboxing).
- Die Tool-Oberfläche schwer missbrauchbar halten: Sub-Agenten erhalten **keine** Sitzungstools standardmäßig.
- Konfigurierbare Verschachtelungstiefe für Orchestrator-Muster unterstützen.

Kostenhinweis: Jeder Sub-Agent hat seinen **eigenen** Kontext und eigenen Tokenverbrauch. Für schwere oder wiederholte Aufgaben setzen Sie ein günstigeres Modell für Sub-Agenten ein und behalten Sie für den Hauptagenten ein hochwertigeres Modell bei.  
Sie können dies über `agents.defaults.subagents.model` oder pro-Agent-Überschreibungen konfigurieren.

## Tool

Verwenden Sie `sessions_spawn`:

- Startet einen Sub-Agent-Lauf (`deliver: false`, globale Lane: `subagent`)
- Führt anschließend einen Announce-Schritt aus und postet die Announce-Antwort in den anfragenden Chat-Kanal
- Standardmodell: erbt vom Aufrufer, sofern Sie nicht `agents.defaults.subagents.model` (oder pro-Agent `agents.list[].subagents.model`) setzen; ein explizites `sessions_spawn.model` hat weiterhin Vorrang.
- Standard-Thinking: erbt vom Aufrufer, sofern Sie nicht `agents.defaults.subagents.thinking` (oder pro-Agent `agents.list[].subagents.thinking`) setzen; ein explizites `sessions_spawn.thinking` hat weiterhin Vorrang.

Tool-Parameter:

- `task` (erforderlich)
- `label?` (optional)
- `agentId?` (optional; unter einer anderen Agent-ID starten, falls erlaubt)
- `model?` (optional; überschreibt das Sub-Agent-Modell; ungültige Werte werden übersprungen und der Sub-Agent läuft mit dem Standardmodell, mit Warnung im Tool-Ergebnis)
- `thinking?` (optional; überschreibt die Denkstufe für den Sub-Agent-Lauf)
- `runTimeoutSeconds?` (Standard `0`; wenn gesetzt, wird der Sub-Agent-Lauf nach N Sekunden abgebrochen)
- `cleanup?` (`delete|keep`, Standard `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: Liste von Agent-IDs, die über `agentId` adressiert werden dürfen (`["*"]` erlaubt alle). Standard: nur der anfragende Agent.

Discovery:

- Verwenden Sie `agents_list`, um zu sehen, welche Agent-IDs aktuell für `sessions_spawn` erlaubt sind.

Auto-Archivierung:

- Sub-Agent-Sitzungen werden automatisch nach `agents.defaults.subagents.archiveAfterMinutes` archiviert (Standard: 60).
- Die Archivierung verwendet `sessions.delete` und benennt das Transkript in `*.deleted.<timestamp>` um (gleicher Ordner).
- `cleanup: "delete"` archiviert unmittelbar nach dem Announce (Transkript bleibt durch Umbenennung erhalten).
- Auto-Archivierung ist Best-Effort; ausstehende Timer gehen bei einem Gateway-Neustart verloren.
- `runTimeoutSeconds` archiviert **nicht** automatisch; es stoppt nur den Lauf. Die Sitzung bleibt bis zur Auto-Archivierung bestehen.
- Auto-Archivierung gilt gleichermaßen für Tiefe 1 und Tiefe 2 Sitzungen.

## Nested Sub-Agents

Standardmäßig können Sub-Agenten keine eigenen Sub-Agenten starten (`maxSpawnDepth: 1`). Sie können eine Ebene Verschachtelung aktivieren, indem Sie `maxSpawnDepth: 2` setzen. Dadurch wird das **Orchestrator-Muster** ermöglicht: Main → Orchestrator-Sub-Agent → Worker-Sub-Sub-Agenten.

### How to enable

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // allow sub-agents to spawn children (default: 1)
        maxChildrenPerAgent: 5, // max active children per agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
      },
    },
  },
}
```

### Depth levels

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Hauptagent                                    | Immer                        |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-Agent (Orchestrator wenn Tiefe 2 erlaubt) | Nur wenn `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-Sub-Agent (Leaf-Worker)                   | Niemals                      |

### Announce chain

Ergebnisse fließen die Kette nach oben:

1. Depth-2-Worker beendet → meldet an seinen Parent (Depth-1-Orchestrator)
2. Depth-1-Orchestrator erhält das Announce, konsolidiert Ergebnisse, beendet → meldet an Main
3. Main-Agent erhält das Announce und liefert an den Benutzer

Jede Ebene sieht nur Announce-Nachrichten ihrer direkten Kinder.

### Tool policy by depth

- **Depth 1 (Orchestrator, wenn `maxSpawnDepth >= 2`)**: Erhält `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`, um Kinder zu verwalten. Andere Session-/System-Tools bleiben verweigert.
- **Depth 1 (Leaf, wenn `maxSpawnDepth == 1`)**: Keine Sitzungstools (aktuelles Standardverhalten).
- **Depth 2 (Leaf-Worker)**: Keine Sitzungstools — `sessions_spawn` ist auf Tiefe 2 immer verweigert. Keine weiteren Kinder möglich.

### Per-agent spawn limit

Jede Agentensitzung (auf jeder Tiefe) kann maximal `maxChildrenPerAgent` (Standard: 5) aktive Kinder gleichzeitig haben. Dies verhindert unkontrolliertes Fan-out eines einzelnen Orchestrators.

### Cascade stop

Das Stoppen eines Depth-1-Orchestrators stoppt automatisch alle seine Depth-2-Kinder:

- `/stop` im Hauptchat stoppt alle Depth-1-Agenten und kaskadiert zu deren Depth-2-Kindern.
- `/subagents kill <id>` stoppt einen bestimmten Sub-Agenten und kaskadiert zu dessen Kindern.
- `/subagents kill all` stoppt alle Sub-Agenten des Anfragenden und kaskadiert.

## Authentication

Die Sub-Agent-Authentifizierung wird nach **Agent-ID**, nicht nach Sitzungstyp, aufgelöst:

- Der Sub-Agent-Sitzungsschlüssel ist `agent:<agentId>:subagent:<uuid>`.
- Der Auth-Store wird aus dem `agentDir` dieses Agenten geladen.
- Die Auth-Profile des Hauptagenten werden als **Fallback** zusammengeführt; Agentenprofile überschreiben Hauptprofile bei Konflikten.

Hinweis: Die Zusammenführung ist additiv, daher sind Hauptprofile immer als Fallback verfügbar. Vollständig isolierte Auth pro Agent wird derzeit nicht unterstützt.

## Announce

Sub-Agenten melden über einen Announce-Schritt zurück:

- Der Announce-Schritt läuft innerhalb der Sub-Agent-Sitzung (nicht der Anfragersitzung).
- Antwortet der Sub-Agent exakt mit `ANNOUNCE_SKIP`, wird nichts gepostet.
- Andernfalls wird die Announce-Antwort über einen Follow-up-`agent`-Aufruf (`deliver=true`) in den anfragenden Chat-Kanal gepostet.
- Announce-Antworten bewahren Thread-/Themen-Routing, sofern verfügbar (Slack-Threads, Telegram-Themen, Matrix-Threads).
- Announce-Nachrichten werden auf ein stabiles Template normalisiert:
  - `Status:` abgeleitet aus dem Laufzeitergebnis (`success`, `error`, `timeout` oder `unknown`).
  - `Result:` die Zusammenfassung aus dem Announce-Schritt (oder `(not available)` falls fehlend).
  - `Notes:` Fehlerdetails und andere hilfreiche Kontextinformationen.
- `Status` wird nicht aus der Modellausgabe abgeleitet, sondern aus Laufzeit-Signalen.

Announce-Payloads enthalten am Ende eine Statistikzeile (auch wenn eingebettet):

- Laufzeit (z. B. `runtime 5m12s`)
- Tokenverbrauch (input/output/total)
- Geschätzte Kosten, wenn Modellpreise konfiguriert sind (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` und Transkriptpfad (damit der Hauptagent per `sessions_history` die Historie abrufen oder die Datei auf der Festplatte prüfen kann)

## Tool Policy (sub-agent tools)

Standardmäßig erhalten Sub-Agenten **alle Tools außer Sitzungstools** und System-Tools:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Wenn `maxSpawnDepth >= 2`, erhalten Depth-1-Orchestrator-Sub-Agenten zusätzlich `sessions_spawn`, `subagents`, `sessions_list` und `sessions_history`, um ihre Kinder zu verwalten.

Überschreiben per Konfiguration:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

Sub-Agenten verwenden eine dedizierte In-Process-Queue-Lane:

- Lane-Name: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (Standard `8`)

## Stopping

- Das Senden von `/stop` im anfragenden Chat bricht die Anfragersitzung ab und stoppt alle aktiven daraus gestarteten Sub-Agent-Läufe, inklusive kaskadierender Kinder.
- `/subagents kill <id>` stoppt einen bestimmten Sub-Agenten und kaskadiert zu dessen Kindern.

## Limitations

- Sub-Agent-Announce ist **Best-Effort**. Wenn das Gateway neu startet, geht ausstehende „announce back“-Arbeit verloren.
- Sub-Agenten teilen sich weiterhin die gleichen Gateway-Prozessressourcen; behandeln Sie `maxConcurrent` als Sicherheitsventil.
- `sessions_spawn` ist immer nicht blockierend: es gibt sofort `{ status: "accepted", runId, childSessionKey }` zurück.
- Der Sub-Agent-Kontext injiziert nur `AGENTS.md` + `TOOLS.md` (kein `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` oder `BOOTSTRAP.md`).
- Maximale Verschachtelungstiefe ist 5 (`maxSpawnDepth` Bereich: 1–5). Tiefe 2 wird für die meisten Anwendungsfälle empfohlen.
- `maxChildrenPerAgent` begrenzt aktive Kinder pro Sitzung (Standard: 5, Bereich: 1–20).
---
summary: "Sub-Agents: Starten isolierter Agentenläufe, die Ergebnisse an den anfordernden Chat zurückmelden"
read_when:
  - Sie möchten Hintergrund-/Parallelarbeit über den Agenten ausführen
  - Sie ändern sessions_spawn oder die Sub-Agent-Werkzeugrichtlinie
title: "Sub-Agents"
---

# Sub-Agenten

Unteragenten ermöglichen es Ihnen, Hintergrundaufgaben auszuführen, ohne die Hauptkonversation zu blockieren. Wenn Sie einen Unteragenten starten, läuft er in einer eigenen isolierten Sitzung, erledigt seine Arbeit und meldet das Ergebnis nach Abschluss im Chat.

## Befehl

Verwenden Sie den Slash-Befehl `/subagents`, um Sub-Agent-Läufe für die aktuelle Sitzung zu inspizieren und zu steuern:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` zeigt Ausführungsmetadaten an (Status, Zeitstempel, Sitzungs-ID, Transkriptpfad, Bereinigung).

Primäre Ziele:

- „Research / lange Aufgabe / langsames Tool“-Arbeiten parallelisieren, ohne die Hauptausführung zu blockieren.
- Sub-Agents standardmäßig isoliert halten (Sitzungstrennung + optionale Sandbox).
- Die Tool-Oberfläche schwer missbrauchbar halten: Sub-Agents erhalten standardmäßig **keine** Sitzungstools.
- Konfigurierbare Verschachtelungstiefe für Orchestrator-Muster unterstützen.

Kostennotiz: Jeder Sub-Agent hat seinen **eigenen** Kontext und Token-Verbrauch. Für umfangreiche oder wiederkehrende
Aufgaben ein günstigeres Modell für Sub-Agents festlegen und Ihren Haupt-Agenten auf einem qualitativ hochwertigeren Modell belassen.
In einer Multi-Agenten-Konfiguration können Sie Sub-Agent-Standards pro Agent festlegen:

## Tool-Richtlinie

Expliziter `thinking`-Parameter im `sessions_spawn`-Aufruf

- Startet eine Sub-Agent-Ausführung (`deliver: false`, globale Lane: `subagent`)
- Führt anschließend einen Ankündigungsschritt aus und veröffentlicht die Antwort auf die Ankündigung im Chat-Kanal des Anfragenden
- Globaler Standard: `agents.defaults.subagents.model`
- Modell: normale Modellauswahl des Zielagenten (sofern `subagents.model` nicht gesetzt ist) Thinking: keine Überschreibung für Unteragenten (sofern `subagents.thinking` nicht gesetzt ist)

Tool-Parameter:

- `task`
- `label`
- Unter einer anderen Agenten-ID starten (muss erlaubt sein)
- <Note>
Ungültige Modellwerte werden stillschweigend übersprungen — der Sub-Agent wird mit dem nächsten gültigen Standard ausgeführt, mit einer Warnung im Tool-Ergebnis.
</Note>
- `thinking?` (optional; überschreibt das Thinking-Level für die Sub-Agent-Ausführung)
- Den Sub-Agenten nach N Sekunden abbrechen
- "delete" | "keep"

Allowlist:

- 2. {
     agents: {
     list: [
     {
     id: "orchestrator",
     subagents: {
     allowAgents: ["researcher", "coder"], // oder ["\*"] um alle zu erlauben
     },
     },
     ],
     },
     } Standard: nur der anfragende Agent.

Erkennung:

- Verwenden Sie das Tool `agents_list`, um herauszufinden, welche Agenten-IDs derzeit für `sessions_spawn` erlaubt sind.
</Tip>

Automatisches Archivieren

- Die Unteragenten-Sitzung wird nach 60 Minuten automatisch archiviert (konfigurierbar).
- <Note>
Das Archivieren benennt das Transkript in `*.deleted.<timestamp>` (gleicher Ordner).
- "delete" archiviert unmittelbar nach der Ankündigung
- Automatische Archivierungs-Timer sind Best-Effort; ausstehende Timer gehen verloren, wenn das Gateway neu startet.
- `runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
- Auto-Archivierung gilt gleichermaßen für Sitzungen der Tiefe 1 und Tiefe 2.

## Stopping Sub-Agents

- **No nested spawning:** Sub-agents cannot spawn their own sub-agents. Sie können eine Verschachtelungsebene aktivieren, indem Sie `maxSpawnDepth: 2` setzen; dies ermöglicht das **Orchestrator-Muster**: Haupt-Agent → Orchestrator-Sub-Agent → Worker-Sub-Sub-Agents.

### So aktivieren Sie es

```json5
{
  agents: {
  defaults: {
  subagents: {
  model: "minimax/MiniMax-M2.1",
  },
  },
  },
  }
```

### Tiefenstufen

| Tiefe | Sitzungsschlüssel-Format                                                                                                                                                                       | Rolle                                                                 | Kann Sub-Agents starten?      |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | ----------------------------- |
| 0     | `agentId`                                                                                                                                                                                      | Pro-Agent-Überschreibungen                                            | Immer                         |
| 1     | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;model: "minimax/MiniMax-M2.1",&#xA;},&#xA;},&#xA;},&#xA;} | Sub-Agent (Orchestrator, wenn Tiefe 2 erlaubt ist) | Nur wenn `maxSpawnDepth >= 2` |
| 2     | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                               | _(Agent des Aufrufers)_                            | Niemals                       |

### Kette ankündigen

Ergebnisse fließen die Kette hinauf zurück:

1. Depth-2-Worker beendet → kündigt bei seinem Parent (Depth-1-Orchestrator) an
2. Depth-1-Orchestrator erhält die Ankündigung, fasst die Ergebnisse zusammen, beendet → kündigt bei Main an
3. Main-Agent erhält die Ankündigung und liefert an den Benutzer

Jede Ebene sieht nur Ankündigungen ihrer direkten Kinder.

### Tool-Richtlinie nach Tiefe

- **Depth 1 (Orchestrator, wenn `maxSpawnDepth >= 2`)**: Erhält `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`, damit er seine Kinder verwalten kann. Andere Session-/System-Tools bleiben verweigert.
- **Depth 1 (Leaf, wenn `maxSpawnDepth == 1`)**: Keine Session-Tools (aktuelles Standardverhalten).
- **Depth 2 (Leaf-Worker)**: Keine Session-Tools — `sessions_spawn` ist auf Depth 2 immer verweigert. Kann keine weiteren Kinder starten.

### Agentenübergreifendes Spawnen

Jede Agent-Session (auf jeder Tiefe) kann höchstens `maxChildrenPerAgent` (Standard: 5) aktive Kinder gleichzeitig haben. Dies verhindert eine unkontrollierte Vervielfachung durch einen einzelnen Orchestrator.

### Kaskadierendes Stoppen

Das Stoppen eines Depth-1-Orchestrators beendet automatisch alle seine Depth-2-Kinder:

- `/stop` im Main-Chat beendet alle Depth-1-Agenten und kaskadiert zu ihren Depth-2-Kindern.
- `/subagents stop <id>`
- `/subagents kill all` beendet alle Sub-Agents des Anforderers und kaskadiert.

## Authentifizierung

Die Sub-Agent-Authentifizierung wird nach **Agent-ID** aufgelöst, nicht nach Sitzungstyp:

- Der Sub-Agent-Session-Key ist `agent:<agentId>:subagent:<uuid>`.
- Der Auth-Store wird aus dem `agentDir` des Zielagenten geladen
- Die Auth-Profile des Hauptagenten werden als **Fallback** zusammengeführt (Agentenprofile gewinnen bei Konflikten)

Die Zusammenführung ist additiv — Hauptprofile sind immer als Fallbacks verfügbar
Fully isolated auth per sub-agent is not currently supported.

## Ankündigungsstatus

Sub-Agents melden über einen Announce-Schritt zurück:

- Der Announce-Schritt wird innerhalb der Sub-Agent-Session ausgeführt (nicht in der Session des Anforderers).
- Wenn der Sub-Agent exakt mit `ANNOUNCE_SKIP` antwortet, wird nichts gepostet.
- Andernfalls wird die Announce-Antwort über einen nachfolgenden `agent`-Aufruf (`deliver=true`) im Chat-Kanal des Anforderers gepostet.
- Announce-Antworten bewahren, sofern verfügbar, Thread-/Themen-Routing (Slack-Threads, Telegram-Themen, Matrix-Threads).
- Announce-Nachrichten werden auf eine stabile Vorlage normalisiert:
  - `Status:` abgeleitet vom Lauf-Ergebnis (`success`, `error`, `timeout` oder `unknown`).
  - `Result:` der Zusammenfassungsinhalt aus dem Announce-Schritt (oder `(not available)` falls nicht vorhanden).
  - `Notes:` Fehlerdetails und anderer nützlicher Kontext.
- Die Ankündigungsnachricht enthält einen Status, der aus dem Laufzeitergebnis abgeleitet ist (nicht aus der Modellausgabe):

Jede Ankündigung enthält eine Statistikzeile mit:

- Laufzeit (z. B. `runtime 5m12s`)
- Tokenverbrauch (Eingabe/Ausgabe/Gesamt)
- Geschätzte Kosten (wenn die Modellbepreisung über `models.providers.*.models[].cost` konfiguriert ist)
- `sessionKey`, `sessionId` und Transkriptpfad (damit der Main-Agent den Verlauf über `sessions_history` abrufen oder die Datei auf der Festplatte einsehen kann)

## Sub-Agent-Tools anpassen

Um Sub-Agenten auf **nur** bestimmte Tools zu beschränken:

-   
</Accordion>
- `sessions_history`
- `sessions_send`
- Das `sessions_spawn`-Tool

Wenn `maxSpawnDepth >= 2`, erhalten Depth-1-Orchestrator-Sub-Agents zusätzlich `sessions_spawn`, `subagents`, `sessions_list` und `sessions_history`, damit sie ihre Kinder verwalten können.

Überschreiben per Konfiguration:

```json5
45. {
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny gewinnt weiterhin, falls gesetzt
      },
    },
  },
}
```

## Konwährung

Sub-Agents verwenden eine dedizierte In-Process-Queue-Spur:

- Lane-Name: `subagent`
- {
  agents: {
  defaults: {
  subagents: {
  maxConcurrent: 4, // default: 8
  },
  },
  },
  }

## Wird gestoppt

- Das Senden von `/stop` im Requester-Chat bricht die Requester-Sitzung ab und beendet alle aktiven Sub-Agent-Läufe, die daraus gestartet wurden, einschließlich verschachtelter untergeordneter Prozesse.
- Aborts the main session **and** all active sub-agent runs spawned from it

## Einschränkungen

- Die Ankündigung von Sub-Agents erfolgt nach dem **Best-Effort**-Prinzip. - **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- - **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- Der Aufruf ist **nicht blockierend** — der Hauptagent erhält sofort `{ status: "accepted", runId, childSessionKey }` zurück.
- Der Sub-Agent-Kontext injiziert nur `AGENTS.md` + `TOOLS.md` (kein `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` oder `BOOTSTRAP.md`).
- Die maximale Verschachtelungstiefe beträgt 5 (`maxSpawnDepth` Bereich: 1–5). Für die meisten Anwendungsfälle wird Tiefe 2 empfohlen.
- `maxChildrenPerAgent` begrenzt die Anzahl aktiver untergeordneter Prozesse pro Sitzung (Standard: 5, Bereich: 1–20).

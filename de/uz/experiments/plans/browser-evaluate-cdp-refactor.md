---
summary: "Plan: Isolierung von browser act:evaluate aus der Playwright-Warteschlange mittels CDP, mit End-to-End-Deadlines und sicherer Ref-Auflösung"
owner: "openclaw"
status: "Entwurf"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor Plan

## Kontext

`act:evaluate` führt vom Benutzer bereitgestelltes JavaScript in der Seite aus. Derzeit läuft es über Playwright
(`page.evaluate` oder `locator.evaluate`). Playwright serialisiert CDP-Befehle pro Seite, sodass ein
hängendes oder lang laufendes Evaluate die Seiten-Befehlsschlange blockieren und jede spätere Aktion
in diesem Tab als "hängend" erscheinen lassen kann.

PR #13498 fügt ein pragmatisches Sicherheitsnetz hinzu (begrenztes Evaluate, Abort-Weitergabe und Best-Effort-
Wiederherstellung). Dieses Dokument beschreibt ein größeres Refactoring, das `act:evaluate` grundsätzlich
von Playwright isoliert, sodass ein hängendes Evaluate normale Playwright-Operationen nicht blockieren kann.

## Ziele

- `act:evaluate` kann spätere Browser-Aktionen im selben Tab nicht dauerhaft blockieren.
- Timeouts sind durchgängig eine Single Source of Truth, sodass sich ein Aufrufer auf ein Budget verlassen kann.
- Abort und Timeout werden über HTTP und In-Process-Dispatch hinweg gleich behandelt.
- Element-Targeting für Evaluate wird unterstützt, ohne alles von Playwright weg zu migrieren.
- Wahrung der Abwärtskompatibilität für bestehende Aufrufer und Payloads.

## Nicht-Ziele

- Ersetzen aller Browser-Aktionen (Klicken, Tippen, Warten usw.) durch CDP-Implementierungen.
- Entfernung des bestehenden Sicherheitsnetzes aus PR #13498 (es bleibt ein nützlicher Fallback).
- Einführung neuer unsicherer Fähigkeiten über das bestehende `browser.evaluateEnabled`-Gate hinaus.
- Hinzufügen von Prozessisolierung (Worker-Prozess/-Thread) für Evaluate. Falls wir nach diesem Refactoring weiterhin schwer wiederherstellbare
  hängende Zustände sehen, ist das eine Idee für einen Folge-Schritt.

## Aktuelle Architektur (Warum sie hängen bleibt)

Auf hoher Ebene:

- Aufrufer senden `act:evaluate` an den Browser-Steuerungsdienst.
- Der Route-Handler ruft Playwright auf, um das JavaScript auszuführen.
- Playwright serialisiert Seitenbefehle, sodass ein evaluate, das nie abgeschlossen wird, die Warteschlange blockiert.
- Eine blockierte Warteschlange führt dazu, dass spätere Klick-/Eingabe-/Warte-Operationen im Tab scheinbar hängen bleiben.

## Vorgeschlagene Architektur

### 1. Weitergabe von Deadlines

Führen Sie ein einheitliches Budgetkonzept ein und leiten Sie alles davon ab:

- Der Aufrufer setzt `timeoutMs` (oder eine Deadline in der Zukunft).
- Das äußere Request-Timeout, die Logik des Route-Handlers und das Ausführungsbudget innerhalb der Seite
  verwenden alle dasselbe Budget, mit geringem Puffer, wo dies für Serialisierungs-Overhead erforderlich ist.
- Abbruch wird überall als `AbortSignal` weitergegeben, sodass die Stornierung konsistent ist.

Umsetzungsrichtung:

- Fügen Sie einen kleinen Helper hinzu (zum Beispiel `createBudget({ timeoutMs, signal })`), der Folgendes zurückgibt:
  - `signal`: das verknüpfte AbortSignal
  - `deadlineAtMs`: absolute Deadline
  - `remainingMs()`: verbleibendes Budget für untergeordnete Operationen
- Verwenden Sie diesen Helper in:
  - `src/browser/client-fetch.ts` (HTTP- und In-Process-Dispatch)
  - `src/node-host/runner.ts` (Proxy-Pfad)
  - Browser-Aktionsimplementierungen (Playwright und CDP)

### 2. Separater Evaluate-Engine (CDP-Pfad)

Fügen Sie eine CDP-basierte Evaluate-Implementierung hinzu, die sich nicht die per-Page-Befehlswarteschlange von Playwright teilt. Die zentrale Eigenschaft ist, dass der Evaluate-Transport eine separate WebSocket-Verbindung
und eine separate CDP-Session ist, die an das Target angehängt ist.

Umsetzungsrichtung:

- Neues Modul, zum Beispiel `src/browser/cdp-evaluate.ts`, das:
  - Sich mit dem konfigurierten CDP-Endpunkt verbindet (Browser-Level-Socket).
  - `Target.attachToTarget({ targetId, flatten: true })` verwendet, um eine `sessionId` zu erhalten.
  - Entweder Folgendes ausführt:
    - `Runtime.evaluate` für ein Evaluate auf Seitenebene, oder
    - `DOM.resolveNode` plus `Runtime.callFunctionOn` für ein Evaluate auf Elementebene.
  - Bei Timeout oder Abbruch:
    - Sendet bestmöglich `Runtime.terminateExecution` für die Session.
    - Schließt die WebSocket-Verbindung und gibt einen klaren Fehler zurück.

Hinweise:

- Dies führt weiterhin JavaScript auf der Seite aus, sodass die Beendigung Nebenwirkungen haben kann. Der Vorteil
  besteht darin, dass die Playwright-Warteschlange nicht blockiert wird und der Vorgang auf Transportebene durch das Beenden der CDP-Session abgebrochen werden kann.

### 3. Ref-Story (Element-Targeting ohne vollständige Neuschreibung)

Der schwierigste Teil ist das gezielte Ansprechen von Elementen. CDP benötigt ein DOM-Handle oder `backendDOMNodeId`, während
die meisten Browser-Aktionen heute Playwright-Locators verwenden, die auf Refs aus Snapshots basieren.

Empfohlener Ansatz: Bestehende Refs beibehalten, aber optional eine CDP-auflösbare ID anhängen.

#### 3.1 Gespeicherte Ref-Informationen erweitern

Erweitern Sie die gespeicherten Role-Ref-Metadaten, sodass sie optional eine CDP-ID enthalten:

- Heute: `{ role, name, nth }`
- Vorgeschlagen: `{ role, name, nth, backendDOMNodeId?: number }`

Dadurch bleiben alle bestehenden Playwright-basierten Aktionen funktionsfähig und CDP evaluate kann
den gleichen `ref`-Wert verwenden, wenn die `backendDOMNodeId` verfügbar ist.

#### 3.2 backendDOMNodeId zum Zeitpunkt des Snapshots befüllen

Beim Erstellen eines Role-Snapshots:

1. Erzeugen Sie die bestehende Role-Ref-Map wie bisher (role, name, nth).
2. Rufen Sie den AX-Baum über CDP (`Accessibility.getFullAXTree`) ab und berechnen Sie eine parallele Map von
   `(role, name, nth) -> backendDOMNodeId` unter Verwendung derselben Regeln zur Behandlung von Duplikaten.
3. Führen Sie die ID wieder in die gespeicherten Ref-Informationen für den aktuellen Tab ein.

Wenn die Zuordnung für einen Ref fehlschlägt, lassen Sie `backendDOMNodeId` undefiniert. Dadurch wird die Funktion
als Best-Effort-Ansatz umgesetzt und kann sicher ausgerollt werden.

#### 3.3 Evaluate-Verhalten mit Ref

In `act:evaluate`:

- Wenn `ref` vorhanden ist und eine `backendDOMNodeId` hat, führen Sie die Element-Evaluate-Operation über CDP aus.
- Wenn `ref` vorhanden ist, aber keine `backendDOMNodeId` hat, greifen Sie auf den Playwright-Pfad zurück (mit
  dem Sicherheitsnetz).

Optionale Ausweichmöglichkeit:

- Erweitern Sie die Request-Struktur so, dass `backendDOMNodeId` direkt akzeptiert wird (für fortgeschrittene Aufrufer und
  für Debugging), während `ref` die primäre Schnittstelle bleibt.

### 4. Einen letzten Recovery-Pfad beibehalten

Selbst mit CDP evaluate gibt es weitere Möglichkeiten, einen Tab oder eine Verbindung zu blockieren. Behalten Sie die
bestehenden Recovery-Mechanismen (Ausführung beenden + Playwright trennen) als letzten Ausweg
bei für:

- Legacy-Aufrufer
- Umgebungen, in denen das Anhängen von CDP blockiert ist
- Unerwartete Playwright-Randfälle

## Implementierungsplan (Einzelne Iteration)

### Liefergegenstände

- Eine CDP-basierte Evaluate-Engine, die außerhalb der Playwright-Pro-Page-Command-Queue ausgeführt wird.
- Ein einheitliches End-to-End-Timeout-/Abort-Budget, das von Aufrufern und Handlern konsistent verwendet wird.
- Ref-Metadaten, die optional `backendDOMNodeId` für Element-Evaluate enthalten können.
- `act:evaluate` bevorzugt, wenn möglich, die CDP-Engine und greift andernfalls auf Playwright zurück.
- Tests, die belegen, dass ein hängendes Evaluate spätere Aktionen nicht blockiert.
- Logs/Metriken, die Fehler und Fallbacks sichtbar machen.

### Implementierungs-Checkliste

1. Fügen Sie einen gemeinsamen "Budget"-Helper hinzu, um `timeoutMs` + Upstream-`AbortSignal` zu verknüpfen in:
   - ein einzelnes `AbortSignal`
   - eine absolute Deadline
   - eine `remainingMs()`-Hilfsfunktion für nachgelagerte Operationen
2. Aktualisiere alle Aufruferpfade so, dass diese Hilfsfunktion verwendet wird, damit `timeoutMs` überall dasselbe bedeutet:
   - `src/browser/client-fetch.ts` (HTTP- und In-Process-Dispatch)
   - `src/node-host/runner.ts` (Node-Proxy-Pfad)
   - CLI-Wrapper, die `/act` aufrufen (füge `--timeout-ms` zu `browser evaluate` hinzu)
3. Implementiere `src/browser/cdp-evaluate.ts`:
   - Verbindung zum browserweiten CDP-Socket herstellen
   - `Target.attachToTarget`, um eine `sessionId` zu erhalten
   - `Runtime.evaluate` für Page-Evaluate ausführen
   - `DOM.resolveNode` + `Runtime.callFunctionOn` für Element-Evaluate ausführen
   - bei Timeout/Abort: nach Möglichkeit `Runtime.terminateExecution` ausführen und dann den Socket schließen
4. Erweitere die gespeicherten Role-Ref-Metadaten optional um `backendDOMNodeId`:
   - behalte das bestehende `{ role, name, nth }`-Verhalten für Playwright-Aktionen bei
   - füge `backendDOMNodeId?: number` für CDP-Element-Targeting hinzu
5. Fülle `backendDOMNodeId` während der Snapshot-Erstellung (nach Möglichkeit):
   - AX-Tree über CDP abrufen (`Accessibility.getFullAXTree`)
   - `(role, name, nth) -> backendDOMNodeId` berechnen und in die gespeicherte Ref-Map integrieren
   - wenn das Mapping mehrdeutig ist oder fehlt, die ID undefiniert lassen
6. Aktualisiere das Routing von `act:evaluate`:
   - wenn kein `ref`: immer CDP-Evaluate verwenden
   - wenn `ref` zu einer `backendDOMNodeId` aufgelöst wird: CDP-Element-Evaluate verwenden
   - andernfalls: auf Playwright-Evaluate zurückfallen (weiterhin begrenzt und abbrechbar)
7. Behalte den bestehenden "Last-Resort"-Recovery-Pfad als Fallback bei, nicht als Standardpfad.
8. Tests hinzufügen:
   - ein blockierendes Evaluate läuft innerhalb des Budgets in ein Timeout und der nächste Click/Type ist erfolgreich
   - Abort bricht Evaluate ab (Client-Disconnect oder Timeout) und gibt nachfolgende Aktionen frei
   - Mapping-Fehler fallen sauber auf Playwright zurück
9. Observability hinzufügen:
   - Evaluate-Dauer- und Timeout-Zähler
   - terminateExecution-Nutzung
   - Fallback-Rate (CDP -> Playwright) und Gründe

### Akzeptanzkriterien

- Ein absichtlich blockierendes `act:evaluate` kehrt innerhalb des Aufrufer-Budgets zurück und blockiert den Tab
  nicht für nachfolgende Aktionen.
- `timeoutMs` verhält sich konsistent über CLI, Agent-Tool, Node-Proxy und In-Process-Aufrufe hinweg.
- Wenn `ref` einer `backendDOMNodeId` zugeordnet werden kann, verwendet Element-Evaluate CDP; andernfalls ist der
  Fallback-Pfad weiterhin begrenzt und wiederherstellbar.

## Testplan

- Unit-Tests:
  - `(role, name, nth)`-Matching-Logik zwischen Role-Refs und AX-Tree-Knoten.
  - Verhalten der Budget-Hilfsfunktion (Headroom, Remaining-Time-Berechnung).
- Integrationstests:
  - CDP-Evaluate-Timeout kehrt innerhalb des Budgets zurück und blockiert nicht die nächste Aktion.
  - Abort bricht Evaluate ab und löst eine Best-Effort-Terminierung aus.
- Vertragstests:
  - Sicherstellen, dass `BrowserActRequest` und `BrowserActResponse` kompatibel bleiben.

## Risiken und Gegenmaßnahmen

- Mapping ist nicht perfekt:
  - Gegenmaßnahme: Best-Effort-Mapping, Fallback auf Playwright Evaluate und Debug-Tooling hinzufügen.
- `Runtime.terminateExecution` hat Nebenwirkungen:
  - Gegenmaßnahme: nur bei Timeout/Abort verwenden und das Verhalten in Fehlermeldungen dokumentieren.
- Zusätzlicher Overhead:
  - Gegenmaßnahme: AX-Tree nur abrufen, wenn Snapshots angefordert werden, pro Target cachen und
    die CDP-Session kurzlebig halten.
- Einschränkungen des Extension-Relays:
  - Gegenmaßnahme: Browser-Level-Attach-APIs verwenden, wenn keine Per-Page-Sockets verfügbar sind, und
    den aktuellen Playwright-Pfad als Fallback beibehalten.

## Offene Fragen

- Soll die neue Engine als `playwright`, `cdp` oder `auto` konfigurierbar sein?
- Möchten wir ein neues "nodeRef"-Format für fortgeschrittene Nutzer bereitstellen oder nur `ref` beibehalten?
- Wie sollen Frame-Snapshots und selector-scoped Snapshots am AX-Mapping beteiligt werden?

---
summary: "Direkte `openclaw agent`-CLI-Ausführungen (mit optionaler Zustellung)"
read_when:
  - Hinzufügen oder Ändern des Agent-CLI-Einstiegspunkts
title: "Agent senden"
---

# `openclaw agent` (direkte Agent-Ausführungen)

`openclaw agent` führt einen einzelnen Agent-Durchlauf aus, ohne dass eine eingehende Chat-Nachricht erforderlich ist.
Standardmäßig erfolgt die Ausführung **über das Gateway**; fügen Sie `--local` hinzu, um die eingebettete
Laufzeit auf dem aktuellen Rechner zu erzwingen.

## Verhalten

- Erforderlich: `--message <text>`
- Sitzungsauswahl:
  - `--to <dest>` leitet den Sitzungsschlüssel ab (Gruppen-/Kanalziele bewahren die Isolation; Direktchats werden zu `main` zusammengeführt), **oder**
  - `--session-id <id>` verwendet eine bestehende Sitzung anhand der ID erneut, **oder**
  - `--agent <id>` adressiert direkt einen konfigurierten Agenten (verwendet den `main`-Sitzungsschlüssel dieses Agenten)
- Führt dieselbe eingebettete Agent-Laufzeit aus wie normale eingehende Antworten.
- Thinking-/Verbose-Flags werden im Sitzungsspeicher persistiert.
- Ausgabe:
  - Standard: gibt den Antworttext aus (zuzüglich `MEDIA:<url>`-Zeilen)
  - `--json`: gibt strukturierte Nutzlast + Metadaten aus
- Optionale Zustellung zurück an einen Kanal mit `--deliver` + `--channel` (Zielformate entsprechen `openclaw message --target`).
- Verwenden Sie `--reply-channel`/`--reply-to`/`--reply-account`, um die Zustellung zu überschreiben, ohne die Sitzung zu ändern.

Wenn das Gateway nicht erreichbar ist, **fällt** die CLI auf die eingebettete lokale Ausführung zurück.

## Beispiele

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Flags

- `--local`: lokal ausführen (erfordert API-Schlüssel des Modellanbieters in Ihrer Shell)
- `--deliver`: sendet die Antwort an den gewählten Kanal
- `--channel`: Zustellungskanal (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, Standard: `whatsapp`)
- `--reply-to`: Überschreibung des Zustellziels
- `--reply-channel`: Überschreibung des Zustellungskanals
- `--reply-account`: Überschreibung der Zustellungs-Konto-ID
- `--thinking <off|minimal|low|medium|high|xhigh>`: Thinking-Level persistieren (nur GPT-5.2- und Codex-Modelle)
- `--verbose <on|full|off>`: Verbose-Level persistieren
- `--timeout <seconds>`: Agent-Timeout überschreiben
- `--json`: strukturierte JSON-Ausgabe


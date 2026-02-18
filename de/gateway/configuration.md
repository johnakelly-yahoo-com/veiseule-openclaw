---
title: "„Konfiguration“"
---

# Konfiguration

OpenClaw liest eine optionale <Tooltip tip="JSON5 unterstützt Kommentare und nachgestellte Kommata">**JSON5**</Tooltip>-Konfiguration aus `~/.openclaw/openclaw.json`.

Fehlt die Datei, verwendet OpenClaw sichere Standardwerte. Häufige Gründe für eine Konfiguration:

- Kanäle verbinden und steuern, wer dem Bot Nachrichten senden darf
- Modelle, Tools, Sandboxing oder Automatisierung (Cron, Hooks) festlegen
- Sitzungen, Medien, Netzwerk oder UI optimieren

Siehe die [vollständige Referenz](/gateway/configuration-reference) für alle verfügbaren Felder.

<Tip>
**Neu bei der Konfiguration?** Starten Sie mit `openclaw onboard` für eine interaktive Einrichtung oder sehen Sie sich den Leitfaden [Configuration Examples](/gateway/configuration-examples) mit vollständigen Copy‑&‑Paste‑Beispielen an.
</Tip>

## Minimale Konfiguration

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Konfiguration bearbeiten

<Tabs>
  <Tab title="Interaktiver Assistent">
    ```bash
    openclaw onboard       # vollständiger Einrichtungsassistent
    openclaw configure     # Konfigurationsassistent
    ```
  </Tab>
  <Tab title="CLI (Einzeiler)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    Öffnen Sie [http://127.0.0.1:18789](http://127.0.0.1:18789) und verwenden Sie den Tab **Config**.  
    Die Control UI rendert ein Formular aus dem Konfigurationsschema mit einem **Raw JSON**‑Editor als Notausgang.
  </Tab>
  <Tab title="Direkt bearbeiten">
    Bearbeiten Sie `~/.openclaw/openclaw.json` direkt.  
    Das Gateway überwacht die Datei und wendet Änderungen automatisch an (siehe [Hot Reload](#config-hot-reload)).
  </Tab>
</Tabs>

## Strikte Validierung

<Warning>
OpenClaw akzeptiert nur Konfigurationen, die vollständig dem Schema entsprechen. Unbekannte Schlüssel, fehlerhafte Typen oder ungültige Werte führen dazu, dass das Gateway **nicht startet**.  
Die einzige Ausnahme auf Root‑Ebene ist `$schema` (String), damit Editoren JSON‑Schema‑Metadaten anhängen können.
</Warning>

Wenn die Validierung fehlschlägt:

- Das Gateway startet nicht
- Nur Diagnosebefehle funktionieren (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Führen Sie `openclaw doctor` aus, um die genauen Probleme zu sehen
- Führen Sie `openclaw doctor --fix` (oder `--yes`) aus, um Reparaturen anzuwenden

## Häufige Aufgaben

<AccordionGroup>
  <Accordion title="Einen Kanal einrichten (WhatsApp, Telegram, Discord, etc.)">
    Jeder Kanal hat einen eigenen Abschnitt unter `channels.<provider>`.  
    Siehe die jeweilige Kanalseite für Einrichtungsanweisungen:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Alle Kanäle verwenden dasselbe DM‑Policy‑Muster:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // nur für allowlist/open
        },
      },
    }
    ```
  </Accordion>

  <Accordion title="Modelle auswählen und konfigurieren">
    Primäres Modell und optionale Fallbacks festlegen:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` definiert den Modellkatalog und fungiert als Allowlist für `/model`.
    - Modellreferenzen verwenden das Format `provider/model` (z. B. `anthropic/claude-opus-4-6`).
    - Siehe [Models CLI](/concepts/models) für Modellwechsel im Chat und [Model Failover](/concepts/model-failover) für Auth‑Rotation und Fallback‑Verhalten.
    - Für benutzerdefinierte/self‑hosted Provider siehe [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls).

  </Accordion>

  <Accordion title="Steuern, wer dem Bot Nachrichten senden darf">
    DM‑Zugriff wird pro Kanal über `dmPolicy` gesteuert:

    - `"pairing"` (Standard): Unbekannte Absender erhalten einen einmaligen Pairing‑Code
    - `"allowlist"`: nur Absender in `allowFrom`
    - `"open"`: alle eingehenden DMs erlauben (erfordert `allowFrom: ["*"]`)
    - `"disabled"`: alle DMs ignorieren

    Für Gruppen verwenden Sie `groupPolicy` + `groupAllowFrom` oder kanalspezifische Allowlists.

    Siehe die [vollständige Referenz](/gateway/configuration-reference#dm-and-group-access).

  </Accordion>

  <Accordion title="Gruppen‑Mention‑Gating konfigurieren">
    Gruppennachrichten erfordern standardmäßig eine **Erwähnung**. Konfiguration pro Agent:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Metadaten‑Erwähnungen**: native @‑Mentions (WhatsApp Tap‑to‑mention, Telegram @bot, etc.)
    - **Textmuster**: Regex‑Patterns in `mentionPatterns`
    - Siehe [vollständige Referenz](/gateway/configuration-reference#group-chat-mention-gating).

  </Accordion>

  <Accordion title="Sitzungen und Resets konfigurieren">
    Sitzungen steuern Gesprächskontinuität und Isolation:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Siehe [Session Management](/concepts/session) und die [vollständige Referenz](/gateway/configuration-reference#session).

  </Accordion>

  <Accordion title="Sandboxing aktivieren">
    Agent‑Sitzungen in isolierten Docker‑Containern ausführen:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    Image einmalig bauen: `scripts/sandbox-setup.sh`

    Siehe [Sandboxing](/gateway/sandboxing).

  </Accordion>

  <Accordion title="Heartbeat (periodische Check‑ins) einrichten">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: Dauer (`30m`, `2h`), `0m` deaktiviert
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Siehe [Heartbeat](/gateway/heartbeat).

  </Accordion>

  <Accordion title="Cron‑Jobs konfigurieren">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    Siehe [Cron jobs](/automation/cron-jobs).

  </Accordion>

  <Accordion title="Webhooks (Hooks) einrichten">
    HTTP‑Webhook‑Endpunkte aktivieren:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    Siehe die [vollständige Referenz](/gateway/configuration-reference#hooks).

  </Accordion>

  <Accordion title="Multi‑Agent‑Routing konfigurieren">
    Mehrere isolierte Agenten mit separaten Workspaces:

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    Siehe [Multi-Agent](/concepts/multi-agent).

  </Accordion>

  <Accordion title="Konfiguration in mehrere Dateien aufteilen ($include)">
    `$include` verwenden, um große Konfigurationen zu organisieren:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **Einzelne Datei**: ersetzt das Objekt
    - **Array von Dateien**: Deep‑Merge in Reihenfolge
    - **Geschwister‑Schlüssel**: überschreiben inkludierte Werte
    - **Verschachtelte Includes**: bis zu 10 Ebenen
    - **Relative Pfade**: relativ zur inkludierenden Datei
    - **Fehlerbehandlung**: klare Fehlermeldungen

  </Accordion>
</AccordionGroup>

## Config Hot Reload

Das Gateway überwacht `~/.openclaw/openclaw.json` und wendet Änderungen automatisch an — kein manueller Neustart für die meisten Einstellungen erforderlich.

### Reload‑Modi

| Modus                   | Verhalten                                                                 |
| ----------------------- | ------------------------------------------------------------------------- |
| **`hybrid`** (Standard) | Sichere Änderungen werden hot‑applied, kritische führen zu Neustart.     |
| **`hot`**               | Nur sichere Änderungen; Neustart muss manuell erfolgen.                   |
| **`restart`**           | Neustart bei jeder Änderung.                                              |
| **`off`**               | Dateiüberwachung deaktiviert.                                             |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

<Note>
`gateway.reload` und `gateway.remote` lösen **keinen** Neustart aus.
</Note>

## Environment‑Variablen

OpenClaw liest Umgebungsvariablen aus dem Elternprozess sowie:

- `.env` im aktuellen Arbeitsverzeichnis
- `~/.openclaw/.env` (globaler Fallback)

Keine Datei überschreibt bestehende Variablen.

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell‑Env‑Import (optional)">
Falls aktiviert und erwartete Schlüssel fehlen, startet OpenClaw Ihre Login‑Shell und importiert nur fehlende Variablen:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env‑Var‑Äquivalent: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Env‑Var‑Substitution in Konfigurationswerten">
Referenzieren Sie Variablen mit `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Regeln:

- Nur Großbuchstaben: `[A-Z_][A-Z0-9_]*`
- Fehlende/leere Variablen verursachen Fehler beim Laden
- Mit `$${VAR}` escapen
- Funktioniert auch in `$include`‑Dateien
- Inline‑Substitution möglich (`"${BASE}/v1"`)

</Accordion>

Siehe [Environment](/help/environment).

## Vollständige Referenz

Für die komplette Feld‑für‑Feld‑Dokumentation siehe **[Configuration Reference](/gateway/configuration-reference)**.

---

_Verwandt: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_

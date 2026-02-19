---
summary: "Konfigurationsübersicht: häufige Aufgaben, Schnellstart und Links zur vollständigen Referenz"
read_when:
  - OpenClaw zum ersten Mal einrichten
  - Suche nach gängigen Konfigurationsmustern
  - Zu bestimmten Konfigurationsabschnitten navigieren
title: "„Konfiguration“"
---

# Konfiguration 🔧

OpenClaw liest eine optionale **JSON5**-Konfiguration aus `~/.openclaw/openclaw.json` (Kommentare + nachgestellte Kommata erlaubt).

Fehlt die Datei, verwendet OpenClaw sichere Standardwerte (eingebetteter Pi‑Agent + Sitzungen pro Absender + Workspace `~/.openclaw/workspace`). In der Regel benötigen Sie eine Konfiguration nur, um:

- Kanäle verbinden und steuern, wer dem Bot Nachrichten senden darf
- Modelle, Tools, Sandboxing oder Automatisierung (cron, Hooks) festlegen
- Sitzungen, Medien, Netzwerk oder UI optimieren

Siehe die [vollständige Referenz](/gateway/configuration-reference) für alle verfügbaren Felder.

<Tip>
„Alle Konfigurationsoptionen für ~/.openclaw/openclaw.json mit Beispielen“
</Tip>

## Minimale Konfiguration

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Konfiguration bearbeiten

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # vollständiger Einrichtungsassistent
    openclaw configure     # Konfigurationsassistent
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    Das Gateway stellt eine JSON-Schema-Darstellung der Konfiguration über `config.schema` für UI-Editoren bereit.
    Die Control UI rendert aus diesem Schema ein Formular, mit einem **Raw JSON**-Editor als Notausgang.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`) Das Gateway überwacht die Datei und wendet Änderungen automatisch an (siehe [Hot Reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Strikte Konfigurationsvalidierung

<Warning>
OpenClaw akzeptiert nur Konfigurationen, die vollständig dem Schema entsprechen. Unbekannte Schlüssel, fehlerhafte Typen oder ungültige Werte führen dazu, dass das Gateway aus Sicherheitsgründen **nicht startet**. Die einzige Ausnahme auf Root-Ebene ist `$schema` (String), damit Editoren JSON-Schema-Metadaten anhängen können.
</Warning>

Wenn die Validierung fehlschlägt:

- Das Gateway startet nicht.
- Es sind nur Diagnosebefehle erlaubt (z. B.: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Führen Sie `openclaw doctor` aus, um die genauen Probleme zu sehen.
- Führen Sie `openclaw doctor --fix` (oder `--yes`) aus, um Migrationen/Reparaturen anzuwenden.

## Häufige Aufgaben

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Jeder Kanal hat seinen eigenen Konfigurationsabschnitt unter `channels.<provider>`. Siehe die jeweilige Kanalseite für Einrichtungsschritte:

    ```
    {
      Kanäle: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowVon: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        Signal: {
          groupPolicy: "allowlist",
          groupAllowVon: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groupAllowVrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org. om"],
        },
        discord: {
          groupPolicy: "allowlist",
          Gilden: {
            GUILD_ID: {
              Kanäle: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          Kanäle: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">
    Primäres Modell und optionale Fallbacks festlegen:

    ```
    Wenn `models.providers` vorhanden ist, schreibt OpenClaw eine `models.json` in
    `~/.openclaw/agents/<agentId>/agent/` beim Start:
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">
    DM-Zugriff wird pro Kanal über `dmPolicy` gesteuert:

    ```
    Gruppen: `channels.mattermost.groupPolicy="allowlist"` standardmäßig (mention-gated). Benutze `channels.mattermost.groupAllowFrom` um Absender zu beschränken.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Gruppieren von Nachrichten standardmäßig **Erwähnung benötigen** (entweder Metadaten Erwähnung oder Regex Pattern). `agents.defaults.subagents` konfiguriert Subagent Standards:

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    Sitzungen steuern Gesprächskontinuität und Isolation:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // empfohlen für Mehrbenutzerumgebungen
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (gemeinsam) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Siehe [Session Management](/concepts/session) für Geltungsbereiche, Identitätsverknüpfungen und Senderegeln.
    - Siehe [full reference](/gateway/configuration-reference#session) für alle Felder.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    Agent-Sitzungen in isolierten Docker-Containern ausführen:

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789, auth: { token: "secret" } },
    
      // Common agent defaults
      agents: {
        defaults: {
          sandbox: { mode: "all", scope: "session" },
        },
        // Merge agent lists from all clients
        list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
      },
    
      // Merge broadcast configs
      broadcast: {
        $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
      },
    
      channels: { whatsapp: { groupPolicy: "allowlist" } },
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
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

    ```
    - `every`: Dauer-String (`30m`, `2h`). Mit `0m` deaktivieren.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Siehe [Heartbeat](/gateway/heartbeat) für die vollständige Anleitung.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    Siehe [Cron Jobs](/automation/cron-jobs) für die Funktionsübersicht und CLI Beispiele.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Aktivieren Sie einen einfachen HTTP-Webhook-Endpunkt auf dem Gateway HTTP-Server.

    ````
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
    
    Siehe [full reference](/gateway/configuration-reference#hooks) für alle Mapping-Optionen und die Gmail-Integration.
    ````

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Führen Sie mehrere isolierte Agenten (separater Arbeitsbereich, `agentDir`, Sitzungen) innerhalb eines Gateways aus.

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Teilen Sie Ihre Konfiguration mithilfe der Direktive `$include` in mehrere Dateien auf. Dies ist nützlich für:

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (Config hot reload)

Der Gateway überwacht `~/.openclaw/openclaw.json` und übernimmt Änderungen automatisch — für die meisten Einstellungen ist kein manueller Neustart erforderlich.

### Neulademodi

| Modus:                       | Verhalten                                                                                                                                                                                      |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (Standard)   | Übernimmt sichere Änderungen sofort (Hot-Apply). Startet bei kritischen Änderungen automatisch neu.                                         |
| **`hot`**                                    | Übernimmt nur sichere Änderungen (Hot-Apply). Protokolliert eine Warnung, wenn ein Neustart erforderlich ist — Sie führen ihn selbst durch. |
| Anwenden + Neustart (RPC) | `restart`: Starten Sie das Gateway bei jeder Konfigurationsänderung neu.                                                                                       |
| **Regeln:**                  | Deaktiviert die Dateiüberwachung. Änderungen werden beim nächsten manuellen Neustart wirksam.                                                                  |

```json5
{
  Gateway: {
    reload: {
      Modus: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### Was wird per Hot-Apply übernommen und was erfordert einen Neustart

Die meisten Felder werden ohne Ausfallzeit per Hot-Apply übernommen. Im `hybrid`-Modus werden Änderungen, die einen Neustart erfordern, automatisch verarbeitet.

| Kategorie                            | Felder:                                                                                         | Neustart erforderlich?                   |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — alle integrierten und Erweiterungskanäle                    | Nein                                     |
| Agent & Modelle  | `model`: Standardmodell, überschreibt `agents.defaults.model` für diesen Agent. | Nein                                     |
| Automatisierung                      | `hooks`, `cron`, `agent.heartbeat`                                                                              | Nein                                     |
| messages                             | `messages.queue`                                                                                                | Nein                                     |
| Tools & Medien   | `tools`, `browser`, `skills`, `audio`, `talk`                                                                   | Nein                                     |
| Schema- und UI-Hinweise              | `ui`, `logging`, `identity`, `bindings`                                                                         | Nein                                     |
| Gateway-Server                       | `gateway` (port/bind/auth/control UI/tailscale)                                              | **Inline‑Substitution:** |
| Infrastruktur                        | `discovery`, `canvasHost`, `plugins`                                                                            | **Erwähnungstypen:**     |

<Note>
`gateway.reload` und `gateway.remote` sind Ausnahmen — deren Änderung löst **keinen** Neustart aus.
</Note>

## Partielle Updates (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Verwenden Sie `config.apply`, um die vollständige Konfiguration in einem Schritt zu validieren, zu schreiben und das Gateway neu zu starten.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">Verwenden Sie `config.patch`, um ein partielles Update in die bestehende Konfiguration zu mergen, ohne
unverwandte Schlüssel zu überschreiben. Es gelten die Semantiken von JSON Merge Patch:

    ````
    - Objekte werden rekursiv zusammengeführt
    - `null` löscht einen Schlüssel
    - Arrays werden ersetzt
    
    Parameter:
    
    - `raw` (string) — JSON5 nur mit den zu ändernden Schlüsseln
    - `baseHash` (erforderlich) — Konfigurations-Hash von `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — wie bei `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Variable

OpenClaw liest Umgebungsvariablen aus dem übergeordneten Prozess sowie zusätzlich:

- `.env` aus dem aktuellen Arbeitsverzeichnis (falls vorhanden)
- eine globale Fallback‑Datei `.env` aus `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Keine der `.env`‑Dateien überschreibt bestehende Umgebungsvariablen. Sie können Umgebungsvariablen auch inline in der Konfiguration angeben.

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

<Accordion title="Shell env import (optional)">Opt‑in‑Komfortfunktion: Wenn aktiviert und noch keiner der erwarteten Schlüssel gesetzt ist,
führt OpenClaw Ihre Login‑Shell aus und importiert nur die fehlenden erwarteten Schlüssel (überschreibt nie).

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">Sie können Umgebungsvariablen direkt in jedem String‑Wert der Konfiguration mit der Syntax
`${VAR_NAME}` referenzieren.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

Standards:

- Es werden nur großgeschriebene Umgebungsvariablennamen gematcht: `[A-Z_][A-Z0-9_]*`
- Fehlende/leere Variablen führen beim Laden zu einem Fehler
- Mit `$${VAR}` escapen, um ein literales `${VAR}` auszugeben
- Funktioniert mit `$include` (auch eingebundene Dateien erhalten Substitution)
- Inline-Substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Siehe [/environment](/help/environment) für vollständige Prioritäten und Quellen.

## Vollständige Referenz

**Neu bei der Konfiguration?** Sehen Sie sich den Leitfaden [Configuration Examples](/gateway/configuration-examples) mit vollständigen Beispielen und detaillierten Erläuterungen an!

---

Beispiel (Provider/modellspezifische Erlaubnisliste):

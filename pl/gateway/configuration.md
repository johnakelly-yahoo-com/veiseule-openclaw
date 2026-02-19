---
summary: "Przegląd konfiguracji: typowe zadania, szybka konfiguracja i linki do pełnej dokumentacji"
read_when:
  - Pierwsza konfiguracja OpenClaw
  - Szukasz typowych wzorców konfiguracji
  - Przechodzenie do konkretnych sekcji konfiguracji
title: "„Konfiguracja”"
---

# Konfiguracja 🔧

OpenClaw odczytuje opcjonalną konfigurację **JSON5** z pliku `~/.openclaw/openclaw.json` (dozwolone są komentarze i przecinki na końcu).

Jeśli plik nie istnieje, OpenClaw używa bezpiecznych (w miarę) ustawień domyślnych (wbudowany agent Pi + sesje per nadawca + obszar roboczy `~/.openclaw/workspace`). Zwykle konfiguracja jest potrzebna tylko po to, aby:

- Podłącz kanały i kontroluj, kto może wysyłać wiadomości do bota
- Ustaw modele, narzędzia, sandboxing lub automatyzację (cron, hooki)
- Dostosuj sesje, media, sieć lub interfejs użytkownika

Zobacz [pełną dokumentację](/gateway/configuration-reference), aby poznać wszystkie dostępne pola.

<Tip>
„Wszystkie opcje konfiguracji dla ~/.openclaw/openclaw.json wraz z przykładami”
</Tip>

## Minimalna konfiguracja (zalecany punkt startowy)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Edytowanie konfiguracji

<Tabs>
  <Tab title="Interactive wizard">
```bash
openclaw onboard       # full setup wizard
openclaw configure     # config wizard
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
    Gateway udostępnia reprezentację JSON Schema konfiguracji poprzez `config.schema` dla edytorów UI.
    Control UI renderuje formularz na podstawie tego schematu, z edytorem **Raw JSON** jako wyjściem awaryjnym.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (lub `OPENCLAW_CONFIG_PATH`) Gateway monitoruje plik i automatycznie stosuje zmiany (zobacz [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Ścisła walidacja

<Warning>
OpenClaw akceptuje wyłącznie konfiguracje, które w pełni odpowiadają schematowi. Nieznane klucze, błędne typy lub nieprawidłowe wartości powodują, że Gateway **odmawia uruchomienia** ze względów bezpieczeństwa. Jedynym wyjątkiem na poziomie głównym jest `$schema` (string), aby edytory mogły dołączać metadane JSON Schema.
</Warning>

Gdy walidacja zakończy się niepowodzeniem:

- Gateway nie startuje.
- Dozwolone są wyłącznie polecenia diagnostyczne (na przykład: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Uruchom `openclaw doctor`, aby zobaczyć dokładne problemy.
- Uruchom `openclaw doctor --fix` (lub `--yes`), aby zastosować migracje/naprawy.

## Typowe zadania

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Każdy kanał ma własną sekcję konfiguracji w `channels.<provider>`. Zobacz dedykowaną stronę kanału, aby poznać kroki konfiguracji:

    ```
    {
      kanałów: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFod: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowFod: ["+15551234567"],
        },
        imesage: {
          groupPolicy: "allowlist",
          groupAllowFod: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org. om"],
        },
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            GUILD_ID: {
              kanałów: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          kanały: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">
    Ustaw model główny oraz opcjonalne modele zapasowe:

    ```
    {
      agents: {
        domyślnie: {
          cliBackends: {
            "claude-cli": {
              command: "/opt/homebrew/bin/claude",
            },
            "my-cli": {
              command: "my-cli",
              args: ["--json"], Wyjście
              : „json”,
              modelArg: "--model", sesja
              Arg: "--session", Tryb sesyjny
              : "istnieje",
              systemPromptArg: "--system",
              systemPromptWhen: "first",
              imageArg: "--image", Tryb obrazka
              : "repeat",
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">
    Dostęp do wiadomości prywatnych (DM) jest kontrolowany osobno dla każdego kanału za pomocą `dmPolicy`:

    ```
    - `"pairing"` (domyślnie): nieznani nadawcy otrzymują jednorazowy kod parowania do zatwierdzenia
    - `"allowlist"`: tylko nadawcy z `allowFrom` (lub z zapisanej listy sparowanych)
    - `"open"`: zezwól na wszystkie przychodzące wiadomości prywatne (wymaga `allowFrom: ["*"]`)
    - `"disabled"`: ignoruj wszystkie wiadomości prywatne
    
    Dla grup użyj `groupPolicy` + `groupAllowFrom` lub list dozwolonych specyficznych dla kanału.
    
    Zobacz [pełną dokumentację](/gateway/configuration-reference#dm-and-group-access), aby poznać szczegóły dla poszczególnych kanałów.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Grupuj wiadomości domyślne do **wymaga wzmianki** (albo wspomnienie o metadanych albo wzory regex). `agents.defaults.subagents` konfiguruje domyślne subagenta:

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
    Sesje kontrolują ciągłość i izolację rozmów:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // zalecane dla wielu użytkowników
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (współdzielona) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Zobacz [Zarządzanie sesją](/concepts/session), aby poznać zakresy, powiązania tożsamości i zasady wysyłania.
    - Zobacz [pełną dokumentację](/gateway/configuration-reference#session), aby poznać wszystkie pola.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    Uruchamiaj sesje agentów w izolowanych kontenerach Docker:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
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
    - `every`: czas trwania (`30m`, `2h`). Ustaw `0m`, aby wyłączyć.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Zobacz [Heartbeat](/gateway/heartbeat), aby zapoznać się z pełnym przewodnikiem.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    Zobacz [Cron jobs](/automation/cron-jobs), aby uzyskać przegląd funkcji i przykłady CLI.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Włącz prosty punkt końcowy HTTP na serwerze HTTP bramy.

    ```
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        ścieżka: "/hooks",
        : ["gmail"],
        transformsDir: "~/. ołówek/haczyki”,
        mapowanie: [
          {
            dopasowanie: { path: "gmail" }, Działanie
            : „agent”,
            wakeMode: "teraz",
            nazwa: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "Od: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
            dostawy: true,
            channel: "last", Model
            : "openai/gpt-5. -mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">
    Uruchamiaj wiele izolowanych agentów z oddzielnymi przestrzeniami roboczymi i sesjami:

    ```
    {
      agents: {
        list: [
          { id: "home", domyślnie: true, obszar roboczy: "~/. penclaw/workspace-home" },
          { id: "work", workspace: "~/. penclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", pasuje: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", ID konta: "biz" } },
      ],
      kanałów: {
        whatsapp: {
          accounts: {
            personal: {},
            biz: {},
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Podziel konfigurację na wiele plików, używając dyrektywy `$include`. Jest to przydatne do:

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

## `gateway.reload` (konfig hot reload)

Gateway monitoruje `~/.openclaw/openclaw.json` i automatycznie stosuje zmiany — w przypadku większości ustawień nie jest wymagany ręczny restart.

### Tryby przeładowania

| Tryb czatu:                | Scal zachowanie                                                                                                                                                |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (domyślny) | Natychmiast stosuje bezpieczne zmiany. Automatycznie uruchamia ponownie przy zmianach krytycznych.                             |
| **`hot`**                                  | Stosuje natychmiast tylko bezpieczne zmiany. Zapisuje ostrzeżenie w logach, gdy wymagany jest restart — wykonujesz go ręcznie. |
| **`restart`**                              | Restartuje Gateway przy każdej zmianie konfiguracji, bez względu na jej rodzaj.                                                                |
| **Podstawianie inline:**   | Wyłącza monitorowanie pliku. Zmiany zaczynają obowiązywać przy następnym ręcznym restarcie.                                    |

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### Co jest stosowane natychmiast, a co wymaga restartu

Większość pól jest stosowana natychmiast bez przestoju. W trybie `hybrid` zmiany wymagające restartu są obsługiwane automatycznie.

| Kategoria                            | Pola                                                                                         | Wymagany restart?                        |
| ------------------------------------ | -------------------------------------------------------------------------------------------- | ---------------------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — wszystkie wbudowane i rozszerzone kanały | Nie                                      |
| Agent i modele                       | `agent`, `agents`, `models`, `routing`                                                       | Nie                                      |
| Automatyzacja                        | `hooks`, `cron`, `agent.heartbeat`                                                           | Nie                                      |
| messages                             | `messages.inbound`                                                                           | Nie                                      |
| Narzędzia i media                    | `tools`, `browser`, `skills`, `audio`, `talk`                                                | Nie                                      |
| UI i różne                           | `ui`, `logging`, `identity`, `bindings`                                                      | Nie                                      |
| Serwer:              | `gateway` (port/bind/auth/control UI/tailscale)                           | **Zasady:**              |
| Infrastruktura                       | `discovery`, `canvasHost`, `plugins`                                                         | **Rodzaje wspominania:** |

<Note>
`gateway.reload` i `gateway.remote` są wyjątkami — ich zmiana **nie** powoduje restartu.
</Note>

## Aktualizacje częściowe (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Użyj `config.apply`, aby zweryfikować i zapisać pełną konfigurację oraz zrestartować Gateway w jednym kroku.

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

  <Accordion title="config.patch (partial update)">Użyj `config.patch`, aby scalić częściową aktualizację z istniejącą konfiguracją bez nadpisywania
niepowiązanych kluczy. Stosowane są semantyki JSON merge patch:

    ````
    - Obiekty są scalane rekurencyjnie
    - `null` usuwa klucz
    - Tablice są zastępowane
    
    Parametry:
    
    - `raw` (string) — JSON5 zawierający tylko klucze do zmiany
    - `baseHash` (wymagane) — hash konfiguracji z `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — takie same jak w `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Zmienne środowiskowe

OpenClaw odczytuje zmienne środowiskowe z procesu nadrzędnego oraz:

- `.env` z bieżącego katalogu roboczego (jeśli istnieje)
- globalny fallback `.env` z `~/.openclaw/.env` (czyli `$OPENCLAW_STATE_DIR/.env`)

Żaden plik `.env` nie nadpisuje istniejących zmiennych środowiskowych. Możesz także podać zmienne środowiskowe inline w konfiguracji.

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

<Accordion title="Shell env import (optional)">Opcjonalne ułatwienie: jeśli włączone i żaden z oczekiwanych kluczy nie jest jeszcze ustawiony,
OpenClaw uruchamia powłokę logowania użytkownika i importuje wyłącznie brakujące oczekiwane klucze
(nigdy nie nadpisuje).

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

<Accordion title="Env var substitution in config values">Możesz bezpośrednio odwoływać się do zmiennych środowiskowych w dowolnej wartości string
konfiguracji, używając składni `${VAR_NAME}`.

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

Zasady:

- Dopasowywane są tylko nazwy zmiennych zapisane wielkimi literami: `[A-Z_][A-Z0-9_]*`
- Brakujące lub puste zmienne powodują błąd podczas ładowania konfiguracji
- Użyj `$${VAR}`, aby wypisać dosłowny `${VAR}`
- Działa z `$include` (dołączane pliki również podlegają podstawianiu)
- Podstawianie inline: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Zobacz [/environment](/help/environment), aby poznać pełną kolejność i źródła.

## Pełna referencja

**Nowy w konfiguracji?** Zobacz przewodnik [Configuration Examples](/gateway/configuration-examples), aby zapoznać się z kompletnymi przykładami wraz ze szczegółowymi wyjaśnieniami!

---

Przykład: konfiguracja prawna dla wielu klientów


---
title: "Dokumentacja konfiguracji"
description: "Kompletny opis wszystkich pól w ~/.openclaw/openclaw.json"
---

# Dokumentacja konfiguracji

Wszystkie pola dostępne w `~/.openclaw/openclaw.json`. Aby zobaczyć przegląd zorientowany na zadania, zobacz [Configuration](/gateway/configuration).

Format konfiguracji to **JSON5** (dozwolone komentarze + przecinki końcowe). Wszystkie pola są opcjonalne — OpenClaw używa bezpiecznych ustawień domyślnych, gdy zostaną pominięte.

---

## Kanały

Każdy kanał uruchamia się automatycznie, gdy istnieje jego sekcja konfiguracji (chyba że `enabled: false`).

### Dostęp do DM i grup

Wszystkie kanały obsługują polityki DM oraz polityki grupowe:

| Polityka DM                              | Zachowanie                                                                           |
| ---------------------------------------- | ------------------------------------------------------------------------------------ |
| `pairing` (domyślnie) | Nieznani nadawcy otrzymują jednorazowy kod parowania; właściciel musi go zatwierdzić |
| `allowlist`                              | Tylko nadawcy z `allowFrom` (lub ze sparowanej listy dozwolonych) |
| `open`                                   | Zezwalaj na wszystkie przychodzące DM (wymaga `allowFrom: ["*"]`) |
| `disabled`                               | Ignoruj wszystkie przychodzące DM                                                    |

| Polityka grupowa                           | Zachowanie                                                                            |
| ------------------------------------------ | ------------------------------------------------------------------------------------- |
| `allowlist` (domyślnie) | Tylko grupy pasujące do skonfigurowanej listy dozwolonych                             |
| `open`                                     | Pomiń listy dozwolonych dla grup (nadal obowiązuje wymóg wzmianki) |
| `disabled`                                 | Blokuj wszystkie wiadomości w grupach/pokojach                                        |

<Note>
`channels.defaults.groupPolicy` ustawia wartość domyślną, gdy `groupPolicy` dostawcy nie jest ustawione.
Kody parowania wygasają po 1 godzinie. Oczekujące żądania parowania DM są ograniczone do **3 na kanał**.
Slack/Discord mają specjalny mechanizm zapasowy: jeśli ich sekcja dostawcy całkowicie nie istnieje, polityka grupowa w czasie działania może zostać ustawiona na `open` (z ostrzeżeniem przy starcie).
</Note>

### WhatsApp

WhatsApp działa przez kanał webowy gateway (Baileys Web). Uruchamia się automatycznie, gdy istnieje powiązana sesja.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blue ticks (false in self-chat mode)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- Polecenia wychodzące domyślnie używają konta `default`, jeśli istnieje; w przeciwnym razie pierwszego skonfigurowanego identyfikatora konta (posortowanego).
- Starszy katalog uwierzytelniania Baileys dla jednego konta jest migrowany przez `openclaw doctor` do `whatsapp/default`.
- Nadpisania per konto: `channels.whatsapp.accounts.<id>.sendReadReceipts`, `channels.whatsapp.accounts.<id>.dmPolicy`, `channels.whatsapp.accounts.<id>.allowFrom`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Token bota: `channels.telegram.botToken` lub `channels.telegram.tokenFile`, z `TELEGRAM_BOT_TOKEN` jako wartością zapasową dla domyślnego konta.
- `configWrites: false` blokuje zapisy konfiguracji inicjowane z Telegrama (migracje ID supergrupy, `/config set|unset`).
- Podgląd strumienia Telegram używa `sendMessage` + `editMessageText` (działa w czatach prywatnych i grupowych).
- Polityka ponowień: zobacz [Retry policy](/concepts/retry).

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Tylko krótkie odpowiedzi.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- Token: `channels.discord.token`, z `DISCORD_BOT_TOKEN` jako wartością zapasową dla domyślnego konta.
- Użyj `user:<id>` (DM) lub `channel:<id>` (kanał guild) jako celów dostarczania; same numeryczne ID są odrzucane.
- Slugi guild są zapisywane małymi literami, a spacje zastępowane `-`; klucze kanałów używają nazwy w formie sluga (bez `#`). Preferuj identyfikatory guild.
- Wiadomości autorstwa bota są domyślnie ignorowane. `allowBots: true` włącza ich obsługę (własne wiadomości nadal są filtrowane).
- `maxLinesPerMessage` (domyślnie 17) dzieli długie wiadomości nawet wtedy, gdy mają mniej niż 2000 znaków.
- `channels.discord.ui.components.accentColor` ustawia kolor akcentu dla kontenerów komponentów Discord v2.

**Tryby powiadomień o reakcjach:** `off` (brak), `own` (wiadomości bota, domyślnie), `all` (wszystkie wiadomości), `allowlist` (z `guilds.<id>`.users\` dla wszystkich wiadomości).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- JSON konta usługi: osadzone (`serviceAccount`) lub z pliku (`serviceAccountFile`).
- Zmienne środowiskowe jako fallback: `GOOGLE_CHAT_SERVICE_ACCOUNT` lub `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Użyj `spaces/<spaceId>` lub `users/<userId|email>` jako celów dostarczania.

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Tylko krótkie odpowiedzi.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **Tryb Socket** wymaga zarówno `botToken`, jak i `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` jako domyślny fallback ze zmiennych środowiskowych).
- **Tryb HTTP** wymaga `botToken` oraz `signingSecret` (na poziomie głównym lub per konto).
- `configWrites: false` blokuje zapisy konfiguracji inicjowane ze Slacka.
- Użyj `user:<id>` (DM) lub `channel:<id>` jako celów dostarczania.

**Tryby powiadomień o reakcjach:** `off`, `own` (domyślnie), `all`, `allowlist` (z `reactionAllowlist`).

**Izolacja sesji w wątkach:** `thread.historyScope` jest per wątek (domyślnie) lub współdzielone w całym kanale. `thread.inheritParent` kopiuje transkrypcję kanału nadrzędnego do nowych wątków.

| Grupa akcji | Domyślne | Uwagi                            |
| ----------- | -------- | -------------------------------- |
| reactions   | włączone | Reagowanie + lista reakcji       |
| messages    | włączone | Odczyt/wysyłanie/edycja/usuwanie |
| pins        | włączone | Przypnij/odepnij/lista           |
| memberInfo  | włączone | Informacje o członku             |
| emojiList   | włączone | Niestandardowa lista emoji       |

### Mattermost

Mattermost jest dostarczany jako wtyczka: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Tryby czatu: `oncall` (odpowiada na wzmiankę @, domyślnie), `onmessage` (na każdą wiadomość), `onchar` (wiadomości zaczynające się od prefiksu wyzwalającego).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Tryby powiadomień o reakcjach:** `off`, `own` (domyślnie), `all`, `allowlist` (z `reactionAllowlist`).

### iMessage

OpenClaw uruchamia `imsg rpc` (JSON-RPC przez stdio). Nie jest wymagany żaden demon ani port.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- Wymaga pełnego dostępu do dysku dla bazy danych Messages.
- Preferuj cele `chat_id:<id>`. Użyj `imsg chats --limit 20`, aby wyświetlić listę czatów.
- `cliPath` może wskazywać na wrapper SSH; ustaw `remoteHost` do pobierania załączników przez SCP.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Wiele kont (wszystkie kanały)

Uruchamiaj wiele kont w ramach jednego kanału (każde z własnym `accountId`):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- `default` jest używane, gdy `accountId` jest pominięte (CLI + routing).
- Tokeny z Env mają zastosowanie tylko do konta **default**.
- Podstawowe ustawienia kanału mają zastosowanie do wszystkich kont, chyba że zostaną nadpisane dla konkretnego konta.
- Użyj `bindings[].match.accountId`, aby kierować każde konto do innego agenta.

### Wymuszanie wzmianki w czatach grupowych

Wiadomości grupowe domyślnie **wymagają wzmianki** (wzmianka w metadanych lub wzorce regex). Dotyczy czatów grupowych w WhatsApp, Telegram, Discord, Google Chat oraz iMessage.

**Typy wzmianek:**

- **Wzmianki w metadanych**: Natywne wzmianki @ na platformie. Ignorowane w trybie czatu własnego WhatsApp.
- **Wzorce tekstowe**: Wzorce regex w `agents.list[].groupChat.mentionPatterns`. Zawsze sprawdzane.
- Wymuszanie wzmianki jest stosowane tylko wtedy, gdy wykrycie jest możliwe (wzmianki natywne lub co najmniej jeden wzorzec).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` ustawia globalną wartość domyślną. Kanały mogą nadpisać ustawienie za pomocą `channels.<channel> .historyLimit` (lub per-konto). Ustaw `0`, aby wyłączyć.

#### Limity historii DM

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

Rozstrzyganie: nadpisanie per-DM → domyślne dostawcy → brak limitu (wszystko zachowane).

Obsługiwane: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Tryb czatu z samym sobą

Dodaj własny numer w `allowFrom`, aby włączyć tryb self-chat (ignoruje natywne wzmianki @, odpowiada tylko na wzorce tekstowe):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### Komendy (obsługa komend czatu)

```json5
{
  commands: {
    native: "auto", // rejestruj natywne komendy, gdy są obsługiwane
    text: true, // analizuj /commands w wiadomościach czatu
    bash: false, // zezwól na ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // zezwól na /config
    debug: false, // zezwól na /debug
    restart: false, // zezwól na /restart + narzędzie restartu gateway
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Komendy tekstowe muszą być **samodzielnymi** wiadomościami rozpoczynającymi się od `/`.
- `native: "auto"` włącza natywne komendy dla Discord/Telegram, pozostawia Slack wyłączony.
- Nadpisanie per kanał: `channels.discord.commands.native` (bool lub `"auto"`). `false` usuwa wcześniej zarejestrowane komendy.
- `channels.telegram.customCommands` dodaje dodatkowe pozycje menu bota Telegram.
- `bash: true` włącza `! <cmd>` dla powłoki hosta. Wymaga `tools.elevated.enabled` oraz nadawcy w `tools.elevated.allowFrom.<channel>`.
- `config: true` włącza `/config` (odczyt/zapis `openclaw.json`).
- `channels.<provider> .configWrites` kontroluje możliwość modyfikacji konfiguracji per kanał (domyślnie: true).
- `allowFrom` jest ustawiane per dostawca. Gdy ustawione, jest to **jedyne** źródło autoryzacji (listy dozwolonych kanałów/parowanie oraz `useAccessGroups` są ignorowane).
- `useAccessGroups: false` pozwala komendom ominąć polityki grup dostępu, gdy `allowFrom` nie jest ustawione.

</Accordion>

---

## Domyślne ustawienia agenta

### `agents.defaults.workspace`

Domyślnie: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Opcjonalny katalog główny repozytorium wyświetlany w wierszu Runtime w system prompt. Jeśli nieustawione, OpenClaw automatycznie wykrywa, przechodząc w górę od katalogu workspace.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Wyłącza automatyczne tworzenie plików bootstrap workspace (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Maksymalna liczba znaków na plik bootstrap przestrzeni roboczej przed obcięciem. Domyślnie: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Maksymalna łączna liczba znaków wstrzykiwanych ze wszystkich plików bootstrap przestrzeni roboczej. Domyślnie: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Strefa czasowa dla kontekstu promptu systemowego (nie znaczników czasu wiadomości). Jeśli nie ustawiono, używana jest strefa czasowa hosta.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Format czasu w prompcie systemowym. Domyślnie: `auto` (preferencja systemu operacyjnego).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: format `provider/model` (np. `anthropic/claude-opus-4-6`). Jeśli pominiesz dostawcę, OpenClaw przyjmie `anthropic` (przestarzałe).
- `models`: skonfigurowany katalog modeli i lista dozwolonych dla `/model`. Każdy wpis może zawierać `alias` (skrót) oraz `params` (specyficzne dla dostawcy: `temperature`, `maxTokens`).
- `imageModel`: używany tylko wtedy, gdy model główny nie obsługuje wejścia obrazów.
- `maxConcurrent`: maksymalna liczba równoległych uruchomień agenta we wszystkich sesjach (każda sesja nadal jest wykonywana sekwencyjnie). Domyślnie: 1.

**Wbudowane skróty aliasów** (mają zastosowanie tylko, gdy model znajduje się w `agents.defaults.models`):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Twoje skonfigurowane aliasy zawsze mają pierwszeństwo przed domyślnymi.

Modele Z.AI GLM-4.x automatycznie włączają tryb myślenia, chyba że ustawisz `--thinking off` lub samodzielnie zdefiniujesz `agents.defaults.models["zai/<model>"].params.thinking`.

### `agents.defaults.cliBackends`

Opcjonalne backendy CLI do uruchomień awaryjnych tylko z tekstem (bez wywołań narzędzi). Przydatne jako kopia zapasowa, gdy dostawcy API zawodzą.

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- Backendy CLI są przede wszystkim tekstowe; narzędzia są zawsze wyłączone.
- Sesje są obsługiwane, gdy ustawiono `sessionArg`.
- Przekazywanie obrazów jest obsługiwane, gdy `imageArg` akceptuje ścieżki do plików.

### `agents.defaults.heartbeat`

Okresowe uruchomienia heartbeat.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: ciąg określający czas trwania (ms/s/m/h). Domyślnie: `30m`.
- Dla pojedynczego agenta: ustaw `agents.list[].heartbeat`. Gdy jakikolwiek agent definiuje `heartbeat`, **tylko ci agenci** uruchamiają heartbeat.
- Heartbeat wykonuje pełne tury agenta — krótsze interwały zużywają więcej tokenów.

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`: `default` lub `safeguard` (fragmentaryczne podsumowywanie dla długich historii). Zobacz [Compaction](/concepts/compaction).
- `memoryFlush`: cicha tura agentowa przed automatyczną kompaktacją w celu zapisania trwałych wspomnień. Pomijane, gdy workspace jest w trybie tylko do odczytu.

### `agents.defaults.contextPruning`

Usuwa **stare wyniki narzędzi** z kontekstu w pamięci przed wysłaniem do LLM. **Nie** modyfikuje historii sesji zapisanej na dysku.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` włącza przebiegi przycinania.
- `ttl` określa, jak często przycinanie może zostać uruchomione ponownie (od ostatniego użycia cache).
- Przycinanie najpierw miękko skraca zbyt duże wyniki narzędzi, a następnie w razie potrzeby całkowicie czyści starsze wyniki.

**Soft-trim** zachowuje początek i koniec oraz wstawia `...` pośrodku.

**Hard-clear** zastępuje cały wynik narzędzia placeholderem.

Uwagi:

- Bloki obrazów nigdy nie są przycinane ani czyszczone.
- Współczynniki opierają się na liczbie znaków (w przybliżeniu), a nie na dokładnej liczbie tokenów.
- Jeśli istnieje mniej niż `keepLastAssistants` wiadomości asystenta, przycinanie jest pomijane.

</Accordion>

Szczegóły działania znajdziesz w [Session Pruning](/concepts/session-pruning).

### Strumieniowanie blokowe

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- Kanały inne niż Telegram wymagają jawnego ustawienia `*.blockStreaming: true`, aby włączyć odpowiedzi blokowe.
- Nadpisania kanałów: `channels.<channel>`.blockStreamingCoalesce`(oraz warianty dla poszczególnych kont). Domyślnie dla Signal/Slack/Discord/Google Chat`minChars: 1500\`.
- `humanDelay`: losowa przerwa między odpowiedziami blokowymi. `natural` = 800–2500ms. Nadpisanie per-agent: `agents.list[].humanDelay`.

Zobacz [Streaming](/concepts/streaming), aby poznać zachowanie + szczegóły chunkowania.

### Wskaźniki pisania

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- Domyślnie: `instant` dla czatów bezpośrednich/wzmianek, `message` dla czatów grupowych bez wzmianek.
- Nadpisania per-sesja: `session.typingMode`, `session.typingIntervalSeconds`.

Zobacz [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

Opcjonalne **sandboxowanie Docker** dla wbudowanego agenta. Zobacz [Sandboxing](/gateway/sandboxing), aby zapoznać się z pełnym przewodnikiem.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

<Accordion title="Sandbox details">

**Dostęp do workspace:**

- `none`: workspace sandboxa w danym zakresie pod `~/.openclaw/sandboxes`
- `ro`: workspace sandboxa w `/workspace`, workspace agenta montowany tylko do odczytu w `/agent`
- `rw`: workspace agenta montowany do odczytu i zapisu w `/workspace`

**Zakres (Scope):**

- `session`: kontener + workspace per-sesja
- `agent`: jeden kontener + workspace na agenta (domyślnie)
- `shared`: współdzielony kontener i workspace (brak izolacji między sesjami)

**`setupCommand`** uruchamiane raz po utworzeniu kontenera (przez `sh -lc`). Wymaga dostępu wychodzącego do sieci, zapisywalnego katalogu root oraz użytkownika root.

**Domyślnie kontenery mają `network: "none"`** — ustaw na `"bridge"`, jeśli agent potrzebuje dostępu wychodzącego.

**Załączniki przychodzące** są umieszczane w `media/inbound/*` w aktywnym workspace.

**`docker.binds`** montuje dodatkowe katalogi hosta; globalne i per-agent bindy są łączone.

**Przeglądarka w sandboxie** (`sandbox.browser.enabled`): Chromium + CDP w kontenerze. Adres URL noVNC jest wstrzykiwany do system promptu. Nie wymaga `browser.enabled` w głównej konfiguracji.

- `allowHostControl: false` (domyślnie) blokuje sesjom sandboxa możliwość kierowania do przeglądarki hosta.
- `sandbox.browser.binds` montuje dodatkowe katalogi hosta wyłącznie do kontenera przeglądarki w sandboxie. Po ustawieniu (również `[]`) zastępuje `docker.binds` dla kontenera przeglądarki.

</Accordion>

Budowanie obrazów:

```bash
scripts/sandbox-setup.sh           # główny obraz sandbox
scripts/sandbox-browser-setup.sh   # opcjonalny obraz przeglądarki
```

### `agents.list` (nadpisania per-agent)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // lub { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: stabilny identyfikator agenta (wymagany).
- `default`: gdy ustawiono wiele, wygrywa pierwszy (rejestrowane jest ostrzeżenie). Jeśli żaden nie jest ustawiony, domyślny jest pierwszy element listy.
- `model`: forma tekstowa nadpisuje tylko `primary`; forma obiektowa `{ primary, fallbacks }` nadpisuje oba (`[]` wyłącza globalne fallbacks).
- `identity.avatar`: ścieżka względna względem workspace, URL `http(s)` lub URI `data:`.
- `identity` wyprowadza wartości domyślne: `ackReaction` z `emoji`, `mentionPatterns` z `name`/`emoji`.
- `subagents.allowAgents`: lista dozwolonych identyfikatorów agentów dla `sessions_spawn` (`["*"]` = dowolny; domyślnie: tylko ten sam agent).

---

## Routing wieloagentowy

Uruchamiaj wiele odizolowanych agentów w jednym Gateway. Zobacz [Multi-Agent](/concepts/multi-agent).

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

### Pola dopasowania binding

- `match.channel` (wymagane)
- `match.accountId` (opcjonalne; `*` = dowolne konto; pominięte = konto domyślne)
- `match.peer` (opcjonalne; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (opcjonalne; zależne od kanału)

**Deterministyczna kolejność dopasowania:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (dokładne, bez peer/guild/team)
5. `match.accountId: "*"` (dla całego kanału)
6. Agent domyślny

W obrębie każdego poziomu wygrywa pierwszy pasujący wpis `bindings`.

### Profile dostępu per agent

<Accordion title="Full access (no sandbox)">

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

</Accordion>

Zobacz [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools), aby poznać szczegóły dotyczące priorytetów.

---

## Sesja

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // legacy (runtime always uses "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: sposób grupowania wiadomości DM.
  - `main`: wszystkie DM współdzielą główną sesję.
  - `per-peer`: izolacja według identyfikatora nadawcy we wszystkich kanałach.
  - `per-channel-peer`: izolacja per kanał + nadawca (zalecane dla skrzynek wieloużytkownikowych).
  - `per-account-channel-peer`: izolacja per konto + kanał + nadawca (zalecane dla wielu kont).
- **`identityLinks`**: mapowanie identyfikatorów kanonicznych na prefiksowane przez dostawcę peer w celu współdzielenia sesji między kanałami.
- **`reset`**: główna polityka resetowania. `daily` resetuje o `atHour` czasu lokalnego; `idle` resetuje po `idleMinutes`. Jeśli skonfigurowane są oba, wygrywa ten, który wygaśnie jako pierwszy.
- **`resetByType`**: nadpisania per typ (`direct`, `group`, `thread`). Starsze `dm` akceptowane jako alias dla `direct`.
- **`mainKey`**: pole przestarzałe. Środowisko uruchomieniowe zawsze używa teraz `"main"` dla głównego koszyka czatu bezpośredniego.
- **`sendPolicy`**: dopasowanie według `channel`, `chatType` (`direct|group|channel`, ze starszym aliasem `dm`), `keyPrefix` lub `rawKeyPrefix`. Pierwsza reguła odrzucenia ma pierwszeństwo.
- **`maintenance`**: `warn` ostrzega aktywną sesję przy usunięciu; `enforce` stosuje przycinanie i rotację.

</Accordion>

---

## Wiadomości

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### Prefiks odpowiedzi

Nadpisania dla kanału/konta: `channels.<channel>`.responsePrefix`, `channels.<channel>`.accounts.<id>`.responsePrefix\`.

Rozstrzyganie (najbardziej szczegółowe ma pierwszeństwo): konto → kanał → globalne. `""` wyłącza i zatrzymuje dziedziczenie. `"auto"` wyprowadza `[{identity.name}]`.

**Zmienne szablonu:**

| Zmienna           | Opis                       | Przykład                                   |
| ----------------- | -------------------------- | ------------------------------------------ |
| `{model}`         | Krótka nazwa modelu        | `claude-opus-4-6`                          |
| `{modelFull}`     | Pełny identyfikator modelu | `anthropic/claude-opus-4-6`                |
| `{provider}`      | Nazwa dostawcy             | `anthropic`                                |
| `{thinkingLevel}` | Bieżący poziom rozumowania | `high`, `low`, `off`                       |
| `{identity.name}` | Nazwa tożsamości agenta    | (tak samo jak `"auto"`) |

Zmienne są niewrażliwe na wielkość liter. `{think}` jest aliasem dla `{thinkingLevel}`.

### Reakcja potwierdzająca (ack)

- Domyślnie używa `identity.emoji` aktywnego agenta, w przeciwnym razie `"👀"`. Ustaw `""`, aby wyłączyć.
- Nadpisania dla poszczególnych kanałów: `channels.<channel>``.ackReaction`, `channels.<channel>``.accounts.<id>``.ackReaction`.
- Kolejność rozstrzygania: konto → kanał → `messages.ackReaction` → domyślna tożsamość.
- Zakres: `group-mentions` (domyślnie), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: usuwa potwierdzenie po odpowiedzi (tylko Slack/Discord/Telegram/Google Chat).

### Debounce przychodzących wiadomości

Łączy szybko następujące po sobie wiadomości tekstowe od tego samego nadawcy w jedną turę agenta. Multimedia/załączniki są przetwarzane natychmiast. Polecenia sterujące pomijają debounce.

### TTS (zamiana tekstu na mowę)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

- `auto` kontroluje automatyczne TTS. `/tts off|always|inbound|tagged` nadpisuje ustawienia dla sesji.
- `summaryModel` nadpisuje `agents.defaults.model.primary` dla automatycznego podsumowania.
- Klucze API domyślnie używają `ELEVENLABS_API_KEY`/`XI_API_KEY` oraz `OPENAI_API_KEY`.

---

## Talk

Ustawienia domyślne dla trybu Talk (macOS/iOS/Android).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Identyfikatory głosu domyślnie używają `ELEVENLABS_VOICE_ID` lub `SAG_VOICE_ID`.
- `apiKey` domyślnie używa `ELEVENLABS_API_KEY`.
- `voiceAliases` umożliwia dyrektywom Talk używanie przyjaznych nazw.

---

## Narzędzia

### Profile narzędzi

`tools.profile` ustawia bazową listę dozwolonych przed `tools.allow`/`tools.deny`:

| Profil      | Zawiera                                                                                   |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | Tylko `session_status`                                                                    |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Brak ograniczeń (tak samo jak nieustawione)                            |

### Grupy narzędzi

| Grupa              | Narzędzia                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` is accepted as an alias for `exec`)         |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Wszystkie wbudowane narzędzia (z wyłączeniem wtyczek dostawców)       |

### `tools.allow` / `tools.deny`

Globalna polityka zezwalania/odmawiania dostępu do narzędzi (deny ma pierwszeństwo). Niewrażliwe na wielkość liter, obsługuje symbole wieloznaczne `*`. Stosowane nawet gdy sandbox Docker jest wyłączony.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Dodatkowe ograniczenia narzędzi dla określonych dostawców lub modeli. Kolejność: profil bazowy → profil dostawcy → allow/deny.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

Kontroluje podwyższony (host) dostęp do exec:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- Nadpisanie per-agent (`agents.list[].tools.elevated`) może jedynie dodatkowo ograniczać.
- `/elevated on|off|ask|full` zapisuje stan per sesję; dyrektywy inline dotyczą pojedynczej wiadomości.
- Podwyższony `exec` działa na hoście i omija sandboxing.

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // or BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

Konfiguruje przetwarzanie przychodzących mediów (image/audio/video):

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Media model entry fields">

**Wpis dostawcy** (`type: "provider"` lub pominięte):

- `provider`: identyfikator dostawcy API (`openai`, `anthropic`, `google`/`gemini`, `groq` itp.)
- `model`: nadpisanie identyfikatora modelu
- `profile` / `preferredProfile`: wybór profilu uwierzytelniania

**Wpis CLI** (`type: "cli"`):

- `command`: plik wykonywalny do uruchomienia
- `args`: argumenty szablonowe (obsługuje `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` itp.)

**Wspólne pola:**

- `capabilities`: opcjonalna lista (`image`, `audio`, `video`). Domyślnie: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: nadpisania dla poszczególnych wpisów.
- W przypadku błędu następuje przejście do kolejnego wpisu.

Uwierzytelnianie dostawcy przebiega według standardowej kolejności: profile auth → zmienne środowiskowe → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: domyślny model dla uruchamianych sub-agentów. Jeśli pominięte, sub-agenci dziedziczą model wywołującego.
- Polityka narzędzi dla pojedynczego sub-agenta: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Niestandardowi dostawcy i bazowe adresy URL

OpenClaw używa katalogu modeli pi-coding-agent. Dodaj niestandardowych dostawców przez `models.providers` w konfiguracji lub `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (default) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- Użyj `authHeader: true` + `headers` dla niestandardowych wymagań uwierzytelniania.
- Nadpisz katalog główny konfiguracji agenta za pomocą `OPENCLAW_AGENT_DIR` (lub `PI_CODING_AGENT_DIR`).

### Przykłady dostawców

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

Użyj `cerebras/zai-glm-4.7` dla Cerebras; `zai/glm-4.7` dla bezpośredniego Z.AI.

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

Ustaw `OPENCODE_API_KEY` (lub `OPENCODE_ZEN_API_KEY`). Skrót: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

Ustaw `ZAI_API_KEY`. `z.ai/*` i `z-ai/*` są akceptowanymi aliasami. Skrót: `openclaw onboard --auth-choice zai-api-key`.

- Ogólny endpoint: `https://api.z.ai/api/paas/v4`
- Endpoint do kodowania (domyślny): `https://api.z.ai/api/coding/paas/v4`
- Dla ogólnego endpointu zdefiniuj niestandardowego dostawcę z nadpisaniem bazowego adresu URL.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Dla punktu końcowego China: `baseUrl: "https://api.moonshot.cn/v1"` lub `openclaw onboard --auth-choice moonshot-api-key-cn`.

</Accordion>

<Accordion title="Kimi Coding">

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

Zgodny z Anthropic, wbudowany dostawca. Skrót: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Bazowy URL nie powinien zawierać `/v1` (klient Anthropic dodaje go automatycznie). Skrót: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Ustaw `MINIMAX_API_KEY`. Skrót: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

Zobacz [Local Models](/gateway/local-models). TL;DR: uruchom MiniMax M2.1 przez LM Studio Responses API na wydajnym sprzęcie; zachowaj hostowane modele jako scalone na potrzeby zapasowe.

</Accordion>

---

## Umiejętności

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: opcjonalna lista dozwolonych tylko dla dołączonych umiejętności (zarządzane/robocze umiejętności pozostają bez zmian).
- `entries.<skillKey>`.enabled: false\` wyłącza umiejętność nawet jeśli jest dołączona/zainstalowana.
- `entries.<skillKey>`.apiKey\`: ułatwienie dla umiejętności deklarujących główną zmienną środowiskową.

---

## Wtyczki

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- Ładowane z `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` oraz `plugins.load.paths`.
- **Zmiany konfiguracji wymagają ponownego uruchomienia gateway.**
- `allow`: opcjonalna lista dozwolonych (ładowane są tylko wymienione wtyczki). `deny` ma pierwszeństwo.

Zobacz [Plugins](/tools/plugin).

---

## Przeglądarka

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` wyłącza `act:evaluate` oraz `wait --fn`.
- Profile zdalne są tylko do podłączania (uruchamianie/zatrzymywanie/resetowanie jest wyłączone).
- Kolejność automatycznego wykrywania: domyślna przeglądarka, jeśli oparta na Chromium → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Usługa kontrolna: tylko loopback (port pochodny z `gateway.port`, domyślnie `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: kolor akcentu dla natywnego interfejsu aplikacji (odcień dymku w Trybie rozmowy itp.).
- `assistant`: nadpisanie tożsamości w interfejsie Control UI. Domyślnie używana jest tożsamość aktywnego agenta.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (uruchom gateway) lub `remote` (połącz ze zdalnym gateway). Gateway odmawia uruchomienia, jeśli tryb nie jest ustawiony na `local`.
- `port`: pojedynczy multipleksowany port dla WS + HTTP. Priorytet: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (domyślnie), `lan` (`0.0.0.0`), `tailnet` (tylko adres IP Tailscale) lub `custom`.
- **Auth**: wymagane domyślnie. Powiązania inne niż loopback wymagają współdzielonego tokenu/hasła. Kreator wdrożeniowy domyślnie generuje token.
- `auth.mode: "trusted-proxy"`: deleguj uwierzytelnianie do reverse proxy ze świadomością tożsamości i ufaj nagłówkom tożsamości z `gateway.trustedProxies` (zobacz [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: gdy ustawione na `true`, nagłówki tożsamości Tailscale Serve spełniają wymagania uwierzytelniania (weryfikowane przez `tailscale whois`). Domyślnie `true`, gdy `tailscale.mode = "serve"`.
- `auth.rateLimit`: opcjonalne ograniczanie liczby nieudanych prób uwierzytelniania. Stosowane per adres IP klienta oraz per zakres uwierzytelniania (shared-secret i device-token są śledzone niezależnie). Zablokowane próby zwracają `429` + `Retry-After`.
  - `auth.rateLimit.exemptLoopback` domyślnie ma wartość `true`; ustaw na `false`, jeśli celowo chcesz, aby ruch z localhost również podlegał limitowaniu (np. w środowiskach testowych lub przy rygorystycznych wdrożeniach proxy).
- `tailscale.mode`: `serve` (tylko tailnet, powiązanie loopback) lub `funnel` (publiczne, wymaga uwierzytelniania).
- `remote.transport`: `ssh` (domyślnie) lub `direct` (ws/wss). Dla `direct`, `remote.url` musi mieć schemat `ws://` lub `wss://`.
- `gateway.remote.token` służy wyłącznie do zdalnych wywołań CLI; nie włącza uwierzytelniania lokalnego gateway.
- `trustedProxies`: adresy IP reverse proxy, które kończą TLS. Wymieniaj tylko proxy, które kontrolujesz.
- `gateway.tools.deny`: dodatkowe nazwy narzędzi blokowane dla HTTP `POST /tools/invoke` (rozszerza domyślną listę blokowanych).
- `gateway.tools.allow`: usuwa nazwy narzędzi z domyślnej listy blokowanych HTTP.

</Accordion>

### Endpointy zgodne z OpenAI

- Chat Completions: domyślnie wyłączone. Włącz za pomocą `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Wzmocnione zabezpieczenia wejścia URL w Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Izolacja wielu instancji

Uruchom wiele gateway na jednym hoście z unikalnymi portami i katalogami stanu:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Flagi pomocnicze: `--dev` (używa `~/.openclaw-dev` + port `19001`), `--profile <name>` (używa `~/.openclaw-<name>`).

Zobacz [Multiple Gateways](/gateway/multiple-gateways).

---

## Hooki

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

Auth: `Authorization: Bearer <token>` lub `x-openclaw-token: <token>`.

**Endpointy:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` z ładunku żądania jest akceptowany tylko gdy `hooks.allowRequestSessionKey=true` (domyślnie: `false`).
- `POST /hooks/<name>` → rozwiązywane przez `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` dopasowuje podścieżkę po `/hooks` (np. `/hooks/gmail` → `gmail`).
- `match.source` dopasowuje pole ładunku dla ścieżek ogólnych.
- Szablony takie jak `{{messages[0].subject}}` odczytują dane z ładunku.
- `transform` może wskazywać na moduł JS/TS zwracający akcję hooka.
  - `transform.module` musi być ścieżką względną i pozostawać w obrębie `hooks.transformsDir` (ścieżki bezwzględne i traversal są odrzucane).
- `agentId` kieruje do konkretnego agenta; nieznane identyfikatory powodują powrót do domyślnego.
- `allowedAgentIds`: ogranicza jawne kierowanie (`*` lub pominięte = zezwól na wszystkie, `[]` = zablokuj wszystkie).
- `defaultSessionKey`: opcjonalny stały klucz sesji dla uruchomień agenta hooka bez jawnego `sessionKey`.
- `allowRequestSessionKey`: pozwala wywołującym `/hooks/agent` ustawić `sessionKey` (domyślnie: `false`).
- `allowedSessionKeyPrefixes`: opcjonalna lista dozwolonych prefiksów dla jawnych wartości `sessionKey` (żądanie + mapowanie), np. `["hook:"]`.
- `deliver: true` wysyła końcową odpowiedź do kanału; `channel` domyślnie ma wartość `last`.
- `model` nadpisuje LLM dla tego uruchomienia hooka (musi być dozwolony, jeśli ustawiono katalog modeli).

</Accordion>

### Integracja Gmail

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- Gateway automatycznie uruchamia `gog gmail watch serve` przy starcie, gdy jest skonfigurowany. Ustaw `OPENCLAW_SKIP_GMAIL_WATCHER=1`, aby wyłączyć.
- Nie uruchamiaj osobnego `gog gmail watch serve` równolegle z Gateway.

---

## Host Canvas

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // lub OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Serwuje edytowalne przez agenta pliki HTML/CSS/JS oraz A2UI przez HTTP pod portem Gateway:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Tylko lokalnie: ustaw `gateway.bind: "loopback"` (domyślnie).
- Powiązania inne niż loopback: trasy canvas wymagają uwierzytelnienia Gateway (token/hasło/trusted-proxy), tak jak inne interfejsy HTTP Gateway.
- WebViews Node zazwyczaj nie wysyłają nagłówków uwierzytelniania; po sparowaniu i połączeniu węzła Gateway zezwala na fallback z prywatnego IP, aby węzeł mógł ładować canvas/A2UI bez ujawniania sekretów w URL.
- Wstrzykuje klienta live-reload do serwowanego HTML.
- Automatycznie tworzy startowy `index.html`, gdy katalog jest pusty.
- Serwuje również A2UI pod `/__openclaw__/a2ui/`.
- Zmiany wymagają ponownego uruchomienia gateway.
- Wyłącz live reload dla dużych katalogów lub w przypadku błędów `EMFILE`.

---

## Wykrywanie

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (domyślnie): pomija `cliPath` + `sshPort` w rekordach TXT.
- `full`: zawiera `cliPath` + `sshPort`.
- Domyślna nazwa hosta to `openclaw`. Zastąp za pomocą `OPENCLAW_MDNS_HOSTNAME`.

### Sieć rozległa (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Zapisuje strefę unicast DNS-SD w `~/.openclaw/dns/`. W celu wykrywania między sieciami połącz z serwerem DNS (zalecany CoreDNS) + Tailscale split DNS.

Konfiguracja: `openclaw dns setup --apply`.

---

## Środowisko

### `env` (wbudowane zmienne środowiskowe)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- Wbudowane zmienne środowiskowe są stosowane tylko wtedy, gdy w środowisku procesu brakuje danego klucza.
- Pliki `.env`: CWD `.env` + `~/.openclaw/.env` (żaden nie nadpisuje istniejących zmiennych).
- `shellEnv`: importuje brakujące oczekiwane klucze z profilu powłoki logowania.
- Zobacz [Environment](/help/environment), aby poznać pełną kolejność priorytetów.

### Podstawianie zmiennych środowiskowych

Odwołuj się do zmiennych środowiskowych w dowolnym ciągu konfiguracyjnym za pomocą `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Dopasowywane są tylko nazwy zapisane wielkimi literami: `[A-Z_][A-Z0-9_]*`.
- Brakujące/puste zmienne powodują błąd podczas wczytywania konfiguracji.
- Użyj `$${VAR}`, aby uzyskać dosłowne `${VAR}`.
- Działa z `$include`.

---

## Przechowywanie uwierzytelniania

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- Profile uwierzytelniania dla poszczególnych agentów są przechowywane w `<agentDir>/auth-profiles.json`.
- Starsze dane OAuth są importowane z `~/.openclaw/credentials/oauth.json`.
- Zobacz [OAuth](/concepts/oauth).

---

## Logowanie

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- Domyślny plik logu: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Ustaw `logging.file`, aby użyć stałej ścieżki.
- `consoleLevel` zwiększa się do `debug` przy użyciu `--verbose`.

---

## Kreator

Metadane zapisywane przez kreatory CLI (`onboard`, `configure`, `doctor`):

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## Tożsamość

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

Zapisywane przez asystenta wdrożeniowego macOS. Wyprowadza wartości domyślne:

- `messages.ackReaction` z `identity.emoji` (w przeciwnym razie 👀)
- `mentionPatterns` z `identity.name`/`identity.emoji`
- `avatar` akceptuje: ścieżkę względną względem workspace, URL `http(s)` lub URI `data:`

---

## Bridge (przestarzałe, usunięte)

Aktualne kompilacje nie zawierają już mostu TCP. Węzły łączą się przez Gateway WebSocket. Klucze `bridge.*` nie są już częścią schematu konfiguracji (walidacja zakończy się błędem, dopóki nie zostaną usunięte; `openclaw doctor --fix` może usunąć nieznane klucze).

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`: jak długo przechowywać zakończone sesje cron przed ich usunięciem. Domyślnie: `24h`.

Zobacz [Cron Jobs](/automation/cron-jobs).

---

## Zmienne szablonu modelu mediów

Symbole zastępcze szablonu rozwijane w `tools.media.*.models[].args`:

| Zmienna            | Opis                                                                                     |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `{{Body}}`         | Pełna treść przychodzącej wiadomości                                                     |
| `{{RawBody}}`      | Surowa treść (bez historii/oznaczeń nadawcy)                          |
| `{{BodyStripped}}` | Treść z usuniętymi wzmiankami grupowymi                                                  |
| `{{From}}`         | Identyfikator nadawcy                                                                    |
| `{{To}}`           | Identyfikator odbiorcy                                                                   |
| `{{MessageSid}}`   | Identyfikator wiadomości kanału                                                          |
| `{{SessionId}}`    | UUID bieżącej sesji                                                                      |
| `{{IsNewSession}}` | `"true"` gdy utworzono nową sesję                                                        |
| `{{MediaUrl}}`     | Pseudo-URL przychodzących mediów                                                         |
| `{{MediaPath}}`    | Lokalna ścieżka do mediów                                                                |
| `{{MediaType}}`    | Typ mediów (image/audio/document/…)                                   |
| `{{Transcript}}`   | Transkrypcja audio                                                                       |
| `{{Prompt}}`       | Rozwiązany prompt mediów dla wpisów CLI                                                  |
| `{{MaxChars}}`     | Rozwiązana maksymalna liczba znaków wyjściowych dla wpisów CLI                           |
| `{{ChatType}}`     | `"direct"` lub `"group"`                                                                 |
| `{{GroupSubject}}` | Temat grupy (najlepsza próba)                                         |
| `{{GroupMembers}}` | Podgląd członków grupy (najlepsza próba)                              |
| `{{SenderName}}`   | Wyświetlana nazwa nadawcy (najlepsza próba)                           |
| `{{SenderE164}}`   | Numer telefonu nadawcy (najlepsza próba)                              |
| `{{Provider}}`     | Wskazówka dostawcy (whatsapp, telegram, discord itp.) |

---

## Konfiguracja zawiera (`$include`)

Podziel konfigurację na wiele plików:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**Zasady scalania:**

- Pojedynczy plik: zastępuje obiekt nadrzędny.
- Tablica plików: scalane głęboko w podanej kolejności (późniejsze nadpisują wcześniejsze).
- Klucze równorzędne: scalane po include (nadpisują dołączone wartości).
- Zagnieżdżone include: do 10 poziomów głębokości.
- Ścieżki: względne (względem pliku dołączającego), bezwzględne lub odwołania do katalogu nadrzędnego `../`.
- Błędy: czytelne komunikaty o brakujących plikach, błędach parsowania i cyklicznych include.

---

_Powiązane: [Konfiguracja](/gateway/configuration) · [Przykłady konfiguracji](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

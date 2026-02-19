---
summary: "„Status wsparcia bota Discord, możliwości i konfiguracja”"
read_when:
  - Prace nad funkcjami kanału Discord
title: "Discord"
---

# Discord (Bot API)

Status: gotowy do DM-ów i tekstowych kanałów gildii przez oficjalną bramę bota Discord.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Wiadomości prywatne Discord domyślnie używają trybu parowania.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Zachowanie natywnych poleceń i katalog poleceń.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostyka międzykanałowa i proces naprawy.
  
</Card>
</CardGroup>

## Szybka konfiguracja

<Steps>
  <Step title="Create a Discord bot and enable intents">
    Utwórz aplikację w Discord Developer Portal, dodaj bota, a następnie włącz:

    ```
    **Server Members Intent** (zalecany; wymagany do niektórych wyszukiwań członków/użytkowników i dopasowań list dozwolonych w gildiach)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Awaryjne użycie zmiennej środowiskowej dla konta domyślnego:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Zaproś bota na swój serwer z uprawnieniami wymaganymi do odczytu/wysyłania wiadomości tam, gdzie chcesz go używać.

```bash
Uruchom gateway.
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Kody parowania wygasają po 1 godzinie.
    ```

  
</Step>
</Steps>

<Note>
Rozwiązywanie tokena jest zależne od konta. Wartości tokena w konfiguracji mają pierwszeństwo przed fallbackiem z env. `DISCORD_BOT_TOKEN` jest używany tylko dla konta domyślnego.
</Note>

## Model wykonawczy

- Gateway zarządza połączeniem z Discord.
- Trasowanie odpowiedzi jest deterministyczne: przychodzące odpowiedzi z Discord wracają na Discord.
- Agent może wywołać `discord` z akcjami takimi jak:
- Czat bezpośredni zwija się do głównej sesji agenta (domyślnie `agent:main:main`); kanały gildii pozostają odizolowane jako `agent:<agentId>:discord:channel:<channelId>` (nazwy wyświetlane używają `discord:<guildSlug>#<channelSlug>`).
- Grupowe DM-y są domyślnie ignorowane; włącz przez `channels.discord.dm.groupEnabled` i opcjonalnie ogranicz przez `channels.discord.dm.groupChannels`.
- Komendy natywne używają izolowanych kluczy sesji (`agent:<agentId>:discord:slash:<userId>`) zamiast współdzielonej sesji `main`.

## Kontrola dostępu i trasowanie

<Tabs>
  <Tab title="DM policy">Aby ignorować wszystkie DM-y: ustaw `channels.discord.dm.enabled=false` lub `channels.discord.dm.policy="disabled"`.

    ```
    Aby zastosować twardą listę dozwolonych: ustaw `channels.discord.dm.policy="allowlist"` i wypisz nadawców w `channels.discord.dm.allowFrom`.
    ```

  
</Tab>

  <Tab title="Guild policy">Zachowanie jest kontrolowane przez `channels.discord.replyToMode`:

    ```
    Komendy natywne respektują te same listy dozwolonych co DM-y/wiadomości gildii (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, reguły per kanał).
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    Jeśli ustawisz tylko `DISCORD_BOT_TOKEN` i nigdy nie utworzysz sekcji `channels.discord`, środowisko wykonawcze
        domyślnie ustawi `groupPolicy` na `open`.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">    Wiadomości na serwerze (Guild) są domyślnie filtrowane na podstawie wzmianki.

    ```
    Wykrywanie wzmianek obejmuje:
    
    - bezpośrednią wzmiankę o bocie
    - skonfigurowane wzorce wzmianek (`agents.list[].groupChat.mentionPatterns`, zapasowo `messages.groupChat.mentionPatterns`)
    - niejawne zachowanie odpowiedzi do bota w obsługiwanych przypadkach
    
    `requireMention` jest konfigurowane per serwer/kanał (`channels.discord.guilds...`).
    
    Grupowe DM:
    
    - domyślnie: ignorowane (`dm.groupEnabled=false`)
    - opcjonalna allowlista przez `dm.groupChannels` (ID kanałów lub slugi)
    ```

  
</Tab>
</Tabs>

### Trasowanie agentów na podstawie ról

Użyj `bindings[].match.roles`, aby kierować członków serwera Discord do różnych agentów na podstawie ID roli. Powiązania oparte na rolach akceptują wyłącznie ID ról i są oceniane po powiązaniach peer lub parent-peer, a przed powiązaniami tylko dla serwera. Jeśli powiązanie ustawia również inne pola dopasowania (na przykład `peer` + `guildId` + `roles`), wszystkie skonfigurowane pola muszą się zgadzać.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Konfiguracja w Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">W **Bot** → **Privileged Gateway Intents** włącz:

    ```
    - Message Content Intent
    - Server Members Intent (zalecane)
    
    Presence intent jest opcjonalny i wymagany tylko wtedy, gdy chcesz otrzymywać aktualizacje statusu. Ustawianie statusu bota (`setPresence`) nie wymaga włączania aktualizacji statusu członków.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">W aplikacji: **OAuth2** → **URL Generator**

    ```
    - scopes: `bot`, `applications.commands`
    
    Typowe podstawowe uprawnienia:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (opcjonalnie)
    
    Unikaj `Administrator`, chyba że jest to bezwzględnie konieczne.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">    Włącz Discord Developer Mode, a następnie skopiuj:

    ```
    - ID serwera
    - ID kanału
    - ID użytkownika
    
    W konfiguracji OpenClaw preferuj numeryczne ID dla wiarygodnych audytów i testów.
    ```

  
</Accordion>
</AccordionGroup>

## Polecenia natywne i autoryzacja poleceń

- Opcjonalne komendy natywne: `commands.native` domyślnie `"auto"` (włączone dla Discord/Telegram, wyłączone dla Slack).
- Lub konfiguracja: `channels.discord.token: "..."`.
- Nadpisz przez `channels.discord.commands.native: true|false|"auto"`; `false` czyści wcześniej zarejestrowane komendy.
- Autoryzacja poleceń natywnych używa tych samych allowlist/polityk Discord co standardowa obsługa wiadomości.
- Komendy slash mogą być nadal widoczne w UI Discord dla użytkowników spoza listy dozwolonych; OpenClaw egzekwuje listy przy wykonaniu i odpowiada „not authorized”.

Zobacz [Exec approvals](/tools/exec-approvals) i [Slash commands](/tools/slash-commands) dla szerszego przepływu zatwierdzeń i komend.

## Szczegóły funkcji

<AccordionGroup>
  <Accordion title="Reply tags and native replies">    Discord obsługuje tagi odpowiedzi w wyjściu agenta:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    Kontrolowane przez `channels.discord.replyToMode`:
    
    - `off` (domyślnie)
    - `first`
    - `all`
    
    Uwaga: `off` wyłącza niejawne wątkowanie odpowiedzi. Jawne tagi `[[reply_to_*]]` są nadal respektowane.
    
    ID wiadomości są udostępniane w kontekście/historii, aby agenci mogli kierować odpowiedzi do konkretnych wiadomości.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">    Kontekst historii serwera (Guild):

    ```
    - `channels.discord.historyLimit` domyślnie `20`
    - zapasowo: `messages.groupChat.historyLimit`
    - `0` wyłącza
    
    Kontrola historii DM:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Zachowanie wątków:
    
    - wątki Discord są routowane jako sesje kanału
    - metadane wątku nadrzędnego mogą być używane do powiązania z sesją nadrzędną
    - konfiguracja wątku dziedziczy konfigurację kanału nadrzędnego, chyba że istnieje wpis specyficzny dla wątku
    
    Tematy kanałów są wstrzykiwane jako kontekst **niezaufany** (nie jako system prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    `guilds.<id> .reactionNotifications`: tryb zdarzeń systemu reakcji (`off`, `own`, `all`, `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">    `ackReaction` wysyła emoji potwierdzenia, gdy OpenClaw przetwarza przychodzącą wiadomość.

    ```
    Kolejność rozstrzygania:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - zapasowo emoji tożsamości agenta (`agents.list[].identity.emoji`, w przeciwnym razie "👀")
    
    Uwagi:
    
    - Discord akceptuje emoji unicode lub nazwy niestandardowych emoji.
    - Użyj `""`, aby wyłączyć reakcję dla kanału lub konta.
    ```

  
</Accordion>

  <Accordion title="Config writes">.channels`, niewymienione kanały są domyślnie odrzucane.

    ```
    Dotyczy to przepływów `/config set|unset` (gdy funkcje poleceń są włączone).
    
    Wyłącz:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">    Kieruj ruch WebSocket Discord gateway przez proxy HTTP(S) za pomocą `channels.discord.proxy`.

```json5
Reakcje: agent może wyzwalać reakcje przez narzędzie `discord` (kontrolowane przez `channels.discord.actions.*`).
```

    ```
    Nadpisanie per konto:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">`pluralkit`: rozwiązywanie wiadomości proxy PluralKit, aby członkowie systemu byli widoczni jako odrębni nadawcy.

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    Uwagi:
    
    - allowlisty mogą używać `pk:<memberId>`
    - wyświetlane nazwy członków są dopasowywane po nazwie/slug
    - wyszukiwania używają oryginalnego ID wiadomości i są ograniczone oknem czasowym
    - jeśli wyszukiwanie się nie powiedzie, wiadomości proxowane są traktowane jako wiadomości bota i odrzucane, chyba że `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">    Aktualizacje statusu są stosowane tylko wtedy, gdy ustawisz pole statusu lub aktywności.

    ```
    Przykład tylko statusu:
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    Przykład aktywności (niestandardowy status jest domyślnym typem aktywności):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Przykład streamingu:
    ```

```json5
Aby zachować stare zachowanie „otwarte dla wszystkich”: ustaw `channels.discord.dm.policy="open"` i `channels.discord.dm.allowFrom=["*"]`.
```

    ```
    Mapa typów aktywności:
    
    - 0: Playing
    - 1: Streaming (wymaga `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (używa tekstu aktywności jako treści statusu; emoji opcjonalne)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    Discord obsługuje zatwierdzanie wykonania oparte na przyciskach w DM i może opcjonalnie publikować prośby o zatwierdzenie w kanale źródłowym.

    ```
    Ścieżka konfiguracji:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, domyślnie: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    Gdy `target` ma wartość `channel` lub `both`, monit o zatwierdzenie jest widoczny na kanale. Tylko skonfigurowani zatwierdzający mogą używać przycisków; inni użytkownicy otrzymają efemeryczną odmowę. Monity o zatwierdzenie zawierają treść polecenia, dlatego włączaj dostarczanie na kanał tylko w zaufanych kanałach. Jeśli nie można wyprowadzić ID kanału z klucza sesji, OpenClaw wraca do dostarczania przez DM.
    
    Jeśli zatwierdzenia kończą się niepowodzeniem z powodu nieznanych ID zatwierdzeń, zweryfikuj listę zatwierdzających oraz włączenie funkcji.
    
    Powiązana dokumentacja: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Akcje narzędzi

Akcje wiadomości Discord obejmują wysyłanie wiadomości, administrację kanałem, moderację, status obecności oraz działania na metadanych.

Główne przykłady:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- `react` / `reactions` (dodawanie lub listowanie reakcji)
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Bramki akcji znajdują się w `channels.discord.actions.*`.

Domyślne zachowanie bramki:

| Grupa akcji                                                                                                   | Domyślne  |
| ------------------------------------------------------------------------------------------------------------- | --------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | enabled   |
| role                                                                                                          | wyłączone |
| moderacja                                                                                                     | wyłączone |
| presence                                                                                                      | wyłączone |

## Interfejs Components v2

OpenClaw używa Discord components v2 do zatwierdzeń exec oraz znaczników między kontekstami. Akcje wiadomości Discord mogą również przyjmować `components` dla niestandardowego UI (zaawansowane; wymaga instancji komponentów Carbon), natomiast starsze `embeds` pozostają dostępne, ale nie są zalecane.

- `channels.discord.ui.components.accentColor` ustawia kolor akcentu używany przez kontenery komponentów Discord (hex).
- Ustaw dla konkretnego konta za pomocą `channels.discord.accounts.<id>.ui.components.accentColor`.
- `embeds` są ignorowane, gdy obecne są components v2.

Przykład:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
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

## messages

Wiadomości głosowe Discord wyświetlają podgląd fali dźwiękowej i wymagają dźwięku OGG/Opus oraz metadanych. OpenClaw generuje falę dźwiękową automatycznie, ale wymaga dostępności `ffmpeg` i `ffprobe` na hoście gateway do analizy i konwersji plików audio.

Wymagania i ograniczenia:

- Podaj **lokalną ścieżkę pliku** (URL-e są odrzucane).
- Pomiń treść tekstową (Discord nie pozwala na tekst + wiadomość głosową w tym samym ładunku).
- Akceptowany jest dowolny format audio; OpenClaw w razie potrzeby konwertuje do OGG/Opus.

Przykład:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Rozwiązywanie problemów

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    **„Used disallowed intents”**: włącz **Message Content Intent** (i prawdopodobnie **Server Members Intent**) w Developer Portal, następnie zrestartuj gateway.
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `groupPolicy`: kontroluje obsługę kanałów gildii (`open|disabled|allowlist`); `allowlist` wymaga list dozwolonych kanałów.
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked"> 
    Typowe przyczyny:

    ```
    Aby dopuścić **żadne kanały**, ustaw `channels.discord.groupPolicy: "disabled"` (lub pozostaw pustą listę dozwolonych).
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**Audyty uprawnień** (`channels status --probe`) sprawdzają tylko numeryczne identyfikatory kanałów.

    ```
    Jeśli używasz kluczy typu slug, dopasowanie w czasie działania nadal może działać, ale probe nie może w pełni zweryfikować uprawnień.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DM-y nie działają**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"` lub nie zostałeś jeszcze zatwierdzony (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">Wiadomości autorstwa bota są domyślnie ignorowane; ustaw `channels.discord.allowBots=true`, aby je dopuścić (własne wiadomości pozostają filtrowane).

    ```
    Ostrzeżenie: jeśli zezwolisz na odpowiedzi do innych botów (`channels.discord.allowBots=true`), zapobiegaj pętlom bot–bot za pomocą list dozwolonych `requireMention`, `channels.discord.guilds.*.channels.<id>
    ```

  
</Accordion>
</AccordionGroup>

## Wskaźniki do referencji konfiguracji

Główna referencja:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Najważniejsze pola Discord:

- `presence` (status/aktywność bota, domyślnie `false`)
- `guilds.<id> .channels.<channel> .allow`: zezwól/zabroń kanału, gdy `groupPolicy="allowlist"`.
- command: `commands.native`, `commands.useAccessGroups`, `configWrites`
- `dmHistoryLimit`: limit historii DM w turach użytkownika. Nadpisania per użytkownik: `dms["<user_id>"].historyLimit`.
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Bezpieczeństwo i operacje

- Traktuj tokeny bota jako sekrety (`DISCORD_BOT_TOKEN` preferowany w środowiskach nadzorowanych).
- Przyznawaj uprawnienia Discord zgodnie z zasadą najmniejszych uprawnień.
- Jeśli polecenie deploy/state jest nieaktualne, uruchom ponownie gateway i sprawdź ponownie za pomocą `openclaw channels status --probe`.

## Powiązane

- [Pairing](/channels/pairing)
- `channels` (tworzenie/edycja/usuwanie kanałów + kategorii + uprawnień)
- [Troubleshooting](/channels/troubleshooting)
- Pełna lista komend + konfiguracja: [Slash commands](/tools/slash-commands)


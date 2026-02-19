---
summary: "Tryb HTTP (Events API)"
read_when:
  - Konfigurowanie Slacka lub debugowanie trybu gniazda/HTTP w Slacku
title: "Slack"
---

# Slack

Status: gotowe do użycia w środowisku produkcyjnym dla DM + kanałów przez integracje aplikacji Slack. Domyślnym trybem jest Socket Mode; obsługiwany jest również tryb HTTP Events API.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    DM w Slack domyślnie działają w trybie pairing.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Natywne zachowanie poleceń i katalog poleceń.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostyka międzykanałowa i procedury naprawcze.
  
</Card>
</CardGroup>

## Szybka konfiguracja

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        W ustawieniach aplikacji Slack:


        ```
        Utwórz **App Token** (`xapp-...`) oraz **Bot Token** (`xoxb-...`).
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Zmienna środowiskowa jako fallback (tylko konto domyślne):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Subskrybuj zdarzenia aplikacji">
          Subskrybuj zdarzenia bota dla:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Włącz także kartę **Messages Tab** w App Home dla DM.
        
</Step>
      
        <Step title="Uruchom gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
            - ustaw tryb na HTTP (`channels.slack.mode="http"`)
            - skopiuj Slack **Signing Secret**
            - ustaw Event Subscriptions + Interactivity + Slash command Request URL na tę samą ścieżkę webhooka (domyślnie `/slack/events`)
        
          
</Step>
        
          <Step title="Skonfiguruj tryb HTTP w OpenClaw">
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      Tryb HTTP dla wielu kont: ustaw `channels.slack.accounts.<id> .mode = "http"` i zapewnij unikalny
      `webhookPath` dla każdego konta, aby każda aplikacja Slack mogła wskazywać własny adres URL.

  
</Tab>
</Tabs>

## Model tokenów

- `botToken` + `appToken` są wymagane dla Socket Mode.
- Tryb HTTP wymaga `botToken` + `signingSecret`.
- Tokeny w konfiguracji mają pierwszeństwo przed fallbackiem zmiennych środowiskowych.
- Zapasowe użycie zmiennych środowiskowych `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` ma zastosowanie tylko do domyślnego konta.
- `userToken` (`xoxp-...`) jest dostępny tylko w konfiguracji (brak fallbacku z env) i domyślnie działa w trybie tylko do odczytu (`userTokenReadOnly: true`).
- Opcjonalnie: dodaj `chat:write.customize`, jeśli chcesz, aby wiadomości wychodzące używały tożsamości aktywnego agenta (niestandardowa `username` i ikona). `icon_emoji` używa składni `:emoji_name:`.

<Tip>
W przypadku akcji/odczytów katalogu można preferować token użytkownika, jeśli jest skonfigurowany. Nawet przy `userTokenReadOnly: false` token bota pozostaje
preferowany do zapisów, gdy jest dostępny.
</Tip>

## Kontrola dostępu i routing

<Tabs>
  <Tab title="DM policy">DM-y ignorowane: nadawca niezatwierdzony, gdy `channels.slack.dm.policy="pairing"`.

    ```
    {
      channels: {
        slack: {
          replyToMode: "off", // default for channels
          replyToModeByChatType: {
            direct: "all", // DMs always thread
            group: "first", // group DMs/MPIM thread first reply
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` kontroluje obsługę kanałów (`open|disabled|allowlist`).

    ```
    Aby zezwolić wszystkim: ustaw `channels.slack.dm.policy="open"` i `channels.slack.dm.allowFrom=["*"]`.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Wiadomości na kanałach są domyślnie ograniczone do wzmianek.


    ```
    .allowBots=true`), zapobiegaj pętlom odpowiedzi bot–bot za pomocą list dozwolonych `requireMention`, `channels.slack.channels.<id>
    ```

  
</Tab>
</Tabs>

## Polecenia i zachowanie slash

- Natywne polecenia są domyślnie wyłączone dla Slacka, chyba że ustawisz `channels.slack.commands.native: true` (globalne `commands.native` ma wartość `"auto"`, co pozostawia Slack wyłączony).
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- Gdy natywne polecenia są włączone, zarejestruj odpowiadające im polecenia slash w Slack (`/<command>` names).
- Jeśli natywne polecenia nie są włączone, możesz uruchomić jedno skonfigurowane polecenie slash przez `channels.slack.slashCommand`.

Domyślne ustawienia polecenia slash:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Sesje slash używają izolowanych kluczy:

- Polecenia slash używają sesji `agent:<agentId>:slack:slash:<userId>` (prefiks konfigurowalny przez `channels.slack.slashCommand.sessionPrefix`).

i nadal kierują wykonanie polecenia do docelowej sesji konwersacji (`CommandTargetSessionKey`).

## Wątki, sesje i znaczniki odpowiedzi

- DM są kierowane jako `direct`; kanały jako `channel`; MPIM jako `group`.
- Przy domyślnym ustawieniu `session.dmScope=main`, wiadomości prywatne Slack są łączone z główną sesją agenta.
- Kanały mapują się na sesje `agent:<agentId>:slack:channel:<channelId>`.
- Odpowiedzi w wątkach mogą tworzyć sufiksy sesji wątku (`:thread:<threadTs>`) w odpowiednich przypadkach.
- Domyślna wartość `channels.slack.thread.historyScope` to `thread`; domyślna wartość `thread.inheritParent` to `false`.
- `channels.slack.thread.initialHistoryLimit` określa, ile istniejących wiadomości w wątku jest pobieranych przy uruchamianiu nowej sesji wątku (domyślnie `20`; ustaw `0`, aby wyłączyć).

Wątkowanie odpowiedzi

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- starszy mechanizm zapasowy dla czatów bezpośrednich: `channels.slack.dm.replyToMode`

Obsługiwane są ręczne znaczniki odpowiedzi:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Uwaga: `replyToMode="off"` wyłącza niejawne wątkowanie odpowiedzi. Jawne znaczniki `[[reply_to_*]]` są nadal respektowane.

## Media, dzielenie na części i dostarczanie

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Załączniki plików Slack są pobierane z prywatnych adresów URL hostowanych przez Slack (przepływ żądań uwierzytelniany tokenem) i zapisywane w magazynie mediów, gdy pobranie się powiedzie i pozwalają na to limity rozmiaru.

    ```
    Przesyłanie mediów jest ograniczone przez `channels.slack.mediaMaxMb` (domyślnie 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - fragmenty tekstu używają `channels.slack.textChunkLimit` (domyślnie 4000)
    - `channels.slack.chunkMode="newline"` włącza dzielenie najpierw według akapitów
    - wysyłanie plików korzysta z API przesyłania Slack i może obejmować odpowiedzi w wątku (`thread_ts`)
    - limit mediów wychodzących jest zgodny z `channels.slack.mediaMaxMb`, jeśli skonfigurowano; w przeciwnym razie wysyłki kanałowe używają domyślnych wartości typu MIME z potoku mediów
  
</Accordion>

  <Accordion title="Delivery targets">
    Preferowane jawne cele:

    ```
    `im:write` (otwieranie DM-ów przez `conversations.open` dla DM-ów użytkowników)
    [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
    ```

  
</Accordion>
</AccordionGroup>

## Akcje i mechanizmy kontrolne

Akcje narzędzi Slacka można bramkować za pomocą `channels.slack.actions.*`:

Dostępne grupy akcji w bieżącym narzędziu Slack:

| Grupa akcji | Domyślnie |
| ----------- | --------- |
| messages    | włączone  |
| reactions   | włączone  |
| pins        | włączone  |
| memberInfo  | włączone  |
| emojiList   | włączone  |

## Zdarzenia i zachowanie operacyjne

- Edycje/usunięcia wiadomości oraz rozgłaszanie wątków są mapowane na zdarzenia systemowe.
- Zdarzenia dodania/usunięcia reakcji są mapowane na zdarzenia systemowe.
- Zdarzenia dołączenia/opuszczenia członka, utworzenia/zmiany nazwy kanału oraz dodania/usunięcia przypięcia są mapowane na zdarzenia systemowe.
- `channel_id_changed` może migrować klucze konfiguracji kanału, gdy włączone jest `configWrites`.
- Metadane tematu/opisu kanału są traktowane jako niezaufany kontekst i mogą być wstrzykiwane do kontekstu routingu.

## Reakcje + lista reakcji

`ackReaction` wysyła emoji potwierdzenia, gdy OpenClaw przetwarza wiadomość przychodzącą.

Kolejność rozwiązywania:

- `lub`channels.slack.channels.<name>`.ackReaction`
- `channels.slack.ackReaction`
- `replyToMode`
- zapasowe emoji tożsamości agenta (`agents.list[].identity.emoji`, w przeciwnym razie "👀")

Uwagi

- Slack oczekuje shortcode’ów (na przykład `"eyes"`).
- Użyj `""`, aby wyłączyć reakcję dla kanału lub konta.

## Lista kontrolna manifestu i zakresów

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    Jeśli konfigurujesz `channels.slack.userToken`, typowe zakresy odczytu to:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Rozwiązywanie problemów

<AccordionGroup>
  <Accordion title="No replies in channels">
    Sprawdź, w kolejności:

    ```
    `allow`: zezwól/zabroń kanału, gdy `groupPolicy="allowlist"`.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Sprawdź:

    ```
    `allowlist` wymaga, aby kanały były wymienione w `channels.slack.channels`.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">Utwórz aplikację Slack i włącz **Socket Mode**.
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Zweryfikuj:

    ```
    - secret podpisu
    - ścieżkę webhooka
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - unikalną wartość `webhookPath` dla każdego konta HTTP
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Sprawdź, czy zamierzałeś:

    ```
    - natywny tryb poleceń (`channels.slack.commands.native: true`) z odpowiednimi poleceniami slash zarejestrowanymi w Slack
    - lub tryb pojedynczego polecenia slash (`channels.slack.slashCommand.enabled: true`)
    
    Sprawdź również `commands.useAccessGroups` oraz listy dozwolonych kanałów/użytkowników.
    ```

  
</Accordion>
</AccordionGroup>

## Odnośniki do dokumentacji konfiguracji

Priorytet:

- Konfiguracja Slacka w trybie gniazda lub webhooka HTTP

  Pola Slack o wysokim znaczeniu:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - Dostęp do DM: `dm.enabled`, `dmPolicy`, `allowFrom` (legacy: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - Aby zezwolić na **brak kanałów**, ustaw `channels.slack.groupPolicy: "disabled"` (lub pozostaw pustą listę dozwolonych).
  - wątki/historia: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - dostarczanie: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - operacje/funkcje: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Powiązane

- [Parowanie](/channels/pairing)
- `channel`: standardowe kanały (publiczne/prywatne)
- Schemat triage: [/channels/troubleshooting](/channels/troubleshooting).
- [Konfiguracja](/gateway/configuration)
- Pełna lista poleceń + konfiguracja: [Polecenia slash](/tools/slash-commands)

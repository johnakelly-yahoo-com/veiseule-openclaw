---
summary: "Status wsparcia bota Telegram, możliwości i konfiguracja"
read_when:
  - Prace nad funkcjami Telegram lub webhookami
title: "Telegram"
---

# Telegram (Bot API)

Status: gotowe do produkcji dla DM-ów bota + grup przez grammY. Domyślnie long-polling; webhook opcjonalny.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Parowanie jest domyślną wymianą tokenów dla DM-ów Telegrama.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostyka międzykanałowa i instrukcje naprawcze.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Pełne wzorce i przykłady konfiguracji kanałów.
  
</Card>
</CardGroup>

## Szybka konfiguracja

<Steps>
  <Step title="Create the bot token in BotFather">Otwórz Telegram i porozmawiaj z **@BotFather** ([bezpośredni link](https://t.me/BotFather)). Potwierdź, że uchwyt to dokładnie `@BotFather`.

    ```
    Uruchom `/newbot`, a następnie postępuj zgodnie z instrukcjami (nazwa + nazwa użytkownika kończąca się na `bot`).
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Opcja env: `TELEGRAM_BOT_TOKEN=...` (działa dla konta domyślnego).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Nieznani nadawcy otrzymują kod parowania; wiadomości są ignorowane do czasu zatwierdzenia (kody wygasają po 1 godzinie).
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Dodaj bota do swojej grupy, a następnie ustaw `channels.telegram.groups` oraz `groupPolicy`, aby dopasować je do swojego modelu dostępu.
  
</Step>
</Steps>

<Note>
Kolejność rozwiązywania tokenów uwzględnia konto. W praktyce wartości z konfiguracji mają pierwszeństwo przed zmiennymi środowiskowymi, a `TELEGRAM_BOT_TOKEN` dotyczy tylko konta domyślnego.
</Note>

## Ustawienia po stronie Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Boty Telegrama domyślnie mają włączony **Tryb prywatności**, który ogranicza, jakie wiadomości grupowe otrzymują.

    ```
    **Uwaga:** Po przełączeniu trybu prywatności Telegram wymaga usunięcia i ponownego dodania bota
    do każdej grupy, aby zmiana zaczęła obowiązywać.
    ```

  
</Accordion>

  <Accordion title="Group permissions">Status administratora ustawia się w obrębie grupy (interfejs Telegrama).

    ```
    Dodaj bota jako **administratora** grupy (boty admini otrzymują wszystkie wiadomości).
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — zezwól/zabroń dodawania bota do grup.
    ```

  
</Accordion>
</AccordionGroup>

## Kontrola dostępu i aktywacja

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` kontroluje dostęp do wiadomości prywatnych:


    ```
    - `pairing` (domyślnie)
    - `allowlist`
    - `open` (wymaga, aby `allowFrom` zawierało `"*"`)
    - `disabled`
    
    `channels.telegram.allowFrom` akceptuje numeryczne identyfikatory użytkowników Telegram. Prefiksy `telegram:` / `tg:` są akceptowane i normalizowane.
    Kreator wdrożeniowy akceptuje dane wejściowe `@username` i przekształca je na identyfikatory numeryczne.
    Jeśli przeprowadziłeś aktualizację i Twoja konfiguracja zawiera wpisy allowlist w postaci `@username`, uruchom `openclaw doctor --fix`, aby je przekształcić (best-effort; wymaga tokena bota Telegram).
    
    ### Jak znaleźć swój identyfikator użytkownika Telegram
    
    Bezpieczniejsza metoda (bez bota zewnętrznego):
    
    1. Wyślij wiadomość prywatną do swojego bota.
    2. Uruchom `openclaw logs --follow`.
    3. Odczytaj `from.id`.
    
    Oficjalna metoda Bot API:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    **Uwaga dotycząca prywatności:** `@userinfobot` to bot podmiotu trzeciego.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">Dwie niezależne kontrole:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">Odpowiedzi w grupach domyślnie wymagają wzmianki (natywna @wzmianka lub `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).

    ```
    Wzmianka może pochodzić z:
    
    - natywnej wzmianki `@botusername`, lub
    - wzorców wzmianek w:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Przełączniki poleceń na poziomie sesji:
    
    - `/activation always`
    - `/activation mention`
    
    Aktualizują one wyłącznie stan sesji. Aby zachować trwałość, użyj konfiguracji.
    
    Przykład trwałej konfiguracji:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    Jeśli wolisz, dodaj bota do grupy, wyślij wiadomość i użyj `openclaw logs --follow`, aby odczytać `chat.id`, albo użyj Bot API `getUpdates`.
    ```

  
</Tab>
</Tabs>

## Zachowanie w czasie działania

- Telegram jest obsługiwany przez proces gateway.
- Deterministyczne routowanie: odpowiedzi wracają do Telegrama; model nigdy nie wybiera kanałów.
- Wiadomości przychodzące są normalizowane do wspólnej koperty kanału z kontekstem odpowiedzi i placeholderami multimediów.
- Uzyskanie identyfikatora czatu grupy Dołącza `:topic:<threadId>` do klucza sesji grupy Telegrama, aby każdy temat był odizolowany.
- Wysyła wskaźniki pisania i odpowiedzi z `message_thread_id`, aby odpowiedzi pozostawały w temacie.
- Long polling używa grammY runner z sekwencjonowaniem per-czat/per-wątek. Long-polling używa runnera grammY z sekwencjonowaniem per czat; całkowita współbieżność jest ograniczona przez `agents.defaults.maxConcurrent`.
- Telegram Bot API nie obsługuje potwierdzeń odczytu; nie ma opcji `sendReadReceipts`.

## Dokumentacja funkcji

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw może strumieniować częściowe odpowiedzi w DM-ach Telegrama przy użyciu `sendMessageDraft`.

    ```
    Wymaganie:
    
    - `channels.telegram.streamMode` nie jest ustawione na `"off"` (domyślnie: `"partial"`)
    
    Tryby:
    
    - `off`: brak podglądu na żywo
    - `partial`: częste aktualizacje podglądu z częściowego tekstu
    - `block`: porcjowane aktualizacje podglądu z użyciem `channels.telegram.draftChunk`
    
    Domyślne wartości `draftChunk` dla `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` jest ograniczane przez `channels.telegram.textChunkLimit`.
    
    Działa to w czatach prywatnych oraz w grupach/tematach.
    
    W przypadku odpowiedzi zawierających wyłącznie tekst OpenClaw zachowuje tę samą wiadomość podglądu i wykonuje końcową edycję w miejscu (bez drugiej wiadomości).
    
    W przypadku złożonych odpowiedzi (np. z multimediami) OpenClaw wraca do standardowego końcowego dostarczania, a następnie usuwa wiadomość podglądu.
    
    `streamMode` jest niezależny od block streaming. Gdy block streaming jest jawnie włączony dla Telegram, OpenClaw pomija strumień podglądu, aby uniknąć podwójnego strumieniowania.
    
    Strumień rozumowania tylko dla Telegram:
    
    - `/reasoning stream` wysyła rozumowanie do podglądu na żywo podczas generowania
    - końcowa odpowiedź jest wysyłana bez tekstu rozumowania
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">Tekst wychodzący Telegrama używa `parse_mode: "HTML"` (obsługiwany podzbiór tagów Telegrama).

    ```
    - Tekst w stylu Markdown jest renderowany do bezpiecznego dla Telegram HTML.
    - Surowy HTML modelu jest escapowany, aby ograniczyć błędy parsowania Telegram.
    - Jeśli Telegram odrzuci sparsowany HTML, OpenClaw ponowi próbę jako zwykły tekst.
    
    Podglądy linków są domyślnie włączone i można je wyłączyć za pomocą `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Some commands can be handled by plugins/skills without being registered in Telegram’s command menu.

    ```
    `commands.native` (domyślnie `"auto"` → włączone dla Telegram/Discord, wyłączone dla Slack), `commands.text`, `commands.useAccessGroups` (zachowanie poleceń). Nadpisz przez `channels.telegram.commands.native`.
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Zasady:
    
    - nazwy są normalizowane (usunięcie wiodącego `/`, małe litery)
    - poprawny wzorzec: `a-z`, `0-9`, `_`, długość `1..32`
    - polecenia niestandardowe nie mogą nadpisywać poleceń natywnych
    - konflikty/duplikaty są pomijane i logowane
    
    Uwagi:
    
    - polecenia niestandardowe są tylko pozycjami menu; nie implementują automatycznie działania
    - polecenia pluginów/Skills mogą nadal działać po wpisaniu, nawet jeśli nie są widoczne w menu Telegram
    
    Jeśli polecenia natywne są wyłączone, wbudowane zostają usunięte. Polecenia niestandardowe/pluginów mogą nadal się rejestrować, jeśli są skonfigurowane.
    
    Częsta przyczyna błędu konfiguracji:
    
    - `setMyCommands failed` zwykle oznacza, że wychodzące DNS/HTTPS do `api.telegram.org` jest zablokowane.
    
    ### Polecenia parowania urządzenia (plugin `device-pair`)
    
    Gdy plugin `device-pair` jest zainstalowany:
    
    1. `/pair` generuje kod konfiguracji
    2. wklej kod w aplikacji iOS
    3. `/pair approve` zatwierdza najnowsze oczekujące żądanie
    
    Więcej szczegółów: [Parowanie](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    Skonfiguruj zakres klawiatury inline:


```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Dla konfiguracji per konto:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `"disabled"` = żadnych wiadomości grupowych w ogóle
      Domyślnie `groupPolicy: "allowlist"` (zablokowane, dopóki nie dodasz `groupAllowFrom`).
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Gdy użytkownik kliknie przycisk, dane callback są wysyłane z powrotem do agenta jako wiadomość w formacie:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">    Działania narzędzi Telegram obejmują:

    ```
    - `sendMessage` (`to`, `content`, opcjonalnie `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Akcje wiadomości kanału udostępniają ergonomiczne aliasy (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Kontrola dostępu:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (domyślnie: wyłączone)
    
    Semantyka usuwania reakcji: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram obsługuje opcjonalne odpowiedzi w wątkach za pomocą tagów:

    ```
    - `[[reply_to_current]]` odpowiada na wiadomość wywołującą
    - `[[reply_to:<id>]]` odpowiada na konkretny identyfikator wiadomości Telegram
    
    `channels.telegram.replyToMode` kontroluje obsługę:
    
    - `off` (domyślnie)
    - `first`
    - `all`
    
    Uwaga: `off` wyłącza domyślne wątkowanie odpowiedzi. Jawne tagi `[[reply_to_*]]` są nadal respektowane.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Tematy (supergrupy forum)

    ```
    **Grupy forum:** Reakcje w grupach forum zawierają `message_thread_id` i używają kluczy sesji takich jak `agent:main:telegram:group:{chatId}:topic:{threadId}`.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Wiadomości audio


    ```
    `[[audio_as_voice]]` — wyślij audio jako notatkę głosową zamiast pliku.
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    Video messages (video vs video note)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Notatki wideo nie obsługują podpisów; dostarczony tekst wiadomości jest wysyłany osobno.
    
    ### Naklejki
    
    Obsługa naklejek przychodzących:
    
    - statyczne WEBP: pobierane i przetwarzane (placeholder `<media:sticker>`)
    - animowane TGS: pomijane
    - wideo WEBM: pomijane
    
    Pola kontekstu naklejki:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Plik cache naklejek:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Naklejki są opisywane jednokrotnie (gdy to możliwe) i zapisywane w cache, aby ograniczyć powtarzane wywołania vision.
    
    Włącz akcje naklejek:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Wysyłanie naklejek
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Zwraca pasujące naklejki z cache:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Otrzymuje aktualizację `message_reaction` z Telegram API

    ```
    Po włączeniu OpenClaw umieszcza w kolejce zdarzenia systemowe, takie jak:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Konfiguracja:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (domyślnie: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (domyślnie: `minimal`)
    
    Uwagi:
    
    - `own` oznacza reakcje użytkowników na wiadomości wysłane przez bota (best-effort na podstawie cache wysłanych wiadomości).
    - Telegram nie udostępnia identyfikatorów wątków w aktualizacjach reakcji.
      - grupy bez forum są kierowane do sesji czatu grupowego
      - grupy forum są kierowane do sesji ogólnego tematu grupy (`:topic:1`), a nie do dokładnego tematu źródłowego
    
    `allowed_updates` dla polling/webhook automatycznie obejmuje `message_reaction`.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` wysyła emoji potwierdzenia, gdy OpenClaw przetwarza wiadomość przychodzącą.


    ```
    Kolejność rozwiązywania:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - emoji tożsamości agenta jako fallback (`agents.list[].identity.emoji`, w przeciwnym razie "👀")
    
    Uwagi:
    
    - Telegram oczekuje emoji w formacie unicode (np. "👀").
    - Użyj `""`, aby wyłączyć reakcję dla kanału lub konta.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Zapisy konfiguracji kanału są domyślnie włączone (`configWrites !== false`).

    ```
    Zapisy wywołane przez Telegram obejmują:
    
    - zdarzenia migracji grupy (`migrate_to_chat_id`) w celu aktualizacji `channels.telegram.groups`
    - `/config set` oraz `/config unset` (wymaga włączenia komend)
    
    Wyłącz:
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">Domyślnie: long-polling (nie wymaga publicznego URL).

    ```
    W trybie webhook reakcje są dołączane do webhooka `allowed_updates`
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Tekst wychodzący jest dzielony na kawałki do `channels.telegram.textChunkLimit` (domyślnie 4000).
    Opcjonalne dzielenie po nowych liniach: ustaw `channels.telegram.chunkMode="newline"`, aby dzielić po pustych liniach (granice akapitów) przed dzieleniem długości.
    Pobieranie/wysyłanie multimediów jest ograniczone do `channels.telegram.mediaMaxMb` (domyślnie 5).
    Żądania Telegram Bot API wygasają po `channels.telegram.timeoutSeconds` (domyślnie 500 przez grammY).
    Kontekst historii grup używa `channels.telegram.historyLimit` (lub `channels.telegram.accounts.*.historyLimit`), z fallbackiem do `messages.groupChat.historyLimit`. Ustaw `0`, aby wyłączyć (domyślnie 50).
    Historia DM-ów może być ograniczona przez `channels.telegram.dmHistoryLimit` (tury użytkownika). Nadpisania per użytkownik: `channels.telegram.dms["<user_id>"].historyLimit`.<user_id>"].historyLimit`
    - ponowienia wywołań wychodzącego API Telegram są konfigurowalne przez `channels.telegram.retry`.

    ```
    Cel wysyłki w CLI może być numerycznym ID czatu lub nazwą użytkownika:
    ```

```bash
Przykład: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Rozwiązywanie problemów

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    Jeśli ustawiono `channels.telegram.groups.*.requireMention=false`, **tryb prywatności** Telegram Bot API musi być wyłączony.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - gdy `channels.telegram.groups` istnieje, grupa musi być wymieniona (lub zawierać `"*"`)
    - zweryfikuj członkostwo bota w grupie
    - sprawdź logi: `openclaw logs --follow`, aby poznać powody pominięcia
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    `setMyCommands failed` w logach zwykle oznacza zablokowane wyjściowe HTTPS/DNS do `api.telegram.org`.
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + niestandardowy fetch/proxy mogą powodować natychmiastowe przerwanie działania, jeśli typy AbortSignal są niezgodne.
    - Niektóre hosty najpierw rozwiązują `api.telegram.org` do IPv6; uszkodzony routing IPv6 może powodować sporadyczne błędy API Telegram.
    - Zweryfikuj odpowiedzi DNS:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Więcej pomocy: [Rozwiązywanie problemów kanałów](/channels/troubleshooting).

## Referencja konfiguracji (Telegram)

Główne odniesienie:

- `channels.telegram.enabled`: włącz/wyłącz start kanału.

- `channels.telegram.botToken`: token bota (BotFather).

- `channels.telegram.tokenFile`: odczytaj token ze ścieżki pliku.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (domyślnie: parowanie).

- `channels.telegram.allowFrom`: lista dozwolonych DM-ów (id/nazwy użytkowników). `open` wymaga `"*"`. `openclaw doctor --fix` może rozwiązać starsze wpisy `@username` do ID.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (domyślnie: lista dozwolonych).

- `channels.telegram.groupAllowFrom`: lista dozwolonych nadawców grupowych (id/nazwy użytkowników). `openclaw doctor --fix` może rozwiązać starsze wpisy `@username` do ID.

- `channels.telegram.groups`: domyślne ustawienia per grupa + lista dozwolonych (użyj `"*"` dla domyślnych globalnych).
  - `channels.telegram.groups.<id>.groupPolicy`: nadpisanie per grupa dla groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: domyślne bramkowanie wzmianek.
  - `channels.telegram.groups.<id>.skills`: filtr skills (pominięcie = wszystkie skills, puste = brak).
  - `channels.telegram.groups.<id>.allowFrom`: nadpisanie listy dozwolonych nadawców per grupa.
  - `channels.telegram.groups.<id>.systemPrompt`: dodatkowy prompt systemowy dla grupy.
  - `channels.telegram.groups.<id>.enabled`: wyłącz grupę, gdy `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: nadpisania per temat (te same pola co grupa).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: nadpisanie per temat dla groupPolicy (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: nadpisanie bramkowania wzmianek per temat.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (domyślnie: lista dozwolonych).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: nadpisanie per konto.

- `channels.telegram.replyToMode`: `off | first | all` (domyślnie: `first`).

- `channels.telegram.textChunkLimit`: rozmiar kawałka wyjściowego (znaki).

- `channels.telegram.chunkMode`: `length` (domyślne) lub `newline`, aby dzielić po pustych liniach (granice akapitów) przed dzieleniem długości.

- `channels.telegram.linkPreview`: przełącz podglądy linków dla wiadomości wychodzących (domyślnie: true).

- `channels.telegram.streamMode`: `off | partial | block` (strumieniowanie szkiców).

- `channels.telegram.mediaMaxMb`: limit multimediów przychodzących/wychodzących (MB).

- `channels.telegram.retry`: polityka ponowień dla wywołań Telegram API (liczba prób, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: nadpisanie Node autoSelectFamily (true=włącz, false=wyłącz). Domyślnie wyłączone na Node 22, aby uniknąć timeoutów Happy Eyeballs.

- `channels.telegram.proxy`: URL proxy dla wywołań Bot API (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: włącz tryb webhook (wymaga `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: sekret webhooka (wymagany, gdy webhookUrl jest ustawiony).

- `channels.telegram.webhookPath`: lokalna ścieżka webhooka (domyślnie `/telegram-webhook`).

- Lokalny listener wiąże się z `0.0.0.0:8787` i domyślnie serwuje `POST /telegram-webhook`.

- `channels.telegram.actions.reactions`: bramkuj reakcje narzędzia Telegrama.

- `channels.telegram.actions.sendMessage`: bramkuj wysyłanie wiadomości narzędzia Telegrama.

- `channels.telegram.actions.deleteMessage`: bramkuj usuwanie wiadomości narzędzia Telegrama.

- `channels.telegram.actions.sticker`: bramkuj akcje naklejek Telegrama — wysyłanie i wyszukiwanie (domyślnie: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — kontroluj, które reakcje wyzwalają zdarzenia systemowe (domyślnie: `own`, gdy nie ustawiono).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — kontroluj zdolność agenta do reagowania (domyślnie: `minimal`, gdy nie ustawiono).

- Uruchomisz `/config set` lub `/config unset` w czacie Telegrama (wymaga `commands.config: true`).

Pola specyficzne dla Telegram o wysokim znaczeniu:

- uruchamianie/uwierzytelnianie: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- `allowlist` — DM-y + grupy, ale tylko nadawcy dozwoleni przez `allowFrom`/`groupAllowFrom` (te same zasady co dla poleceń sterujących)
- komendy/menu: `commands.native`, `customCommands`
- wątki/odpowiedzi: `replyToMode`
- Opcjonalnie (tylko dla `streamMode: "block"`):
- formatowanie/dostarczanie: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/sieć: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- Tryb webhook: ustaw `channels.telegram.webhookUrl` i `channels.telegram.webhookSecret` (opcjonalnie `channels.telegram.webhookPath`).
- akcje/możliwości: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- Powiadomienia o reakcjach
- zapisy/historia: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Powiązane

- Szczegóły: [Parowanie](/channels/pairing)
- [Routing kanałów](/channels/channel-routing)
- Rozwiązywanie problemów konfiguracji (polecenia)


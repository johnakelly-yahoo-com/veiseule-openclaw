---
title: "Telegram"
---

# Telegram (Bot API)

Status: gotowe do produkcji dla DM-ów bota + grup przez grammY. Domyślnie long-polling; webhook opcjonalny.

## Szybka konfiguracja (dla początkujących)

1. Utwórz bota za pomocą **@BotFather** ([bezpośredni link](https://t.me/BotFather)). Potwierdź, że uchwyt to dokładnie `@BotFather`, a następnie skopiuj token.
2. Ustaw token:
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - Lub konfiguracja: `channels.telegram.botToken: "..."`.
   - Jeśli oba są ustawione, konfiguracja ma pierwszeństwo (fallback do env dotyczy tylko konta domyślnego).
3. Uruchom gateway.
4. Dostęp do DM-ów domyślnie wymaga parowania; zatwierdź kod parowania przy pierwszym kontakcie.

Minimalna konfiguracja:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
    },
  },
}
```

## Czym to jest

- Kanał Telegram Bot API należący do Gateway.
- Deterministyczne routowanie: odpowiedzi wracają do Telegrama; model nigdy nie wybiera kanałów.
- DM-y współdzielą główną sesję agenta; grupy pozostają odizolowane (`agent:<agentId>:telegram:group:<chatId>`).

## Konfiguracja (szybka ścieżka)

### 1. Utwórz token bota (BotFather)

1. Otwórz Telegram i porozmawiaj z **@BotFather** ([bezpośredni link](https://t.me/BotFather)). Potwierdź, że uchwyt to dokładnie `@BotFather`.
2. Uruchom `/newbot`, a następnie postępuj zgodnie z instrukcjami (nazwa + nazwa użytkownika kończąca się na `bot`).
3. Skopiuj token i przechowuj go w bezpiecznym miejscu.

Opcjonalne ustawienia BotFather:

- `/setjoingroups` — zezwól/zabroń dodawania bota do grup.
- `/setprivacy` — kontroluj, czy bot widzi wszystkie wiadomości w grupach.

### 2. Skonfiguruj token (env lub konfiguracja)

Przykład:

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

Opcja env: `TELEGRAM_BOT_TOKEN=...` (działa dla konta domyślnego).
Jeśli ustawione są zarówno env, jak i konfiguracja, pierwszeństwo ma konfiguracja.

Obsługa wielu kont: użyj `channels.telegram.accounts` z tokenami per konto i opcjonalnym `name`. Zobacz [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) dla wspólnego wzorca.

3. Uruchom gateway. Telegram startuje, gdy token zostanie rozpoznany (najpierw konfiguracja, fallback do env).
4. Dostęp do DM-ów domyślnie wymaga parowania. Zatwierdź kod przy pierwszym kontakcie z botem.
5. Dla grup: dodaj bota, zdecyduj o zachowaniu prywatności/uprawnieniach admina (poniżej), a następnie ustaw `channels.telegram.groups`, aby kontrolować bramkowanie wzmianek + listy dozwolonych.

## Token + prywatność + uprawnienia (po stronie Telegrama)

### Tworzenie tokena (BotFather)

- `/newbot` tworzy bota i zwraca token (zachowaj go w tajemnicy).
- Jeśli token wycieknie, unieważnij/wygeneruj go ponownie przez @BotFather i zaktualizuj konfigurację.

### Widoczność wiadomości w grupach (Tryb prywatności)

Boty Telegrama domyślnie mają włączony **Tryb prywatności**, który ogranicza, jakie wiadomości grupowe otrzymują.
Jeśli bot musi widzieć _wszystkie_ wiadomości w grupie, masz dwie opcje:

- Wyłącz tryb prywatności za pomocą `/setprivacy` **lub**
- Dodaj bota jako **administratora** grupy (boty admini otrzymują wszystkie wiadomości).

**Uwaga:** Po przełączeniu trybu prywatności Telegram wymaga usunięcia i ponownego dodania bota
do każdej grupy, aby zmiana zaczęła obowiązywać.

### Uprawnienia grupowe (prawa administratora)

Status administratora ustawia się w obrębie grupy (interfejs Telegrama). Boty admini zawsze otrzymują wszystkie
wiadomości w grupie, więc użyj admina, jeśli potrzebujesz pełnej widoczności.

## Jak to działa (zachowanie)

- Wiadomości przychodzące są normalizowane do wspólnej koperty kanału z kontekstem odpowiedzi i placeholderami multimediów.
- Odpowiedzi w grupach domyślnie wymagają wzmianki (natywna @wzmianka lub `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
- Nadpisanie wieloagentowe: ustaw wzorce per agent w `agents.list[].groupChat.mentionPatterns`.
- Odpowiedzi zawsze wracają do tego samego czatu Telegrama.
- Long-polling używa runnera grammY z sekwencjonowaniem per czat; całkowita współbieżność jest ograniczona przez `agents.defaults.maxConcurrent`.
- Telegram Bot API nie obsługuje potwierdzeń odczytu; nie ma opcji `sendReadReceipts`.

## Szkic streamingu

OpenClaw może strumieniować częściowe odpowiedzi w DM-ach Telegrama przy użyciu `sendMessageDraft`.

Wymagania:

- Włączony Tryb wątków dla bota w @BotFather (tryb tematów forum).
- Tylko prywatne wątki czatu (Telegram dołącza `message_thread_id` do wiadomości przychodzących).
- `channels.telegram.streamMode` nie ustawione na `"off"` (domyślnie: `"partial"`, `"block"` włącza aktualizacje szkicu w kawałkach).

Strumieniowanie szkiców działa tylko w DM-ach; Telegram nie obsługuje go w grupach ani kanałach.

## Formatowanie (HTML Telegrama)

- Tekst wychodzący Telegrama używa `parse_mode: "HTML"` (obsługiwany podzbiór tagów Telegrama).
- Wejście „markdownopodobne” jest renderowane do **bezpiecznego HTML dla Telegrama** (pogrubienie/kursywa/przekreślenie/kod/linki); elementy blokowe są spłaszczane do tekstu z nowymi liniami/punktami.
- Surowy HTML z modeli jest escapowany, aby uniknąć błędów parsowania Telegrama.
- Jeśli Telegram odrzuci ładunek HTML, OpenClaw ponawia wysyłkę tej samej wiadomości jako zwykły tekst.

## Polecenia (natywne + niestandardowe)

OpenClaw rejestruje natywne polecenia (takie jak `/status`, `/reset`, `/model`) w menu bota Telegrama przy starcie.
Możesz dodać niestandardowe polecenia do menu przez konfigurację:

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

## Rozwiązywanie problemów konfiguracji (polecenia)

- `setMyCommands failed` w logach zwykle oznacza zablokowane wyjściowe HTTPS/DNS do `api.telegram.org`.
- Jeśli widzisz błędy `sendMessage` lub `sendChatAction`, sprawdź trasowanie IPv6 i DNS.

Więcej pomocy: [Rozwiązywanie problemów kanałów](/channels/troubleshooting).

Uwagi:

- Niestandardowe polecenia to **wyłącznie wpisy menu**; OpenClaw ich nie implementuje, chyba że obsłużysz je gdzie indziej.
- Some commands can be handled by plugins/skills without being registered in Telegram’s command menu. These still work when typed (they just won't show up in `/commands` / the menu).
- Nazwy poleceń są normalizowane (usuwany wiodący `/`, zamieniane na małe litery) i muszą pasować do `a-z`, `0-9`, `_` (1–32 znaki).
- Niestandardowe polecenia **nie mogą nadpisywać poleceń natywnych**. Konflikty są ignorowane i logowane.
- Jeśli `commands.native` jest wyłączone, rejestrowane są tylko niestandardowe polecenia (lub czyszczone, jeśli ich brak).

### Device pairing commands (`device-pair` plugin)

If the `device-pair` plugin is installed, it adds a Telegram-first flow for pairing a new phone:

1. `/pair` generates a setup code (sent as a separate message for easy copy/paste).
2. Wklej kod konfiguracji w aplikacji iOS, aby się połączyć.
3. `/pair approve` approves the latest pending device request.

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## Limity

- Tekst wychodzący jest dzielony na kawałki do `channels.telegram.textChunkLimit` (domyślnie 4000).
- Opcjonalne dzielenie po nowych liniach: ustaw `channels.telegram.chunkMode="newline"`, aby dzielić po pustych liniach (granice akapitów) przed dzieleniem długości.
- Pobieranie/wysyłanie multimediów jest ograniczone do `channels.telegram.mediaMaxMb` (domyślnie 5).
- Żądania Telegram Bot API wygasają po `channels.telegram.timeoutSeconds` (domyślnie 500 przez grammY). Ustaw niżej, aby uniknąć długich zawieszeń.
- Kontekst historii grup używa `channels.telegram.historyLimit` (lub `channels.telegram.accounts.*.historyLimit`), z fallbackiem do `messages.groupChat.historyLimit`. Ustaw `0`, aby wyłączyć (domyślnie 50).
- Historia DM-ów może być ograniczona przez `channels.telegram.dmHistoryLimit` (tury użytkownika). Nadpisania per użytkownik: `channels.telegram.dms["<user_id>"].historyLimit`.

## Tryby aktywacji w grupach

Domyślnie bot odpowiada w grupach tylko na wzmianki (`@botname` lub wzorce w `agents.list[].groupChat.mentionPatterns`). Aby zmienić to zachowanie:

### Przez konfigurację (zalecane)

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // always respond in this group
      },
    },
  },
}
```

**Ważne:** Ustawienie `channels.telegram.groups` tworzy **listę dozwolonych** — akceptowane będą tylko wymienione grupy (lub `"*"`).
Tematy forum dziedziczą konfigurację grupy nadrzędnej (allowFrom, requireMention, skills, prompty), chyba że dodasz nadpisania per temat w `channels.telegram.groups.<groupId>.topics.<topicId>`.

Aby zezwolić wszystkim grupom na zawsze-odpowiadanie:

```json5
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

Aby zachować tryb tylko-wzmianki dla wszystkich grup (zachowanie domyślne):

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

### Przez polecenie (poziom sesji)

Wyślij w grupie:

- `/activation always` — odpowiadaj na wszystkie wiadomości
- `/activation mention` — wymagaj wzmianek (domyślne)

**Uwaga:** Polecenia aktualizują tylko stan sesji. Dla trwałego zachowania po restartach użyj konfiguracji.

### Uzyskanie identyfikatora czatu grupy

Przekaż dowolną wiadomość z grupy do `@userinfobot` lub `@getidsbot` w Telegramie, aby zobaczyć identyfikator czatu (liczba ujemna, np. `-1001234567890`).

**Wskazówka:** Aby poznać własny identyfikator użytkownika, wyślij DM do bota — odpowie identyfikatorem (wiadomość parowania), albo użyj `/whoami` po włączeniu poleceń.

**Uwaga dotycząca prywatności:** `@userinfobot` to bot podmiotu trzeciego. Jeśli wolisz, dodaj bota do grupy, wyślij wiadomość i użyj `openclaw logs --follow`, aby odczytać `chat.id`, albo użyj Bot API `getUpdates`.

## Zapisy konfiguracji

Domyślnie Telegram ma prawo zapisywać aktualizacje konfiguracji wyzwalane zdarzeniami kanału lub `/config set|unset`.

Dzieje się to, gdy:

- Grupa zostanie uaktualniona do supergrupy i Telegram wyemituje `migrate_to_chat_id` (zmiana ID czatu). OpenClaw może automatycznie migrować `channels.telegram.groups`.
- Uruchomisz `/config set` lub `/config unset` w czacie Telegrama (wymaga `commands.config: true`).

Wyłącz za pomocą:

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## Tematy (supergrupy forum)

Tematy forum Telegrama zawierają `message_thread_id` na wiadomość. OpenClaw:

- Dołącza `:topic:<threadId>` do klucza sesji grupy Telegrama, aby każdy temat był odizolowany.
- Wysyła wskaźniki pisania i odpowiedzi z `message_thread_id`, aby odpowiedzi pozostawały w temacie.
- Temat ogólny (id wątku `1`) jest specjalny: wysyłanie wiadomości pomija `message_thread_id` (Telegram je odrzuca), ale wskaźniki pisania nadal je zawierają.
- Udostępnia `MessageThreadId` + `IsForum` w kontekście szablonu do routingu/templatingu.
- Konfiguracja specyficzna dla tematu jest dostępna w `channels.telegram.groups.<chatId>.topics.<threadId>` (skills, listy dozwolonych, auto-odpowiedź, prompty systemowe, wyłączenie).
- Konfiguracje tematów dziedziczą ustawienia grupy (requireMention, listy dozwolonych, skills, prompty, włączone), chyba że zostaną nadpisane per temat.

Czaty prywatne mogą w niektórych przypadkach zawierać `message_thread_id`. OpenClaw pozostawia klucz sesji DM bez zmian, ale nadal używa identyfikatora wątku do odpowiedzi/strumieniowania szkiców, gdy jest obecny.

## Przyciski inline

Telegram obsługuje klawiatury inline z przyciskami callback.

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

Dla konfiguracji per konto:

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

Koszyki:

- `off` — przyciski inline wyłączone
- `dm` — tylko DM-y (cele grupowe zablokowane)
- `group` — tylko grupy (cele DM zablokowane)
- `all` — DM-y + grupy
- `allowlist` — DM-y + grupy, ale tylko nadawcy dozwoleni przez `allowFrom`/`groupAllowFrom` (te same zasady co dla poleceń sterujących)

Domyślnie: `allowlist`.
Starsze: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

### Wysyłanie przycisków

Użyj narzędzia wiadomości z parametrem `buttons`:

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

Gdy użytkownik kliknie przycisk, dane callback są wysyłane z powrotem do agenta jako wiadomość w formacie:
`callback_data: value`

### Opcje konfiguracji

Możliwości Telegrama można konfigurować na dwóch poziomach (pokazana forma obiektowa; starsze tablice stringów nadal obsługiwane):

- `channels.telegram.capabilities`: Globalna domyślna konfiguracja możliwości stosowana do wszystkich kont Telegrama, chyba że zostanie nadpisana.
- `channels.telegram.accounts.<account>.capabilities`: Możliwości per konto, które nadpisują globalne domyślne dla danego konta.

Użyj ustawienia globalnego, gdy wszystkie boty/konta Telegrama mają zachowywać się tak samo. Użyj konfiguracji per konto, gdy różne boty potrzebują różnych zachowań (np. jedno konto obsługuje tylko DM-y, a inne jest dozwolone w grupach).

## Kontrola dostępu (DM-y + grupy)

### Dostęp do DM-ów

- Domyślnie: `channels.telegram.dmPolicy = "pairing"`. Nieznani nadawcy otrzymują kod parowania; wiadomości są ignorowane do czasu zatwierdzenia (kody wygasają po 1 godzinie).
- Zatwierdzanie przez:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- Parowanie jest domyślną wymianą tokenów dla DM-ów Telegrama. Szczegóły: [Parowanie](/channels/pairing)
- `channels.telegram.allowFrom` akceptuje numeryczne identyfikatory użytkowników (zalecane) lub wpisy `@username`. To **nie** jest nazwa użytkownika bota; użyj identyfikatora nadawcy (człowieka). Kreator akceptuje `@username` i w miarę możliwości rozwiązuje go do identyfikatora numerycznego.

#### Znajdowanie identyfikatora użytkownika Telegrama

Bezpieczniej (bez bota podmiotu trzeciego):

1. Uruchom gateway i wyślij DM do bota.
2. Uruchom `openclaw logs --follow` i poszukaj `from.id`.

Alternatywa (oficjalne Bot API):

1. Wyślij DM do bota.
2. Pobierz aktualizacje z tokenem bota i odczytaj `message.from.id`:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Podmiot trzeci (mniej prywatne):

- Wyślij DM do `@userinfobot` lub `@getidsbot` i użyj zwróconego identyfikatora użytkownika.

### Dostęp do grup

Dwie niezależne kontrole:

**1. Które grupy są dozwolone** (lista dozwolonych grup przez `channels.telegram.groups`):

- Brak konfiguracji `groups` = wszystkie grupy dozwolone
- Z konfiguracją `groups` = dozwolone tylko wymienione grupy lub `"*"`
- Przykład: `"groups": { "-1001234567890": {}, "*": {} }` zezwala na wszystkie grupy

**2. Którzy nadawcy są dozwoleni** (filtrowanie nadawców przez `channels.telegram.groupPolicy`):

- `"open"` = wszyscy nadawcy w dozwolonych grupach mogą pisać
- `"allowlist"` = tylko nadawcy z `channels.telegram.groupAllowFrom` mogą pisać
- `"disabled"` = żadnych wiadomości grupowych w ogóle
  Domyślnie `groupPolicy: "allowlist"` (zablokowane, dopóki nie dodasz `groupAllowFrom`).

Większość użytkowników chce: `groupPolicy: "allowlist"` + `groupAllowFrom` + konkretne grupy wymienione w `channels.telegram.groups`

Aby zezwolić **dowolnemu członkowi grupy** na rozmowę w konkretnej grupie (zachowując jednocześnie ograniczenia poleceń sterujących do autoryzowanych nadawców), ustaw nadpisanie per grupę:

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

## Long-polling vs webhook

- Domyślnie: long-polling (nie wymaga publicznego URL).
- Tryb webhook: ustaw `channels.telegram.webhookUrl` i `channels.telegram.webhookSecret` (opcjonalnie `channels.telegram.webhookPath`).
  - Lokalny listener wiąże się z `0.0.0.0:8787` i domyślnie serwuje `POST /telegram-webhook`.
  - Jeśli publiczny URL jest inny, użyj reverse proxy i skieruj `channels.telegram.webhookUrl` na publiczny endpoint.

## Wątkowanie odpowiedzi

Telegram obsługuje opcjonalne odpowiedzi w wątkach za pomocą tagów:

- `[[reply_to_current]]` -- odpowiedź na wyzwalającą wiadomość.
- `[[reply_to:<id>]]` -- odpowiedź na konkretny identyfikator wiadomości.

Sterowane przez `channels.telegram.replyToMode`:

- `first` (domyślnie), `all`, `off`.

## Wiadomości audio (głos vs plik)

Telegram rozróżnia **notatki głosowe** (okrągła chmurka) od **plików audio** (karta z metadanymi).
OpenClaw domyślnie używa plików audio dla zgodności wstecznej.

Aby wymusić chmurkę notatki głosowej w odpowiedziach agenta, dołącz ten tag w dowolnym miejscu odpowiedzi:

- `[[audio_as_voice]]` — wyślij audio jako notatkę głosową zamiast pliku.

Tag jest usuwany z dostarczonego tekstu. Inne kanały ignorują ten tag.

Dla wysyłek narzędziem wiadomości ustaw `asVoice: true` z kompatybilnym z głosem adresem URL `media`
(`message` jest opcjonalne, gdy media są obecne):

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## Video messages (video vs video note)

Telegram distinguishes **video notes** (round bubble) from **video files** (rectangular).
OpenClaw defaults to video files.

For message tool sends, set `asVideoNote: true` with a video `media` URL:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(Uwaga: Notatki wideo nie obsługują napisów. If you provide a message text, it will be sent as a separate message.)

## Naklejki

OpenClaw obsługuje odbieranie i wysyłanie naklejek Telegrama z inteligentnym buforowaniem.

### Odbieranie naklejek

Gdy użytkownik wysyła naklejkę, OpenClaw obsługuje ją w zależności od typu:

- **Naklejki statyczne (WEBP):** Pobierane i przetwarzane przez wizję. Naklejka pojawia się jako placeholder `<media:sticker>` w treści wiadomości.
- **Naklejki animowane (TGS):** Pomijane (format Lottie nie jest obsługiwany do przetwarzania).
- **Naklejki wideo (WEBM):** Pomijane (format wideo nie jest obsługiwany do przetwarzania).

Pole kontekstu szablonu dostępne przy odbieraniu naklejek:

- `Sticker` — obiekt z:
  - `emoji` — emoji powiązane z naklejką
  - `setName` — nazwa zestawu naklejek
  - `fileId` — identyfikator pliku Telegrama (umożliwia odesłanie tej samej naklejki)
  - `fileUniqueId` — stabilny identyfikator do wyszukiwania w cache
  - `cachedDescription` — zbuforowany opis wizji, gdy dostępny

### Cache naklejek

Naklejki są przetwarzane przez możliwości wizyjne AI w celu generowania opisów. Ponieważ te same naklejki są często wysyłane wielokrotnie, OpenClaw buforuje te opisy, aby uniknąć zbędnych wywołań API.

**Jak to działa:**

1. **Pierwsze spotkanie:** Obraz naklejki jest wysyłany do AI do analizy wizyjnej. AI generuje opis (np. „Kreskówkowy kot entuzjastycznie machający”).
2. **Zapis w cache:** Opis jest zapisywany wraz z identyfikatorem pliku, emoji i nazwą zestawu.
3. **Kolejne spotkania:** Gdy ta sama naklejka pojawi się ponownie, używany jest opis z cache. Obraz nie jest wysyłany do AI.

**Lokalizacja cache:** `~/.openclaw/telegram/sticker-cache.json`

**Format wpisu cache:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Korzyści:**

- Redukcja kosztów API dzięki unikaniu powtarzanych wywołań wizji dla tej samej naklejki
- Szybsze czasy odpowiedzi dla zbuforowanych naklejek (brak opóźnienia przetwarzania wizji)
- Umożliwia wyszukiwanie naklejek na podstawie zbuforowanych opisów

Cache jest wypełniany automatycznie w miarę odbierania naklejek. Nie jest wymagana ręczna administracja cache.

### Wysyłanie naklejek

Agent może wysyłać i wyszukiwać naklejki za pomocą akcji `sticker` i `sticker-search`. Są one domyślnie wyłączone i muszą zostać włączone w konfiguracji:

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

**Wyślij naklejkę:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Parametry:

- `fileId` (wymagane) — identyfikator pliku Telegrama naklejki. Uzyskaj go z `Sticker.fileId` przy odbiorze naklejki lub z wyniku `sticker-search`.
- `replyTo` (opcjonalne) — identyfikator wiadomości, na którą odpowiedzieć.
- `threadId` (opcjonalne) — identyfikator wątku wiadomości dla tematów forum.

**Wyszukiwanie naklejek:**

Agent może przeszukiwać zbuforowane naklejki po opisie, emoji lub nazwie zestawu:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Zwraca pasujące naklejki z cache:

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

Wyszukiwanie używa dopasowania rozmytego w tekście opisu, znakach emoji i nazwach zestawów.

**Przykład z wątkowaniem:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## Strumieniowanie (szkicy)

Telegram może strumieniować **dymki szkiców** podczas generowania odpowiedzi przez agenta.
OpenClaw używa Bot API `sendMessageDraft` (nie są to prawdziwe wiadomości), a następnie wysyła
odpowiedź końcową jako zwykłą wiadomość.

Wymagania (Telegram Bot API 9.3+):

- **Prywatne czaty z włączonymi tematami** (tryb tematów forum dla bota).
- Wiadomości przychodzące muszą zawierać `message_thread_id` (prywatny wątek tematu).
- Strumieniowanie jest ignorowane dla grup/supergrup/kanałów.

Konfiguracja:

- `channels.telegram.streamMode: "off" | "partial" | "block"` (domyślnie: `partial`)
  - `partial`: aktualizuj dymek szkicu najnowszym tekstem strumieniowania.
  - `block`: aktualizuj dymek szkicu w większych blokach (kawałkami).
  - `off`: wyłącz strumieniowanie szkiców.
- Opcjonalnie (tylko dla `streamMode: "block"`):
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    - domyślne: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (ograniczone do `channels.telegram.textChunkLimit`).

Uwaga: strumieniowanie szkiców jest oddzielne od **strumieniowania blokowego** (wiadomości kanału).
Strumieniowanie blokowe jest domyślnie wyłączone i wymaga `channels.telegram.blockStreaming: true`,
jeśli chcesz wczesne wiadomości Telegrama zamiast aktualizacji szkicu.

Strumień rozumowania (tylko Telegram):

- `/reasoning stream` strumieniuje rozumowanie do dymka szkicu podczas generowania odpowiedzi,
  a następnie wysyła odpowiedź końcową bez rozumowania.
- Jeśli `channels.telegram.streamMode` to `off`, strumień rozumowania jest wyłączony.
  Więcej kontekstu: [Strumieniowanie + dzielenie](/concepts/streaming).

## Polityka ponowień

Wywołania Telegram API wychodzące są ponawiane przy przejściowych błędach sieci/429 z wykładniczym opóźnieniem i jitterem. Skonfiguruj przez `channels.telegram.retry`. Zobacz [Polityka ponowień](/concepts/retry).

## Narzędzie agenta (wiadomości + reakcje)

- Narzędzie: `telegram` z akcją `sendMessage` (`to`, `content`, opcjonalnie `mediaUrl`, `replyToMessageId`, `messageThreadId`).
- Narzędzie: `telegram` z akcją `react` (`chatId`, `messageId`, `emoji`).
- Narzędzie: `telegram` z akcją `deleteMessage` (`chatId`, `messageId`).
- Semantyka usuwania reakcji: zobacz [/tools/reactions](/tools/reactions).
- Bramkowanie narzędzi: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (domyślnie: włączone) oraz `channels.telegram.actions.sticker` (domyślnie: wyłączone).

## Powiadomienia o reakcjach

**Jak działają reakcje:**
Reakcje Telegrama docierają jako **oddzielne zdarzenia `message_reaction`**, a nie jako właściwości w ładunkach wiadomości. Gdy użytkownik doda reakcję, OpenClaw:

1. Otrzymuje aktualizację `message_reaction` z Telegram API
2. Konwertuje ją na **zdarzenie systemowe** w formacie: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Kolejkuje zdarzenie systemowe używając **tego samego klucza sesji** co zwykłe wiadomości
4. Gdy nadejdzie kolejna wiadomość w tej rozmowie, zdarzenia systemowe są opróżniane i dołączane na początku kontekstu agenta

Agent widzi reakcje jako **powiadomienia systemowe** w historii rozmowy, a nie jako metadane wiadomości.

**Konfiguracja:**

- `channels.telegram.reactionNotifications`: Kontroluje, które reakcje wyzwalają powiadomienia
  - `"off"` — ignoruj wszystkie reakcje
  - `"own"` — powiadamiaj, gdy użytkownicy reagują na wiadomości bota (best-effort; w pamięci) (domyślnie)
  - `"all"` — powiadamiaj o wszystkich reakcjach

- `channels.telegram.reactionLevel`: Kontroluje zdolność agenta do reagowania
  - `"off"` — agent nie może reagować na wiadomości
  - `"ack"` — bot wysyła reakcje potwierdzające (👀 podczas przetwarzania) (domyślnie)
  - `"minimal"` — agent może reagować oszczędnie (wytyczna: 1 na 5–10 wymian)
  - `"extensive"` — agent może reagować swobodnie, gdy to właściwe

**Grupy forum:** Reakcje w grupach forum zawierają `message_thread_id` i używają kluczy sesji takich jak `agent:main:telegram:group:{chatId}:topic:{threadId}`. Zapewnia to, że reakcje i wiadomości w tym samym temacie pozostają razem.

**Przykładowa konfiguracja:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**Wymagania:**

- Boty Telegrama muszą jawnie zażądać `message_reaction` w `allowed_updates` (konfigurowane automatycznie przez OpenClaw)
- W trybie webhook reakcje są dołączane do webhooka `allowed_updates`
- W trybie pollingu reakcje są dołączane do `getUpdates` `allowed_updates`

## Cele dostarczania (CLI/cron)

- Użyj identyfikatora czatu (`123456789`) lub nazwy użytkownika (`@name`) jako celu.
- Przykład: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

## Rozwiązywanie problemów

**Bot nie odpowiada na wiadomości bez wzmianek w grupie:**

- Jeśli ustawiono `channels.telegram.groups.*.requireMention=false`, **tryb prywatności** Telegram Bot API musi być wyłączony.
  - BotFather: `/setprivacy` → **Wyłącz** (następnie usuń i dodaj bota ponownie do grupy)
- `openclaw channels status` pokazuje ostrzeżenie, gdy konfiguracja oczekuje nieoznaczonych wzmianek wiadomości grupowych.
- `openclaw channels status --probe` może dodatkowo sprawdzić członkostwo dla jawnych numerycznych identyfikatorów grup (nie potrafi audytować reguł z symbolem wieloznacznym `"*"`).
- Szybki test: `/activation always` (tylko sesja; użyj konfiguracji dla trwałości)

**Bot w ogóle nie widzi wiadomości grupowych:**

- Jeśli ustawiono `channels.telegram.groups`, grupa musi być wymieniona lub używać `"*"`
- Sprawdź Ustawienia prywatności w @BotFather → „Group Privacy” powinno być **OFF**
- Zweryfikuj, czy bot faktycznie jest członkiem (a nie tylko adminem bez dostępu do odczytu)
- Sprawdź logi gateway: `openclaw logs --follow` (szukaj „skipping group message”)

**Bot odpowiada na wzmianki, ale nie na `/activation always`:**

- Polecenie `/activation` aktualizuje stan sesji, ale nie zapisuje się do konfiguracji
- Dla trwałego zachowania dodaj grupę do `channels.telegram.groups` z `requireMention: false`

**Polecenia takie jak `/status` nie działają:**

- Upewnij się, że Twój identyfikator użytkownika Telegrama jest autoryzowany (przez parowanie lub `channels.telegram.allowFrom`)
- Polecenia wymagają autoryzacji nawet w grupach z `groupPolicy: "open"`

**Long-polling przerywa się natychmiast na Node 22+ (często z proxy/własnym fetch):**

- Node 22+ jest bardziej rygorystyczny wobec instancji `AbortSignal`; obce sygnały mogą natychmiast przerywać wywołania `fetch`.
- Zaktualizuj do kompilacji OpenClaw, która normalizuje sygnały abort, lub uruchamiaj gateway na Node 20 do czasu aktualizacji.

**Bot startuje, a potem po cichu przestaje odpowiadać (lub loguje `HttpError: Network request ... failed`):**

- Niektóre hosty rozwiązują `api.telegram.org` najpierw do IPv6. Jeśli serwer nie ma działającego wyjścia IPv6, grammY może utknąć na żądaniach tylko-IPv6.
- Naprawa: włącz wyjście IPv6 **lub** wymuś rozwiązywanie IPv4 dla `api.telegram.org` (np. dodaj wpis `/etc/hosts` używając rekordu A IPv4 lub preferuj IPv4 w stosie DNS systemu), a następnie zrestartuj gateway.
- Szybka kontrola: `dig +short api.telegram.org A` i `dig +short api.telegram.org AAAA`, aby potwierdzić, co zwraca DNS.

## Referencja konfiguracji (Telegram)

Pełna konfiguracja: [Konfiguracja](/gateway/configuration)

Opcje dostawcy:

- `channels.telegram.enabled`: włącz/wyłącz start kanału.
- `channels.telegram.botToken`: token bota (BotFather).
- `channels.telegram.tokenFile`: odczytaj token ze ścieżki pliku.
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (domyślnie: parowanie).
- `channels.telegram.allowFrom`: lista dozwolonych DM-ów (id/nazwy użytkowników). `open` wymaga `"*"`.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (domyślnie: lista dozwolonych).
- `channels.telegram.groupAllowFrom`: lista dozwolonych nadawców grupowych (id/nazwy użytkowników).
- `channels.telegram.groups`: domyślne ustawienia per grupa + lista dozwolonych (użyj `"*"` dla domyślnych globalnych).
  - `channels.telegram.groups.<id>.groupPolicy`: nadpisanie per grupa dla groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: domyślne bramkowanie wzmianek.
  - `channels.telegram.groups.<id>.skills`: filtr skills (pominięcie = wszystkie skills, puste = brak).
  - `channels.telegram.groups.<id>.allowFrom`: nadpisanie listy dozwolonych nadawców per grupa.
  - `channels.telegram.groups.<id>.systemPrompt`: dodatkowy prompt systemowy dla grupy.
  - `channels.telegram.groups.<id>.enabled`: wyłącz grupę, gdy `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: nadpisania per temat (te same pola co grupa).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: nadpisanie per temat dla groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: nadpisanie bramkowania wzmianek per temat.
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
- `channels.telegram.actions.reactions`: bramkuj reakcje narzędzia Telegrama.
- `channels.telegram.actions.sendMessage`: bramkuj wysyłanie wiadomości narzędzia Telegrama.
- `channels.telegram.actions.deleteMessage`: bramkuj usuwanie wiadomości narzędzia Telegrama.
- `channels.telegram.actions.sticker`: bramkuj akcje naklejek Telegrama — wysyłanie i wyszukiwanie (domyślnie: false).
- `channels.telegram.reactionNotifications`: `off | own | all` — kontroluj, które reakcje wyzwalają zdarzenia systemowe (domyślnie: `own`, gdy nie ustawiono).
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — kontroluj zdolność agenta do reagowania (domyślnie: `minimal`, gdy nie ustawiono).

Powiązane opcje globalne:

- `agents.list[].groupChat.mentionPatterns` (wzorce bramkowania wzmianek).
- `messages.groupChat.mentionPatterns` (globalny fallback).
- `commands.native` (domyślnie `"auto"` → włączone dla Telegram/Discord, wyłączone dla Slack), `commands.text`, `commands.useAccessGroups` (zachowanie poleceń). Nadpisz przez `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.



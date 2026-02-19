---
summary: "Obsługa Signal przez signal-cli (JSON-RPC + SSE), konfiguracja i model numeru"
read_when:
  - Konfigurowanie obsługi Signal
  - Debugowanie wysyłania/odbierania w Signal
title: "Signal"
---

# Signal (signal-cli)

Status: integracja zewnętrznego CLI. Gateway komunikuje się z `signal-cli` przez HTTP JSON-RPC + SSE.

## Wymagania wstępne

- OpenClaw zainstalowany na Twoim serwerze (poniższy proces dla Linux przetestowano na Ubuntu 24).
- `signal-cli` dostępny na hoście, na którym działa gateway.
- Numer telefonu, który może odebrać jedną wiadomość SMS weryfikacyjną (dla ścieżki rejestracji SMS).
- Dostęp do przeglądarki dla captcha Signal (`signalcaptchas.org`) podczas rejestracji.

## Szybka konfiguracja (dla początkujących)

1. Użyj **oddzielnego numeru Signal** dla bota (zalecane).
2. Zainstaluj `signal-cli` (wymagana Java).
3. Wybierz jedną ścieżkę konfiguracji:
   - `signal-cli link -n "OpenClaw"`
   - **Ścieżka B (rejestracja SMS):** zarejestruj dedykowany numer z captcha + weryfikacją SMS.
4. Skonfiguruj OpenClaw i uruchom gateway.
5. Wyślij pierwszą wiadomość DM i zatwierdź parowanie (`openclaw pairing approve signal <CODE>`).

Minimalna konfiguracja:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Opis pól:

| Pole                                             | Opis                                                                                     |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| `account`                                        | Numer telefonu bota w formacie E.164 (`+15551234567`) |
| Konfiguracja (szybka ścieżka) | Ścieżka do `signal-cli` (`signal-cli`, jeśli znajduje się w `PATH`)   |
| `dmPolicy`                                       | Polityka dostępu do DM (zalecane `pairing`)                           |
| `allowFrom`                                      | Numery telefonów lub wartości `uuid:&lt;id&gt;` uprawnione do wysyłania DM                     |

## Czym to jest

- Kanał Signal przez `signal-cli` (nie wbudowana libsignal).
- Deterministyczne routowanie: odpowiedzi zawsze wracają do Signal.
- DM-y współdzielą główną sesję agenta; grupy są izolowane (`agent:<agentId>:signal:group:<groupId>`).

## Zapisy konfiguracji

Domyślnie Signal ma prawo zapisywać aktualizacje konfiguracji wyzwalane przez `/config set|unset` (wymaga `commands.config: true`).

Wyłącz za pomocą:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Model numeru (ważne)

- Gateway łączy się z **urządzeniem Signal** (konto `signal-cli`).
- Jeśli uruchamiasz bota na **swoim osobistym koncie Signal**, będzie on ignorował Twoje własne wiadomości (ochrona przed pętlą).
- Aby uzyskać scenariusz „piszę do bota i on odpowiada”, użyj **oddzielnego numeru bota**.

## Ścieżka konfiguracji A: połącz istniejące konto Signal (QR)

1. Zainstaluj `signal-cli` (wymagana Java).
2. Sparuj konto bota:
   - `signal-cli link -n "OpenClaw"`, a następnie zeskanuj kod QR w Signal.
3. Skonfiguruj Signal i uruchom gateway.

Przykład:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Obsługa wielu kont: użyj `channels.signal.accounts` z konfiguracją per konto oraz opcjonalnym `name`. Zobacz [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) dla wspólnego wzorca.

## Ścieżka konfiguracji B: zarejestruj dedykowany numer bota (SMS, Linux)

Użyj tej opcji, jeśli chcesz mieć dedykowany numer bota zamiast łączyć istniejące konto aplikacji Signal.

1. Uzyskaj numer, który może odbierać SMS (lub weryfikację głosową dla numerów stacjonarnych).
   - Użyj dedykowanego numeru bota, aby uniknąć konfliktów konta/sesji.
2. Zainstaluj `signal-cli` na hoście gateway:

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Jeśli używasz wersji JVM (`signal-cli-${VERSION}.tar.gz`), najpierw zainstaluj JRE 25+.
Utrzymuj `signal-cli` w najnowszej wersji; upstream informuje, że starsze wydania mogą przestać działać wraz ze zmianami w API serwera Signal.

3. Zarejestruj i zweryfikuj numer:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Jeśli wymagane jest captcha:

1. Otwórz `https://signalcaptchas.org/registration/generate.html`.
2. Ukończ captcha, skopiuj adres linku `signalcaptcha://...` z „Open Signal”.
3. Jeśli to możliwe, wykonaj polecenie z tego samego zewnętrznego adresu IP co sesja przeglądarki.
4. Natychmiast ponownie uruchom rejestrację (tokeny captcha szybko wygasają):

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. Sparuj urządzenie bota i uruchom demona:

```bash
# Jeśli uruchamiasz gateway jako usługę systemd użytkownika:
systemctl --user restart openclaw-gateway

# Następnie zweryfikuj:
openclaw doctor
openclaw channels status --probe
```

5. Częste awarie:
   - Wyślij dowolną wiadomość na numer bota.
   - Zatwierdź kod na serwerze: `openclaw pairing approve signal <PAIRING_CODE>`.
   - Zapisz numer bota jako kontakt w telefonie, aby uniknąć komunikatu „Nieznany kontakt”.

Ważne: rejestracja konta numeru telefonu za pomocą `signal-cli` może wylogować główną sesję aplikacji Signal dla tego numeru. Preferuj dedykowany numer bota lub użyj trybu łączenia QR, jeśli chcesz zachować istniejącą konfigurację aplikacji w telefonie.

Źródła upstream:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Proces captcha: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Proces łączenia: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Tryb zewnętrznego demona (httpUrl)

Jeśli chcesz samodzielnie zarządzać `signal-cli` (wolne zimne starty JVM, inicjalizacja kontenera lub współdzielone CPU), uruchom demona osobno i wskaż go w OpenClaw:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

To pomija automatyczne uruchamianie oraz oczekiwanie na start wewnątrz OpenClaw. Przy wolnych startach podczas auto-spawn ustaw `channels.signal.startupTimeoutMs`.

## Kontrola dostępu (DM-y + grupy)

DM-y:

- Domyślnie: `channels.signal.dmPolicy = "pairing"`.
- Nieznani nadawcy otrzymują kod parowania; wiadomości są ignorowane do momentu zatwierdzenia (kody wygasają po 1 godzinie).
- Zatwierdzanie przez:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Parowanie jest domyślną wymianą tokenów dla DM-ów Signal. Szczegóły: [Pairing](/channels/pairing)
- Nadawcy tylko z UUID (z `sourceUuid`) są zapisywani jako `uuid:<id>` w `channels.signal.allowFrom`.

Grupy:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` kontroluje, kto może wyzwalać w grupach, gdy ustawione jest `allowlist`.

## Jak to działa (zachowanie)

- `signal-cli` działa jako demon; gateway odczytuje zdarzenia przez SSE.
- Wiadomości przychodzące są normalizowane do wspólnej koperty kanału.
- Odpowiedzi zawsze wracają do tego samego numeru lub grupy.

## Media i limity

- Tekst wychodzący jest dzielony na fragmenty do `channels.signal.textChunkLimit` (domyślnie 4000).
- Opcjonalne dzielenie po nowych liniach: ustaw `channels.signal.chunkMode="newline"`, aby dzielić po pustych liniach (granice akapitów) przed dzieleniem według długości.
- Obsługiwane są załączniki (base64 pobierane z `signal-cli`).
- Domyślny limit mediów: `channels.signal.mediaMaxMb` (domyślnie 8).
- Użyj `channels.signal.ignoreAttachments`, aby pominąć pobieranie mediów.
- Kontekst historii grup używa `channels.signal.historyLimit` (lub `channels.signal.accounts.*.historyLimit`), z przejściem awaryjnym do `messages.groupChat.historyLimit`. Ustaw `0`, aby wyłączyć (domyślnie 50).

## Pisanie + potwierdzenia odczytu

- **Wskaźniki pisania**: OpenClaw wysyła sygnały pisania przez `signal-cli sendTyping` i odświeża je podczas generowania odpowiedzi.
- **Potwierdzenia odczytu**: gdy `channels.signal.sendReadReceipts` jest true, OpenClaw przekazuje potwierdzenia odczytu dla dozwolonych DM-ów.
- Signal-cli nie udostępnia potwierdzeń odczytu dla grup.

## Reakcje (narzędzie wiadomości)

- Użyj `message action=react` z `channel=signal`.
- Cele: nadawca E.164 lub UUID (użyj `uuid:<id>` z wyjścia parowania; goły UUID też działa).
- `messageId` to znacznik czasu Signal dla wiadomości, na którą reagujesz.
- Reakcje w grupach wymagają `targetAuthor` lub `targetAuthorUuid`.

Przykłady:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Konfiguracja:

- `channels.signal.actions.reactions`: włącz/wyłącz akcje reakcji (domyślnie true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` wyłącza reakcje agenta (narzędzie wiadomości `react` zgłosi błąd).
  - `minimal`/`extensive` włącza reakcje agenta i ustawia poziom wskazówek.
- Nadpisania per konto: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Cele dostarczania (CLI/cron)

- DM-y: `signal:+15551234567` (lub zwykły E.164).
- DM-y UUID: `uuid:<id>` (lub goły UUID).
- Grupy: `signal:group:<groupId>`.
- Nazwy użytkowników: `username:<name>` (jeśli obsługiwane przez Twoje konto Signal).

## Rozwiązywanie problemów

Najpierw uruchom tę drabinę:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Następnie, w razie potrzeby, potwierdź stan parowania DM-ów:

```bash
openclaw pairing list signal
```

Częste awarie:

- Demon osiągalny, ale brak odpowiedzi: zweryfikuj ustawienia konta/demona (`httpUrl`, `account`) oraz tryb odbioru.
- DM-y ignorowane: nadawca oczekuje na zatwierdzenie parowania.
- Wiadomości grupowe ignorowane: bramkowanie nadawcy/wzmianek w grupach blokuje dostarczenie.
- Błędy walidacji konfiguracji po edycji: uruchom `openclaw doctor --fix`.
- Brak Signal w diagnostyce: potwierdź `channels.signal.enabled: true`.

Dodatkowe kontrole:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Schemat triage: [/channels/troubleshooting](/channels/troubleshooting).

## Uwagi dotyczące bezpieczeństwa

- `signal-cli` przechowuje klucze konta lokalnie (zwykle `~/.local/share/signal-cli/data/`).
- Wykonaj kopię zapasową stanu konta Signal przed migracją lub przebudową serwera.
- Zachowaj `channels.signal.dmPolicy: "pairing"`, chyba że wyraźnie chcesz umożliwić szerszy dostęp do DM.
- Weryfikacja SMS jest wymagana tylko podczas rejestracji lub odzyskiwania konta, ale utrata kontroli nad numerem/kontem może skomplikować ponowną rejestrację.

## Referencja konfiguracji (Signal)

Pełna konfiguracja: [Configuration](/gateway/configuration)

Opcje dostawcy:

- `channels.signal.enabled`: włącz/wyłącz uruchamianie kanału.
- `channels.signal.account`: E.164 dla konta bota.
- `channels.signal.cliPath`: ścieżka do `signal-cli`.
- `channels.signal.httpUrl`: pełny URL demona (nadpisuje host/port).
- `channels.signal.httpHost`, `channels.signal.httpPort`: powiązanie demona (domyślnie 127.0.0.1:8080).
- `channels.signal.autoStart`: automatyczne uruchamianie demona (domyślnie true, jeśli `httpUrl` nieustawione).
- `channels.signal.startupTimeoutMs`: limit czasu oczekiwania na start w ms (limit 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: pomiń pobieranie załączników.
- `channels.signal.ignoreStories`: ignoruj relacje z demona.
- `channels.signal.sendReadReceipts`: przekazuj potwierdzenia odczytu.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (domyślnie: parowanie).
- `channels.signal.allowFrom`: lista dozwolonych DM-ów (E.164 lub `uuid:<id>`). `open` wymaga `"*"`. Signal nie ma nazw użytkowników; użyj identyfikatorów telefonu/UUID.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (domyślnie: lista dozwolonych).
- `channels.signal.groupAllowFrom`: lista dozwolonych nadawców w grupach.
- `channels.signal.historyLimit`: maksymalna liczba wiadomości grupowych do uwzględnienia jako kontekst (0 wyłącza).
- `channels.signal.dmHistoryLimit`: limit historii DM-ów w turach użytkownika. Nadpisania per użytkownik: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: rozmiar fragmentów wychodzących (znaki).
- `channels.signal.chunkMode`: `length` (domyślne) lub `newline`, aby dzielić po pustych liniach (granice akapitów) przed dzieleniem według długości.
- `channels.signal.mediaMaxMb`: limit mediów przychodzących/wychodzących (MB).

Powiązane opcje globalne:

- `agents.list[].groupChat.mentionPatterns` (Signal nie obsługuje natywnych wzmianek).
- `messages.groupChat.mentionPatterns` (globalny fallback).
- `messages.responsePrefix`.

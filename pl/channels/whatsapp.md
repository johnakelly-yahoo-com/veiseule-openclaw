---
title: "„WhatsApp”"
---

# WhatsApp (kanał webowy)

Status: wyłącznie WhatsApp Web przez Baileys. Sesje należą do Gateway.

## Szybki start (dla początkujących)

1. Użyj **oddzielnego numeru telefonu**, jeśli to możliwe (zalecane).
2. Skonfiguruj WhatsApp w `~/.openclaw/openclaw.json`.
3. Uruchom `openclaw channels login`, aby zeskanować kod QR (Połączone urządzenia).
4. Uruchom gateway.

Minimalna konfiguracja:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

## Cele

- Wiele kont WhatsApp (multi‑account) w jednym procesie Gateway.
- Deterministyczne routowanie: odpowiedzi wracają do WhatsApp, bez routowania modelu.
- Model widzi wystarczający kontekst, aby rozumieć cytowane odpowiedzi.

## Zapisy konfiguracji

Domyślnie WhatsApp może zapisywać aktualizacje konfiguracji wyzwalane przez `/config set|unset` (wymaga `commands.config: true`).

Wyłącz za pomocą:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## Architektura (kto jest właścicielem czego)

- **Gateway** jest właścicielem gniazda Baileys i pętli skrzynki odbiorczej.
- **CLI / aplikacja na macOS** komunikują się z gateway; brak bezpośredniego użycia Baileys.
- **Aktywny listener** jest wymagany do wysyłek wychodzących; w przeciwnym razie wysyłka kończy się natychmiastowym błędem.

## Pozyskanie numeru telefonu (dwa tryby)

WhatsApp wymaga prawdziwego numeru komórkowego do weryfikacji. Numery VoIP i wirtualne są zwykle blokowane. Istnieją dwie obsługiwane metody uruchomienia OpenClaw na WhatsApp:

### Dedykowany numer (zalecane)

Użyj **oddzielnego numeru telefonu** dla OpenClaw. Najlepsze UX, czyste routowanie, brak osobliwości czatu z samym sobą. Idealna konfiguracja: **zapasowy/stary telefon z Androidem + eSIM**. Pozostaw go na Wi‑Fi i zasilaniu oraz połącz przez QR.

**WhatsApp Business:** Możesz używać WhatsApp Business na tym samym urządzeniu z innym numerem. Świetne do oddzielenia prywatnego WhatsApp — zainstaluj WhatsApp Business i zarejestruj tam numer OpenClaw.

**Przykładowa konfiguracja (dedykowany numer, lista dozwolonych jednego użytkownika):**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

**Tryb parowania (opcjonalnie):**  
Jeśli zamiast listy dozwolonych chcesz parowanie, ustaw `channels.whatsapp.dmPolicy` na `pairing`. Nieznani nadawcy otrzymają kod parowania; zatwierdź poleceniem:
`openclaw pairing approve whatsapp <code>`

### Numer osobisty (fallback)

Szybki wariant awaryjny: uruchom OpenClaw na **własnym numerze**. Do testów pisz do siebie (WhatsApp „Napisz do siebie”), aby nie spamować kontaktów. Podczas konfiguracji i eksperymentów spodziewaj się odczytywać kody weryfikacyjne na głównym telefonie. **Musisz włączyć tryb self‑chat.**  
Gdy kreator poprosi o Twój osobisty numer WhatsApp, wpisz telefon, z którego będziesz pisać (właściciel/nadawca), a nie numer asystenta.

**Przykładowa konfiguracja (numer osobisty, self‑chat):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Odpowiedzi self‑chat domyślnie używają `[{identity.name}]`, gdy jest ustawione (w przeciwnym razie `[openclaw]`),  
jeśli `messages.responsePrefix` jest nieustawione. Ustaw jawnie, aby dostosować lub wyłączyć  
prefiks (użyj `""`, aby go usunąć).

### Wskazówki dot. pozyskania numeru

- **Lokalny eSIM** od operatora w Twoim kraju (najbardziej niezawodne)
  - Austria: [hot.at](https://www.hot.at)
  - UK: [giffgaff](https://www.giffgaff.com) — darmowa karta SIM, bez umowy
- **Karta prepaid** — tania, wymaga jedynie odebrania jednego SMS‑a weryfikacyjnego

**Unikaj:** TextNow, Google Voice, większości usług „free SMS” — WhatsApp agresywnie je blokuje.

**Wskazówka:** Numer musi odebrać tylko jeden SMS weryfikacyjny. Potem sesje WhatsApp Web utrzymują się dzięki `creds.json`.

## Dlaczego nie Twilio?

- Wczesne wersje OpenClaw obsługiwały integrację WhatsApp Business od Twilio.
- Numery WhatsApp Business słabo pasują do osobistego asystenta.
- Meta wymusza 24‑godzinne okno odpowiedzi; jeśli nie odpowiedziano w ciągu ostatnich 24 godzin, numer biznesowy nie może inicjować nowych wiadomości.
- Wysoki wolumen lub „gadatliwe” użycie wywołuje agresywne blokady, bo konta biznesowe nie są przeznaczone do wysyłania dziesiątek wiadomości asystenta.
- Efekt: zawodna dostarczalność i częste blokady, dlatego wsparcie zostało usunięte.

## Logowanie + poświadczenia

- Polecenie logowania: `openclaw channels login` (QR przez Połączone urządzenia).
- Logowanie wielu kont: `openclaw channels login --account <id>` (`<id>` = `accountId`).
- Konto domyślne (gdy pominięto `--account`): `default`, jeśli obecne; w przeciwnym razie pierwszy skonfigurowany identyfikator konta (sortowany).
- Poświadczenia przechowywane w `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
- Kopia zapasowa w `creds.json.bak` (odtwarzana przy uszkodzeniu).
- Zgodność wsteczna: starsze instalacje przechowywały pliki Baileys bezpośrednio w `~/.openclaw/credentials/`.
- Wylogowanie: `openclaw channels logout` (lub `--account <id>`) usuwa stan uwierzytelnienia WhatsApp (zachowuje współdzielone `oauth.json`).
- Wylogowane gniazdo => błąd z instrukcją ponownego połączenia.

## Przepływ przychodzący (DM + grupy)

- Zdarzenia WhatsApp pochodzą z `messages.upsert` (Baileys).
- Listenery skrzynki odbiorczej są odłączane przy zamykaniu, aby uniknąć kumulowania handlerów zdarzeń w testach/restartach.
- Czaty statusowe/broadcast są ignorowane.
- Czaty bezpośrednie używają E.164; grupy używają JID grupowego.
- **Polityka DM**: `channels.whatsapp.dmPolicy` kontroluje dostęp do czatów bezpośrednich (domyślnie: `pairing`).
  - Parowanie: nieznani nadawcy otrzymują kod parowania (zatwierdzenie przez `openclaw pairing approve whatsapp <code>`; kody wygasają po 1 godzinie).
  - Otwarte: wymaga, aby `channels.whatsapp.allowFrom` zawierało `"*"`.
  - Twój połączony numer WhatsApp jest domyślnie zaufany, więc wiadomości do siebie pomijają sprawdzenia `channels.whatsapp.dmPolicy` i `channels.whatsapp.allowFrom`.

### Tryb numeru osobistego (fallback)

Jeśli uruchamiasz OpenClaw na **osobistym numerze WhatsApp**, włącz `channels.whatsapp.selfChatMode` (zob. przykład powyżej).

Zachowanie:

- Wychodzące DM‑y nigdy nie wyzwalają odpowiedzi parowania (zapobiega spamowaniu kontaktów).
- Przychodzący nieznani nadawcy nadal podlegają `channels.whatsapp.dmPolicy`.
- Tryb self‑chat (allowFrom zawiera Twój numer) unika automatycznych potwierdzeń odczytu i ignoruje JID wzmianek.
- Potwierdzenia odczytu są wysyłane dla DM‑ów innych niż self‑chat.

## Potwierdzenia odczytu

Domyślnie gateway oznacza przychodzące wiadomości WhatsApp jako przeczytane (niebieskie haczyki) po ich zaakceptowaniu.

Wyłącz globalnie:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

Wyłącz per konto:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

Uwagi:

- Tryb self‑chat zawsze pomija potwierdzenia odczytu.

## WhatsApp FAQ: wysyłanie wiadomości + parowanie

**Czy OpenClaw wyśle wiadomości do losowych kontaktów po połączeniu WhatsApp?**  
Nie. Domyślna polityka DM to **parowanie**, więc nieznani nadawcy otrzymują tylko kod parowania, a ich wiadomość **nie jest przetwarzana**. OpenClaw odpowiada wyłącznie na czaty, które otrzymuje, lub na wysyłki, które jawnie wyzwolisz (agent/CLI).

**Jak działa parowanie na WhatsApp?**  
Parowanie to bramka DM dla nieznanych nadawców:

- Pierwsza DM od nowego nadawcy zwraca krótki kod (wiadomość nie jest przetwarzana).
- Zatwierdź poleceniem: `openclaw pairing approve whatsapp <code>` (lista przez `openclaw pairing list whatsapp`).
- Kody wygasają po 1 godzinie; oczekujące żądania są ograniczone do 3 na kanał.

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Replies still come from the **same WhatsApp account**, and direct chats collapse to each agent's main session, so use **one agent per person**. Kontrola dostępu DM (`dmPolicy`/`allowFrom`) jest globalna per konto WhatsApp. Zobacz [Multi‑Agent Routing](/concepts/multi-agent).

**Dlaczego kreator pyta o mój numer telefonu?**  
Kreator używa go do ustawienia **listy dozwolonych/właściciela**, aby Twoje własne DM‑y były dozwolone. Nie służy do automatycznego wysyłania. Jeśli działasz na osobistym numerze WhatsApp, użyj tego samego numeru i włącz `channels.whatsapp.selfChatMode`.

## Normalizacja wiadomości (co widzi model)

- `Body` to bieżąca treść wiadomości z kopertą.

- Kontekst cytowanej odpowiedzi jest **zawsze dołączany**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- Ustawiane są także metadane odpowiedzi:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = cytowana treść lub placeholder medium
  - `ReplyToSender` = E.164, gdy znany

- Wiadomości przychodzące zawierające wyłącznie media używają placeholderów:
  - `<media:image|video|audio|document|sticker>`

## Grupy

- Grupy mapują się na sesje `agent:<agentId>:whatsapp:group:<jid>`.
- Polityka grup: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (domyślnie `allowlist`).
- Tryby aktywacji:
  - `mention` (domyślny): wymaga @wzmianki lub dopasowania regex.
  - `always`: zawsze wyzwala.
- `/activation mention|always` jest tylko dla właściciela i musi być wysłane jako samodzielna wiadomość.
- Właściciel = `channels.whatsapp.allowFrom` (lub własny E.164, jeśli nieustawione).
- **Wstrzykiwanie historii** (tylko oczekujące):
  - Ostatnie _nieprzetworzone_ wiadomości (domyślnie 50) wstawiane pod:
    `[Chat messages since your last reply - for context]` (wiadomości już w sesji nie są wstrzykiwane ponownie)
  - Bieżąca wiadomość pod:
    `[Current message - respond to this]`
  - Dołączany sufiks nadawcy: `[from: Name (+E164)]`
- Metadane grup są buforowane 5 min (temat + uczestnicy).

## Dostarczanie odpowiedzi (wątki)

- WhatsApp Web wysyła standardowe wiadomości (brak wątkowania cytowanych odpowiedzi w bieżącym gateway).
- Tagi odpowiedzi są ignorowane w tym kanale.

## Reakcje potwierdzające (auto‑reakcja przy odbiorze)

WhatsApp może automatycznie wysyłać reakcje emoji na przychodzące wiadomości natychmiast po ich odebraniu, zanim bot wygeneruje odpowiedź. Zapewnia to natychmiastowe potwierdzenie dla użytkowników, że wiadomość została odebrana.

**Konfiguracja:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Opcje:**

- `emoji` (string): Emoji używane do potwierdzenia (np. „👀”, „✅”, „📨”). Puste lub pominięte = funkcja wyłączona.
- `direct` (boolean, domyślnie: `true`): Wysyłaj reakcje w czatach bezpośrednich/DM.
- `group` (string, domyślnie: `"mentions"`): Zachowanie w czatach grupowych:
  - `"always"`: Reaguj na wszystkie wiadomości grupowe (nawet bez @wzmianki)
  - `"mentions"`: Reaguj tylko, gdy bot jest @wspomniany
  - `"never"`: Nigdy nie reaguj w grupach

**Nadpisanie per konto:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Uwagi dot. zachowania:**

- Reakcje są wysyłane **natychmiast** po odebraniu wiadomości, przed wskaźnikami pisania lub odpowiedziami bota.
- W grupach z `requireMention: false` (aktywacja: zawsze) `group: "mentions"` zareaguje na wszystkie wiadomości (nie tylko @wzmianki).
- Fire‑and‑forget: błędy reakcji są logowane, ale nie blokują odpowiedzi bota.
- JID uczestnika jest automatycznie dołączany dla reakcji grupowych.
- WhatsApp ignoruje `messages.ackReaction`; użyj zamiast tego `channels.whatsapp.ackReaction`.

## Narzędzie agenta (reakcje)

- Narzędzie: `whatsapp` z akcją `react` (`chatJid`, `messageId`, `emoji`, opcjonalnie `remove`).
- Opcjonalnie: `participant` (nadawca w grupie), `fromMe` (reakcja na własną wiadomość), `accountId` (wiele kont).
- Semantyka usuwania reakcji: zob. [/tools/reactions](/tools/reactions).
- Bramka narzędzia: `channels.whatsapp.actions.reactions` (domyślnie: włączone).

## Limity

- Tekst wychodzący jest dzielony na kawałki do `channels.whatsapp.textChunkLimit` (domyślnie 4000).
- Opcjonalne dzielenie po nowych liniach: ustaw `channels.whatsapp.chunkMode="newline"`, aby dzielić po pustych liniach (granice akapitów) przed dzieleniem długości.
- Zapisy mediów przychodzących są ograniczone przez `channels.whatsapp.mediaMaxMb` (domyślnie 50 MB).
- Elementy mediów wychodzących są ograniczone przez `agents.defaults.mediaMaxMb` (domyślnie 5 MB).

## Wysyłka wychodząca (tekst + media)

- Używa aktywnego listenera webowego; błąd, jeśli gateway nie działa.
- Dzielenie tekstu: maks. 4k na wiadomość (konfigurowalne przez `channels.whatsapp.textChunkLimit`, opcjonalnie `channels.whatsapp.chunkMode`).
- Media:
  - Obsługiwane: obraz/wideo/audio/dokument.
  - Audio wysyłane jako PTT; `audio/ogg` => `audio/ogg; codecs=opus`.
  - Podpis (caption) tylko przy pierwszym elemencie medium.
  - Pobieranie mediów obsługuje HTTP(S) i ścieżki lokalne.
  - Animowane GIF‑y: WhatsApp oczekuje MP4 z `gifPlayback: true` dla pętli inline.
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: parametry `send` obejmują `gifPlayback: true`

## Notatki głosowe (audio PTT)

WhatsApp wysyła audio jako **notatki głosowe** (dymek PTT).

- Najlepsze rezultaty: OGG/Opus. OpenClaw przepisuje `audio/ogg` na `audio/ogg; codecs=opus`.
- `[[audio_as_voice]]` jest ignorowane dla WhatsApp (audio i tak jest wysyłane jako notatka głosowa).

## Limity mediów + optymalizacja

- Domyślny limit wysyłki: 5 MB (na element medium).
- Nadpisanie: `agents.defaults.mediaMaxMb`.
- Obrazy są automatycznie optymalizowane do JPEG poniżej limitu (zmiana rozmiaru + dobór jakości).
- Zbyt duże media => błąd; odpowiedź z medium przechodzi na ostrzeżenie tekstowe.

## Sygnały heartbeat

- **Heartbeat Gateway** loguje kondycję połączenia (`web.heartbeatSeconds`, domyślnie 60 s).
- **Heartbeat agenta** można skonfigurować per agent (`agents.list[].heartbeat`) lub globalnie
  przez `agents.defaults.heartbeat` (fallback, gdy brak wpisów per agent).
  - Używa skonfigurowanego promptu heartbeat (domyślnie: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + zachowania pomijania `HEARTBEAT_OK`.
  - Dostarczanie domyślnie do ostatnio używanego kanału (lub skonfigurowanego celu).

## Zachowanie ponownego łączenia

- Polityka backoff: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- Po osiągnięciu maxAttempts monitoring webowy zatrzymuje się (tryb zdegradowany).
- Wylogowane => zatrzymaj i wymagaj ponownego połączenia.

## Szybka mapa konfiguracji

- `channels.whatsapp.dmPolicy` (polityka DM: pairing/allowlist/open/disabled).
- `channels.whatsapp.selfChatMode` (konfiguracja „ten sam telefon”; bot używa Twojego osobistego numeru WhatsApp).
- `channels.whatsapp.allowFrom` (lista dozwolonych DM). WhatsApp używa numerów E.164 (bez nazw użytkowników).
- `channels.whatsapp.mediaMaxMb` (limit zapisu mediów przychodzących).
- `channels.whatsapp.ackReaction` (auto‑reakcja przy odbiorze wiadomości: `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*` (ustawienia per konto + opcjonalnie `authDir`).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (limit mediów przychodzących per konto).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (nadpisanie reakcji potwierdzających per konto).
- `channels.whatsapp.groupAllowFrom` (lista dozwolonych nadawców grup).
- `channels.whatsapp.groupPolicy` (polityka grup).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (kontekst historii grup; `0` wyłącza).
- `channels.whatsapp.dmHistoryLimit` (limit historii DM w turach użytkownika). Nadpisania per użytkownik: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (lista dozwolonych grup + domyślne bramkowanie wzmianek; użyj `"*"`, aby zezwolić wszystkim)
- `channels.whatsapp.actions.reactions` (bramkowanie reakcji narzędzia WhatsApp).
- `agents.list[].groupChat.mentionPatterns` (lub `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (prefiks przychodzący; per konto: `channels.whatsapp.accounts.<accountId>.messagePrefix`; przestarzałe: `messages.messagePrefix`)
- `messages.responsePrefix` (prefiks wychodzący)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (opcjonalne nadpisanie)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (nadpisania per agent)
- `session.*` (zakres, bezczynność, magazyn, mainKey)
- `web.enabled` (wyłącza start kanału, gdy false)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Logi + rozwiązywanie problemów

- Podsystemy: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- Plik logów: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (konfigurowalny).
- Przewodnik rozwiązywania problemów: [Gateway troubleshooting](/gateway/troubleshooting).

## Rozwiązywanie problemów (szybkie)

**Brak połączenia / wymagane logowanie QR**

- Objaw: `channels status` pokazuje `linked: false` lub ostrzega „Not linked”.
- Naprawa: uruchom `openclaw channels login` na hoście Gateway i zeskanuj QR (WhatsApp → Ustawienia → Połączone urządzenia).

**Połączone, ale rozłączone / pętla ponownego łączenia**

- Objaw: `channels status` pokazuje `running, disconnected` lub ostrzega „Linked but disconnected”.
- Naprawa: `openclaw doctor` (lub zrestartuj gateway). Jeśli problem się utrzymuje, połącz ponownie przez `channels login` i sprawdź `openclaw logs --follow`.

**Runtime Bun**

- Bun **nie jest zalecany**. WhatsApp (Baileys) i Telegram są niestabilne na Bun.
  Uruchamiaj gateway na **Node**. (Zob. uwaga o runtime w Pierwsze kroki.)



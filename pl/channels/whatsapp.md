---
summary: "Obsługa kanału WhatsApp, kontrola dostępu, sposób dostarczania i operacje"
read_when:
  - Praca nad zachowaniem kanału WhatsApp/web lub routowaniem skrzynki odbiorczej
title: "„WhatsApp”"
---

# WhatsApp (kanał webowy)

Status: wyłącznie WhatsApp Web przez Baileys. Sesje należą do Gateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Domyślną polityką DM jest parowanie dla nieznanych nadawców.
  
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
  <Step title="Configure WhatsApp access policy">

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

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Dla konkretnego konta:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
Uruchom gateway.
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
Zatwierdź poleceniem: `openclaw pairing approve whatsapp <code>` (lista przez `openclaw pairing list whatsapp`).
```

    ```
    Kody wygasają po 1 godzinie; oczekujące żądania są ograniczone do 3 na kanał.
    ```

  
</Step>
</Steps>

<Note>
Świetne do oddzielenia prywatnego WhatsApp — zainstaluj WhatsApp Business i zarejestruj tam numer OpenClaw. (Metadane kanału i proces wdrożenia są zoptymalizowane pod tę konfigurację, ale obsługiwane są również konfiguracje z numerem prywatnym.)
</Note>

## Wzorce wdrożeniowe

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    To najczystszy tryb operacyjny:

    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">    Onboarding obsługuje tryb numeru osobistego i zapisuje bazową konfigurację przyjazną dla czatu z samym sobą:

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">    Kanał platformy komunikacyjnej opiera się na WhatsApp Web (`Baileys`) w obecnej architekturze kanałów OpenClaw.

    ```
    Nie ma oddzielnego kanału wiadomości Twilio WhatsApp w wbudowanym rejestrze chat-channel.
    ```

  
</Accordion>
</AccordionGroup>

## Model działania (runtime)

- Gateway zarządza gniazdem WhatsApp oraz pętlą ponownego łączenia.
- Wysyłki wychodzące wymagają aktywnego listenera WhatsApp dla docelowego konta.
- Czaty statusowe/broadcast są ignorowane.
- Czaty bezpośrednie używają reguł sesji DM (`session.dmScope`; domyślnie `main` łączy DM z główną sesją agenta).
- Grupy mapują się na sesje `agent:<agentId>:whatsapp:group:<jid>`.

## Kontrola dostępu i aktywacja

<Tabs>
  <Tab title="DM policy">**Polityka DM**: `channels.whatsapp.dmPolicy` kontroluje dostęp do czatów bezpośrednich (domyślnie: `pairing`).

    ```
    Polityka grup: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (domyślnie `allowlist`).
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">    Dostęp do grup ma dwie warstwy:

    ```
    `channels.whatsapp.groupAllowFrom` (lista dozwolonych nadawców grup).
    ```

  
</Tab>

  <Tab title="Mentions + /activation">    Odpowiedzi w grupach domyślnie wymagają wzmianki.

    ```
    Wykrywanie wzmianki obejmuje:
    
    - jawne wzmianki WhatsApp o tożsamości bota
    - skonfigurowane wzorce regex wzmianki (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - niejawne wykrywanie odpowiedzi do bota (nadawca odpowiedzi odpowiada tożsamości bota)
    
    Polecenie aktywacji na poziomie sesji:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` aktualizuje stan sesji (nie globalną konfigurację). Jest ograniczone do właściciela.
    ```

  
</Tab>
</Tabs>

## Zachowanie numeru osobistego i czatu z samym sobą

Gdy połączony numer własny jest również obecny w `allowFrom`, aktywowane są zabezpieczenia czatu własnego WhatsApp:

- pomijanie potwierdzeń odczytu dla wiadomości w czacie z samym sobą
- ignorowanie automatycznego wyzwalania wzmianki-JID, które w przeciwnym razie powodowałoby pingowanie samego siebie
- Odpowiedzi self‑chat domyślnie używają `[{identity.name}]`, gdy jest ustawione (w przeciwnym razie `[openclaw]`),  
  jeśli `messages.responsePrefix` jest nieustawione.

## Normalizacja wiadomości i kontekst

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">    Przychodzące wiadomości WhatsApp są opakowywane we współdzieloną strukturę inbound envelope.

    ````
    Jeśli istnieje cytowana odpowiedź, kontekst jest dołączany w następującej formie:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Pola metadanych odpowiedzi są również uzupełniane, gdy są dostępne (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, nadawca JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Wiadomości przychodzące zawierające wyłącznie media używają placeholderów:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Ładunki lokalizacji i kontaktów są normalizowane do kontekstu tekstowego przed przekazaniem dalej.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">    W przypadku grup nieprzetworzone wiadomości mogą być buforowane i wstrzykiwane jako kontekst, gdy bot zostanie ostatecznie wyzwolony.

    ```
    Ostatnie _nieprzetworzone_ wiadomości (domyślnie 50) wstawiane pod:
    `[Chat messages since your last reply - for context]` (wiadomości już w sesji nie są wstrzykiwane ponownie)
    ```

  
</Accordion>

  <Accordion title="Read receipts">Domyślnie gateway oznacza przychodzące wiadomości WhatsApp jako przeczytane (niebieskie haczyki) po ich zaakceptowaniu.

    ```
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

  
</Accordion>
</AccordionGroup>

## Dostarczanie, dzielenie na części i multimedia

<AccordionGroup>
  <Accordion title="Text chunking">Opcjonalne dzielenie po nowych liniach: ustaw `channels.whatsapp.chunkMode="newline"`, aby dzielić po pustych liniach (granice akapitów) przed dzieleniem długości.
</Accordion>

  <Accordion title="Outbound media behavior">    - obsługuje ładunki image, video, audio (PTT voice-note) oraz document
    - `audio/ogg` jest przepisywane na `audio/ogg; codecs=opus` dla kompatybilności z voice-note
    - odtwarzanie animowanych GIF jest obsługiwane przez `gifPlayback: true` przy wysyłaniu video
    - podpisy (captions) są stosowane do pierwszego elementu multimedialnego przy wysyłaniu odpowiedzi z wieloma mediami
    - źródłem mediów może być HTTP(S), `file://` lub ścieżki lokalne
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">    - limit zapisu mediów przychodzących: `channels.whatsapp.mediaMaxMb` (domyślnie `50`)
    - limit mediów wychodzących dla automatycznych odpowiedzi: `agents.defaults.mediaMaxMb` (domyślnie `5MB`)
    - obrazy są automatycznie optymalizowane (zmiana rozmiaru/jakości), aby zmieścić się w limitach
    - w przypadku niepowodzenia wysyłki mediów, fallback dla pierwszego elementu wysyła ostrzeżenie tekstowe zamiast cicho pomijać odpowiedź
  
</Accordion>
</AccordionGroup>

## Reakcje potwierdzające

`channels.whatsapp.ackReaction` (auto‑reakcja przy odbiorze wiadomości: `{emoji, direct, group}`).

```json5
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

Uwagi:

- wysyłane natychmiast po zaakceptowaniu wiadomości przychodzącej (przed odpowiedzią)
- błędy są logowane, ale nie blokują normalnego dostarczenia odpowiedzi
- tryb grupowy `mentions` reaguje na tury wyzwolone wzmianką; aktywacja grupy `always` działa jako obejście tego sprawdzenia
- WhatsApp ignoruje `messages.ackReaction`; użyj zamiast tego `channels.whatsapp.ackReaction`.

## Wiele kont i poświadczenia

<AccordionGroup>
  <Accordion title="Account selection and defaults">Konto domyślne (gdy pominięto `--account`): `default`, jeśli obecne; w przeciwnym razie pierwszy skonfigurowany identyfikator konta (sortowany).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">Poświadczenia przechowywane w `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.<accountId>/creds.json`
    - plik kopii zapasowej: `creds.json.bak`
    - starsza domyślna autoryzacja w `~/.openclaw/credentials/` jest nadal rozpoznawana/migrowana dla przepływów domyślnego konta
  
</Accordion>

  <Accordion title="Logout behavior">Logowanie wielu kont: `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>]` czyści stan autoryzacji WhatsApp dla tego konta.

    ```
    W starszych katalogach autoryzacji plik `oauth.json` jest zachowywany, podczas gdy pliki autoryzacji Baileys są usuwane.
    ```

  
</Accordion>
</AccordionGroup>

## Narzędzia, akcje i zapisy konfiguracji

- Narzędzie agenta (reakcje)
- Bramki akcji:
  - `channels.whatsapp.actions.reactions` (bramkowanie reakcji narzędzia WhatsApp).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## Rozwiązywanie problemów (szybkie)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Objaw: `channels status` pokazuje `linked: false` lub ostrzega „Not linked”.

    ```
    Wylogowanie: `openclaw channels logout` (lub `--account <id>`) usuwa stan uwierzytelnienia WhatsApp (zachowuje współdzielone `oauth.json`).
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">Objaw: połączone konto z powtarzającymi się rozłączeniami lub próbami ponownego połączenia.

    ```
    Naprawa: `openclaw doctor` (lub zrestartuj gateway). Jeśli problem się utrzymuje, połącz ponownie przez `channels login` i sprawdź `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">Wysyłki wychodzące kończą się natychmiastowym błędem, gdy nie istnieje aktywny listener gateway dla docelowego konta.

    ```
    Upewnij się, że gateway działa i konto jest połączone.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Sprawdź w tej kolejności:

    ```
    `channels.whatsapp.groups` (lista dozwolonych grup + domyślne bramkowanie wzmianek; użyj `"*"`, aby zezwolić wszystkim)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Środowisko wykonawcze bramki WhatsApp powinno używać Node. WhatsApp (Baileys) i Telegram są niestabilne na Bun. Uruchamiaj gateway na **Node**.
  
</Accordion>
</AccordionGroup>

## Odnośniki do dokumentacji konfiguracji

Główne źródło:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

Kluczowe pola WhatsApp:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>``.enabled`, `accounts.<id>``.authDir`, nadpisania na poziomie konta
- operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- zachowanie sesji: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## Powiązane

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- Przewodnik rozwiązywania problemów: [Gateway troubleshooting](/gateway/troubleshooting).


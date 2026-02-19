---
summary: "Starsza obsługa iMessage przez imsg (JSON-RPC przez stdio). Dla nowych konfiguracji należy używać BlueBubbles."
read_when:
  - Konfigurowanie obsługi iMessage
  - Debugowanie wysyłania/odbierania iMessage
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
**Zalecane:** Do nowych konfiguracji iMessage używaj [BlueBubbles](/channels/bluebubbles).

Kanał `imsg` jest starszą integracją zewnętrznego CLI i może zostać usunięty w przyszłej wersji. 
</Warning>

Status: starsza integracja zewnętrznego CLI. Gateway uruchamia `imsg rpc` (JSON-RPC przez stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">Skonfiguruj iMessage i uruchom gateway.
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">Parowanie jest domyślną wymianą tokenów dla DM-ów iMessage.
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Pełne odniesienie do pól iMessage.
  
</Card>
</CardGroup>

## Szybka konfiguracja

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="Skonfiguruj OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Uruchom gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Zatwierdź pierwsze parowanie DM (domyślna dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            Żądania parowania wygasają po 1 godzinie.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">Jeśli chcesz używać iMessage na innym Macu, ustaw `channels.imessage.cliPath` na opakowanie, które uruchamia `imsg` na zdalnym hoście macOS przez SSH. OpenClaw potrzebuje tylko stdio.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Zalecana konfiguracja, gdy załączniki są włączone:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    Jeśli `remoteHost` nie jest ustawione, OpenClaw próbuje wykryć je automatycznie, parsując polecenie SSH w skrypcie opakowującym.
    ```

  
</Tab>
</Tabs>

## Wymagania i uprawnienia (macOS)

- Kanał iMessage oparty na `imsg` w systemie macOS.
- Pełny dostęp do dysku dla OpenClaw + `imsg` (dostęp do bazy danych Messages).
- Uprawnienie Automatyzacji jest wymagane do wysyłania wiadomości przez Messages.app.

<Tip>
Uprawnienia są przyznawane dla każdego kontekstu procesu osobno. Jeśli gateway działa bez interfejsu graficznego (LaunchAgent/SSH), uruchom jednorazowe polecenie interaktywne w tym samym kontekście, aby wywołać monity:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Kontrola dostępu i routowanie

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` kontroluje wiadomości bezpośrednie:

    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (domyślnie: lista dozwolonych).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom`: lista dozwolonych nadawców w grupach.

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DM używają routowania bezpośredniego; grupy używają routowania grupowego.
    - Przy domyślnym `session.dmScope=main` wiadomości iMessage DM są scalane do głównej sesji agenta.
    DM-y współdzielą główną sesję agenta; grupy są izolowane (`agent:<agentId>:imessage:group:<chat_id>`).<agentId>Grupy:<chat_id>`).
    - Odpowiedzi są kierowane z powrotem do iMessage z użyciem metadanych źródłowego kanału/celu.

    ```
    Jeśli wątek z wieloma uczestnikami dotrze z `is_group=false`, nadal możesz go izolować, `chat_id` używając `channels.imessage.groups` (zobacz „Wątki typu grupowego” poniżej).
    ```

  
</Tab>
</Tabs>

## Wzorce wdrożeniowe

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Jeśli chcesz, aby bot wysyłał z **oddzielnej tożsamości iMessage** (i zachować porządek w osobistych Wiadomościach), użyj dedykowanego Apple ID + dedykowanego użytkownika macOS.

    ```
    Otwórz Messages na tym użytkowniku macOS i zaloguj się do iMessage przy użyciu Apple ID bota.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Typowa topologia:

    ```
    Jeśli Gateway działa na hoście/VM z Linuxem, ale iMessage musi działać na Macu, Tailscale jest najprostszym mostem: Gateway komunikuje się z Makiem przez tailnet, uruchamia `imsg` przez SSH i pobiera załączniki przez SCP.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    Używaj kluczy SSH, aby zarówno SSH, jak i SCP działały bez interakcji.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">Dla konfiguracji z jednym kontem użyj opcji płaskich (`channels.imessage.cliPath`, `channels.imessage.dbPath`) zamiast mapy `accounts`.

    ```
    Każde konto może nadpisywać pola takie jak `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` oraz ustawienia historii.
    ```

  
</Accordion>
</AccordionGroup>

## Media, dzielenie na części i cele dostarczania

<AccordionGroup>
  <Accordion title="Attachments and media">Wysyłanie mediów jest ograniczone przez `channels.imessage.mediaMaxMb` (domyślnie 16).
</Accordion>

  <Accordion title="Outbound chunking">Tekst wychodzący jest dzielony na fragmenty do `channels.imessage.textChunkLimit` (domyślnie 4000).
</Accordion>

  <Accordion title="Addressing formats">
    Preferowane jawne cele:

    ```
    - `chat_id:123` (zalecane dla stabilnego routowania)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Obsługiwane są również cele typu handle:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Zapisy konfiguracji

Domyślnie iMessage może zapisywać aktualizacje konfiguracji wyzwalane przez `/config set|unset` (wymaga `commands.config: true`).

Wyłącz za pomocą:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Rozwiązywanie problemów

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Zweryfikuj plik binarny i obsługę RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Jeśli probe zgłasza brak obsługi RPC, zaktualizuj `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">
    Sprawdź:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (domyślnie: parowanie).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    Sprawdź:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">
    Sprawdź:

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    Uruchom ponownie w interaktywnym terminalu GUI w tym samym kontekście użytkownika/sesji i zatwierdź monity:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Uruchom gateway i zatwierdź wszystkie monity macOS (Automatyzacja + Pełny dostęp do dysku).
    ```

  
</Accordion>
</AccordionGroup>

## Wskaźniki do dokumentacji konfiguracji

- Referencja konfiguracji (iMessage)
- Pełna konfiguracja: [Konfiguracja](/gateway/configuration)
- Szczegóły: [Parowanie](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

---
title: "Parowanie"
---

# Parowanie

„Parowanie” to jawny krok **zatwierdzania przez właściciela** w OpenClaw.
Jest używane w dwóch miejscach:

1. **Parowanie DM-ów** (kto ma prawo rozmawiać z botem)
2. **Parowanie węzłów** (które urządzenia/węzły mogą dołączyć do sieci Gateway)

Kontekst bezpieczeństwa: [Security](/gateway/security)

## 1. Parowanie DM-ów (dostęp do czatu przychodzącego)

Gdy kanał jest skonfigurowany z polityką DM `pairing`, nieznani nadawcy otrzymują krótki kod, a ich wiadomość **nie jest przetwarzana** do momentu zatwierdzenia.

Domyślne polityki DM są opisane w: [Security](/gateway/security)

Kody parowania:

- 8 znaków, wielkie litery, bez znaków dwuznacznych (`0O1I`).
- **Wygasają po 1 godzinie**. Bot wysyła wiadomość parowania tylko wtedy, gdy tworzony jest nowy wniosek (w przybliżeniu raz na godzinę na nadawcę).
- Oczekujące wnioski parowania DM są domyślnie ograniczone do **3 na kanał**; dodatkowe wnioski są ignorowane, dopóki jeden nie wygaśnie lub nie zostanie zatwierdzony.

### Zatwierdź nadawcę

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Obsługiwane kanały: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

### Gdzie przechowywany jest stan

Przechowywane w `~/.openclaw/credentials/`:

- Oczekujące wnioski: `<channel>-pairing.json`
- Zatwierdzona lista dozwolonych: `<channel>-allowFrom.json`

Traktuj je jako wrażliwe (kontrolują dostęp do Twojego asystenta).

## 2. Parowanie urządzeń węzłów (iOS/Android/macOS/węzły headless)

Węzły łączą się z Gateway jako **urządzenia** z `role: node`. Gateway
tworzy wniosek parowania urządzenia, który musi zostać zatwierdzony.

### Parowanie przez Telegram (zalecane dla iOS)

Jeśli używasz wtyczki `device-pair`, możesz przeprowadzić pierwsze parowanie urządzenia w całości z poziomu Telegrama:

1. W Telegramie wyślij do swojego bota wiadomość: `/pair`
2. Bot odpowie dwiema wiadomościami: wiadomością z instrukcjami oraz oddzielną wiadomością z **kodem konfiguracji** (łatwym do skopiowania/wklejenia w Telegramie).
3. Na telefonie otwórz aplikację OpenClaw na iOS → Ustawienia → Gateway.
4. Wklej kod konfiguracji i połącz się.
5. Z powrotem w Telegramie: `/pair approve`

Kod konfiguracji to ładunek JSON zakodowany w base64, który zawiera:

- `url`: adres URL WebSocket Gateway (`ws://...` lub `wss://...`)
- `token`: krótkotrwały token parowania

Traktuj kod konfiguracji jak hasło, dopóki jest ważny.

### Zatwierdzanie urządzenia węzła

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Przechowywanie stanu parowania węzłów

Przechowywane w `~/.openclaw/devices/`:

- `pending.json` (krótkotrwałe; oczekujące wnioski wygasają)
- `paired.json` (sparowane urządzenia + tokeny)

### Uwagi

- Starsze API `node.pair.*` (CLI: `openclaw nodes pending/approve`) to
  osobny, należący do gateway, magazyn parowania. Węzły WS nadal wymagają parowania urządzeń.

## Powiązana dokumentacja

- Model bezpieczeństwa + prompt injection: [Security](/gateway/security)
- Bezpieczne aktualizowanie (uruchom doctor): [Updating](/install/updating)
- Konfiguracje kanałów:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  - iMessage (legacy): [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)



---
summary: "Runbook dla usługi Gateway, jej cyklu życia i operacji"
read_when:
  - Uruchamianie lub debugowanie procesu gateway
title: "Runbook Gateway"
---

# Runbook usługi Gateway

Użyj tej strony do uruchomienia w dniu 1 oraz operacji w dniu 2 usługi Gateway.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Diagnostyka zaczynająca się od objawów z dokładnymi sekwencjami poleceń i sygnaturami logów.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Przewodnik konfiguracji zorientowany na zadania + pełna referencja konfiguracji.
  
</Card>
</CardGroup>

## 5-minutowe uruchomienie lokalne

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
Linux: `openclaw-gateway-<profile>.service`
```

Zdrowa baza: `Runtime: running` i `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Przeładowanie konfiguracji Gateway obserwuje aktywną ścieżkę pliku konfiguracyjnego (rozwiązaną na podstawie domyślnych ustawień profilu/stanu lub `OPENCLAW_CONFIG_PATH`, gdy ustawione).
Tryb domyślny: `gateway.reload.mode="hybrid"` (bezpieczne zmiany stosowane „na gorąco”, restart przy krytycznych).
</Note>

## Model runtime

- Jeden zawsze działający proces do routingu, control plane i połączeń kanałów.
- Multipleks jednego portu.
  - WebSocket control/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - Control UI i hooki
- Domyślny tryb bindowania: `loopback`.
- Uwierzytelnianie Gateway jest domyślnie wymagane: ustaw `gateway.auth.token` (lub `OPENCLAW_GATEWAY_TOKEN`) albo `gateway.auth.password`.

### Port i priorytet bindowania

| Ustawienie                                                         | Kolejność rozwiązywania                                                                                                   |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| Porty pochodne (reguły kciuka): | Priorytet portów: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > domyślny `18789`. |
| Tryb bindowania                                                    | CLI/override → `gateway.bind` → `loopback`                                                                                |

### Tryby hot reload

| Wyłącz za pomocą `gateway.reload.mode="off"`. | Zachowanie                                                                 |
| ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `off`                                                         | Brak przeładowania konfiguracji                                            |
| `hot`                                                         | Zastosuj tylko zmiany bezpieczne dla trybu hot                             |
| `restart`                                                     | Uruchom ponownie przy zmianach wymagających restartu                       |
| `hybrid` (domyślnie)                       | Zastosuj w trybie hot, gdy bezpieczne, w przeciwnym razie uruchom ponownie |

## Zestaw poleceń operatora

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Zdalny dostęp

Preferowane Tailscale/VPN; w przeciwnym razie tunel SSH:
Zapasowo: tunel SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Następnie klienci łączą się z `ws://127.0.0.1:18789` przez tunel.

<Warning>
Jeśli skonfigurowano token, klienci muszą dołączyć go w `connect.params.auth.token`, nawet przez tunel.
</Warning>

Zobacz: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Nadzór i cykl życia usługi

Używaj uruchomień nadzorowanych dla niezawodności na poziomie produkcyjnym.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Etykiety LaunchAgent to `ai.openclaw.gateway` (domyślnie) lub `ai.openclaw.<profile>`, gdy uruchamiasz nazwany profil. `openclaw doctor` sprawdza i naprawia rozbieżności w konfiguracji usługi.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Włącz lingering (wymagane, aby usługa użytkownika przetrwała wylogowanie/bezczynność):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Użyj jednostki systemowej dla hostów wieloużytkownikowych/zawsze włączonych.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Wiele gatewayów (ten sam host)

W większości konfiguracji należy uruchomić **jedną** bramę Gateway.
Używaj wielu Gatewayów tylko dla redundancji lub ścisłej izolacji (np. bot ratunkowy).

Lista kontrolna per instancję:

- unikatowy `gateway.port`
- unikatowy `OPENCLAW_CONFIG_PATH`
- unikatowy `OPENCLAW_STATE_DIR`
- unikatowy `agents.defaults.workspace`

Przykład:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Zobacz [Multiple gateways](/gateway/multiple-gateways).

### Szybka ścieżka dla profilu deweloperskiego

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

Ustawienia domyślne obejmują izolowany stan/konfigurację oraz bazowy port bramy `19001`.

## Protokół (widok operatora)

- Pierwsza ramka klienta musi mieć typ `connect`.
- Gateway zwraca migawkę `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy).
- Żądania: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Typowe zdarzenia: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Uruchomienia agenta są dwuetapowe:

1. Natychmiastowe potwierdzenie przyjęcia (`status:"accepted"`)
2. Odpowiedzi `agent` są dwuetapowe: najpierw potwierdzenie `res` `{runId,status:"accepted"}`, a następnie końcowe `res` `{runId,status:"ok"|"error",summary}` po zakończeniu wykonania; strumieniowane wyjście dociera jako `event:"agent"`.

Pełna dokumentacja: [Gateway protocol](/gateway/protocol) oraz [Bridge protocol (legacy)](/gateway/bridge-protocol).

## Kontrole operacyjne

### Żywotność

- Otwórz połączenie WS i wyślij `connect`.
- Oczekuj odpowiedzi `hello-ok` z migawką.

### Gotowość

```bash
`openclaw gateway health|status` — żądanie health/status przez WS Gateway.
```

### Odzyskiwanie po przerwie

Zdarzenia nie są odtwarzane. W przypadku luk w sekwencji odśwież stan (`health`, `system-presence`) przed kontynuowaniem.

## Typowe sygnatury błędów

| Sygnatura                                                      | Prawdopodobny problem                                    |
| -------------------------------------------------------------- | -------------------------------------------------------- |
| `refusing to bind gateway ... `without auth\`                  | Powiązanie z adresem innym niż loopback bez tokena/hasła |
| `another gateway instance is already listening` / `EADDRINUSE` | Konflikt portów                                          |
| `Gateway start blocked: set gateway.mode=local`                | Konfiguracja ustawiona na tryb zdalny                    |
| `unauthorized` podczas łączenia                                | Niezgodność uwierzytelniania między klientem a Gateway   |

Aby uzyskać pełne ścieżki diagnostyczne, użyj [Gateway Troubleshooting](/gateway/troubleshooting).

## Gwarancje bezpieczeństwa

- Klienci protokołu Gateway szybko przerywają działanie, gdy Gateway jest niedostępny (brak domyślnego przełączenia na kanał bezpośredni).
- Ramki inne niż pierwsza ramka połączenia lub niepoprawny JSON są odrzucane, a gniazdo jest zamykane.
- Łagodne zamknięcie: emituj zdarzenie `shutdown` przed zamknięciem; klienci muszą obsłużyć zamknięcie + ponowne połączenie.

---

Powiązane:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

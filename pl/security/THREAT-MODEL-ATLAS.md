# Model zagrożeń OpenClaw v1.0

## Framework MITRE ATLAS

**Wersja:** 1.0-draft  
**Ostatnia aktualizacja:** 2026-02-04  
**Metodologia:** MITRE ATLAS + Diagramy przepływu danych  
**Framework:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Źródło frameworka

Ten model zagrożeń opiera się na [MITRE ATLAS](https://atlas.mitre.org/), branżowym standardzie dokumentowania zagrożeń adversarialnych dla systemów AI/ML. ATLAS jest utrzymywany przez [MITRE](https://www.mitre.org/) we współpracy ze społecznością bezpieczeństwa AI.

**Kluczowe zasoby ATLAS:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contributing to ATLAS](https://atlas.mitre.org/resources/contribute)

### Współtworzenie tego modelu zagrożeń

To żywy dokument utrzymywany przez społeczność OpenClaw. Zobacz [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md), aby zapoznać się z wytycznymi dotyczącymi współtworzenia:

- Zgłaszanie nowych zagrożeń
- Aktualizowanie istniejących zagrożeń
- Proponowanie łańcuchów ataków
- Sugerowanie mechanizmów mitygacji

---

## 1. Wprowadzenie

### 1.1 Cel

Ten model zagrożeń dokumentuje zagrożenia adversarialne dla platformy agenta AI OpenClaw oraz marketplace’u umiejętności ClawHub, wykorzystując framework MITRE ATLAS zaprojektowany specjalnie dla systemów AI/ML.

### 1.2 Zakres

| Komponent               | Ujęty | Uwagi                                            |
| ----------------------- | ----- | ------------------------------------------------ |
| OpenClaw Agent Runtime  | Tak   | Wykonywanie agenta, wywołania narzędzi, sesje   |
| Gateway                 | Tak   | Uwierzytelnianie, routing, integracje kanałów   |
| Integracje kanałów      | Tak   | WhatsApp, Telegram, Discord, Signal, Slack itp. |
| ClawHub Marketplace     | Tak   | Publikowanie, moderacja, dystrybucja umiejętności |
| MCP Servers             | Tak   | Zewnętrzni dostawcy narzędzi                     |
| Urządzenia użytkowników | Częściowo | Aplikacje mobilne, klienci desktopowi     |

### 1.3 Poza zakresem

W tym modelu zagrożeń nic nie zostało jednoznacznie wyłączone z zakresu.

---

## 2. Architektura systemu

### 2.1 Granice zaufania

```
┌─────────────────────────────────────────────────────────────────┐
│                    STREFA NIEZAUFANA                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│        GRANICA ZAUFANIA 1: Dostęp przez kanał                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Parowanie urządzeń (30 s okresu karencji)              │   │
│  │  • Walidacja AllowFrom / AllowList                        │   │
│  │  • Uwierzytelnianie Token/Password/Tailscale              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        GRANICA ZAUFANIA 2: Izolacja sesji                        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   SESJE AGENTA                            │   │
│  │  • Klucz sesji = agent:channel:peer                       │   │
│  │  • Polityki narzędzi per agent                            │   │
│  │  • Logowanie transkryptów                                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        GRANICA ZAUFANIA 3: Wykonywanie narzędzi                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                PIASKOWNICA WYKONAWCZA                     │   │
│  │  • Piaskownica Docker LUB Host (exec-approvals)           │   │
│  │  • Zdalne wykonanie Node                                   │   │
│  │  • Ochrona SSRF (DNS pinning + blokowanie IP)              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        GRANICA ZAUFANIA 4: Treści zewnętrzne                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │          POBRANE URL / E-MAILE / WEBHOOKI                │   │
│  │  • Owijanie treści zewnętrznych (tagi XML)               │   │
│  │  • Wstrzykiwanie komunikatu bezpieczeństwa               │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        GRANICA ZAUFANIA 5: Łańcuch dostaw                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Publikowanie umiejętności (semver, wymagany SKILL.md)  │   │
│  │  • Flagi moderacji oparte na wzorcach                     │   │
│  │  • Skanowanie VirusTotal (wkrótce)                        │   │
│  │  • Weryfikacja wieku konta GitHub                         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Przepływy danych

| Przepływ | Źródło  | Cel      | Dane                | Ochrona              |
| -------- | ------- | -------- | ------------------- | -------------------- |
| F1       | Kanał   | Gateway  | Wiadomości użytkownika | TLS, AllowFrom    |
| F2       | Gateway | Agent    | Przekierowane wiadomości | Izolacja sesji |
| F3       | Agent   | Narzędzia| Wywołania narzędzi  | Egzekwowanie polityk |
| F4       | Agent   | Zewnętrzne | Żądania web_fetch | Blokowanie SSRF      |
| F5       | ClawHub | Agent    | Kod umiejętności    | Moderacja, skanowanie |
| F6       | Agent   | Kanał    | Odpowiedzi          | Filtrowanie wyjścia  |

---

## 3. Analiza zagrożeń według taktyk ATLAS

### 3.1 Rozpoznanie (AML.TA0002)

#### T-RECON-001: Odkrywanie endpointów agenta

| Atrybut                 | Wartość                                                             |
| ----------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                         |
| **Opis**                | Atakujący skanuje w poszukiwaniu wystawionych endpointów gateway OpenClaw |
| **Wektor ataku**        | Skanowanie sieci, zapytania shodan, enumeracja DNS                 |
| **Dotknięte komponenty**| Gateway, wystawione endpointy API                                   |
| **Obecne mitygacje**    | Opcjonalne uwierzytelnianie Tailscale, domyślne bindowanie do loopback |
| **Ryzyko rezydualne**   | Średnie – publiczne gatewaye możliwe do wykrycia                   |
| **Rekomendacje**        | Udokumentować bezpieczne wdrożenia, dodać rate limiting dla endpointów wykrywania |

#### T-RECON-002: Sondowanie integracji kanałów

| Atrybut                 | Wartość                                                             |
| ----------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                         |
| **Opis**                | Atakujący sonduje kanały komunikacyjne, aby zidentyfikować konta zarządzane przez AI |
| **Wektor ataku**        | Wysyłanie wiadomości testowych, obserwacja wzorców odpowiedzi      |
| **Dotknięte komponenty**| Wszystkie integracje kanałów                                        |
| **Obecne mitygacje**    | Brak specyficznych                                                  |
| **Ryzyko rezydualne**   | Niskie – samo wykrycie ma ograniczoną wartość                      |
| **Rekomendacje**        | Rozważyć losowe opóźnienia odpowiedzi                               |

---

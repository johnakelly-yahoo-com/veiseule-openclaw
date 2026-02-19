---
summary: "Przegląd opcji i ścieżek onboardingu OpenClaw"
read_when:
  - Wybór ścieżki onboardingu
  - Konfigurowanie nowego środowiska
title: "Przegląd wdrożenia"
sidebarTitle: "Przegląd wdrożenia"
---

# Przegląd wdrożenia

OpenClaw obsługuje wiele ścieżek wdrożenia w zależności od tego, gdzie działa Gateway
i jak wolisz konfigurować dostawców.

## Wybierz ścieżkę wdrożenia

- **Kreator CLI** dla macOS, Linux i Windows (przez WSL2).
- **Aplikacja macOS** dla prowadzonego pierwszego uruchomienia na Macach z Apple silicon lub Intel.

## Kreator wdrożenia CLI

Uruchom kreator w terminalu:

```bash
openclaw onboard
```

Użyj kreatora CLI, gdy chcesz mieć pełną kontrolę nad Gateway, przestrzenią roboczą,
kanałami i umiejętnościami. Dokumentacja:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## Wdrożenie przez aplikację macOS

Użyj aplikacji OpenClaw, gdy chcesz w pełni prowadzonej konfiguracji w macOS. Dokumentacja:

- [Onboarding (macOS App)](/start/onboarding)

## Niestandardowy dostawca

Jeśli potrzebujesz punktu końcowego, którego nie ma na liście, w tym hostowanych dostawców,
udostępniających standardowe API OpenAI lub Anthropic, wybierz **Custom Provider** w
kreatorze CLI. Zostaniesz poproszony o:

- Wybranie opcji zgodnej z OpenAI, zgodnej z Anthropic lub **Unknown** (automatyczne wykrywanie).
- Wprowadzenie bazowego URL i klucza API (jeśli wymagany przez dostawcę).
- Podanie identyfikatora modelu oraz opcjonalnego aliasu.
- Wybranie identyfikatora Endpoint, aby wiele niestandardowych endpointów mogło współistnieć.

Aby uzyskać szczegółowe instrukcje, skorzystaj z dokumentacji wdrożenia CLI powyżej.


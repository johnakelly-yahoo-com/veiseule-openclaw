# Współtworzenie modelu zagrożeń OpenClaw

Dziękujemy za pomoc w zwiększaniu bezpieczeństwa OpenClaw. Ten model zagrożeń jest żywym dokumentem i zapraszamy do współtworzenia każdego — nie musisz być ekspertem ds. bezpieczeństwa.

## Ways to Contribute

### Dodaj zagrożenie

Zauważyłeś wektor ataku lub ryzyko, którego nie uwzględniliśmy? Otwórz zgłoszenie w [openclaw/trust](https://github.com/openclaw/trust/issues) i opisz je własnymi słowami. Nie musisz znać żadnych frameworków ani wypełniać wszystkich pól — po prostu opisz scenariusz.

**Warto uwzględnić (opcjonalnie):**

- Scenariusz ataku i sposób, w jaki mógłby zostać wykorzystany
- Które części OpenClaw są dotknięte (CLI, gateway, kanały, ClawHub, serwery MCP itp.)
- Jak oceniasz jego powagę (niska / średnia / wysoka / krytyczna)
- Linki do powiązanych badań, CVE lub przykładów z rzeczywistego świata

We'll handle the ATLAS mapping, threat IDs, and risk assessment during review. If you want to include those details, great - but it's not expected.

> **This is for adding to the threat model, not reporting live vulnerabilities.** If you've found an exploitable vulnerability, see our [Trust page](https://trust.openclaw.ai) for responsible disclosure instructions.

### Suggest a Mitigation

Have an idea for how to address an existing threat? Open an issue or PR referencing the threat. Useful mitigations are specific and actionable - for example, "per-sender rate limiting of 10 messages/minute at the gateway" is better than "implement rate limiting."

### Propose an Attack Chain

Attack chains show how multiple threats combine into a realistic attack scenario. If you see a dangerous combination, describe the steps and how an attacker would chain them together. A short narrative of how the attack unfolds in practice is more valuable than a formal template.

### Fix or Improve Existing Content

Typos, clarifications, outdated info, better examples - PRs welcome, no issue needed.

## Czego używamy

### MITRE ATLAS

Ten model zagrożeń opiera się na [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), ramach zaprojektowanych specjalnie dla zagrożeń AI/ML, takich jak wstrzykiwanie promptów, niewłaściwe użycie narzędzi i wykorzystywanie agentów. Nie musisz znać ATLAS, aby wnieść wkład — mapujemy zgłoszenia do frameworku podczas przeglądu.

### Identyfikatory zagrożeń

Każde zagrożenie otrzymuje identyfikator, taki jak `T-EXEC-003`. Kategorie to:

| Kod     | Kategoria                                   |
| ------- | ------------------------------------------- |
| RECON   | Rozpoznanie — zbieranie informacji          |
| ACCESS  | Dostęp początkowy — uzyskanie wejścia       |
| EXEC    | Wykonanie — uruchamianie złośliwych działań |
| PERSIST | Trwałość — utrzymywanie dostępu             |
| EVADE   | Unikanie obrony — unikanie wykrycia         |
| DISC    | Odkrywanie — poznawanie środowiska          |
| EXFIL   | Eksfiltracja — kradzież danych              |
| IMPACT  | Wpływ — szkody lub zakłócenia               |

Identyfikatory są przypisywane przez opiekunów podczas przeglądu. Nie musisz wybierać jednego.

### Poziomy ryzyka

| Poziom                                | 28. Znaczenie                                          |
| ------------------------------------- | ----------------------------------------------------------------------------- |
| **Krytyczny**                         | Pełne przejęcie systemu lub wysokie prawdopodobieństwo + krytyczny wpływ      |
| 31. **Wysoki** | Znaczne szkody prawdopodobne lub średnie prawdopodobieństwo + krytyczny wpływ |
| **Średni**                            | Umiarkowane ryzyko lub niskie prawdopodobieństwo + wysoki wpływ               |
| 35. **Niski**  | Mało prawdopodobne i o ograniczonym wpływie                                   |

Jeśli nie masz pewności co do poziomu ryzyka, po prostu opisz wpływ, a my go ocenimy.

## Proces przeglądu

1. **Triaging** — Przeglądamy nowe zgłoszenia w ciągu 48 godzin
2. **Ocena** — Weryfikujemy wykonalność, przypisujemy mapowanie ATLAS i identyfikator zagrożenia, weryfikujemy poziom ryzyka
3. **Dokumentacja** — Zapewniamy, że wszystko jest sformatowane i kompletne
4. **Scalenie** — Dodanie do modelu zagrożeń i wizualizacji

## 43) Zasoby

- [Strona ATLAS](https://atlas.mitre.org/)
- [Techniki ATLAS](https://atlas.mitre.org/techniques/)
- [Studia przypadków ATLAS](https://atlas.mitre.org/studies/)
- [Model zagrożeń OpenClaw](./THREAT-MODEL-ATLAS.md)

## Kontakt

- **Luki bezpieczeństwa:** Zobacz naszą [stronę Zaufanie](https://trust.openclaw.ai), aby uzyskać instrukcje zgłaszania
- **Pytania dotyczące modelu zagrożeń:** Otwórz zgłoszenie w [openclaw/trust](https://github.com/openclaw/trust/issues)
- **General chat:** Discord #security channel

## Recognition

Contributors to the threat model are recognized in the threat model acknowledgments, release notes, and the OpenClaw security hall of fame for significant contributions.


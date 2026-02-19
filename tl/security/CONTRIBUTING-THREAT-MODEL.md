# Pag-aambag sa OpenClaw Threat Model

Thanks for helping make OpenClaw more secure. This threat model is a living document and we welcome contributions from anyone - you don't need to be a security expert.

## Mga Paraan ng Pag-aambag

### Magdagdag ng Banta

Spotted an attack vector or risk we haven't covered? Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues) and describe it in your own words. You don't need to know any frameworks or fill in every field - just describe the scenario.

**Makakatulong isama (ngunit hindi kailangan):**

- Ang senaryo ng pag-atake at kung paano ito maaaring maabuso
- Aling bahagi ng OpenClaw ang apektado (CLI, gateway, channels, ClawHub, MCP servers, atbp.)
- Gaano ito kalubha sa tingin mo (low / medium / high / critical)
- Mga link sa kaugnay na pananaliksik, CVEs, o totoong halimbawa

We'll handle the ATLAS mapping, threat IDs, and risk assessment during review. If you want to include those details, great - but it's not expected.

> **Ito ay para sa pagdaragdag sa threat model, hindi para sa pagre-report ng aktibong vulnerabilities.** Kung may natuklasan kang vulnerability na maaaring ma-exploit, tingnan ang aming [Trust page](https://trust.openclaw.ai) para sa mga tagubilin sa responsableng pag-disclose.

### Magmungkahi ng Mitigation

Have an idea for how to address an existing threat? Open an issue or PR referencing the threat. Useful mitigations are specific and actionable - for example, "per-sender rate limiting of 10 messages/minute at the gateway" is better than "implement rate limiting."

### Magmungkahi ng Attack Chain

Attack chains show how multiple threats combine into a realistic attack scenario. If you see a dangerous combination, describe the steps and how an attacker would chain them together. A short narrative of how the attack unfolds in practice is more valuable than a formal template.

### Ayusin o Pagbutihin ang Umiiral na Nilalaman

Mga typo, paglilinaw, luma nang impormasyon, mas magagandang halimbawa — malugod ang PRs, hindi na kailangan ng issue.

## Ano ang Ginagamit Namin

### MITRE ATLAS

This threat model is built on [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), a framework designed specifically for AI/ML threats like prompt injection, tool misuse, and agent exploitation. You don't need to know ATLAS to contribute - we map submissions to the framework during review.

### Mga ID ng Banta

Each threat gets an ID like `T-EXEC-003`. The categories are:

| Kodigo  | Kategorya                                          |
| ------- | -------------------------------------------------- |
| RECON   | Reconnaissance - pangangalap ng impormasyon        |
| ACCESS  | Initial access - pagkakaroon ng paunang pagpasok   |
| EXEC    | Execution - pagpapatakbo ng mga malisyosong aksyon |
| PERSIST | Persistence - pagpapanatili ng access              |
| EVADE   | Defense evasion - pag-iwas sa pagtukoy             |
| DISC    | Discovery - pag-aaral tungkol sa kapaligiran       |
| EXFIL   | Exfiltration - pagnanakaw ng data                  |
| IMPACT  | Impact - pinsala o pagkaantala                     |

IDs are assigned by maintainers during review. You don't need to pick one.

### Mga Antas ng Panganib

| Antas          | Kahulugan                                                                         |
| -------------- | --------------------------------------------------------------------------------- |
| **Kritikal**   | Buong kompromiso ng sistema, o mataas ang posibilidad + kritikal ang epekto       |
| **Mataas**     | Malamang na may malaking pinsala, o katamtamang posibilidad + kritikal ang epekto |
| **Katamtaman** | Katamtamang panganib, o mababang posibilidad + mataas na epekto                   |
| **Mababa**     | Hindi malamang at limitado ang epekto                                             |

Kung hindi ka sigurado sa antas ng panganib, ilarawan lamang ang epekto at kami na ang magtatasa.

## Proseso ng Review

1. **Triage** - Nirerepaso namin ang mga bagong submission sa loob ng 48 oras
2. **Assessment** - Tinitiyak namin ang pagiging posible, nagtatalaga ng ATLAS mapping at threat ID, at bineberipika ang antas ng panganib
3. **Documentation** - Tinitiyak naming maayos ang format at kumpleto ang lahat
4. **Merge** - Idinadagdag sa threat model at visualization

## Mga Resource

- [ATLAS Website](https://atlas.mitre.org/)
- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [OpenClaw Threat Model](./THREAT-MODEL-ATLAS.md)

## Contact

- **Security vulnerabilities:** Tingnan ang aming [Trust page](https://trust.openclaw.ai) para sa mga tagubilin sa pagre-report
- **Mga tanong tungkol sa threat model:** Magbukas ng issue sa [openclaw/trust](https://github.com/openclaw/trust/issues)
- **Pangkalahatang usapan:** Discord #security channel

## Pagkilala

Ang mga nag-ambag sa threat model ay kinikilala sa acknowledgments ng threat model, mga release notes, at sa OpenClaw security hall of fame para sa mahahalagang kontribusyon.


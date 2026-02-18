# Pag-aambag sa OpenClaw Threat Model

Salamat sa pagtulong na gawing mas secure ang OpenClaw. Ang threat model na ito ay isang buhay na dokumento at tinatanggap namin ang mga ambag mula sa kahit sino — hindi mo kailangang maging eksperto sa seguridad.

## Mga Paraan ng Pag-aambag

### Magdagdag ng Banta

May napansin ka bang attack vector o panganib na hindi pa namin natutukoy? Magbukas ng issue sa [openclaw/trust](https://github.com/openclaw/trust/issues) at ilarawan ito sa sarili mong mga salita. Hindi mo kailangang alam ang anumang framework o punan ang bawat field — ilarawan lamang ang senaryo.

**Makakatulong isama (ngunit hindi kailangan):**

- Ang senaryo ng pag-atake at kung paano ito maaaring maabuso
- Aling bahagi ng OpenClaw ang apektado (CLI, gateway, channels, ClawHub, MCP servers, atbp.)
- Gaano ito kalubha sa tingin mo (low / medium / high / critical)
- Mga link sa kaugnay na pananaliksik, CVEs, o totoong halimbawa

Kami na ang bahala sa ATLAS mapping, threat IDs, at risk assessment sa panahon ng review. Kung nais mong isama ang mga detalyeng iyon, ayos lang — pero hindi ito inaasahan.

> **Ito ay para sa pagdaragdag sa threat model, hindi para sa pagre-report ng aktibong vulnerabilities.** Kung may natuklasan kang vulnerability na maaaring ma-exploit, tingnan ang aming [Trust page](https://trust.openclaw.ai) para sa mga tagubilin sa responsableng pag-disclose.

### Magmungkahi ng Mitigation

May ideya ka ba kung paano tugunan ang isang umiiral na banta? Magbukas ng issue o PR na tumutukoy sa threat. Ang kapaki-pakinabang na mitigations ay tiyak at naaaksyunan — halimbawa, mas mabuti ang "per-sender rate limiting na 10 mensahe/minuto sa gateway" kaysa sa "magpatupad ng rate limiting."

### Magmungkahi ng Attack Chain

Ipinapakita ng attack chains kung paano nagsasama-sama ang maraming banta upang makabuo ng isang makatotohanang senaryo ng pag-atake. Kung may nakikita kang mapanganib na kombinasyon, ilarawan ang mga hakbang at kung paano ito pagsasamahin ng isang attacker. Mas mahalaga ang isang maikling salaysay kung paano nagaganap ang pag-atake sa aktwal na sitwasyon kaysa sa isang pormal na template.

### Ayusin o Pagbutihin ang Umiiral na Nilalaman

Mga typo, paglilinaw, luma nang impormasyon, mas magagandang halimbawa — malugod ang PRs, hindi na kailangan ng issue.

## Ano ang Ginagamit Namin

### MITRE ATLAS

Ang threat model na ito ay nakabatay sa [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), isang framework na sadyang dinisenyo para sa mga banta sa AI/ML tulad ng prompt injection, maling paggamit ng tools, at agent exploitation. Hindi mo kailangang alam ang ATLAS upang makapag-ambag — kami ang nagmamapa ng mga submission sa framework sa panahon ng review.

### Mga ID ng Banta

Bawat banta ay may ID tulad ng `T-EXEC-003`. Ang mga kategorya ay:

| Kodigo    | Kategorya                                   |
| ------- | ------------------------------------------ |
| RECON   | Reconnaissance - pangangalap ng impormasyon |
| ACCESS  | Initial access - pagkakaroon ng paunang pagpasok |
| EXEC    | Execution - pagpapatakbo ng mga malisyosong aksyon |
| PERSIST | Persistence - pagpapanatili ng access      |
| EVADE   | Defense evasion - pag-iwas sa pagtukoy     |
| DISC    | Discovery - pag-aaral tungkol sa kapaligiran |
| EXFIL   | Exfiltration - pagnanakaw ng data          |
| IMPACT  | Impact - pinsala o pagkaantala             |

Ang mga ID ay itinatalaga ng mga maintainer sa panahon ng review. Hindi mo kailangang pumili ng isa.

### Mga Antas ng Panganib

| Antas        | Kahulugan                                                           |
| ------------ | ----------------------------------------------------------------- |
| **Kritikal** | Buong kompromiso ng sistema, o mataas ang posibilidad + kritikal ang epekto |
| **Mataas**     | Malamang na may malaking pinsala, o katamtamang posibilidad + kritikal ang epekto |
| **Katamtaman**   | Katamtamang panganib, o mababang posibilidad + mataas na epekto |
| **Mababa**      | Hindi malamang at limitado ang epekto                            |

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


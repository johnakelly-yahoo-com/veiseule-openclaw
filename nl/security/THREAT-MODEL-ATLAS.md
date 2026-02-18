# OpenClaw Dreigingsmodel v1.0

## MITRE ATLAS-framework

**Versie:** 1.0-draft  
**Laatst bijgewerkt:** 2026-02-04  
**Methodologie:** MITRE ATLAS + Data Flow Diagrams  
**Framework:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Framework-toeschrijving

Dit dreigingsmodel is gebaseerd op [MITRE ATLAS](https://atlas.mitre.org/), het industriestandaardframework voor het documenteren van adversariële dreigingen voor AI/ML-systemen. ATLAS wordt onderhouden door [MITRE](https://www.mitre.org/) in samenwerking met de AI-beveiligingsgemeenschap.

**Belangrijke ATLAS-bronnen:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Bijdragen aan ATLAS](https://atlas.mitre.org/resources/contribute)

### Bijdragen aan dit dreigingsmodel

Dit is een levend document dat wordt onderhouden door de OpenClaw-community. Zie [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) voor richtlijnen om bij te dragen:

- Nieuwe dreigingen melden
- Bestaande dreigingen bijwerken
- Aanvalsketens voorstellen
- Mitigaties suggereren

---

## 1. Inleiding

### 1.1 Doel

Dit dreigingsmodel documenteert adversariële dreigingen voor het OpenClaw AI-agentplatform en de ClawHub-skillmarktplaats, met gebruik van het MITRE ATLAS-framework dat specifiek is ontworpen voor AI/ML-systemen.

### 1.2 Reikwijdte

| Component              | Inbegrepen | Opmerkingen                                        |
| ---------------------- | ---------- | -------------------------------------------------- |
| OpenClaw Agent Runtime | Ja         | Kernuitvoering van agenten, toolaanroepen, sessies |
| Gateway                | Ja         | Authenticatie, routering, kanaalintegratie         |
| Kanaalintegraties      | Ja         | WhatsApp, Telegram, Discord, Signal, Slack, enz.  |
| ClawHub Marktplaats    | Ja         | Publicatie, moderatie, distributie van skills      |
| MCP-servers            | Ja         | Externe toolproviders                              |
| Gebruikersapparaten    | Gedeeltelijk | Mobiele apps, desktopclients                     |

### 1.3 Buiten scope

Er is niets expliciet buiten scope voor dit dreigingsmodel.

---

## 2. Systeemarchitectuur

### 2.1 Vertrouwensgrenzen

```
┌─────────────────────────────────────────────────────────────────┐
│                    NIET-VERTROUWDE ZONE                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│            VERTROUWENSGRENS 1: Kanaaltoegang                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Apparaatkoppeling (30s respijtperiode)                 │   │
│  │  • AllowFrom / AllowList-validatie                        │   │
│  │  • Token/Password/Tailscale-authenticatie                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          VERTROUWENSGRENS 2: Sessie-isolatie                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENTSESSIES                            │   │
│  │  • Sessiesleutel = agent:channel:peer                     │   │
│  │  • Toolbeleid per agent                                   │   │
│  │  • Transcriptlogging                                       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          VERTROUWENSGRENS 3: Tooluitvoering                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                UITVOERINGSSANDBOX                         │   │
│  │  • Docker-sandbox OF Host (exec-approvals)                │   │
│  │  • Node remote execution                                   │   │
│  │  • SSRF-bescherming (DNS-pinning + IP-blokkering)         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        VERTROUWENSGRENS 4: Externe content                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │        OPGEHAALDE URL's / E-MAILS / WEBHOOKS             │   │
│  │  • Verpakken van externe content (XML-tags)              │   │
│  │  • Injectie van beveiligingsmelding                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        VERTROUWENSGRENS 5: Supply chain                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Skillpublicatie (semver, SKILL.md verplicht)          │   │
│  │  • Patroon-gebaseerde moderatievlaggen                   │   │
│  │  • VirusTotal-scanning (binnenkort beschikbaar)          │   │
│  │  • Verificatie van GitHub-accountleeftijd                │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Datastromen

| Stroom | Bron    | Bestemming | Data               | Beveiliging          |
| ---- | ------- | ---------- | ------------------ | -------------------- |
| F1   | Kanaal  | Gateway    | Gebruikersberichten | TLS, AllowFrom       |
| F2   | Gateway | Agent      | Gerouteerde berichten | Sessie-isolatie   |
| F3   | Agent   | Tools      | Toolaanroepen      | Beleidsafdwinging    |
| F4   | Agent   | Extern     | web_fetch-verzoeken | SSRF-blokkering     |
| F5   | ClawHub | Agent      | Skillcode          | Moderatie, scanning  |
| F6   | Agent   | Kanaal     | Antwoorden         | Outputfiltering      |

---

## 3. Dreigingsanalyse per ATLAS-tactiek

### 3.1 Verkenning (AML.TA0002)

#### T-RECON-001: Ontdekking van agent-endpoints

| Attribuut               | Waarde                                                               |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                          |
| **Beschrijving**        | Aanvaller scant naar blootgestelde OpenClaw-gateway-endpoints       |
| **Aanvalsvector**       | Netwerkscans, shodan-queries, DNS-enumeratie                        |
| **Getroffen componenten** | Gateway, blootgestelde API-endpoints                              |
| **Huidige mitigaties**  | Tailscale-authoptie, standaard binden aan loopback                  |
| **Restrisico**          | Middel - Publieke gateways zijn vindbaar                             |
| **Aanbevelingen**       | Documenteer veilige uitrol, voeg rate limiting toe op discovery-endpoints |

#### T-RECON-002: Probing van kanaalintegraties

| Attribuut               | Waarde                                                             |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                        |
| **Beschrijving**        | Aanvaller test berichtkanalen om AI-beheerde accounts te identificeren |
| **Aanvalsvector**       | Testberichten sturen, responspatronen observeren                  |
| **Getroffen componenten** | Alle kanaalintegraties                                         |
| **Huidige mitigaties**  | Geen specifieke                                                   |
| **Restrisico**          | Laag - Beperkte waarde van enkel ontdekking                       |
| **Aanbevelingen**       | Overweeg randomisatie van responstiming                           |

---



# OpenClaw Hotmodell v1.0

## MITRE ATLAS-ramverket

**Version:** 1.0-utkast  
**Senast uppdaterad:** 2026-02-04  
**Metodik:** MITRE ATLAS + Dataflödesdiagram  
**Ramverk:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Ramverksattribution

Denna hotmodell är byggd på [MITRE ATLAS](https://atlas.mitre.org/), branschstandarden för att dokumentera adversariella hot mot AI/ML-system. ATLAS förvaltas av [MITRE](https://www.mitre.org/) i samarbete med AI-säkerhetscommunityn.

**Viktiga ATLAS-resurser:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contributing to ATLAS](https://atlas.mitre.org/resources/contribute)

### Bidra till denna hotmodell

Detta är ett levande dokument som underhålls av OpenClaw-communityn. Se [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) för riktlinjer för hur du bidrar:

- Rapportera nya hot
- Uppdatera befintliga hot
- Föreslå attackkedjor
- Föreslå skyddsåtgärder

---

## 1. Introduktion

### 1.1 Syfte

Denna hotmodell dokumenterar adversariella hot mot OpenClaw AI-agentplattformen och ClawHub-marknadsplatsen för skills, med hjälp av MITRE ATLAS-ramverket som är särskilt utformat för AI/ML-system.

### 1.2 Omfattning

| Komponent              | Ingår | Kommentar                                         |
| ---------------------- | ----- | ------------------------------------------------- |
| OpenClaw Agent-körtidsmiljö | Ja    | Kärnexekvering av agent, verktygsanrop, sessioner |
| Gateway                | Ja    | Autentisering, routing, kanalintegrationer        |
| Kanalkopplingar        | Ja    | WhatsApp, Telegram, Discord, Signal, Slack, etc.  |
| ClawHub-marknadsplats    | Ja    | Publicering, moderering, distribution av skills   |
| MCP-servrar            | Ja    | Externa verktygsleverantörer                      |
| Användarenheter        | Delvis| Mobilappar, skrivbordsklienter                    |

### 1.3 Utanför omfattning

Inget är uttryckligen utanför omfattningen för denna hotmodell.

---

## 2. Systemarkitektur

### 2.1 Förtroendegränser

```
┌─────────────────────────────────────────────────────────────────┐
│                    ICKE BETRODD ZON                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│             FÖRTROENDEGRÄNS 1: Kanalåtkomst                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Enhetsparning (30 s respitperiod)                      │   │
│  │  • AllowFrom / AllowList-validering                       │   │
│  │  • Token-/lösenords-/Tailscale-autentisering              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           FÖRTROENDEGRÄNS 2: Sessionsisolering                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENTSESSIONER                         │   │
│  │  • Sessionsnyckel = agent:channel:peer                   │   │
│  │  • Verktygspolicyer per agent                            │   │
│  │  • Loggning av transkript                                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            FÖRTROENDEGRÄNS 3: Verktygsexekvering                │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  EXEKVERINGSSANDBOX                      │   │
│  │  • Docker-sandbox ELLER Host (exec-approvals)            │   │
│  │  • Fjärrkörning via Node                                 │   │
│  │  • SSRF-skydd (DNS-pinning + IP-blockering)              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            FÖRTROENDEGRÄNS 4: Externt innehåll                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │            HÄMTADE URL:ER / E-POST / WEBHOOKS           │   │
│  │  • Inpackning av externt innehåll (XML-taggar)           │   │
│  │  • Injektion av säkerhetsmeddelande                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│            FÖRTROENDEGRÄNS 5: Leverantörskedja                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Publicering av skills (semver, SKILL.md krävs)        │   │
│  │  • Mönsterbaserade modereringsflaggor                    │   │
│  │  • VirusTotal-skanning (kommer snart)                    │   │
│  │  • Verifiering av GitHub-kontots ålder                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Dataflöden

| Flöde | Källa   | Destination | Data                 | Skydd                |
| ----- | ------- | ----------- | -------------------- | -------------------- |
| F1    | Kanal   | Gateway     | Användarmeddelanden  | TLS, AllowFrom       |
| F2    | Gateway | Agent       | Routade meddelanden  | Sessionsisolering    |
| F3    | Agent   | Verktyg     | Verktygsanrop        | Policytillämpning    |
| F4    | Agent   | Extern      | web_fetch-förfrågningar | SSRF-blockering   |
| F5    | ClawHub | Agent       | Skill-kod            | Moderering, skanning |
| F6    | Agent   | Kanal       | Svar                 | Utdatafiltrering     |

---

## 3. Hotanalys per ATLAS-taktik

### 3.1 Rekognosering (AML.TA0002)

#### T-RECON-001: Upptäckt av agentändpunkt

| Attribut                | Värde                                                               |
| ----------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                         |
| **Beskrivning**         | Angripare skannar efter exponerade OpenClaw gateway-ändpunkter     |
| **Attackvektor**        | Nätverksskanning, shodan-frågor, DNS-enumerering                   |
| **Påverkade komponenter** | Gateway, exponerade API-ändpunkter                              |
| **Nuvarande skydd**     | Tailscale-autentisering som alternativ, binder till loopback som standard |
| **Kvarstående risk**    | Medel - Publika gateways kan upptäckas                             |
| **Rekommendationer**    | Dokumentera säker driftsättning, lägg till rate limiting på upptäcktsändpunkter |

#### T-RECON-002: Sondering av kanalintegration

| Attribut                | Värde                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                        |
| **Beskrivning**         | Angripare sonderar meddelandekanaler för att identifiera AI-hanterade konton |
| **Attackvektor**        | Skicka testmeddelanden, observera svarsmönster                    |
| **Påverkade komponenter** | Alla kanalintegrationer                                         |
| **Nuvarande skydd**     | Inga specifika                                                     |
| **Kvarstående risk**    | Låg - Begränsat värde enbart från upptäckt                        |
| **Rekommendationer**    | Överväg slumpmässig variation av svarstider                       |

---



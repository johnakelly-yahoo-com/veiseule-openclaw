# OpenClaw Bedrohungsmodell v1.0

## MITRE ATLAS-Framework

**Version:** 1.0-draft  
**Zuletzt aktualisiert:** 2026-02-04  
**Methodik:** MITRE ATLAS + Datenflussdiagramme  
**Framework:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Framework-Zuordnung

Dieses Bedrohungsmodell basiert auf [MITRE ATLAS](https://atlas.mitre.org/), dem branchenüblichen Standard-Framework zur Dokumentation adversarialer Bedrohungen für KI/ML-Systeme. ATLAS wird von [MITRE](https://www.mitre.org/) in Zusammenarbeit mit der KI-Sicherheits-Community gepflegt.

**Zentrale ATLAS-Ressourcen:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contributing to ATLAS](https://atlas.mitre.org/resources/contribute)

### Mitwirkung an diesem Bedrohungsmodell

Dies ist ein lebendiges Dokument, das von der OpenClaw-Community gepflegt wird. Siehe [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) für Richtlinien zur Mitwirkung:

- Melden neuer Bedrohungen  
- Aktualisieren bestehender Bedrohungen  
- Vorschlagen von Angriffsketten  
- Empfehlen von Gegenmaßnahmen  

---

## 1. Einführung

### 1.1 Zweck

Dieses Bedrohungsmodell dokumentiert adversariale Bedrohungen für die OpenClaw-KI-Agentenplattform und den ClawHub-Skill-Marktplatz unter Verwendung des speziell für KI/ML-Systeme entwickelten MITRE-ATLAS-Frameworks.

### 1.2 Geltungsbereich

| Komponente              | Enthalten | Hinweise                                          |
| ----------------------- | --------- | ------------------------------------------------- |
| OpenClaw Agent Runtime  | Ja        | Kern-Ausführung von Agenten, Tool-Aufrufe, Sitzungen |
| Gateway                 | Ja        | Authentifizierung, Routing, Kanalintegration      |
| Kanalintegrationen      | Ja        | WhatsApp, Telegram, Discord, Signal, Slack usw.  |
| ClawHub Marktplatz      | Ja        | Veröffentlichung, Moderation, Verteilung von Skills |
| MCP-Server              | Ja        | Externe Tool-Anbieter                             |
| Benutzergeräte          | Teilweise | Mobile Apps, Desktop-Clients                      |

### 1.3 Nicht im Geltungsbereich

Für dieses Bedrohungsmodell ist nichts explizit ausgeschlossen.

---

## 2. Systemarchitektur

### 2.1 Vertrauensgrenzen

```
┌─────────────────────────────────────────────────────────────────┐
│                    NICHT VERTRAUENSWÜRDIGE ZONE                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│              VERTRAUENSGRENZE 1: Kanalzugriff                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Geräte-Kopplung (30s Schonfrist)                      │   │
│  │  • AllowFrom / AllowList-Validierung                     │   │
│  │  • Token/Passwort/Tailscale-Authentifizierung            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           VERTRAUENSGRENZE 2: Sitzungsisolierung                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENT-SITZUNGEN                         │   │
│  │  • Sitzungsschlüssel = agent:channel:peer                │   │
│  │  • Tool-Richtlinien pro Agent                            │   │
│  │  • Transkript-Protokollierung                             │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           VERTRAUENSGRENZE 3: Tool-Ausführung                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                AUSFÜHRUNGS-SANDBOX                        │   │
│  │  • Docker-Sandbox ODER Host (exec-approvals)             │   │
│  │  • Remote-Ausführung auf Node                             │   │
│  │  • SSRF-Schutz (DNS-Pinning + IP-Blockierung)            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           VERTRAUENSGRENZE 4: Externe Inhalte                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │          ABGERUFENE URLs / E-MAILS / WEBHOOKS            │   │
│  │  • Einbettung externer Inhalte (XML-Tags)                │   │
│  │  • Einfügung von Sicherheitshinweisen                    │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           VERTRAUENSGRENZE 5: Supply Chain                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Skill-Veröffentlichung (semver, SKILL.md erforderlich)│   │
│  │  • Musterbasierte Moderations-Flags                      │   │
│  │  • VirusTotal-Scanning (demnächst)                       │   │
│  │  • Verifizierung des GitHub-Account-Alters               │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Datenflüsse

| Ablauf | Quelle  | Ziel        | Daten              | Schutzmaßnahmen        |
| ---- | ------- | ----------- | ------------------ | ---------------------- |
| F1   | Channel | Gateway     | Benutzernachrichten | TLS, AllowFrom         |
| F2   | Gateway | Agent       | Weitergeleitete Nachrichten | Sitzungsisolierung |
| F3   | Agent   | Tools       | Tool-Aufrufe       | Durchsetzung von Richtlinien |
| F4   | Agent   | External    | web_fetch-Anfragen | SSRF-Blockierung       |
| F5   | ClawHub | Agent       | Skill-Code         | Moderation, Scanning   |
| F6   | Agent   | Channel     | Antworten          | Ausgabefilterung       |

---

## 3. Bedrohungsanalyse nach ATLAS-Taktik

### 3.1 Aufklärung (AML.TA0002)

#### T-RECON-001: Entdeckung von Agent-Endpunkten

| Attribut                | Wert                                                               |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                        |
| **Beschreibung**        | Angreifer scannt nach exponierten OpenClaw-Gateway-Endpunkten     |
| **Angriffsvektor**      | Netzwerkscans, Shodan-Abfragen, DNS-Enumeration                   |
| **Betroffene Komponenten** | Gateway, exponierte API-Endpunkte                              |
| **Aktuelle Gegenmaßnahmen** | Tailscale-Auth-Option, standardmäßig Bindung an Loopback     |
| **Restrisiko**          | Mittel – Öffentliche Gateways auffindbar                          |
| **Empfehlungen**        | Sichere Bereitstellung dokumentieren, Rate Limiting für Discovery-Endpunkte hinzufügen |

#### T-RECON-002: Sondierung von Kanalintegrationen

| Attribut                | Wert                                                              |
| ----------------------- | ----------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Active Scanning                                       |
| **Beschreibung**        | Angreifer sondiert Messaging-Kanäle zur Identifizierung KI-verwalteter Konten |
| **Angriffsvektor**      | Senden von Testnachrichten, Beobachtung von Antwortmustern       |
| **Betroffene Komponenten** | Alle Kanalintegrationen                                     |
| **Aktuelle Gegenmaßnahmen** | Keine spezifischen                                          |
| **Restrisiko**          | Gering – Begrenzter Nutzen durch reine Entdeckung                |
| **Empfehlungen**        | Randomisierung der Antwortzeiten erwägen                         |

---

_This threat model is a living document. Report security issues to security@openclaw.ai_



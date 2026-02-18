# Modèle de menace OpenClaw v1.0

## Cadre MITRE ATLAS

**Version :** 1.0-draft  
**Dernière mise à jour :** 2026-02-04  
**Méthodologie :** MITRE ATLAS + diagrammes de flux de données  
**Cadre :** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Attribution du cadre

Ce modèle de menace s’appuie sur [MITRE ATLAS](https://atlas.mitre.org/), le cadre de référence du secteur pour documenter les menaces adverses visant les systèmes d’IA/ML. ATLAS est maintenu par [MITRE](https://www.mitre.org/) en collaboration avec la communauté de la sécurité de l’IA.

**Ressources clés ATLAS :**

- [Techniques ATLAS](https://atlas.mitre.org/techniques/)
- [Tactiques ATLAS](https://atlas.mitre.org/tactics/)
- [Études de cas ATLAS](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contribuer à ATLAS](https://atlas.mitre.org/resources/contribute)

### Contribuer à ce modèle de menace

Il s’agit d’un document évolutif maintenu par la communauté OpenClaw. Consultez [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) pour les directives de contribution :

- Signaler de nouvelles menaces
- Mettre à jour des menaces existantes
- Proposer des chaînes d’attaque
- Suggérer des mesures d’atténuation

---

## 1. Introduction

### 1.1 Objectif

Ce modèle de menace documente les menaces adverses pesant sur la plateforme d’agents IA OpenClaw et la marketplace de skills ClawHub, en utilisant le cadre MITRE ATLAS conçu spécifiquement pour les systèmes d’IA/ML.

### 1.2 Périmètre

| Composant              | Inclus | Remarques                                           |
| ---------------------- | ------ | --------------------------------------------------- |
| Runtime d’agent OpenClaw | Oui    | Exécution des agents, appels d’outils, sessions     |
| Gateway                | Oui    | Authentification, routage, intégration des canaux  |
| Intégrations de canaux | Oui    | WhatsApp, Telegram, Discord, Signal, Slack, etc.   |
| Marketplace ClawHub    | Oui    | Publication, modération, distribution des skills   |
| Serveurs MCP           | Oui    | Fournisseurs d’outils externes                     |
| Appareils utilisateurs | Partiel| Applications mobiles, clients desktop               |

### 1.3 Hors périmètre

Rien n’est explicitement exclu du périmètre de ce modèle de menace.

---

## 2. Architecture du système

### 2.1 Frontières de confiance

```
┌─────────────────────────────────────────────────────────────────┐
│                    ZONE NON FIABLE                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│          FRONTIÈRE DE CONFIANCE 1 : Accès aux canaux            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Appairage des appareils (période de grâce de 30 s)    │   │
│  │  • Validation AllowFrom / AllowList                       │   │
│  │  • Authentification Token/Password/Tailscale              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        FRONTIÈRE DE CONFIANCE 2 : Isolation des sessions        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   SESSIONS D’AGENT                       │   │
│  │  • Clé de session = agent:channel:peer                   │   │
│  │  • Politiques d’outils par agent                         │   │
│  │  • Journalisation des transcriptions                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│        FRONTIÈRE DE CONFIANCE 3 : Exécution des outils          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                SANDBOX D’EXÉCUTION                        │   │
│  │  • Sandbox Docker OU Hôte (exec-approvals)               │   │
│  │  • Exécution distante Node                                │   │
│  │  • Protection SSRF (DNS pinning + blocage IP)            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│      FRONTIÈRE DE CONFIANCE 4 : Contenu externe                │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │        URL / EMAILS / WEBHOOKS RÉCUPÉRÉS                 │   │
│  │  • Encapsulation du contenu externe (balises XML)        │   │
│  │  • Injection d’avis de sécurité                           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│      FRONTIÈRE DE CONFIANCE 5 : Chaîne d’approvisionnement     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Publication de skills (semver, SKILL.md requis)       │   │
│  │  • Indicateurs de modération basés sur des motifs        │   │
│  │  • Analyse VirusTotal (bientôt disponible)               │   │
│  │  • Vérification de l’ancienneté du compte GitHub         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Flux de données

| Flux | Source  | Destination | Données              | Protection                |
| ---- | ------- | ----------- | -------------------- | ------------------------- |
| F1   | Canal   | Gateway     | Messages utilisateur | TLS, AllowFrom            |
| F2   | Gateway | Agent       | Messages routés      | Isolation de session      |
| F3   | Agent   | Outils      | Appels d’outils      | Application des politiques|
| F4   | Agent   | Externe     | Requêtes web_fetch   | Blocage SSRF              |
| F5   | ClawHub | Agent       | Code de skill        | Modération, analyse       |
| F6   | Agent   | Canal       | Réponses             | Filtrage en sortie        |

---

## 3. Analyse des menaces par tactique ATLAS

### 3.1 Reconnaissance (AML.TA0002)

#### T-RECON-001 : Découverte des endpoints d’agent

| Attribut                | Valeur                                                               |
| ----------------------- | -------------------------------------------------------------------- |
| **ID ATLAS**            | AML.T0006 - Active Scanning                                          |
| **Description**         | L’attaquant scanne les endpoints Gateway OpenClaw exposés           |
| **Vecteur d’attaque**   | Scan réseau, requêtes shodan, énumération DNS                       |
| **Composants affectés** | Gateway, endpoints API exposés                                       |
| **Mesures actuelles**   | Option d’authentification Tailscale, liaison par défaut en loopback |
| **Risque résiduel**     | Moyen - Gateways publics découvrables                                |
| **Recommandations**     | Documenter le déploiement sécurisé, ajouter un rate limiting sur les endpoints de découverte |

#### T-RECON-002 : Sondage des intégrations de canaux

| Attribut                | Valeur                                                             |
| ----------------------- | ------------------------------------------------------------------ |
| **ID ATLAS**            | AML.T0006 - Active Scanning                                        |
| **Description**         | L’attaquant sonde les canaux de messagerie pour identifier les comptes gérés par une IA |
| **Vecteur d’attaque**   | Envoi de messages de test, observation des schémas de réponse     |
| **Composants affectés** | Toutes les intégrations de canaux                                  |
| **Mesures actuelles**   | Aucune spécifique                                                   |
| **Risque résiduel**     | Faible - Valeur limitée de la simple découverte                    |
| **Recommandations**     | Envisager une randomisation du temps de réponse                    |

---


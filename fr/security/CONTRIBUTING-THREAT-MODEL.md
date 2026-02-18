# Contribuer au modèle de menace d’OpenClaw

Merci d’aider à rendre OpenClaw plus sûr. Ce modèle de menace est un document évolutif et nous accueillons les contributions de tous — il n’est pas nécessaire d’être expert en sécurité.

## Façons de contribuer

### Ajouter une menace

Vous avez repéré un vecteur d’attaque ou un risque que nous n’avons pas couvert ? Ouvrez un ticket sur [openclaw/trust](https://github.com/openclaw/trust/issues) et décrivez-le avec vos propres mots. Vous n’avez pas besoin de connaître des frameworks ni de remplir tous les champs — décrivez simplement le scénario.

**Utile à inclure (mais non obligatoire) :**

- Le scénario d’attaque et la manière dont il pourrait être exploité
- Quelles parties d’OpenClaw sont affectées (CLI, gateway, channels, ClawHub, serveurs MCP, etc.)
- La gravité estimée (faible / moyenne / élevée / critique)
- Tout lien vers des recherches associées, des CVE ou des exemples réels

Nous gérerons la cartographie ATLAS, les identifiants de menace et l’évaluation des risques lors de la revue. Si vous souhaitez inclure ces détails, tant mieux — mais ce n’est pas attendu.

> **Ceci sert à enrichir le modèle de menace, pas à signaler des vulnérabilités actives.** Si vous avez trouvé une vulnérabilité exploitable, consultez notre [page Trust](https://trust.openclaw.ai) pour les instructions de divulgation responsable.

### Suggérer une mesure d’atténuation

Vous avez une idée pour traiter une menace existante ? Ouvrez un ticket ou une PR en faisant référence à la menace. Les mesures d’atténuation utiles sont spécifiques et actionnables — par exemple, « limitation par expéditeur à 10 messages/minute au niveau de la gateway » est préférable à « implémenter une limitation de débit ».

### Proposer une chaîne d’attaque

Les chaînes d’attaque montrent comment plusieurs menaces se combinent en un scénario d’attaque réaliste. Si vous voyez une combinaison dangereuse, décrivez les étapes et comment un attaquant les enchaînerait. Un court récit décrivant le déroulement pratique de l’attaque a plus de valeur qu’un modèle formel.

### Corriger ou améliorer le contenu existant

Fautes de frappe, clarifications, informations obsolètes, meilleurs exemples — les PR sont les bienvenues, sans nécessité d’ouvrir un ticket.

## Ce que nous utilisons

### MITRE ATLAS

Ce modèle de menace s’appuie sur [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), un cadre conçu spécifiquement pour les menaces liées à l’IA/ML telles que l’injection de prompt, l’usage abusif d’outils et l’exploitation d’agents. Vous n’avez pas besoin de connaître ATLAS pour contribuer — nous associons les soumissions au cadre lors de la phase de revue.

### Identifiants de menace

Chaque menace reçoit un identifiant comme `T-EXEC-003`. Les catégories sont :

| Code    | Catégorie                                   |
| ------- | ------------------------------------------ |
| RECON   | Reconnaissance - collecte d’informations   |
| ACCESS  | Accès initial — obtention d’un accès             |
| EXEC    | Exécution — réalisation d’actions malveillantes      |
| PERSIST | Persistance — maintien de l’accès           |
| EVADE   | Évasion des défenses — contournement de la détection       |
| DISC    | Discovery - learning about the environment |
| EXFIL   | Exfiltration - stealing data               |
| IMPACT  | Impact - damage or disruption              |

IDs are assigned by maintainers during review. You don't need to pick one.

### Risk Levels

| Level        | Meaning                                                                   |
| ------------ | ------------------------------------------------------------------------- |
| **Critique** | Compromission complète du système, ou forte probabilité + impact critique |
| **High**     | Significant damage likely, or medium likelihood + critical impact         |
| **Medium**   | Moderate risk, or low likelihood + high impact                            |
| **Low**      | Unlikely and limited impact                                               |

If you're unsure about the risk level, just describe the impact and we'll assess it.

## Review Process

1. **Triage** - We review new submissions within 48 hours
2. **Assessment** - We verify feasibility, assign ATLAS mapping and threat ID, validate risk level
3. **Documentation** - We ensure everything is formatted and complete
4. **Merge** - Added to the threat model and visualization

## Resources

- [ATLAS Website](https://atlas.mitre.org/)
- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [OpenClaw Threat Model](./THREAT-MODEL-ATLAS.md)

## Contact

- **Security vulnerabilities:** See our [Trust page](https://trust.openclaw.ai) for reporting instructions
- **Threat model questions:** Open an issue on [openclaw/trust](https://github.com/openclaw/trust/issues)
- **General chat:** Discord #security channel

## Recognition

Contributors to the threat model are recognized in the threat model acknowledgments, release notes, and the OpenClaw security hall of fame for significant contributions.

---
summary: "Assistant de prise en main CLI : configuration guidee pour la Gateway (passerelle), l’espace de travail, les canaux et les Skills"
read_when:
  - Execution ou configuration de l’assistant de prise en main
  - Mise en place d’une nouvelle machine
title: "Assistant de prise en main (CLI)"
sidebarTitle: "Prise en main : CLI"
---

# Assistant de prise en main (CLI)

L’assistant de prise en main est la methode **recommandee** pour configurer OpenClaw sur macOS,
Linux ou Windows (via WSL2 ; fortement recommande).
Il configure une Gateway (passerelle) locale ou une connexion a une Gateway distante, ainsi que les canaux, les Skills
et les parametres par defaut de l’espace de travail, au sein d’un flux guide unique.

```bash
openclaw onboard
```

<Info>
Premier chat le plus rapide : ouvrez l’UI de controle (aucune configuration de canal requise). Executez
`openclaw dashboard` et discutez dans le navigateur. Docs : [Dashboard](/web/dashboard).
</Info>

Pour reconfigurer ulterieurement :

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` n’implique pas un mode non interactif. Pour les scripts, utilisez `--non-interactive`.
</Note>

<Tip>
Recommande : configurez une cle d’API Brave Search afin que l’agent puisse utiliser `web_search`
(`web_fetch` fonctionne sans cle). La voie la plus simple : `openclaw configure --section web`,
qui stocke `tools.web.search.apiKey`. Docs : [Web tools](/tools/web).
</Tip>

## Démarrage rapide vs Avancé

L’assistant demarre avec **Démarrage rapide** (parametres par defaut) ou **Avancé** (controle total).

<Tabs>
  <Tab title="QuickStart (defaults)">
    - Gateway (passerelle) locale (loopback)
    - Espace de travail par defaut (ou espace de travail existant)
    - Port de la Gateway **18789**
    - Authentification de la Gateway **Token** (genere automatiquement, meme en loopback)
    - Exposition Tailscale **Desactivee**
    - Les Messages prives Telegram + WhatsApp sont par defaut en **allowlist** (vous serez invite a saisir votre numero de telephone)
  
</Tab>
  <Tab title="Advanced (full control)">
    - Expose chaque etape (mode, espace de travail, Gateway, canaux, daemon, Skills).
  
</Tab>
</Tabs>

## Ce que configure l’assistant

Le **mode local (par defaut)** vous guide a travers les etapes suivantes :

1. **Modele/Auth** — Cle d’API Anthropic (recommandee), OAuth, OpenAI ou autres fournisseurs. Choisissez un modele par defaut.
2. **Espace de travail** — Emplacement des fichiers de l’agent (par defaut `~/.openclaw/workspace`). Initialise les fichiers de demarrage.
3. **Gateway** — Port, adresse de liaison, mode d’authentification, exposition Tailscale.
4. **Canaux** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles ou iMessage.
5. **Daemon** — Installe un LaunchAgent (macOS) ou une unite systemd utilisateur (Linux/WSL2).
6. **Verification de sante** — Demarre la Gateway et verifie qu’elle s’execute.
7. **Skills** — Installe les Skills recommandes et les dependances optionnelles.

<Note>
Relancer l’assistant ne **supprime** rien, sauf si vous choisissez explicitement **Reset** (ou passez `--reset`).
Si la configuration est invalide ou contient des cles heritees, l’assistant vous demande d’executer `openclaw doctor` au prealable.
</Note>

Le **mode distant** configure uniquement le client local pour se connecter a une Gateway ailleurs.
Il n’installe ni ne modifie **rien** sur l’hote distant.

## Ajouter un autre agent

Utilisez `openclaw agents add <name>` pour creer un agent distinct avec son propre espace de travail,
ses sessions et ses profils d’authentification. L’execution sans `--workspace` lance l’assistant.

Ce que cela configure :

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

Notes :

- Les espaces de travail par defaut suivent `~/.openclaw/workspace-<agentId>`.
- Ajoutez `bindings` pour router les messages entrants (l’assistant peut le faire).
- Options non interactives : `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Reference complete

Pour des detail pas a pas, le scripting non interactif, la configuration de Signal,
l’API RPC et la liste complete des champs de configuration que l’assistant ecrit, consultez la
[Reference de l’assistant](/reference/wizard).

## Docs associees

- Reference des commandes CLI : [`openclaw onboard`](/cli/onboard)
- Aperçu de la configuration : [Onboarding Overview](/start/onboarding-overview)
- Prise en main de l’app macOS : [Onboarding](/start/onboarding)
- Rituel du premier demarrage de l’agent : [Agent Bootstrapping](/start/bootstrapping)

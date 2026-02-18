---
title: "Amorçage de l’agent"
sidebarTitle: "Amorçage"
---

# Amorçage de l’agent

L’amorçage est le rituel de **première exécution** qui prépare l’espace de travail
de l’agent et collecte les informations d’identité. Il a lieu après la prise en
main, lorsque l’agent démarre pour la première fois.

## Qu'est-ce que l'amorçage fait

Lors de la première exécution de l’agent, OpenClaw amorce l’espace de travail
(par défaut `~/.openclaw/workspace`) :

- Initialise `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`.
- Exécute un court rituel de questions-réponses (une question à la fois).
- Écrit l’identité et les préférences dans `IDENTITY.md`, `USER.md`, `SOUL.md`.
- Supprime `BOOTSTRAP.md` une fois terminé afin que cela ne s’exécute qu’une seule fois.

## Où cela s’exécute

L’amorçage s’exécute toujours sur l’**hôte de la Gateway (passerelle)**. Si
l’application macOS se connecte à une Gateway distante, l’espace de travail et
les fichiers d’amorçage se trouvent sur cette machine distante.

<Note>
Lorsque la Gateway s’exécute sur une autre machine, modifiez les fichiers de
l’espace de travail sur l’hôte de la Gateway (par exemple, `user@gateway-host:~/.openclaw/workspace`).
</Note>

## Documentation associée

- Prise en main de l’application macOS : [Onboarding](/start/onboarding)
- Organisation de l’espace de travail : [Agent workspace](/concepts/agent-workspace)



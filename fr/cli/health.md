---
title: "sante"
---

# `openclaw health`

Recupere l'etat de sante de la Gateway (passerelle) en cours d'execution.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notes :

- `--verbose` execute des sondes en temps reel et affiche des temps par compte lorsque plusieurs comptes sont configures.
- La sortie inclut des magasins de sessions par agent lorsque plusieurs agents sont configures.


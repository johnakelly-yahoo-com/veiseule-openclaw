---
title: "securite"
---

# `openclaw security`

Outils de securite (audit + correctifs optionnels).

Liens connexes :

- Guide de securite : [Security](/gateway/security)

## Audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

L’audit avertit lorsque plusieurs expéditeurs de Messages prives partagent la session principale et recommande le **mode Message prive securise** : `session.dmScope="per-channel-peer"` (ou `per-account-channel-peer` pour les canaux multi-comptes) pour les boites de reception partagees.
Il avertit egalement lorsque de petits modeles (`<=300B`) sont utilises sans sandboxing et avec des outils web/navigateur actives.



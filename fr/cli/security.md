---
summary: "Reference CLI pour `openclaw security` (audit et correction des erreurs de securite courantes)"
read_when:
  - Vous souhaitez executer un audit de securite rapide sur la configuration/l'etat
  - Vous souhaitez appliquer des suggestions de « correctifs » surs (chmod, durcissement des valeurs par defaut)
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
Pour l’ingestion webhook, un avertissement est affiché lorsque `hooks.defaultSessionKey` n’est pas défini, lorsque les remplacements `sessionKey` de requête sont activés, et lorsque ces remplacements sont activés sans `hooks.allowedSessionKeyPrefixes`.
Un avertissement est également affiché lorsque les paramètres Docker du sandbox sont configurés alors que le mode sandbox est désactivé, lorsque `gateway.nodes.denyCommands` utilise des entrées inefficaces de type motif ou inconnues, lorsque le paramètre global `tools.profile="minimal"` est remplacé par des profils d’outils d’agent, et lorsque les outils de plugins d’extension installés peuvent être accessibles sous une politique d’outils permissive.

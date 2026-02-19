---
summary: "Reference CLI pour `openclaw plugins` (liste, installation, activation/desactivation, diagnostic)"
read_when:
  - Vous souhaitez installer ou gerer des plugins Gateway (passerelle) en processus
  - Vous souhaitez depanner des echecs de chargement de plugins
title: "plugins"
---

# `openclaw plugins`

Gerez les plugins/extensions du Gateway (passerelle) (charges en processus).

Liens connexes :

- Systeme de plugins : [Plugins](/tools/plugin)
- Manifeste de plugin + schema : [Plugin manifest](/plugins/manifest)
- Renforcement de la securite : [Security](/gateway/security)

## Commandes

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Les plugins fournis sont livres avec OpenClaw mais demarrent desactives. Utilisez `plugins enable` pour
les activer.

Tous les plugins doivent fournir un fichier `openclaw.plugin.json` avec un Schema JSON en ligne
(`configSchema`, meme s'il est vide). Les manifestes ou schemas manquants/invalides empechent
le chargement du plugin et font echouer la validation de la configuration.

### Installation

```bash
openclaw plugins install <path-or-spec>
```

Note de securite : traitez l'installation de plugins comme l'execution de code. Preferez des versions epinglees.

Les spécifications Npm sont **uniquement issues du registre** (nom du package + version/tag optionnel). Les spécifications Git/URL/file
sont rejetées. Les installations de dépendances s’exécutent avec `--ignore-scripts` pour des raisons de sécurité.

Archives prises en charge : `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Utilisez `--link` pour eviter de copier un repertoire local (ajoute a `plugins.load.paths`) :

```bash
openclaw plugins install -l ./my-plugin
```

### Désinstaller

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` supprime les enregistrements du plugin dans `plugins.entries`, `plugins.installs`,
la liste d’autorisation des plugins, ainsi que les entrées liées de `plugins.load.paths` lorsque applicable.
Pour les plugins mémoire actifs, l’emplacement mémoire est réinitialisé à `memory-core`.

Par défaut, la désinstallation supprime également le répertoire d’installation du plugin sous la racine des extensions du répertoire d’état actif (`$OPENCLAW_STATE_DIR/extensions/<id>`). Utilisez
`--keep-files` pour conserver les fichiers sur le disque.

`--keep-config` est pris en charge comme alias obsolète de `--keep-files`.

### Mise a jour

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Les mises a jour ne s'appliquent qu'aux plugins installes depuis npm (suivis dans `plugins.installs`).

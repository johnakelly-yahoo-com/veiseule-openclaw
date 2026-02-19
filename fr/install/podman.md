---
summary: "Exécuter OpenClaw dans un conteneur Podman rootless"
read_when:
  - Vous souhaitez un gateway conteneurisé avec Podman au lieu de Docker
title: "Podman"
---

# Podman

Exécutez le gateway OpenClaw dans un conteneur Podman **rootless**. Utilise la même image que Docker (construite à partir du [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) du dépôt).

## Prérequis

- Podman (rootless)
- Sudo pour la configuration initiale (création de l’utilisateur, construction de l’image)

## Démarrage rapide

**1. Configuration initiale unique** (depuis la racine du dépôt ; crée l’utilisateur, construit l’image, installe le script de lancement) :

```bash
./setup-podman.sh
```

Cela crée également un `~openclaw/.openclaw/openclaw.json` minimal (définit `gateway.mode="local"`) afin que le gateway puisse démarrer sans exécuter l’assistant.

Par défaut, le conteneur **n’est pas** installé en tant que service systemd ; vous le démarrez manuellement (voir ci-dessous). Pour une configuration de type production avec démarrage automatique et redémarrages, installez-le plutôt comme service utilisateur systemd Quadlet :

```bash
./setup-podman.sh --quadlet
```

(Ou définissez `OPENCLAW_PODMAN_QUADLET=1` ; utilisez `--container` pour installer uniquement le conteneur et le script de lancement.)

**2. Démarrer le gateway** (manuel, pour un test rapide) :

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Assistant d’onboarding** (par exemple pour ajouter des canaux ou des fournisseurs) :

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Ouvrez ensuite `http://127.0.0.1:18789/` et utilisez le token présent dans `~openclaw/.openclaw/.env` (ou la valeur affichée par la configuration).

## Systemd (Quadlet, optionnel)

Si vous avez exécuté `./setup-podman.sh --quadlet` (ou `OPENCLAW_PODMAN_QUADLET=1`), une unité [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) est installée afin que le gateway s’exécute comme service utilisateur systemd pour l’utilisateur openclaw. Le service est activé et démarré à la fin de la configuration.

- **Démarrer :** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Arrêter :** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Statut :** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs :** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Le fichier quadlet se trouve dans `~openclaw/.config/containers/systemd/openclaw.container`. Pour modifier les ports ou les variables d’environnement, éditez ce fichier (ou le `.env` qu’il source), puis exécutez `sudo systemctl --machine openclaw@ --user daemon-reload` et redémarrez le service. Au démarrage, le service se lance automatiquement si le lingering est activé pour openclaw (la configuration le fait lorsque loginctl est disponible).

Pour ajouter quadlet **après** une configuration initiale qui ne l’utilisait pas, relancez : `./setup-podman.sh --quadlet`.

## L’utilisateur openclaw (sans connexion interactive)

`setup-podman.sh` crée un utilisateur système dédié `openclaw` :

- **Shell :** `nologin` — aucune connexion interactive ; réduit la surface d’attaque.

- **Home :** par ex. `/home/openclaw` — contient `~/.openclaw` (config, espace de travail) et le script de lancement `run-openclaw-podman.sh`.

- **Rootless Podman :** L’utilisateur doit disposer d’une plage **subuid** et **subgid**. De nombreuses distributions les attribuent automatiquement lors de la création de l’utilisateur. Si la configuration affiche un avertissement, ajoutez des lignes à `/etc/subuid` et `/etc/subgid` :

  ```text
  openclaw:100000:65536
  ```

  Ensuite, démarrez le gateway avec cet utilisateur (par ex. via cron ou systemd) :

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config :** Seuls `openclaw` et root peuvent accéder à `/home/openclaw/.openclaw`. Pour modifier la configuration : utilisez l’interface de contrôle une fois le gateway en cours d’exécution, ou `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Environnement et configuration

- **Token :** Stocké dans `~openclaw/.openclaw/.env` en tant que `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` et `run-openclaw-podman.sh` le génèrent s’il est absent (utilise `openssl`, `python3` ou `od`).
- **Optionnel :** Dans ce `.env`, vous pouvez définir des clés fournisseur (par ex. `GROQ_API_KEY`, `OLLAMA_API_KEY`) et d’autres variables d’environnement OpenClaw.
- **Ports hôte :** Par défaut, le script mappe `18789` (gateway) et `18790` (bridge). Remplacez le mapping du port **hôte** avec `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` et `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` au lancement.
- **Chemins :** La configuration et l’espace de travail sur l’hôte sont par défaut `~openclaw/.openclaw` et `~openclaw/.openclaw/workspace`. Remplacez les chemins hôte utilisés par le script de lancement avec `OPENCLAW_CONFIG_DIR` et `OPENCLAW_WORKSPACE_DIR`.

## Commandes utiles

- **Logs :** Avec quadlet : `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Avec script : `sudo -u openclaw podman logs -f openclaw`
- **Arrêter :** Avec quadlet : `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Avec script : `sudo -u openclaw podman stop openclaw`
- **Redémarrer :** Avec quadlet : `sudo systemctl --machine openclaw@ --user start openclaw.service`. Avec script : relancez le script de lancement ou `podman start openclaw`
- **Supprimer le conteneur :** `sudo -u openclaw podman rm -f openclaw` — la configuration et l’espace de travail sur l’hôte sont conservés

## Dépannage

- **Permission denied (EACCES) sur config ou auth-profiles :** Le conteneur utilise par défaut `--userns=keep-id` et s’exécute avec le même uid/gid que l’utilisateur hôte qui lance le script. Assurez-vous que vos répertoires hôte `OPENCLAW_CONFIG_DIR` et `OPENCLAW_WORKSPACE_DIR` appartiennent à cet utilisateur.
- **Démarrage du gateway bloqué (`gateway.mode=local` manquant) :** Assurez-vous que `~openclaw/.openclaw/openclaw.json` existe et définit `gateway.mode="local"`. `setup-podman.sh` crée ce fichier s’il est absent.
- **Rootless Podman échoue pour l’utilisateur openclaw :** Vérifiez que `/etc/subuid` et `/etc/subgid` contiennent une ligne pour `openclaw` (par ex. `openclaw:100000:65536`). Ajoutez-la si elle est absente et redémarrez.
- **Nom de conteneur déjà utilisé :** Le script de lancement utilise `podman run --replace`, donc le conteneur existant est remplacé lorsque vous redémarrez. Pour nettoyer manuellement : `podman rm -f openclaw`.
- **Script introuvable lors de l’exécution en tant qu’openclaw :** Assurez-vous que `setup-podman.sh` a été exécuté afin que `run-openclaw-podman.sh` soit copié dans le répertoire personnel d’openclaw (par ex. `/home/openclaw/run-openclaw-podman.sh`).
- **Service Quadlet introuvable ou échec au démarrage :** Exécutez `sudo systemctl --machine openclaw@ --user daemon-reload` après avoir modifié le fichier `.container`. Quadlet nécessite cgroups v2 : `podman info --format '{{.Host.CgroupsVersion}}'` doit afficher `2`.

## Optionnel : exécuter en tant que votre propre utilisateur

Pour exécuter la gateway en tant qu’utilisateur normal (sans utilisateur openclaw dédié) : construisez l’image, créez `~/.openclaw/.env` avec `OPENCLAW_GATEWAY_TOKEN`, puis lancez le conteneur avec `--userns=keep-id` et des montages vers votre `~/.openclaw`. Le script de lancement est conçu pour le flux avec l’utilisateur openclaw ; pour une configuration mono‑utilisateur, vous pouvez à la place exécuter manuellement la commande `podman run` du script, en pointant la configuration et l’espace de travail vers votre répertoire personnel. Recommandé pour la plupart des utilisateurs : utilisez `setup-podman.sh` et exécutez en tant qu’utilisateur openclaw afin d’isoler la configuration et le processus.


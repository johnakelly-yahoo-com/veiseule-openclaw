------

# Liste de controle de publication (npm + macOS)

Utilisez `pnpm` (Node 22+) depuis la racine du depot. Gardez l’arborescence de travail propre avant de taguer/publier.

## Declencheur operateur

Lorsque l’operateur dit « release », effectuez immediatement ce precontrole (pas de questions supplementaires sauf blocage) :

- Lisez ce document et `docs/platforms/mac/release.md`.
- Chargez l’environnement depuis `~/.profile` et confirmez que `SPARKLE_PRIVATE_KEY_FILE` + les variables App Store Connect sont definies (SPARKLE_PRIVATE_KEY_FILE doit se trouver dans `~/.profile`).
- Utilisez les cles Sparkle depuis `~/Library/CloudStorage/Dropbox/Backup/Sparkle` si necessaire.

1. **Version & metadonnees**

- [ ] Incrementer la version de `package.json` (ex. `2026.1.29`).
- [ ] Executer `pnpm plugins:sync` pour aligner les versions des packages d’extensions + les journaux de modifications.
- [ ] Mettre a jour les chaines CLI/version : [`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) et l’agent utilisateur Baileys dans [`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts).
- [ ] Confirmer les metadonnees du package (nom, description, depot, mots-cles, licence) et que le mapping `bin` pointe vers [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) pour `openclaw`.
- [ ] Si les dependances ont change, executer `pnpm install` afin que `pnpm-lock.yaml` soit a jour.

2. **Build & artefacts**

- [ ] Si les entrees A2UI ont change, executer `pnpm canvas:a2ui:bundle` et commiter toute mise a jour de [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js).
- [ ] `pnpm run build` (regenere `dist/`).
- [ ] Verifier que le package npm `files` inclut tous les dossiers `dist/*` requis (notamment `dist/node-host/**` et `dist/acp/**` pour node headless + le CLI ACP).
- [ ] Confirmer que `dist/build-info.json` existe et inclut le hash `commit` attendu (la banniere CLI l’utilise pour les installations npm).
- [ ] Optionnel : `npm pack --pack-destination /tmp` apres le build ; inspectez le contenu du tarball et gardez-le a disposition pour la release GitHub (ne **pas** le commiter).

3. **Changelog & documentation**

- [ ] Mettre a jour `CHANGELOG.md` avec les points saillants cote utilisateur (creer le fichier s’il manque) ; conserver un ordre strictement decroissant par version.
- [ ] S’assurer que les exemples/flags du README correspondent au comportement actuel du CLI (notamment les nouvelles commandes ou options).

4. **Validation**

- [ ] `pnpm build`
- [ ] `pnpm check`
- [ ] `pnpm test` (ou `pnpm test:coverage` si vous avez besoin de la couverture)
- [ ] `pnpm release:check` (verifie le contenu de npm pack)
- [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (test fumee d’installation Docker, voie rapide ; requis avant publication)
  - Si la version npm immediate precedente est connue comme defectueuse, definir `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` ou `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` pour l’etape de preinstall.
- [ ] (Optionnel) Test fumee complet de l’installateur (ajoute couverture non-root + CLI) : `pnpm test:install:smoke`
- [ ] (Optionnel) E2E de l’installateur (Docker, execute `curl -fsSL https://openclaw.ai/install.sh | bash`, prise en main, puis appels d’outils reels) :
  - `pnpm test:install:e2e:openai` (necessite `OPENAI_API_KEY`)
  - `pnpm test:install:e2e:anthropic` (necessite `ANTHROPIC_API_KEY`)
  - `pnpm test:install:e2e` (necessite les deux cles ; execute les deux fournisseurs)
- [ ] (Optionnel) Verifier ponctuellement la Gateway (passerelle) web si vos changements affectent les chemins d’envoi/reception.

5. **Application macOS (Sparkle)**

- [ ] Builder + signer l’application macOS, puis la zipper pour la distribution.
- [ ] Generer l’appcast Sparkle (notes HTML via [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) et mettre a jour `appcast.xml`.
- [ ] Garder le zip de l’app (et le zip dSYM optionnel) pret a etre joint a la release GitHub.
- [ ] Suivre [macOS release](/platforms/mac/release) pour les commandes exactes et les variables d’environnement requises.
  - `APP_BUILD` doit etre numerique et monotone (pas de `-beta`) afin que Sparkle compare correctement les versions.
  - En cas de notarisation, utiliser le profil de trousseau `openclaw-notary` cree a partir des variables d’environnement API App Store Connect (voir [macOS release](/platforms/mac/release)).

6. **Publication (npm)**

- [ ] Confirmer que l’etat git est propre ; commiter et pousser si necessaire.
- [ ] `npm login` (verifier la 2FA) si necessaire.
- [ ] `npm publish --access public` (utiliser `--tag beta` pour les pre-releases).
- [ ] Verifier le registre : `npm view openclaw version`, `npm view openclaw dist-tags`, et `npx -y openclaw@X.Y.Z --version` (ou `--help`).

### Depannage (notes de la release 2.0.0-beta2)

- **npm pack/publish se bloque ou produit un tarball enorme** : le bundle de l’application macOS dans `dist/OpenClaw.app` (et les zips de release) est inclus dans le package. Corriger en listant explicitement le contenu publie via `package.json` `files` (inclure les sous-dossiers dist, docs, skills ; exclure les bundles d’app). Confirmer avec `npm pack --dry-run` que `dist/OpenClaw.app` n’est pas liste.
- **Boucle d’authentification npm web pour les dist-tags** : utiliser l’authentification legacy pour obtenir une invite OTP :
  - `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
- **La verification `npx` echoue avec `ECOMPROMISED: Lock compromised`** : reessayer avec un cache neuf :
  - `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
- **Le tag doit etre repointe apres une correction tardive** : forcer la mise a jour et pousser le tag, puis s’assurer que les artefacts de la release GitHub correspondent toujours :
  - `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **Release GitHub + appcast**

- [ ] Taguer et pousser : `git tag vX.Y.Z && git push origin vX.Y.Z` (ou `git push --tags`).
- [ ] Creer/rafraichir la release GitHub pour `vX.Y.Z` avec **le titre `openclaw X.Y.Z`** (pas seulement le tag) ; le corps doit inclure la section de changelog **complete** pour cette version (Highlights + Changes + Fixes), en ligne (sans liens nus), et **ne doit pas repeter le titre dans le corps**.
- [ ] Joindre les artefacts : tarball `npm pack` (optionnel), `OpenClaw-X.Y.Z.zip`, et `OpenClaw-X.Y.Z.dSYM.zip` (si genere).
- [ ] Commiter le `appcast.xml` mis a jour et le pousser (Sparkle alimente depuis main).
- [ ] Depuis un repertoire temporaire propre (sans `package.json`), executer `npx -y openclaw@X.Y.Z send --help` pour confirmer que l’installation et les points d’entree du CLI fonctionnent.
- [ ] Annoncer/partager les notes de release.

## Perimetre de publication des plugins (npm)

Nous ne publions que les **plugins npm existants** sous le scope `@openclaw/*`. Les
plugins bundles qui ne sont pas sur npm restent **uniquement dans l’arborescence disque**
(toujours livres dans `extensions/**`).

Processus pour etablir la liste :

1. `npm search @openclaw --json` et capturer les noms de packages.
2. Comparer avec les noms `extensions/*/package.json`.
3. Publier uniquement **l’intersection** (deja sur npm).

Liste actuelle des plugins npm (mettre a jour si necessaire) :

- @openclaw/bluebubbles
- @openclaw/diagnostics-otel
- @openclaw/discord
- @openclaw/feishu
- @openclaw/lobster
- @openclaw/matrix
- @openclaw/msteams
- @openclaw/nextcloud-talk
- @openclaw/nostr
- @openclaw/voice-call
- @openclaw/zalo
- @openclaw/zalouser

Les notes de release doivent egalement mentionner les **nouveaux plugins bundles optionnels** qui **ne sont pas actives par defaut** (exemple : `tlon`).



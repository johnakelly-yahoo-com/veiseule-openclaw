---
summary: "Plan : isoler browser act:evaluate de la file Playwright à l’aide de CDP, avec des délais de bout en bout et une résolution de références plus sûre"
owner: "openclaw"
status: "brouillon"
last_updated: "2026-02-10"
title: "Refonte CDP de Browser Evaluate"
---

# Plan de refonte CDP de Browser Evaluate

## Contexte

`act:evaluate` exécute du JavaScript fourni par l’utilisateur dans la page. Aujourd’hui, il s’exécute via Playwright
(`page.evaluate` ou `locator.evaluate`). Playwright sérialise les commandes CDP par page ; ainsi, un
`evaluate` bloqué ou de longue durée peut bloquer la file de commandes de la page et donner l’impression que toute action ultérieure
sur cet onglet est « bloquée ».

La PR #13498 ajoute un filet de sécurité pragmatique (evaluate borné, propagation d’abandon et récupération au mieux). Ce document décrit une refonte plus large qui rend `act:evaluate` intrinsèquement
isolé de Playwright afin qu’un evaluate bloqué ne puisse pas bloquer les opérations Playwright normales.

## Objectifs

- `act:evaluate` ne peut pas bloquer définitivement les actions ultérieures du navigateur sur le même onglet.
- Les timeouts constituent une source unique de vérité de bout en bout afin qu’un appelant puisse se fier à un budget.
- L’abandon et le timeout sont traités de la même manière via HTTP et via l’exécution en processus.
- Le ciblage d’élément pour evaluate est pris en charge sans désactiver entièrement Playwright.
- Maintenir la rétrocompatibilité pour les appelants et payloads existants.

## Non-objectifs

- Remplacer toutes les actions du navigateur (click, type, wait, etc.) par des implémentations CDP.
- Supprimer le filet de sécurité existant introduit dans la PR #13498 (il reste un mécanisme de secours utile).
- Introduire de nouvelles capacités non sûres au-delà du contrôle existant `browser.evaluateEnabled`.
- Ajouter une isolation de processus (processus/thread worker) pour evaluate. Si nous observons encore des
  états bloqués difficiles à récupérer après cette refonte, ce sera une piste à explorer ultérieurement.

## Architecture actuelle (Pourquoi cela se bloque)

À un niveau élevé :

- Les appelants envoient `act:evaluate` au service de contrôle du navigateur.
- Le gestionnaire de route appelle Playwright pour exécuter le JavaScript.
- Playwright sérialise les commandes de page, donc un evaluate qui ne se termine jamais bloque la file d’attente.
- Une file d’attente bloquée signifie que les opérations suivantes (click/type/wait) sur l’onglet peuvent sembler figées.

## Architecture proposée

### 1. Propagation du délai

Introduire un concept unique de budget et tout en dériver :

- L’appelant définit `timeoutMs` (ou une échéance dans le futur).
- Le délai d’expiration de la requête externe, la logique du gestionnaire de route et le budget d’exécution dans la page
  utilisent tous le même budget, avec une légère marge lorsque nécessaire pour compenser la surcharge de sérialisation.
- L’abandon est propagé sous forme d’`AbortSignal` partout afin que l’annulation soit cohérente.

Orientation d’implémentation :

- Ajouter un petit utilitaire (par exemple `createBudget({ timeoutMs, signal })`) qui retourne :
  - `signal` : l’AbortSignal lié
  - `deadlineAtMs` : échéance absolue
  - `remainingMs()` : budget restant pour les opérations enfants
- Utiliser cet utilitaire dans :
  - `src/browser/client-fetch.ts` (HTTP et dispatch en processus)
  - `src/node-host/runner.ts` (chemin proxy)
  - implémentations des actions navigateur (Playwright et CDP)

### 2. Moteur d’Evaluate séparé (chemin CDP)

Ajouter une implémentation d’evaluate basée sur CDP qui ne partage pas la file de commandes par page de Playwright. La propriété clé est que le transport d’evaluate utilise une connexion WebSocket distincte
et une session CDP distincte attachée à la cible.

Orientation d’implémentation :

- Nouveau module, par exemple `src/browser/cdp-evaluate.ts`, qui :
  - Se connecte au endpoint CDP configuré (socket au niveau du navigateur).
  - Utilise `Target.attachToTarget({ targetId, flatten: true })` pour obtenir un `sessionId`.
  - Exécute soit :
    - `Runtime.evaluate` pour un evaluate au niveau de la page, ou
    - `DOM.resolveNode` puis `Runtime.callFunctionOn` pour un evaluate sur un élément.
  - En cas de timeout ou d’abandon :
    - Envoie `Runtime.terminateExecution` en best-effort pour la session.
    - Ferme la WebSocket et renvoie une erreur claire.

Remarques :

- Cela exécute toujours du JavaScript dans la page, donc l’interruption peut avoir des effets secondaires. L’avantage
  est que cela ne bloque pas la file Playwright, et que c’est annulable au niveau du transport
  en fermant la session CDP.

### 3. Histoire des refs (ciblage d’éléments sans réécriture complète)

La partie difficile est le ciblage des éléments. CDP nécessite un handle DOM ou un `backendDOMNodeId`, alors qu’aujourd’hui la plupart des actions navigateur utilisent des locators Playwright basés sur des refs issues de snapshots.

Approche recommandée : conserver les refs existantes, mais y associer un id CDP résolvable optionnel.

#### 3.1 Étendre les informations de ref stockées

Étendre les métadonnées de ref de rôle stockées afin d’inclure éventuellement un id CDP :

- Aujourd’hui : `{ role, name, nth }`
- Proposé : `{ role, name, nth, backendDOMNodeId?: number }`

Cela permet de conserver le fonctionnement de toutes les actions existantes basées sur Playwright et d’autoriser CDP evaluate à accepter
la même valeur de `ref` lorsque le `backendDOMNodeId` est disponible.

#### 3.2 Renseigner backendDOMNodeId au moment du snapshot

Lors de la génération d’un snapshot de rôle :

1. Générer la map de refs de rôle existante comme aujourd’hui (role, name, nth).
2. Récupérer l’arbre AX via CDP (`Accessibility.getFullAXTree`) et calculer une map parallèle de
   `(role, name, nth) -> backendDOMNodeId` en utilisant les mêmes règles de gestion des doublons.
3. Fusionner l’id dans les informations de ref stockées pour l’onglet courant.

Si le mapping échoue pour une ref, laisser `backendDOMNodeId` indéfini. Cela rend la fonctionnalité
en mode best-effort et sûre à déployer.

#### 3.3 Comportement de evaluate avec ref

Dans `act:evaluate` :

- Si `ref` est présent et contient `backendDOMNodeId`, exécuter l’évaluation de l’élément via CDP.
- Si `ref` est présent mais ne contient pas `backendDOMNodeId`, revenir au chemin Playwright (avec
  le filet de sécurité).

Option d’échappement facultative :

- Étendre la structure de requête pour accepter directement `backendDOMNodeId` pour les appelants avancés (et
  pour le débogage), tout en conservant `ref` comme interface principale.

### 4. Conserver un chemin de récupération en dernier recours

Même avec CDP evaluate, il existe d’autres moyens de bloquer un onglet ou une connexion. Conserver les
mécanismes de récupération existants (interruption de l’exécution + déconnexion de Playwright) en dernier recours
pour :

- appelants legacy
- environnements où l’attachement CDP est bloqué
- cas limites inattendus de Playwright

## Plan d’implémentation (itération unique)

### Livrables

- Un moteur evaluate basé sur CDP qui s’exécute en dehors de la file de commandes Playwright par page.
- Un budget unique de timeout/annulation de bout en bout, utilisé de manière cohérente par les appelants et les gestionnaires.
- Des métadonnées de ref pouvant éventuellement inclure `backendDOMNodeId` pour l’évaluation d’éléments.
- `act:evaluate` privilégie le moteur CDP lorsque c’est possible et revient à Playwright dans le cas contraire.
- Des tests prouvant qu’un evaluate bloqué n’empêche pas les actions suivantes.
- Des logs/métriques rendant visibles les échecs et les retours en fallback.

### Checklist d’implémentation

1. Ajouter un helper "budget" partagé pour lier `timeoutMs` + `AbortSignal` amont en :
   - un `AbortSignal` unique
   - une deadline absolue
   - un helper `remainingMs()` pour les opérations en aval
2. Mettre à jour tous les chemins appelants pour utiliser ce helper afin que `timeoutMs` signifie la même chose partout :
   - `src/browser/client-fetch.ts` (HTTP et dispatch en processus interne)
   - `src/node-host/runner.ts` (chemin du proxy node)
   - Wrappers CLI qui appellent `/act` (ajouter `--timeout-ms` à `browser evaluate`)
3. Implémenter `src/browser/cdp-evaluate.ts` :
   - se connecter au socket CDP au niveau du navigateur
   - `Target.attachToTarget` pour obtenir un `sessionId`
   - exécuter `Runtime.evaluate` pour l’évaluation de page
   - exécuter `DOM.resolveNode` + `Runtime.callFunctionOn` pour l’évaluation d’élément
   - en cas de timeout/abort : `Runtime.terminateExecution` au mieux, puis fermer le socket
4. Étendre les métadonnées de référence de rôle stockées pour inclure éventuellement `backendDOMNodeId` :
   - conserver le comportement existant `{ role, name, nth }` pour les actions Playwright
   - ajouter `backendDOMNodeId?: number` pour le ciblage d’éléments CDP
5. Renseigner `backendDOMNodeId` lors de la création du snapshot (au mieux) :
   - récupérer l’arbre AX via CDP (`Accessibility.getFullAXTree`)
   - calculer `(role, name, nth) -> backendDOMNodeId` et fusionner dans la map de références stockée
   - si le mapping est ambigu ou manquant, laisser l’id indéfini
6. Mettre à jour le routage `act:evaluate` :
   - si aucun `ref` : toujours utiliser l’évaluation CDP
   - si `ref` correspond à un `backendDOMNodeId` : utiliser l’évaluation d’élément CDP
   - sinon : revenir à l’évaluation Playwright (toujours limitée et annulable)
7. Conserver le chemin de récupération « dernier recours » existant comme solution de repli, et non comme chemin par défaut.
8. Ajouter des tests :
   - une évaluation bloquée expire dans le budget imparti et le clic/saisie suivant réussit
   - l’abandon annule l’évaluation (déconnexion client ou timeout) et débloque les actions suivantes
   - les échecs de mapping reviennent proprement à Playwright
9. Ajouter de l’observabilité :
   - durée d’évaluation et compteurs de timeout
   - utilisation de terminateExecution
   - taux de fallback (CDP -> Playwright) et raisons

### Critères d’acceptation

- Un `act:evaluate` volontairement bloqué se termine dans le budget de l’appelant et ne bloque pas l’onglet
  pour les actions ultérieures.
- `timeoutMs` se comporte de manière cohérente entre le CLI, l’outil agent, le proxy node et les appels en processus interne.
- Si `ref` peut être mappé à `backendDOMNodeId`, l’évaluation d’élément utilise CDP ; sinon le
  chemin de repli reste limité et récupérable.

## Plan de test

- Tests unitaires :
  - logique de correspondance `(role, name, nth)` entre les références de rôle et les nœuds de l’arbre AX.
  - comportement de l’assistant de budget (marge, calcul du temps restant).
- Tests d’intégration :
  - le timeout d’évaluation CDP se produit dans le budget et ne bloque pas l’action suivante.
  - L’abandon annule l’évaluation et déclenche la tentative de terminaison au mieux.
- Tests de contrat :
  - Assurez-vous que `BrowserActRequest` et `BrowserActResponse` restent compatibles.

## Risques et mesures d’atténuation

- Le mapping est imparfait :
  - Mesure d’atténuation : mapping au mieux, repli sur Playwright evaluate, et ajout d’outils de debug.
- `Runtime.terminateExecution` a des effets secondaires :
  - Mesure d’atténuation : utiliser uniquement en cas de timeout/annulation et documenter le comportement dans les erreurs.
- Surcharge supplémentaire :
  - Mesure d’atténuation : récupérer l’arbre AX uniquement lorsque des snapshots sont demandés, mettre en cache par cible et conserver
    une session CDP de courte durée.
- Limitations du relais d’extension :
  - Mesure d’atténuation : utiliser les API d’attachement au niveau du navigateur lorsque les sockets par page ne sont pas disponibles, et
    conserver le chemin Playwright actuel comme solution de repli.

## Questions ouvertes

- Le nouveau moteur doit-il être configurable en tant que `playwright`, `cdp` ou `auto` ?
- Souhaitons-nous exposer un nouveau format "nodeRef" pour les utilisateurs avancés, ou conserver uniquement `ref` ?
- Comment les snapshots de frame et les snapshots limités par sélecteur doivent-ils participer au mapping AX ?

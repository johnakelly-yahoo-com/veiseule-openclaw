---
summary: "Comment les notes audio/vocales entrantes sont telechargees, transcrites et injectees dans les reponses"
read_when:
  - Modification de la transcription audio ou de la gestion des medias
title: "Audio et notes vocales"
---

# Audio / Notes vocales — 2026-01-17

## Ce qui fonctionne

- **Compréhension des medias (audio)** : Si la compréhension audio est activée (ou auto‑détectée), OpenClaw :
  1. Localise la premiere piece jointe audio (chemin local ou URL) et la telecharge si necessaire.
  2. Applique `maxBytes` avant l’envoi a chaque entree de modele.
  3. Execute la premiere entree de modele eligible dans l’ordre (fournisseur ou CLI).
  4. S'il échoue ou saute (taille/timeout), il essaye l'entrée suivante.
  5. En cas de succes, remplace `Body` par un bloc `[Audio]` et definit `{{Transcript}}`.
- **Analyse des commandes** : Lorsque la transcription reussit, `CommandBody`/`RawBody` sont definis sur la transcription afin que les commandes slash continuent de fonctionner.
- **Journalisation verbeuse** : Dans `--verbose`, nous journalisons quand la transcription s’execute et quand elle remplace le corps.

## Auto‑detection (par defaut)

Si vous **ne configurez pas de modeles** et que `tools.media.audio.enabled` n’est **pas** defini sur `false`,
OpenClaw auto‑detecte dans l’ordre suivant et s’arrete a la premiere option fonctionnelle :

1. **CLIs locales** (si installees)
   - `sherpa-onnx-offline` (necessite `SHERPA_ONNX_MODEL_DIR` avec encodeur/decodeur/assembleur/tokens)
   - `whisper-cli` (depuis `whisper-cpp` ; utilise `WHISPER_CPP_MODEL` ou le modele tiny integre)
   - `whisper` (CLI Python ; telecharge automatiquement les modeles)
2. **Gemini CLI** (`gemini`) utilisant `read_many_files`
3. **Cles de fournisseur** (OpenAI → Groq → Deepgram → Google)

Pour desactiver l’auto‑detection, definissez `tools.media.audio.enabled: false`.
Pour personnaliser, definissez `tools.media.audio.models`.
Remarque : La detection des binaires est au mieux de ses capacites sur macOS/Linux/Windows ; assurez‑vous que la CLI est sur `PATH` (nous developpons `~`), ou definissez un modele CLI explicite avec un chemin de commande complet.

## Exemples de configuration

### Fournisseur + secours CLI (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### Fournisseur seulement avec la portée de la portée

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### Fournisseur uniquement (Deepgram)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Notes et limites

- L’authentification des fournisseurs suit l’ordre standard d’authentification des modeles (profils d’authentification, variables d’environnement, `models.providers.*.apiKey`).
- Deepgram recupere `DEEPGRAM_API_KEY` lorsque `provider: "deepgram"` est utilise.
- Details de configuration Deepgram : [Deepgram (transcription audio)](/providers/deepgram).
- Les fournisseurs audio peuvent surcharger `baseUrl`, `headers` et `providerOptions` via `tools.media.audio`.
- La limite de taille par defaut est de 20 Mo (`tools.media.audio.maxBytes`). Les audios depassant cette taille sont ignores pour ce modele et l’entree suivante est essayee.
- La valeur `maxChars` par defaut pour l’audio est **non definie** (transcription complete). Definissez `tools.media.audio.maxChars` ou `maxChars` par entree pour reduire la sortie.
- La valeur par defaut automatique d’OpenAI est `gpt-4o-mini-transcribe` ; definissez `model: "gpt-4o-transcribe"` pour une meilleure precision.
- Utilisez `tools.media.audio.attachments` pour traiter plusieurs notes vocales (`mode: "all"` + `maxAttachments`).
- La transcription est disponible pour les modeles de templates sous `{{Transcript}}`.
- La sortie stdout de la CLI est plafonnee (5 Mo) ; gardez la sortie de la CLI concise.

## Détection de mention dans les groupes

Lorsque `requireMention: true` est défini pour un chat de groupe, OpenClaw transcrit désormais l’audio **avant** de vérifier les mentions. Cela permet de traiter les messages vocaux même lorsqu’ils contiennent des mentions.

**Comment cela fonctionne :**

1. Si un message vocal n’a pas de corps de texte et que le groupe exige des mentions, OpenClaw effectue une transcription « preflight ».
2. La transcription est analysée pour détecter des motifs de mention (par ex., `@BotName`, déclencheurs emoji).
3. Si une mention est trouvée, le message passe par le pipeline complet de réponse.
4. La transcription est utilisée pour la détection des mentions afin que les messages vocaux puissent franchir le filtre de mention.

**Comportement de secours :**

- Si la transcription échoue pendant le preflight (délai dépassé, erreur API, etc.), le message est traité sur la base de la détection de mention textuelle uniquement.
- Cela garantit que les messages mixtes (texte + audio) ne sont jamais ignorés par erreur.

**Exemple :** Un utilisateur envoie un message vocal disant « Hey @Claude, what’s the weather? » dans un groupe Telegram avec `requireMention: true`. Le message vocal est transcrit, la mention est détectée et l’agent répond.

## Gotchas

- Les regles de portee utilisent le principe du premier correspondant gagnant. `chatType` est normalise en `direct`, `group` ou `room`.
- Assurez‑vous que votre CLI se termine avec le code 0 et affiche du texte brut ; le JSON doit etre adapte via `jq -r .text`.
- Gardez des delais d’attente raisonnables (`timeoutSeconds`, par defaut 60 s) afin d’eviter de bloquer la file de reponses.
- La transcription preflight ne traite que la **première** pièce jointe audio pour la détection des mentions. Les audios supplémentaires sont traités lors de la phase principale de compréhension des médias.

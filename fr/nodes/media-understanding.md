---
title: "Compréhension des médias"
---

# Compréhension des médias (entrant) — 2026-01-17

OpenClaw peut **résumer les médias entrants** (image/audio/vidéo) avant l’exécution du pipeline de réponse. Il détecte automatiquement la disponibilité d’outils locaux ou de clés fournisseur, et peut être désactivé ou personnalisé. Si la compréhension est désactivée, les modèles reçoivent toujours les fichiers/URL d’origine comme d’habitude.

## Objectifs

- Optionnel : pré‑digérer les médias entrants en texte court pour un routage plus rapide et une meilleure analyse des commandes.
- Préserver la livraison des médias d’origine au modèle (toujours).
- Prendre en charge les **API de fournisseurs** et les **solutions de repli CLI**.
- Autoriser plusieurs modèles avec une solution de repli ordonnée (erreur/taille/délai).

## Comportement de haut niveau

1. Collecter les pièces jointes entrantes (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Pour chaque capacité activée (image/audio/vidéo), sélectionner les pièces jointes selon la politique (par défaut : **première**).
3. Choisir la première entrée de modèle éligible (taille + capacité + authentification).
4. Si un modèle échoue ou si le média est trop volumineux, **basculer vers l’entrée suivante**.
5. En cas de succès :
   - `Body` devient un bloc `[Image]`, `[Audio]` ou `[Video]`.
   - L’audio définit `{{Transcript}}` ; l’analyse des commandes utilise le texte de légende lorsqu’il est présent,
     sinon la transcription.
   - Les légendes sont conservées en tant que `User text:` à l’intérieur du bloc.

Si la compréhension échoue ou est désactivée, **le flux de réponse se poursuit** avec le corps d’origine + les pièces jointes.

## Aperçu de la configuration

`tools.media` prend en charge des **modèles partagés** ainsi que des remplacements par capacité :

- `tools.media.models` : liste de modèles partagés (utilisez `capabilities` pour le contrôle).
- `tools.media.image` / `tools.media.audio` / `tools.media.video` :
  - valeurs par défaut (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - remplacements de fournisseur (`baseUrl`, `headers`, `providerOptions`)
  - options audio Deepgram via `tools.media.audio.providerOptions.deepgram`
  - **liste `models` par capacité** optionnelle (prioritaire avant les modèles partagés)
  - politique `attachments` (`mode`, `maxAttachments`, `prefer`)
  - `scope` (contrôle optionnel par canal/chatType/clé de session)
- `tools.media.concurrency` : nombre maximal d’exécutions simultanées par capacité (par défaut **2**).

```json5
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### Entrées de modèle

Chaque entrée `models[]` peut être **fournisseur** ou **CLI** :

```json5
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

Les modèles CLI peuvent également utiliser :

- `{{MediaDir}}` (répertoire contenant le fichier média)
- `{{OutputDir}}` (répertoire de travail créé pour cette exécution)
- `{{OutputBase}}` (chemin de base du fichier de travail, sans extension)

## Valeurs par défaut et limites

Valeurs par défaut recommandées :

- `maxChars` : **500** pour image/vidéo (court, adapté aux commandes)
- `maxChars` : **non défini** pour l’audio (transcription complète sauf si vous fixez une limite)
- `maxBytes` :
  - image : **10MB**
  - audio : **20MB**
  - vidéo : **50MB**

Règles :

- Si le média dépasse `maxBytes`, ce modèle est ignoré et **le modèle suivant est essayé**.
- Si le modèle renvoie plus de `maxChars`, la sortie est tronquée.
- `prompt` utilise par défaut une simple instruction « Décrire le {media}. » plus les consignes `maxChars` (image/vidéo uniquement).
- Si `<capability>.enabled: true` mais qu’aucun modèle n’est configuré, OpenClaw essaie le
  **modèle de réponse actif** lorsque son fournisseur prend en charge la capacité.

### Détection automatique de la compréhension des médias (par défaut)

Si `tools.media.<capability>.enabled` n’est **pas** défini sur `false` et que vous n’avez pas
configuré de modèles, OpenClaw détecte automatiquement dans cet ordre et **s’arrête à la première option fonctionnelle** :

1. **CLIs locales** (audio uniquement ; si installées)
   - `sherpa-onnx-offline` (nécessite `SHERPA_ONNX_MODEL_DIR` avec encodeur/décodeur/assembleur/tokens)
   - `whisper-cli` (`whisper-cpp` ; utilise `WHISPER_CPP_MODEL` ou le petit modèle intégré)
   - `whisper` (CLI Python ; télécharge automatiquement les modèles)
2. **Gemini CLI** (`gemini`) utilisant `read_many_files`
3. **Clés fournisseur**
   - Audio : OpenAI → Groq → Deepgram → Google
   - Image : OpenAI → Anthropic → Google → MiniMax
   - Vidéo : Google

Pour désactiver la détection automatique, définissez :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

Remarque : la détection des binaires est au mieux‑effort sur macOS/Linux/Windows ; assurez‑vous que la CLI est sur `PATH` (nous étendons `~`), ou définissez un modèle CLI explicite avec un chemin de commande complet.

## Capacités (optionnel)

Si vous définissez `capabilities`, l’entrée ne s’exécute que pour ces types de médias. Pour les
listes partagées, OpenClaw peut déduire les valeurs par défaut :

- `openai`, `anthropic`, `minimax` : **image**
- `google` (API Gemini) : **image + audio + vidéo**
- `groq` : **audio**
- `deepgram` : **audio**

Pour les entrées CLI, **définissez `capabilities` explicitement** afin d’éviter des correspondances surprenantes.
Si vous omettez `capabilities`, l’entrée est éligible pour la liste dans laquelle elle apparaît.

## Matrice de prise en charge des fournisseurs (intégrations OpenClaw)

| Capacité | Intégration du fournisseur                       | Notes                                                                                   |
| -------- | ------------------------------------------------ | --------------------------------------------------------------------------------------- |
| Image    | OpenAI / Anthropic / Google / autres via `pi-ai` | Tout modèle capable d’images dans le registre fonctionne.               |
| Audio    | OpenAI, Groq, Deepgram, Google                   | Transcription fournisseur (Whisper/Deepgram/Gemini). |
| Vidéo    | Google (API Gemini)           | Compréhension vidéo du fournisseur.                                     |

## Fournisseurs recommandés

**Image**

- Préférez votre modèle actif s’il prend en charge les images.
- Bons choix par défaut : `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Audio**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` ou `deepgram/nova-3`.
- Solution de repli CLI : `whisper-cli` (whisper-cpp) ou `whisper`.
- Configuration Deepgram : [Deepgram (transcription audio)](/providers/deepgram).

**Vidéo**

- `google/gemini-3-flash-preview` (rapide), `google/gemini-3-pro-preview` (plus riche).
- Solution de repli CLI : CLI `gemini` (prend en charge `read_file` pour la vidéo/audio).

## Politique des pièces jointes

La `attachments` par capacité contrôle quelles pièces jointes sont traitées :

- `mode` : `first` (par défaut) ou `all`
- `maxAttachments` : limite le nombre traité (par défaut **1**)
- `prefer` : `first`, `last`, `path`, `url`

Lorsque `mode: "all"`, les sorties sont étiquetées `[Image 1/2]`, `[Audio 2/2]`, etc.

## Exemples de configuration

### 1. Liste de modèles partagés + remplacements

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2. Audio + Vidéo uniquement (image désactivée)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3. Compréhension d’image optionnelle

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4. Entrée unique multimodale (capacités explicites)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## Sortie de statut

Lorsque la compréhension des médias s’exécute, `/status` inclut une courte ligne de résumé :

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Cela affiche les résultats par capacité et le fournisseur/modèle choisi le cas échéant.

## Notes

- La compréhension est **au mieux‑effort**. Les erreurs ne bloquent pas les réponses.
- Les pièces jointes sont toujours transmises aux modèles même lorsque la compréhension est désactivée.
- Utilisez `scope` pour limiter où la compréhension s'exécute (par exemple uniquement les DM).

## Documentation associée

- [Configuration](/gateway/configuration)
- [Prise en charge des images et des médias](/nodes/images)

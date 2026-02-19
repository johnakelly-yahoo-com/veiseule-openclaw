---
summary: "Configuration de Hugging Face Inference (authentification + sélection du modèle)"
read_when:
  - Vous souhaitez utiliser Hugging Face Inference avec OpenClaw
  - Vous avez besoin de la variable d’environnement du token HF ou du choix d’authentification CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) proposent des complétions de chat compatibles OpenAI via une API routeur unique. Vous avez accès à de nombreux modèles (DeepSeek, Llama, et plus encore) avec un seul token. OpenClaw utilise le **endpoint compatible OpenAI** (complétions de chat uniquement) ; pour la génération texte-vers-image, les embeddings ou la parole, utilisez directement les [clients d’inférence HF](https://huggingface.co/docs/api-inference/quicktour).

- Fournisseur : `huggingface`
- Authentification : `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN` (token à granularité fine avec l’autorisation **Make calls to Inference Providers**)
- API : Compatible OpenAI (`https://router.huggingface.co/v1`)
- Facturation : Token HF unique ; la [tarification](https://huggingface.co/docs/inference-providers/pricing) suit les tarifs des fournisseurs avec un niveau gratuit.

## Démarrage rapide

1. Créez un token à granularité fine sur [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) avec l’autorisation **Make calls to Inference Providers**.
2. Lancez l’onboarding et choisissez **Hugging Face** dans la liste déroulante des fournisseurs, puis saisissez votre clé API lorsque cela vous est demandé :

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. Dans la liste déroulante **Default Hugging Face model**, sélectionnez le modèle souhaité (la liste est chargée depuis l’API Inference lorsque vous disposez d’un token valide ; sinon une liste intégrée est affichée). Votre choix est enregistré comme modèle par défaut.
4. Vous pouvez également définir ou modifier le modèle par défaut plus tard dans la configuration :

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Exemple non interactif

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Cela définira `huggingface/deepseek-ai/DeepSeek-R1` comme modèle par défaut.

## Remarque sur l’environnement

Si le Gateway s’exécute en tant que démon (launchd/systemd), assurez-vous que `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN`
est disponible pour ce processus (par exemple, dans `~/.openclaw/.env` ou via
`env.shellEnv`).

## Découverte des modèles et menu déroulant d’onboarding

OpenClaw découvre les modèles en appelant **directement le endpoint d’Inference** :

```bash
GET https://router.huggingface.co/v1/models
```

(Optionnel : envoyez `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` ou `$HF_TOKEN` pour obtenir la liste complète ; certains endpoints renvoient un sous-ensemble sans authentification.) La réponse est au format OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Lorsque vous configurez une clé API Hugging Face (via l’onboarding, `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN`), OpenClaw utilise ce GET pour découvrir les modèles de chat-completion disponibles. Pendant l’**onboarding interactif**, après avoir saisi votre token, vous voyez un menu déroulant **Default Hugging Face model** alimenté à partir de cette liste (ou du catalogue intégré si la requête échoue). À l’exécution (par ex. au démarrage du Gateway), lorsqu’une clé est présente, OpenClaw appelle à nouveau **GET** `https://router.huggingface.co/v1/models` pour actualiser le catalogue. La liste est fusionnée avec un catalogue intégré (pour les métadonnées telles que la fenêtre de contexte et le coût). Si la requête échoue ou si aucune clé n’est définie, seul le catalogue intégré est utilisé.

## Noms des modèles et options modifiables

- **Nom depuis l’API :** Le nom d’affichage du modèle est **récupéré depuis GET /v1/models** lorsque l’API renvoie `name`, `title` ou `display_name` ; sinon, il est dérivé de l’identifiant du modèle (par ex. `deepseek-ai/DeepSeek-R1` → « DeepSeek R1 »).
- **Remplacer le nom d’affichage :** Vous pouvez définir un libellé personnalisé par modèle dans la configuration afin qu’il apparaisse comme vous le souhaitez dans le CLI et l’UI :

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Sélection du fournisseur / de la politique :** Ajoutez un suffixe à l’**identifiant du modèle** pour choisir comment le routeur sélectionne le backend :

  - **`:fastest`** — débit le plus élevé (sélectionné par le routeur ; le choix du fournisseur est **verrouillé** — aucun sélecteur interactif de backend).
  - **`:cheapest`** — coût le plus bas par token de sortie (sélectionné par le routeur ; le choix du fournisseur est **verrouillé**).
  - **`:provider`** — forcer un backend spécifique (par ex. `:sambanova`, `:together`).

  Lorsque vous sélectionnez **:cheapest** ou **:fastest** (par ex. dans le menu déroulant de modèle lors de l’onboarding), le fournisseur est verrouillé : le routeur décide en fonction du coût ou de la vitesse et aucune étape optionnelle « préférer un backend spécifique » n’est affichée. Vous pouvez les ajouter comme entrées distinctes dans `models.providers.huggingface.models` ou définir `model.primary` avec le suffixe. Vous pouvez également définir votre ordre par défaut dans les [paramètres Inference Provider](https://hf.co/settings/inference-providers) (sans suffixe = utiliser cet ordre).

- **Fusion de configuration :** Les entrées existantes dans `models.providers.huggingface.models` (par ex. dans `models.json`) sont conservées lors de la fusion de la configuration. Ainsi, tout `name`, `alias` ou option de modèle personnalisé que vous y définissez est préservé.

## Identifiants de modèles et exemples de configuration

Les références de modèle utilisent le format `huggingface/<org>/<model>` (identifiants de type Hub). La liste ci-dessous provient de **GET** `https://router.huggingface.co/v1/models` ; votre catalogue peut en inclure davantage.

**Exemples d’identifiants (depuis le endpoint d’inference) :**

| Modèle                                 | Réf (préfixer avec `huggingface/`) |
| -------------------------------------- | ----------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                             |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                           |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                       |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                            |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                      |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                   |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                    |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                 |
| GLM 4.7                | `zai-org/GLM-4.7`                                     |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                |

Vous pouvez ajouter `:fastest`, `:cheapest` ou `:provider` (par ex. `:together`, `:sambanova`) à l’identifiant du modèle. Définissez votre ordre par défaut dans les [paramètres des fournisseurs d’inférence](https://hf.co/settings/inference-providers) ; consultez [Inference Providers](https://huggingface.co/docs/inference-providers) et **GET** `https://router.huggingface.co/v1/models` pour la liste complète.

### Exemples complets de configuration

**DeepSeek R1 principal avec Qwen en secours :**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen par défaut, avec variantes :cheapest et :fastest :**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS avec alias :**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Forcer un backend spécifique avec :provider :**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Plusieurs modèles Qwen et DeepSeek avec suffixes de politique :**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```


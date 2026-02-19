---
summary: "Exécutez OpenClaw via le proxy LiteLLM pour un accès unifié aux modèles et un suivi des coûts"
read_when:
  - Vous souhaitez faire passer OpenClaw par un proxy LiteLLM
  - Vous avez besoin d’un suivi des coûts, de journaux ou d’un routage des modèles via LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) est une passerelle LLM open source qui fournit une API unifiée vers plus de 100 fournisseurs de modèles. Faites passer OpenClaw par LiteLLM pour obtenir un suivi centralisé des coûts, des journaux, et la possibilité de changer de backend sans modifier votre configuration OpenClaw.

## Pourquoi utiliser LiteLLM avec OpenClaw ?

- **Suivi des coûts** — Visualisez exactement ce que OpenClaw dépense sur l’ensemble des modèles
- **Routage des modèles** — Passez de Claude à GPT-4, Gemini, Bedrock sans modifier la configuration
- **Clés virtuelles** — Créez des clés avec des limites de dépenses pour OpenClaw
- **Journalisation** — Journaux complets des requêtes/réponses pour le débogage
- **Basculement automatique** — Basculement automatique si votre fournisseur principal est indisponible

## Démarrage rapide

### Via l’onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Configuration manuelle

1. Démarrez le proxy LiteLLM :

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Configurez OpenClaw pour utiliser LiteLLM :

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

C’est tout. OpenClaw route désormais via LiteLLM.

## Configuration

### Variables d’environnement

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Fichier de configuration

```json5
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Clés virtuelles

Créez une clé dédiée pour OpenClaw avec des limites de dépenses :

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

Utilisez la clé générée comme `LITELLM_API_KEY`.

## Routage des modèles

LiteLLM peut router les requêtes de modèles vers différents backends. Configurez dans votre `config.yaml` LiteLLM :

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw continue de demander `claude-opus-4-6` — LiteLLM gère le routage.

## Consulter l’utilisation

Consultez le tableau de bord ou l’API de LiteLLM :

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Notes

- LiteLLM s’exécute par défaut sur `http://localhost:4000`
- OpenClaw se connecte via l’endpoint compatible OpenAI `/v1/chat/completions`
- Toutes les fonctionnalités de OpenClaw fonctionnent via LiteLLM — aucune limitation

## Voir aussi

- [LiteLLM Docs](https://docs.litellm.ai)
- [Fournisseurs de modèles](/concepts/model-providers)

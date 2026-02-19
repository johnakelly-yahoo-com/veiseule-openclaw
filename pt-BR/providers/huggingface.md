---
summary: "Configuração do Hugging Face Inference (auth + seleção de modelo)"
read_when:
  - Você quer usar o Hugging Face Inference com OpenClaw
  - Você precisa da variável de ambiente do token HF ou escolher a autenticação via CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) oferecem chat completions compatíveis com OpenAI por meio de uma única API de roteador. Você obtém acesso a vários modelos (DeepSeek, Llama e outros) com um único token. O OpenClaw usa o **endpoint compatível com OpenAI** (apenas chat completions); para text-to-image, embeddings ou fala, use os [clientes de inferência do HF](https://huggingface.co/docs/api-inference/quicktour) diretamente.

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN` (token granular com **Make calls to Inference Providers**)
- API: compatível com OpenAI (`https://router.huggingface.co/v1`)
- Billing: Token HF único; [pricing](https://huggingface.co/docs/inference-providers/pricing) segue as tarifas do provedor com um nível gratuito.

## Início rápido

1. Crie um token granular em [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) com a permissão **Make calls to Inference Providers**.
2. Execute o onboarding e escolha **Hugging Face** no menu suspenso de provedor, depois insira sua chave de API quando solicitado:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. No menu suspenso **Default Hugging Face model**, selecione o modelo desejado (a lista é carregada da API de Inference quando você tem um token válido; caso contrário, uma lista interna é exibida). Sua escolha é salva como o modelo padrão.
4. Você também pode definir ou alterar o modelo padrão posteriormente na configuração:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Exemplo não interativo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Isso definirá `huggingface/deepseek-ai/DeepSeek-R1` como o modelo padrão.

## Observação sobre o ambiente

Se o Gateway for executado como um daemon (launchd/systemd), certifique-se de que `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN`
está disponível para esse processo (por exemplo, em `~/.openclaw/.env` ou via
`env.shellEnv`).

## Descoberta de modelos e menu suspenso de onboarding

O OpenClaw descobre modelos chamando o **endpoint de Inference diretamente**:

```bash
GET https://router.huggingface.co/v1/models
```

(Opcional: envie `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` ou `$HF_TOKEN` para obter a lista completa; alguns endpoints retornam um subconjunto sem autenticação.) A resposta segue o padrão OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Quando você configura uma chave de API do Hugging Face (via onboarding, `HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN`), o OpenClaw usa esse GET para descobrir os modelos de chat-completion disponíveis. Durante o **onboarding interativo**, após inserir seu token, você verá um menu suspenso de **Modelo padrão do Hugging Face** preenchido com essa lista (ou com o catálogo interno, caso a requisição falhe). Em tempo de execução (por exemplo, na inicialização do Gateway), quando uma chave está presente, o OpenClaw chama novamente **GET** `https://router.huggingface.co/v1/models` para atualizar o catálogo. A lista é mesclada com um catálogo interno (para metadados como janela de contexto e custo). Se a requisição falhar ou nenhuma chave estiver configurada, apenas o catálogo interno será usado.

## Nomes de modelos e opções editáveis

- **Nome da API:** O nome de exibição do modelo é **preenchido a partir de GET /v1/models** quando a API retorna `name`, `title` ou `display_name`; caso contrário, é derivado do id do modelo (por exemplo, `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Substituir nome de exibição:** Você pode definir um rótulo personalizado por modelo na configuração para que ele apareça como desejar no CLI e na UI:

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

- **Seleção de provedor / política:** Adicione um sufixo ao **id do modelo** para escolher como o roteador seleciona o backend:

  - **`:fastest`** — maior taxa de processamento (o roteador escolhe; a escolha do provedor é **bloqueada** — sem seletor interativo de backend).
  - **`:cheapest`** — menor custo por token de saída (o roteador escolhe; a escolha do provedor é **bloqueada**).
  - **`:provider`** — força um backend específico (por exemplo, `:sambanova`, `:together`).

  Ao selecionar **:cheapest** ou **:fastest** (por exemplo, no menu suspenso de modelo durante o onboarding), o provedor fica bloqueado: o roteador decide com base em custo ou velocidade e nenhuma etapa opcional de “preferir backend específico” é exibida. Você pode adicioná-los como entradas separadas em `models.providers.huggingface.models` ou definir `model.primary` com o sufixo. Você também pode definir sua ordem padrão em [Configurações do Provedor de Inferência](https://hf.co/settings/inference-providers) (sem sufixo = usar essa ordem).

- **Mesclagem de configuração:** Entradas existentes em `models.providers.huggingface.models` (por exemplo, em `models.json`) são mantidas quando a configuração é mesclada. Portanto, qualquer `name`, `alias` ou opção de modelo personalizada definida ali é preservada.

## IDs de modelos e exemplos de configuração

As referências de modelo usam o formato `huggingface/<org>/<model>` (IDs no estilo do Hub). A lista abaixo é de **GET** `https://router.huggingface.co/v1/models`; seu catálogo pode incluir mais.

**IDs de exemplo (do endpoint de inferência):**

| Modelo                                 | Ref (prefixe com `huggingface/`) |
| -------------------------------------- | --------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                           |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                         |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                     |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                          |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                    |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                 |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                  |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                               |
| GLM 4.7                | `zai-org/GLM-4.7`                                   |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                              |

Você pode adicionar `:fastest`, `:cheapest` ou `:provider` (por exemplo, `:together`, `:sambanova`) ao ID do modelo. Defina sua ordem padrão em [Configurações do Provedor de Inferência](https://hf.co/settings/inference-providers); consulte [Provedores de Inferência](https://huggingface.co/docs/inference-providers) e **GET** `https://router.huggingface.co/v1/models` para a lista completa.

### Exemplos completos de configuração

**DeepSeek R1 principal com fallback para Qwen:**

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

**Qwen como padrão, com variantes :cheapest e :fastest:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (mais barato)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (mais rápido)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS com aliases:**

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

**Forçar um backend específico com :provider:**

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

**Múltiplos modelos Qwen e DeepSeek com sufixos de política:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (barato)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (rápido)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```


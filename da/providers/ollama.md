---
summary: "Kør OpenClaw med Ollama (lokal LLM-runtime)"
read_when:
  - Du vil køre OpenClaw med lokale modeller via Ollama
  - Du har brug for vejledning til opsætning og konfiguration af Ollama
title: "Ollama"
---

# Ollama

Ollama er en lokal LLM runtime, der gør det nemt at køre open-source modeller på din maskine. OpenClaw integrerer med Ollamas native API (`/api/chat`), understøtter streaming og tool calling og kan **automatisk finde modeller med tool-understøttelse**, når du tilvælger det med `OLLAMA_API_KEY` (eller en auth-profil) og ikke definerer en eksplicit `models.providers.ollama`-post.

## Hurtig start

1. Installér Ollama: [https://ollama.ai](https://ollama.ai)

2. Hent en model:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. Aktivér Ollama for OpenClaw (enhver værdi virker; Ollama kræver ikke en rigtig nøgle):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Brug Ollama-modeller:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Model discovery (implicit udbyder)

Når du sætter `OLLAMA_API_KEY` (eller en auth-profil) og **ikke** definerer `models.providers.ollama`, finder OpenClaw modeller fra den lokale Ollama-instans på `http://127.0.0.1:11434`:

- Forespørger `/api/tags` og `/api/show`
- Beholder kun modeller, der rapporterer `tools`-kapabilitet
- Marker­er `reasoning`, når modellen rapporterer `thinking`
- Læser `contextWindow` fra `model_info["<arch>.context_length"]`, når det er tilgængeligt
- Sætter `maxTokens` til 10× kontekstvinduet
- Sætter alle omkostninger til `0`

Dette undgår manuelle modelposter, samtidig med at kataloget holdes på linje med Ollamas kapabiliteter.

For at se hvilke modeller der er tilgængelige:

```bash
ollama list
openclaw models list
```

For at tilføje en ny model skal du blot hente den med Ollama:

```bash
ollama pull mistral
```

Den nye model vil automatisk blive fundet og være klar til brug.

Hvis du sætter `models.providers.ollama` eksplicit, springes auto-discovery over, og du skal definere modeller manuelt (se nedenfor).

## Konfiguration

### Grundlæggende opsætning (implicit discovery)

Den enkleste måde at aktivere Ollama på er via en miljøvariabel:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Eksplicit opsætning (manuelle modeller)

Brug eksplicit konfiguration når:

- Ollama kører på en anden vært/port.
- Du vil gennemtvinge specifikke kontekstvinduer eller modellister.
- Du vil inkludere modeller, der ikke rapporterer værktøjsunderstøttelse.

```json5
{
  models: {
    providers: {
      ollama: {
        // Use a host that includes /v1 for OpenAI-compatible APIs
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Hvis `OLLAMA_API_KEY` er sat, kan du udelade `apiKey` i udbyderposten, og OpenClaw vil udfylde den til tilgængelighedstjek.

### Brugerdefineret base-URL (eksplicit konfiguration)

Hvis Ollama kører på en anden vært eller port (eksplicit konfiguration deaktiverer auto-discovery, så definér modeller manuelt):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1",
      },
    },
  },
}
```

### Modelvalg

Når det er konfigureret, er alle dine Ollama-modeller tilgængelige:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## Avanceret

### Reasoning-modeller

OpenClaw markerer modeller som reasoning-kompatible, når Ollama rapporterer `thinking` i `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Modelomkostninger

Ollama er gratis og kører lokalt, så alle modelomkostninger er sat til $0.

### Streaming-konfiguration

OpenClaws Ollama-integration bruger som standard den **native Ollama API** (`/api/chat`), som fuldt ud understøtter streaming og tool calling samtidigt. Der kræves ingen særlig konfiguration.

#### Legacy OpenAI-kompatibel tilstand

Hvis du i stedet skal bruge det OpenAI-kompatible endpoint (f.eks. bag en proxy, der kun understøtter OpenAI-format), skal du angive `api: "openai-completions"` eksplicit:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Bemærk: Det OpenAI-kompatible endpoint understøtter muligvis ikke streaming + tool calling samtidigt. Du kan være nødt til at deaktivere streaming med `params: { streaming: false }` i modelkonfigurationen.

### Deaktivér streaming for andre udbydere

For automatisk opdagede modeller, bruger OpenClaw kontekstvinduet rapporteret af Ollama, når det er tilgængeligt, ellers er det standard `8192`. Du kan tilsidesætte `contextWindow` og `maxTokens` i eksplicit leverandørkonfiguration.

## Fejlfinding

### Kontekstvinduer

Sørg for, at Ollama kører, og at du har sat `OLLAMA_API_KEY` (eller en auth-profil), og at du **ikke** har defineret en eksplicit `models.providers.ollama`-post:

```bash
ollama serve
```

Og at API’et er tilgængeligt:

```bash
curl http://localhost:11434/api/tags
```

### Ingen modeller tilgængelige

OpenClaw opdager kun automatisk modeller, der rapporterer værktøj støtte. Hvis din model ikke er angivet, enten:

- Hente en værktøjskompatibel model, eller
- Definere modellen eksplicit i `models.providers.ollama`.

For at tilføje modeller:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Forbindelse afvist

For at tilføje modeller:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## Forbindelse afvist

- [Modeludbydere](/concepts/model-providers) – Overblik over alle udbydere
- [Modelvalg](/concepts/models) – Sådan vælger du modeller
- [Konfiguration](/gateway/configuration) – Fuld konfigurationsreference


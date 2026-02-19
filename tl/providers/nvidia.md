---
summary: "Gamitin ang OpenAI-compatible API ng NVIDIA sa OpenClaw"
read_when:
  - Gusto mong gumamit ng mga NVIDIA model sa OpenClaw
  - Kailangan mong i-setup ang NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

Nagbibigay ang NVIDIA ng OpenAI-compatible API sa `https://integrate.api.nvidia.com/v1` para sa mga modelong Nemotron at NeMo. Mag-authenticate gamit ang API key mula sa [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## CLI setup

I-export ang key nang isang beses, pagkatapos ay patakbuhin ang onboarding at mag-set ng NVIDIA model:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Kung gagamit ka pa rin ng `--token`, tandaan na mase-save ito sa shell history at `ps` output; mas mainam gamitin ang env var kung maaari.

## Config snippet

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## Mga Model ID

- `nvidia/llama-3.1-nemotron-70b-instruct` (default)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Mga Tala

- OpenAI-compatible na `/v1` endpoint; gumamit ng API key mula sa NVIDIA NGC.
- Awtomatikong nag-e-enable ang provider kapag naka-set ang `NVIDIA_API_KEY`; gumagamit ng static defaults (131,072-token context window, 4,096 max tokens).

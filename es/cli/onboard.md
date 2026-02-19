---
summary: "Referencia de la CLI para `openclaw onboard` (asistente interactivo de incorporación)"
read_when:
  - Desea una configuración guiada para el Gateway, el espacio de trabajo, la autenticación, los canales y las Skills
title: "onboard"
---

# `openclaw onboard`

Asistente interactivo de incorporación (configuración del Gateway local o remoto).

## Guías relacionadas

- Centro de incorporación de la CLI: [Onboarding Wizard (CLI)](/start/wizard)
- Resumen de incorporación: [Onboarding Overview](/start/onboarding-overview)
- Automatización de la CLI: [CLI Automation](/start/wizard-cli-automation)
- Referencia de incorporación de la CLI: [CLI Onboarding Reference](/start/wizard-cli-reference)
- Incorporación en macOS: [Onboarding (macOS App)](/start/onboarding)

## Ejemplos

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Proveedor personalizado no interactivo:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` es opcional en modo no interactivo. Si se omite, el proceso de incorporación comprueba `CUSTOM_API_KEY`.

Opciones de endpoint Z.AI no interactivo:

Nota: `--auth-choice zai-api-key` ahora detecta automáticamente el mejor endpoint de Z.AI para tu clave (prefiere la API general con `zai/glm-5`).
Si quieres específicamente los endpoints del GLM Coding Plan, elige `zai-coding-global` o `zai-coding-cn`.

```bash
# Selección de endpoint sin prompts
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Otras opciones de endpoint Z.AI:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Notas del flujo:

- `quickstart`: indicaciones mínimas, genera automáticamente un token del Gateway.
- `manual`: indicaciones completas para puerto/enlace/autenticación (alias de `advanced`).
- Primer chat más rápido: `openclaw dashboard` (UI de control, sin configuración de canal).
- Proveedor personalizado: conecta cualquier endpoint compatible con OpenAI o Anthropic,
  incluidos proveedores alojados que no estén en la lista. Usa Unknown para la detección automática.

## Comandos comunes posteriores

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` no implica modo no interactivo. Use `--non-interactive` para scripts.
</Note>


---
summary: "Pangkalahatang-ideya ng mga opsyon at daloy ng onboarding ng OpenClaw"
read_when:
  - Pagpili ng onboarding path
  - Pag-set up ng bagong environment
title: "Pangkalahatang-ideya ng Onboarding"
sidebarTitle: "Pangkalahatang-ideya ng Onboarding"
---

# Pangkalahatang-ideya ng Onboarding

Sinusuportahan ng OpenClaw ang maraming onboarding path depende sa kung saan tumatakbo ang Gateway
at kung paano mo gustong i-configure ang mga provider.

## Piliin ang iyong onboarding path

- **CLI wizard** para sa macOS, Linux, at Windows (sa pamamagitan ng WSL2).
- **macOS app** para sa gabay na unang pagtakbo sa Apple silicon o Intel Macs.

## CLI onboarding wizard

Patakbuhin ang wizard sa isang terminal:

```bash
openclaw onboard
```

Gamitin ang CLI wizard kung gusto mo ng ganap na kontrol sa Gateway, workspace,
channels, at skills. Docs:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS app onboarding

Gamitin ang OpenClaw app kung gusto mo ng ganap na may gabay na setup sa macOS. Docs:

- [Onboarding (macOS App)](/start/onboarding)

## Custom Provider

Kung kailangan mo ng endpoint na wala sa listahan, kabilang ang mga hosted provider na naglalabas ng karaniwang OpenAI o Anthropic APIs, piliin ang **Custom Provider** sa CLI wizard. Hihilingin sa iyo na:

- Pumili ng OpenAI-compatible, Anthropic-compatible, o **Unknown** (auto-detect).
- Maglagay ng base URL at API key (kung kinakailangan ng provider).
- Magbigay ng model ID at opsyonal na alias.
- Pumili ng Endpoint ID upang maaaring magkasabay ang maraming custom endpoint.

Para sa detalyadong mga hakbang, sundan ang CLI onboarding docs sa itaas.


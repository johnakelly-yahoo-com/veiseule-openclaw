---
title: "Mga Development Channel"
---

# Mga development channel

Huling na-update: 2026-01-21

Nagpapadala ang OpenClaw ng tatlong update channel:

- **matatag**: npm dist-tag `latest`.
- **beta**: npm dist-tag `beta` (mga build na sinusubok).
- Maaaring **hindi** kasama sa beta at dev builds ang isang macOS app release. 10. npm dist-tag: `dev` (kapag na-publish).

Nagpapadala kami ng mga build sa **beta**, sinusubok ang mga ito, at pagkatapos ay **pino-promote ang isang na-validate na build sa `latest`**
nang hindi binabago ang version number — ang mga dist-tag ang pinagmumulan ng katotohanan para sa mga npm install.

## Paglipat ng channel

Pag-checkout ng Git:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

- `stable`/`beta` nagche-check out ng pinakabagong tumutugmang tag (madalas ay parehong tag).
- `dev` lumilipat sa `main` at nagre-rebase sa upstream.

Global na pag-install ng npm/pnpm:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Ina-update nito sa pamamagitan ng katumbas na npm dist-tag (`latest`, `beta`, `dev`).

Kapag **hayagan** kang lumipat ng channel gamit ang `--channel`, ina-align din ng OpenClaw
ang paraan ng pag-install:

- `dev` tinitiyak ang isang git checkout (default `~/openclaw`, i-override gamit ang `OPENCLAW_GIT_DIR`),
  ina-update ito, at ini-install ang global CLI mula sa checkout na iyon.
- `stable`/`beta` nag-i-install mula sa npm gamit ang katugmang dist-tag.

Tip: kung gusto mo ng stable + dev nang sabay, magpanatili ng dalawang clone at ituro ang iyong gateway sa stable na isa.

## Mga plugin at channel

Kapag lumipat ka ng channel gamit ang `openclaw update`, sini-sync din ng OpenClaw ang mga source ng plugin:

- `dev` mas pinipili ang mga bundled plugin mula sa git checkout.
- `stable` at `beta` ibinabalik ang mga npm-installed na plugin package.

## Mga pinakamahusay na kasanayan sa pag-tag

- I-tag ang mga release na gusto mong landingan ng mga git checkout (`vYYYY.M.D` o `vYYYY.M.D-<patch>`).
- Panatilihing immutable ang mga tag: huwag kailanman ilipat o muling gamitin ang isang tag.
- Nanatiling pinagmumulan ng katotohanan ang mga npm dist-tag para sa mga npm install:
  - `latest` → stable
  - `beta` → kandidatong build
  - `dev` → snapshot ng main (opsyonal)

## Availability ng macOS app

Tingnan ang [Sandboxing](/gateway/sandboxing). 12. OK lang iyon:

- Maaari pa ring i-publish ang git tag at npm dist-tag.
- Banggitin ang “walang macOS build para sa beta na ito” sa mga release note o changelog.



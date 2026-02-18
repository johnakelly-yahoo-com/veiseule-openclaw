---
title: "Mga Kasanayan"
---

# Mga Kasanayan (macOS)

Ipinapakita ng macOS app ang OpenClaw Skills sa pamamagitan ng Gateway; hindi nito pinoproseso ang Skills nang lokal.

## Pinagmulan ng Datos

- Ang `skills.status` (gateway) ay nagbabalik ng lahat ng Skills kasama ang eligibility at mga kulang na kinakailangan
  (kasama ang mga allowlist block para sa mga bundled na Skills).
- Ang mga kinakailangan ay hinango mula sa `metadata.openclaw.requires` sa bawat `SKILL.md`.

## Mga aksyon sa pag-install

- Tinutukoy ng `metadata.openclaw.install` ang mga opsyon sa pag-install (brew/node/go/uv).
- Tinatawag ng app ang `skills.install` upang patakbuhin ang mga installer sa host ng Gateway.
- Ipinapakita ng Gateway ang iisang preferred installer lamang kapag may maraming ibinigay
  (brew kapag available, kung hindi ay node manager mula sa `skills.install`, default npm).

## Mga Env/API key

- 22. Iniimbak ng app ang mga key sa `~/.openclaw/openclaw.json` sa ilalim ng \`skills.entries.<skillKey>.
- Ang `skills.update` ay nagpa-patch ng `enabled`, `apiKey`, at `env`.

## Malayuang Mode

- Ang pag-install at mga update sa config ay nangyayari sa host ng Gateway (hindi sa lokal na Mac).

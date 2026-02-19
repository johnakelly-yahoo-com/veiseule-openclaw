---
summary: "Sanggunian ng CLI para sa `openclaw plugins` (listahan, install, enable/disable, doctor)"
read_when:
  - Gusto mong mag-install o mag-manage ng mga in-process na plugin ng Gateway
  - Gusto mong mag-debug ng mga failure sa pag-load ng plugin
title: "mga plugin"
---

# `openclaw plugins`

I-manage ang mga plugin/extension ng Gateway (nilo-load in-process).

Kaugnay:

- Plugin system: [Mga Plugin](/tools/plugin)
- Manifest + schema ng plugin: [Manifest ng plugin](/plugins/manifest)
- Pagpapatibay ng seguridad: [Security](/gateway/security)

## Mga command

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Ang mga bundled plugin ay kasama ng OpenClaw ngunit nagsisimulang naka-disable. Gamitin ang `plugins enable` upang
i-activate ang mga ito.

Dapat maglaman ang lahat ng plugin ng isang `openclaw.plugin.json` file na may inline JSON Schema
(`configSchema`, kahit walang laman). Ang nawawala/invalid na mga manifest o schema ay pumipigil
sa pag-load ng plugin at nagdudulot ng pagkabigo sa config validation.

### I-install

```bash
openclaw plugins install <path-or-spec>
```

Paalaala sa seguridad: ituring ang pag-install ng plugin na parang pagpapatakbo ng code. Mas mainam ang mga pinned na bersyon.

Ang Npm specs ay **registry-only** (pangalan ng package + opsyonal na bersyon/tag). Ang Git/URL/file
specs ay tinatanggihan. Ang pag-install ng dependency ay tumatakbo gamit ang `--ignore-scripts` para sa seguridad.

Gamitin ang `--link` para iwasan ang pagkopya ng lokal na directory (idinadagdag sa `plugins.load.paths`):

Gamitin ang `--link` para iwasan ang pagkopya ng lokal na directory (idinadagdag sa `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

### I-uninstall

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

Ang `uninstall` ay nag-aalis ng mga tala ng plugin mula sa `plugins.entries`, `plugins.installs`,
allowlist ng plugin, at mga naka-link na `plugins.load.paths` na entry kung naaangkop.
Para sa mga aktibong memory plugin, ang memory slot ay nire-reset sa `memory-core`.

Bilang default, ang uninstall ay nag-aalis din ng plugin install directory sa ilalim ng aktibong
state dir extensions root (`$OPENCLAW_STATE_DIR/extensions/<id>`). Gamitin ang
`--keep-files` upang panatilihin ang mga file sa disk.

Ang `--keep-config` ay sinusuportahan bilang deprecated na alias para sa `--keep-files`.

### I-update

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Ang mga update ay nalalapat lamang sa mga plugin na naka-install mula sa npm (sinusubaybayan sa `plugins.installs`).


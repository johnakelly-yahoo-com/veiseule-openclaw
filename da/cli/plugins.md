---
summary: "CLI-reference for `openclaw plugins` (liste, installér, aktivér/deaktivér, doctor)"
read_when:
  - Du vil installere eller administrere in-process Gateway-plugins
  - Du vil fejlfinde fejl ved indlæsning af plugins
title: "plugins"
---

# `openclaw plugins`

Administrér Gateway-plugins/udvidelser (indlæst in-process).

Relateret:

- Pluginsystem: [Plugins](/tools/plugin)
- Pluginmanifest + skema: [Plugin manifest](/plugins/manifest)
- Sikkerhedshærdning: [Security](/gateway/security)

## Kommandoer

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Bundtede plugins skib med OpenClaw men begynde deaktiveret. Brug 'plugins aktivere' til
aktivere dem.

Alle plugins skal sende en `openclaw.plugin.json` fil med en inline JSON Schema
(`configSchema`, selvom tom). Manglende / ugyldige manifester eller skemaer forhindrer
plugin i at indlæse og mislykkes config validering.

### Installér

```bash
openclaw plugins install <path-or-spec>
```

Sikkerhedsnote: behandl plugin installeres som kørende kode. Foretræk fastgjorte versioner.

Npm-specifikationer er **kun registry-baserede** (pakkenavn + valgfri version/tag). Git/URL/file
specifikationer afvises. Installation af afhængigheder køres med `--ignore-scripts` af sikkerhedshensyn.

Brug `--link` for at undgå at kopiere en lokal mappe (tilføjer til `plugins.load.paths`):

Brug `--link` for at undgå at kopiere en lokal mappe (tilføjer til `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

### Afinstaller

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` fjerner plugin-poster fra `plugins.entries`, `plugins.installs`,
plugin-allowlisten og tilknyttede `plugins.load.paths`-poster, når det er relevant.
For aktive memory-plugins nulstilles memory-slot til `memory-core`.

Som standard fjerner uninstall også pluginets installationsmappe under den aktive
state dir extensions-rod (`$OPENCLAW_STATE_DIR/extensions/<id>`). Brug
`--keep-files` for at beholde filer på disken.

`--keep-config` understøttes som et forældet alias for `--keep-files`.

### Opdatér

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Opdateringer gælder kun for plugins installeret fra npm (sporet i `plugins.installs`).

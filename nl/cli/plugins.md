---
summary: "CLI-referentie voor `openclaw plugins` (lijst, installeren, in-/uitschakelen, diagnose)"
read_when:
  - Je wilt in-process Gateway-plugins installeren of beheren
  - Je wilt fouten bij het laden van plugins debuggen
title: "plugins"
---

# `openclaw plugins`

Beheer Gateway-plugins/extensies (in-process geladen).

Gerelateerd:

- Pluginsysteem: [Plugins](/tools/plugin)
- Pluginmanifest + schema: [Plugin manifest](/plugins/manifest)
- Beveiligingsverharding: [Security](/gateway/security)

## Commando's

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Gebundelde plugins worden met OpenClaw geleverd maar starten uitgeschakeld. Gebruik `plugins enable` om ze
te activeren.

Alle plugins moeten een `openclaw.plugin.json`-bestand meeleveren met een inline JSON Schema
(`configSchema`, zelfs als het leeg is). Ontbrekende/ongeldige manifesten of schema’s voorkomen
dat de plugin wordt geladen en laten de configvalidatie falen.

### Installeren

```bash
openclaw plugins install <path-or-spec>
```

Beveiligingsopmerking: behandel plugininstallaties alsof je code uitvoert. Geef de voorkeur aan vastgepinde versies.

Npm-specs zijn **alleen registry** (pakketnaam + optionele versie/tag). Git/URL/file
specs worden afgewezen. Installaties van afhankelijkheden worden uitgevoerd met `--ignore-scripts` voor veiligheid.

Ondersteunde archieven: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Gebruik `--link` om het kopiëren van een lokale map te vermijden (voegt toe aan `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

### Deïnstalleren

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files>
```

`uninstall` verwijdert pluginvermeldingen uit `plugins.entries`, `plugins.installs`, de plugin-allowlist en gekoppelde `plugins.load.paths`-items indien van toepassing.
Voor actieve geheugenplugins wordt de geheugenslot teruggezet naar `memory-core`.

Standaard verwijdert uninstall ook de plugin-installatiemap onder de actieve state-dir extensions-root (`$OPENCLAW_STATE_DIR/extensions/<id>`). Gebruik
`--keep-files` om bestanden op schijf te behouden.

`--keep-config` wordt ondersteund als een verouderde alias voor `--keep-files`.

### Bijwerken

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Updates zijn alleen van toepassing op plugins die via npm zijn geïnstalleerd (bijgehouden in `plugins.installs`).


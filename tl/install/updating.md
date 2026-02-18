---
summary: "Ligtas na pag-update ng OpenClaw (global install o source), kasama ang estratehiya sa rollback"
read_when:
  - Pag-update ng OpenClaw
  - May nasira pagkatapos ng update
title: "Pag-update"
---

# Pag-update

Ang OpenClaw ay mabilis ang pag-unlad (bago ang “1.0”). Ituring ang mga update na parang shipping infra: mag-update → magpatakbo ng mga check → mag-restart (o gamitin ang `openclaw update`, na awtomatikong nagre-restart) → mag-verify.

## Inirerekomenda: patakbuhin muli ang website installer (upgrade in place)

Ang **mas pinipiling** paraan ng pag-update ay ang muling patakbuhin ang installer mula sa website. Ito ay
detects existing installs, upgrades in place, and runs `openclaw doctor` when
needed.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Mga tala:

- Idagdag ang `--no-onboard` kung ayaw mong tumakbo muli ang onboarding wizard.

- Para sa **source installs**, gamitin ang:

  ```bash
  curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
  ```

  Ang installer ay `git pull --rebase` **lamang** kung malinis ang repo.

- Para sa **global installs**, ginagamit ng script ang `npm install -g openclaw@latest` sa likod ng eksena.

- Legacy note: nananatiling available ang `clawdbot` bilang compatibility shim.

## Bago ka mag-update

- Alamin kung paano ka nag-install: **global** (npm/pnpm) vs **from source** (git clone).
- Alamin kung paano tumatakbo ang iyong Gateway: **foreground terminal** vs **supervised service** (launchd/systemd).
- Mag-snapshot ng iyong tailoring:
  - Config: `~/.openclaw/openclaw.json`
  - Credentials: `~/.openclaw/credentials/`
  - Workspace: `~/.openclaw/workspace`

## Update (global na install)

Global install (pumili ng isa):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

**Hindi** namin inirerekomenda ang Bun para sa Gateway runtime (may mga bug sa WhatsApp/Telegram).

Para magpalit ng update channels (git + npm installs):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Gamitin ang `--tag <dist-tag|version>` para sa one-off install na tag/version.

Tingnan ang [Development channels](/install/development-channels) para sa kahulugan ng mga channel at release notes.

Tandaan: sa mga npm install, ang gateway ay naglalabas ng update hint sa startup (sinusuri ang kasalukuyang channel tag). I-disable ito sa pamamagitan ng `update.checkOnStart: false`.

Pagkatapos:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Mga tala:

- Kung tumatakbo ang iyong Gateway bilang service, mas mainam ang `openclaw gateway restart` kaysa sa pagpatay ng mga PID.
- Kung naka-pin ka sa isang partikular na version, tingnan ang “Rollback / pinning” sa ibaba.

## Update (`openclaw update`)

Para sa **source installs** (git checkout), mas mainam ang:

```bash
openclaw update
```

Nagpapatakbo ito ng medyo ligtas na update flow:

- Nangangailangan ng malinis na worktree.
- Lumilipat sa napiling channel (tag o branch).
- Nagfe-fetch + nagre-rebase laban sa naka-configure na upstream (dev channel).
- Nag-i-install ng deps, nagbi-build, nagbi-build ng Control UI, at nagpapatakbo ng `openclaw doctor`.
- Nagre-restart ng gateway bilang default (gamitin ang `--no-restart` para laktawan).

Kung nag-install ka sa pamamagitan ng **npm/pnpm** (walang git metadata), susubukan ng `openclaw update` na mag-update gamit ang iyong package manager. Kung hindi nito matukoy ang pag-install, gamitin na lang ang “Update (global install)”.

## Update (Control UI / RPC)

Ang Control UI ay may **Update & Restart** (RPC: `update.run`). Ito ay:

1. Pinapatakbo ang parehong source-update flow gaya ng `openclaw update` (git checkout lamang).
2. Nagsusulat ng restart sentinel na may structured report (stdout/stderr tail).
3. Nire-restart ang gateway at pini-ping ang huling aktibong session kasama ang report.

Kung pumalya ang rebase, ina-abort ng gateway ang proseso at nagre-restart nang hindi ina-apply ang update.

## Update (from source)

Mula sa repo checkout:

Mas mainam:

```bash
openclaw update
```

Manu-manong paraan (halos katumbas):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # auto-installs UI deps on first run
openclaw doctor
openclaw health
```

Mga tala:

- Mahalaga ang `pnpm build` kapag pinapatakbo mo ang packaged na `openclaw` binary ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) o gumagamit ng Node para patakbuhin ang `dist/`.
- Kung tumatakbo ka mula sa repo checkout nang walang global install, gamitin ang `pnpm openclaw ...` para sa mga CLI command.
- Kung direkta kang tumatakbo mula sa TypeScript (`pnpm openclaw ...`), kadalasan ay hindi na kailangan ang rebuild, pero **naaangkop pa rin ang mga config migration** → patakbuhin ang doctor.
- Madali ang paglipat sa pagitan ng global at git installs: i-install ang kabilang variant, pagkatapos ay patakbuhin ang `openclaw doctor` para muling maisulat ang gateway service entrypoint sa kasalukuyang install.

## Palaging Patakbuhin: `openclaw doctor`

Doctor is the “safe update” command. It’s intentionally boring: repair + migrate + warn.

Tala: kung nasa **source install** ka (git checkout), mag-aalok ang `openclaw doctor` na patakbuhin muna ang `openclaw update`.

Mga tipikal na ginagawa nito:

- Pag-migrate ng deprecated na config keys / legacy config file locations.
- Pag-audit ng DM policies at pagbibigay-babala sa mga mapanganib na “open” na setting.
- Pag-check ng Gateway health at maaaring mag-alok na mag-restart.
- Pag-detect at pag-migrate ng mas lumang gateway services (launchd/systemd; legacy schtasks) papunta sa kasalukuyang OpenClaw services.
- Sa Linux, pagtiyak ng systemd user lingering (para manatiling buhay ang Gateway kahit mag-logout).

Mga detalye: [Doctor](/gateway/doctor)

## Simulan / ihinto / i-restart ang Gateway

CLI (gumagana kahit anong OS):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Kung supervised ka:

- macOS launchd (app-bundled LaunchAgent): `launchctl kickstart -k gui/$UID/bot.molt.gateway` (use `bot.molt.<profile>`; legacy `com.openclaw.*` still works)
- Linux systemd user service: `systemctl --user restart openclaw-gateway[-<profile>].service`
- Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profile>].service`
  - Gumagana lang ang `launchctl`/`systemctl` kung naka-install ang service; kung hindi, patakbuhin ang `openclaw gateway install`.

Runbook + eksaktong service labels: [Gateway runbook](/gateway)

## Rollback / pinning (kapag may nasira)

### Pin (global install)

Mag-install ng subok-na-maayos na version (palitan ang `<version>` ng huling gumaganang version):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Tip: para makita ang kasalukuyang published version, patakbuhin ang `npm view openclaw version`.

Pagkatapos ay i-restart + patakbuhin muli ang doctor:

```bash
openclaw doctor
openclaw gateway restart
```

### Pin (source) ayon sa petsa

Pumili ng commit mula sa isang petsa (halimbawa: “estado ng main noong 2026-01-01”):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Pagkatapos ay muling mag-install ng deps + restart:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Kung gusto mong bumalik sa pinakabago sa susunod:

```bash
git checkout main
git pull
```

## Kung na-stuck ka

- Patakbuhin muli ang `openclaw doctor` at basahin nang mabuti ang output (madalas nitong sinasabi ang solusyon).
- Tingnan: [Pag-troubleshoot](/gateway/troubleshooting)
- Magtanong sa Discord: [https://discord.gg/clawd](https://discord.gg/clawd)

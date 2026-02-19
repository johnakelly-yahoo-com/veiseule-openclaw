---
summary: "Patakbuhin ang OpenClaw sa isang rootless na Podman container"
read_when:
  - Gusto mo ng containerized na gateway gamit ang Podman sa halip na Docker
title: "Podman"
---

# Podman

Patakbuhin ang OpenClaw gateway sa isang **rootless** na Podman container. Gumagamit ng parehong image tulad ng Docker (i-build mula sa repo [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Mga Kinakailangan

- Podman (rootless)
- Sudo para sa one-time setup (gumawa ng user, i-build ang image)

## Mabilisang pagsisimula

**1. One-time setup** (mula sa root ng repo; gumagawa ng user, nagbi-build ng image, nag-i-install ng launch script):

```bash
./setup-podman.sh
```

Lumilikha rin ito ng minimal na `~openclaw/.openclaw/openclaw.json` (itinatakda ang `gateway.mode="local"`) upang makapagsimula ang gateway nang hindi pinapatakbo ang wizard.

Bilang default, ang container ay **hindi** naka-install bilang systemd service; mano-mano mo itong sisimulan (tingnan sa ibaba). Para sa production-style na setup na may auto-start at restarts, i-install ito bilang systemd Quadlet user service sa halip:

```bash
./setup-podman.sh --quadlet
```

(O itakda ang `OPENCLAW_PODMAN_QUADLET=1`; gamitin ang `--container` upang i-install lamang ang container at launch script.)

**2. Simulan ang gateway** (mano-mano, para sa mabilis na smoke testing):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding wizard** (hal. para magdagdag ng mga channel o provider):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Pagkatapos ay buksan ang `http://127.0.0.1:18789/` at gamitin ang token mula sa `~openclaw/.openclaw/.env` (o ang value na ipinakita ng setup).

## Systemd (Quadlet, opsyonal)

Kung pinatakbo mo ang `./setup-podman.sh --quadlet` (o `OPENCLAW_PODMAN_QUADLET=1`), mag-i-install ng [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) unit upang tumakbo ang gateway bilang systemd user service para sa openclaw user. Ang service ay naka-enable at sinimulan sa dulo ng setup.

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stop:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Ang quadlet file ay matatagpuan sa `~openclaw/.config/containers/systemd/openclaw.container`. Upang baguhin ang mga port o env, i-edit ang file na iyon (o ang `.env` na ginagamit nito), pagkatapos ay patakbuhin ang `sudo systemctl --machine openclaw@ --user daemon-reload` at i-restart ang service. Sa pag-boot, awtomatikong magsisimula ang service kung naka-enable ang lingering para sa openclaw (ginagawa ito ng setup kapag available ang loginctl).

Upang magdagdag ng quadlet **pagkatapos** ng paunang setup na hindi ito ginamit, patakbuhin muli: `./setup-podman.sh --quadlet`.

## Ang openclaw user (non-login)

Lumilikha ang `setup-podman.sh` ng nakalaang system user na `openclaw`:

- **Shell:** `nologin` — walang interactive login; binabawasan ang attack surface.

- **Home:** hal. `/home/openclaw` — naglalaman ng `~/.openclaw` (config, workspace) at ng launch script na `run-openclaw-podman.sh`.

- **Rootless Podman:** Dapat may **subuid** at **subgid** range ang user. Maraming distro ang awtomatikong nagtatalaga ng mga ito kapag nilikha ang user. Kung may babala na ipinakita ang setup, magdagdag ng mga linya sa `/etc/subuid` at `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Pagkatapos ay simulan ang gateway bilang user na iyon (hal. mula sa cron o systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** Tanging `openclaw` at root lamang ang may access sa `/home/openclaw/.openclaw`. Upang i-edit ang config: gamitin ang Control UI kapag tumatakbo na ang gateway, o `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Environment at config

- **Token:** Naka-store sa `~openclaw/.openclaw/.env` bilang `OPENCLAW_GATEWAY_TOKEN`. Ang `setup-podman.sh` at `run-openclaw-podman.sh` ang gagawa nito kung wala pa (gumagamit ng `openssl`, `python3`, o `od`).
- **Optional:** Sa `.env` na iyon maaari kang magtakda ng mga provider key (hal. `GROQ_API_KEY`, `OLLAMA_API_KEY`) at iba pang OpenClaw env vars.
- **Host ports:** Bilang default, mina-map ng script ang `18789` (gateway) at `18790` (bridge). I-override ang **host** port mapping gamit ang `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` at `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` kapag nagla-launch.
- **Paths:** Ang host config at workspace ay naka-default sa `~openclaw/.openclaw` at `~openclaw/.openclaw/workspace`. I-override ang mga host path na ginagamit ng launch script gamit ang `OPENCLAW_CONFIG_DIR` at `OPENCLAW_WORKSPACE_DIR`.

## Mga kapaki-pakinabang na command

- **Logs:** Sa quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Sa script: `sudo -u openclaw podman logs -f openclaw`
- **Stop:** Sa quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Sa script: `sudo -u openclaw podman stop openclaw`
- **Start again:** Sa quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Sa script: patakbuhin muli ang launch script o `podman start openclaw`
- **Remove container:** `sudo -u openclaw podman rm -f openclaw` — mananatili ang config at workspace sa host

## Troubleshooting

- **Tinanggihan ang pahintulot (EACCES) sa config o auth-profiles:** Ang container ay naka-default sa `--userns=keep-id` at tumatakbo gamit ang parehong uid/gid ng host user na nagpapatakbo ng script. Tiyaking ang iyong host `OPENCLAW_CONFIG_DIR` at `OPENCLAW_WORKSPACE_DIR` ay pag-aari ng user na iyon.
- **Naharang ang pagsisimula ng Gateway (kulang ang `gateway.mode=local`):** Tiyaking umiiral ang `~openclaw/.openclaw/openclaw.json` at nakatakda ang `gateway.mode="local"`. Nililikha ng `setup-podman.sh` ang file na ito kung wala pa.
- **Nabigo ang rootless Podman para sa user na openclaw:** Suriin kung ang `/etc/subuid` at `/etc/subgid` ay may linyang para sa `openclaw` (hal. `openclaw:100000:65536`). Idagdag ito kung wala at i-restart.
- **Ginagamit na ang pangalan ng container:** Gumagamit ang launch script ng `podman run --replace`, kaya papalitan ang umiiral na container kapag muli kang nag-start. Para maglinis nang mano-mano: `podman rm -f openclaw`.
- **Hindi makita ang script kapag tumatakbo bilang openclaw:** Tiyaking naipatupad ang `setup-podman.sh` upang makopya ang `run-openclaw-podman.sh` sa home ng openclaw (hal. `/home/openclaw/run-openclaw-podman.sh`).
- **Hindi makita ang Quadlet service o nabigong mag-start:** Patakbuhin ang `sudo systemctl --machine openclaw@ --user daemon-reload` pagkatapos i-edit ang `.container` file. Kinakailangan ng Quadlet ang cgroups v2: ang `podman info --format '{{.Host.CgroupsVersion}}'` ay dapat magpakita ng `2`.

## Opsyonal: patakbuhin gamit ang sarili mong user

Upang patakbuhin ang gateway bilang iyong karaniwang user (walang nakatalagang openclaw user): buuin ang image, lumikha ng `~/.openclaw/.env` na may `OPENCLAW_GATEWAY_TOKEN`, at patakbuhin ang container gamit ang `--userns=keep-id` at mga mount papunta sa iyong `~/.openclaw`. Ang launch script ay dinisenyo para sa openclaw-user flow; para sa single-user setup maaari mong patakbuhin nang mano-mano ang `podman run` command mula sa script, ituro ang config at workspace sa iyong home. Inirerekomenda para sa karamihan ng user: gamitin ang `setup-podman.sh` at patakbuhin bilang openclaw user upang maihiwalay ang config at proseso.

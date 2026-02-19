---
summary: "Kør OpenClaw i en rootless Podman-container"
read_when:
  - Du ønsker en containeriseret gateway med Podman i stedet for Docker
title: "Podman"
---

# Podman

Kør OpenClaw-gatewayen i en **rootless** Podman-container. Bruger det samme image som Docker (byg fra repoets [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Krav

- Podman (rootless)
- Sudo til engangsopsætning (opret bruger, byg image)

## Hurtig start

**1. Engangsopsætning** (fra repo-roden; opretter bruger, bygger image, installerer startscript):

```bash
./setup-podman.sh
```

Dette opretter også en minimal `~openclaw/.openclaw/openclaw.json` (sætter `gateway.mode="local"`), så gatewayen kan starte uden at køre guiden.

Som standard installeres containeren **ikke** som en systemd-tjeneste; du starter den manuelt (se nedenfor). For en produktionslignende opsætning med automatisk opstart og genstarter skal du i stedet installere den som en systemd Quadlet-brugertjeneste:

```bash
./setup-podman.sh --quadlet
```

(Eller sæt `OPENCLAW_PODMAN_QUADLET=1`; brug `--container` for kun at installere containeren og startscriptet.)

**2. Start gateway** (manuelt, til hurtig smoke-test):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding-guide** (f.eks. for at tilføje kanaler eller udbydere):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Åbn derefter `http://127.0.0.1:18789/` og brug tokenet fra `~openclaw/.openclaw/.env` (eller værdien, der blev vist af setup).

## Systemd (Quadlet, valgfrit)

Hvis du kørte `./setup-podman.sh --quadlet` (eller `OPENCLAW_PODMAN_QUADLET=1`), installeres en [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)-unit, så gatewayen kører som en systemd-brugertjeneste for openclaw-brugeren. Tjenesten aktiveres og startes i slutningen af opsætningen.

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stop:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Quadlet-filen findes på `~openclaw/.config/containers/systemd/openclaw.container`. For at ændre porte eller miljøvariabler skal du redigere den fil (eller den `.env`, den indlæser), derefter køre `sudo systemctl --machine openclaw@ --user daemon-reload` og genstarte tjenesten. Ved opstart starter tjenesten automatisk, hvis lingering er aktiveret for openclaw (opsætningen gør dette, når loginctl er tilgængelig).

For at tilføje quadlet **efter** en indledende opsætning, der ikke brugte det, skal du køre igen: `./setup-podman.sh --quadlet`.

## Brugeren openclaw (uden login)

`setup-podman.sh` opretter en dedikeret systembruger `openclaw`:

- **Shell:** `nologin` — ingen interaktiv login; reducerer angrebsfladen.

- **Home:** f.eks. `/home/openclaw` — indeholder `~/.openclaw` (konfiguration, workspace) og startscriptet `run-openclaw-podman.sh`.

- **Rootless Podman:** Brugeren skal have et **subuid**- og **subgid**-interval. Mange distributioner tildeler disse automatisk, når brugeren oprettes. Hvis opsætningen viser en advarsel, skal du tilføje linjer i `/etc/subuid` og `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Start derefter gatewayen som den bruger (f.eks. fra cron eller systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Konfiguration:** Kun `openclaw` og root kan få adgang til `/home/openclaw/.openclaw`. For at redigere konfigurationen: brug Control UI, når gatewayen kører, eller `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Miljø og konfiguration

- **Token:** Gemt i `~openclaw/.openclaw/.env` som `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` og `run-openclaw-podman.sh` genererer det, hvis det mangler (bruger `openssl`, `python3` eller `od`).
- **Valgfrit:** I den `.env` kan du angive provider-nøgler (f.eks. `GROQ_API_KEY`, `OLLAMA_API_KEY`) og andre OpenClaw-miljøvariabler.
- **Host-porte:** Som standard mapper scriptet `18789` (gateway) og `18790` (bridge). Tilsidesæt **host**-portmappingen med `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` og `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` ved opstart.
- **Stier:** Hostens konfiguration og workspace er som standard `~openclaw/.openclaw` og `~openclaw/.openclaw/workspace`. Tilsidesæt host-stierne, der bruges af startscriptet, med `OPENCLAW_CONFIG_DIR` og `OPENCLAW_WORKSPACE_DIR`.

## Nyttige kommandoer

- **Logs:** Med quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Med script: `sudo -u openclaw podman logs -f openclaw`
- **Stop:** Med quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Med script: `sudo -u openclaw podman stop openclaw`
- **Start igen:** Med quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Med script: kør startscriptet igen eller `podman start openclaw`
- **Fjern container:** `sudo -u openclaw podman rm -f openclaw` — konfiguration og workspace på hosten bevares

## Fejlfinding

- **Permission denied (EACCES) på konfiguration eller auth-profiles:** Containeren bruger som standard `--userns=keep-id` og kører med samme uid/gid som host-brugeren, der kører scriptet. Sørg for, at din host `OPENCLAW_CONFIG_DIR` og `OPENCLAW_WORKSPACE_DIR` ejes af den bruger.
- **Gateway-opstart blokeret (mangler `gateway.mode=local`):** Sørg for, at `~openclaw/.openclaw/openclaw.json` findes og angiver `gateway.mode="local"`. `setup-podman.sh` opretter denne fil, hvis den mangler.
- **Rootless Podman mislykkes for brugeren openclaw:** Kontrollér, at `/etc/subuid` og `/etc/subgid` indeholder en linje for `openclaw` (f.eks. `openclaw:100000:65536`). Tilføj den, hvis den mangler, og genstart.
- **Container-navn er i brug:** Startscriptet bruger `podman run --replace`, så den eksisterende container erstattes, når du starter igen. For at rydde op manuelt: `podman rm -f openclaw`.
- **Script ikke fundet ved kørsel som openclaw:** Sørg for, at `setup-podman.sh` er kørt, så `run-openclaw-podman.sh` kopieres til openclaws home (f.eks. `/home/openclaw/run-openclaw-podman.sh`).
- **Quadlet-service ikke fundet eller kan ikke starte:** Kør `sudo systemctl --machine openclaw@ --user daemon-reload` efter redigering af `.container`-filen. Quadlet kræver cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` skal vise `2`.

## Valgfrit: kør som din egen bruger

For at køre gatewayen som din normale bruger (ingen dedikeret openclaw-bruger): byg imaget, opret `~/.openclaw/.env` med `OPENCLAW_GATEWAY_TOKEN`, og kør containeren med `--userns=keep-id` og mounts til din `~/.openclaw`. Launch-scriptet er designet til openclaw-user-flowet; for en enkeltbrugeropsætning kan du i stedet køre `podman run`-kommandoen fra scriptet manuelt og pege konfiguration og workspace til din home-mappe. Anbefalet for de fleste brugere: brug `setup-podman.sh` og kør som openclaw-brugeren, så konfiguration og proces er isoleret.


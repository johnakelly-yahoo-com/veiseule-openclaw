---
summary: "Voer OpenClaw uit in een rootless Podman-container"
read_when:
  - Je wilt een gecontaineriseerde gateway met Podman in plaats van Docker
title: "Podman"
---

# Podman

Voer de OpenClaw gateway uit in een **rootless** Podman-container. Gebruikt dezelfde image als Docker (build vanuit de repo [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Vereisten

- Podman (rootless)
- Sudo voor eenmalige setup (gebruiker aanmaken, image builden)

## Snelle start

**1. Eenmalige setup** (vanuit de root van de repo; maakt gebruiker aan, buildt image, installeert startscript):

```bash
./setup-podman.sh
```

Dit maakt ook een minimale `~openclaw/.openclaw/openclaw.json` aan (zet `gateway.mode="local"`) zodat de gateway kan starten zonder de wizard uit te voeren.

Standaard wordt de container **niet** geïnstalleerd als een systemd-service; je start deze handmatig (zie hieronder). Voor een productie-achtige setup met automatische start en herstarts, installeer het als een systemd Quadlet user service:

```bash
./setup-podman.sh --quadlet
```

(Of stel `OPENCLAW_PODMAN_QUADLET=1` in; gebruik `--container` om alleen de container en het startscript te installeren.)

**2. Gateway starten** (handmatig, voor een snelle rooktest):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding-wizard** (bijv. om kanalen of providers toe te voegen):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Open daarna `http://127.0.0.1:18789/` en gebruik de token uit `~openclaw/.openclaw/.env` (of de waarde die door setup is weergegeven).

## Systemd (Quadlet, optioneel)

Als je `./setup-podman.sh --quadlet` (of `OPENCLAW_PODMAN_QUADLET=1`) hebt uitgevoerd, wordt een [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)-unit geïnstalleerd zodat de gateway als een systemd user service voor de openclaw-gebruiker draait. De service wordt aan het einde van de setup ingeschakeld en gestart.

- **Starten:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stoppen:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Het quadlet-bestand bevindt zich op `~openclaw/.config/containers/systemd/openclaw.container`. Om poorten of env te wijzigen, bewerk dat bestand (of de `.env` waarnaar het verwijst), voer daarna `sudo systemctl --machine openclaw@ --user daemon-reload` uit en herstart de service. Bij het opstarten start de service automatisch als lingering is ingeschakeld voor openclaw (setup doet dit wanneer loginctl beschikbaar is).

Om quadlet **na** een initiële setup zonder quadlet toe te voegen, voer opnieuw uit: `./setup-podman.sh --quadlet`.

## De openclaw-gebruiker (geen login)

`setup-podman.sh` maakt een dedicated systeemgebruiker `openclaw` aan:

- **Shell:** `nologin` — geen interactieve login; verkleint het aanvalsoppervlak.

- **Home:** bijv. `/home/openclaw` — bevat `~/.openclaw` (configuratie, workspace) en het opstartscript `run-openclaw-podman.sh`.

- **Rootless Podman:** De gebruiker moet een **subuid**- en **subgid**-bereik hebben. Veel distributies wijzen deze automatisch toe wanneer de gebruiker wordt aangemaakt. Als setup een waarschuwing toont, voeg dan regels toe aan `/etc/subuid` en `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Start vervolgens de gateway als die gebruiker (bijv. via cron of systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** Alleen `openclaw` en root hebben toegang tot `/home/openclaw/.openclaw`. Om de configuratie te bewerken: gebruik de Control UI zodra de gateway draait, of `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Omgeving en configuratie

- **Token:** Opgeslagen in `~openclaw/.openclaw/.env` als `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` en `run-openclaw-podman.sh` genereren deze indien ontbrekend (gebruikt `openssl`, `python3` of `od`).
- **Optioneel:** In dat `.env`-bestand kun je provider-sleutels instellen (bijv. `GROQ_API_KEY`, `OLLAMA_API_KEY`) en andere OpenClaw-omgevingsvariabelen.
- **Hostpoorten:** Standaard koppelt het script `18789` (gateway) en `18790` (bridge). Overschrijf de **host**-poortkoppeling met `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` en `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` bij het starten.
- **Paden:** Hostconfiguratie en workspace staan standaard op `~openclaw/.openclaw` en `~openclaw/.openclaw/workspace`. Overschrijf de hostpaden die door het opstartscript worden gebruikt met `OPENCLAW_CONFIG_DIR` en `OPENCLAW_WORKSPACE_DIR`.

## Handige commando’s

- **Logs:** Met quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Met script: `sudo -u openclaw podman logs -f openclaw`
- **Stoppen:** Met quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Met script: `sudo -u openclaw podman stop openclaw`
- **Opnieuw starten:** Met quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Met script: voer het opstartscript opnieuw uit of `podman start openclaw`
- **Container verwijderen:** `sudo -u openclaw podman rm -f openclaw` — configuratie en workspace op de host blijven behouden

## Probleemoplossing

- **Permission denied (EACCES) op config of auth-profiles:** De container gebruikt standaard `--userns=keep-id` en draait met dezelfde uid/gid als de hostgebruiker die het script uitvoert. Zorg ervoor dat je host-`OPENCLAW_CONFIG_DIR` en `OPENCLAW_WORKSPACE_DIR` eigendom zijn van die gebruiker.
- **Gateway-start geblokkeerd (ontbrekend `gateway.mode=local`):** Zorg dat `~openclaw/.openclaw/openclaw.json` bestaat en `gateway.mode="local"` instelt. `setup-podman.sh` maakt dit bestand aan als het ontbreekt.
- **Rootless Podman faalt voor gebruiker openclaw:** Controleer of `/etc/subuid` en `/etc/subgid` een regel voor `openclaw` bevatten (bijv. `openclaw:100000:65536`). Voeg deze toe indien ontbrekend en herstart.
- **Containernaam al in gebruik:** Het opstartscript gebruikt `podman run --replace`, waardoor de bestaande container wordt vervangen wanneer je opnieuw start. Handmatig opschonen: `podman rm -f openclaw`.
- **Script niet gevonden bij uitvoeren als openclaw:** Zorg dat `setup-podman.sh` is uitgevoerd zodat `run-openclaw-podman.sh` is gekopieerd naar de home van openclaw (bijv. `/home/openclaw/run-openclaw-podman.sh`).
- **Quadlet-service niet gevonden of start niet:** Voer `sudo systemctl --machine openclaw@ --user daemon-reload` uit na het bewerken van het `.container`-bestand. Quadlet vereist cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` moet `2` tonen.

## Optioneel: uitvoeren als je eigen gebruiker

Om de gateway als je normale gebruiker uit te voeren (geen aparte openclaw-gebruiker): bouw de image, maak `~/.openclaw/.env` aan met `OPENCLAW_GATEWAY_TOKEN`, en start de container met `--userns=keep-id` en mounts naar je `~/.openclaw`. Het opstartscript is ontworpen voor de openclaw-gebruikerflow; voor een single-user-setup kun je in plaats daarvan het `podman run`-commando uit het script handmatig uitvoeren, waarbij je configuratie en workspace naar je home verwijst. Aanbevolen voor de meeste gebruikers: gebruik `setup-podman.sh` en voer uit als de openclaw-gebruiker zodat configuratie en proces geïsoleerd zijn.


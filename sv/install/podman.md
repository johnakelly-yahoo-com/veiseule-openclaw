---
summary: "Kör OpenClaw i en rootless Podman-container"
read_when:
  - Du vill ha en containerbaserad gateway med Podman istället för Docker
title: "Podman"
---

# Podman

Kör OpenClaw-gatewayen i en **rootless** Podman-container. Använder samma image som Docker (bygg från repots [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Krav

- Podman (rootless)
- Sudo för engångsinstallation (skapa användare, bygga image)

## Snabbstart

**1. Engångsinstallation** (från repots rotkatalog; skapar användare, bygger image, installerar startskript):

```bash
./setup-podman.sh
```

Detta skapar också en minimal `~openclaw/.openclaw/openclaw.json` (sätter `gateway.mode="local"`) så att gatewayen kan starta utan att köra guiden.

Som standard installeras containern **inte** som en systemd-tjänst, du startar den manuellt (se nedan). För en produktionsliknande installation med automatisk start och omstarter, installera den istället som en systemd Quadlet-användartjänst:

```bash
./setup-podman.sh --quadlet
```

(Eller sätt `OPENCLAW_PODMAN_QUADLET=1`; använd `--container` för att endast installera containern och startskriptet.)

**2. Starta gateway** (manuellt, för snabb smoke-testning):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding-guide** (t.ex. för att lägga till kanaler eller leverantörer):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Öppna sedan `http://127.0.0.1:18789/` och använd token från `~openclaw/.openclaw/.env` (eller värdet som skrevs ut av setup).

## Systemd (Quadlet, valfritt)

Om du körde `./setup-podman.sh --quadlet` (eller `OPENCLAW_PODMAN_QUADLET=1`), installeras en [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)-enhet så att gatewayen körs som en systemd-användartjänst för användaren openclaw. Tjänsten aktiveras och startas i slutet av installationen.

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stoppa:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Loggar:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Quadlet-filen finns på `~openclaw/.config/containers/systemd/openclaw.container`. För att ändra portar eller miljövariabler, redigera den filen (eller `.env`-filen som den använder), kör sedan `sudo systemctl --machine openclaw@ --user daemon-reload` och starta om tjänsten. Vid uppstart startar tjänsten automatiskt om lingering är aktiverat för openclaw (installationen gör detta när loginctl är tillgängligt).

För att lägga till quadlet **efter** en initial installation som inte använde den, kör igen: `./setup-podman.sh --quadlet`.

## Användaren openclaw (utan inloggning)

`setup-podman.sh` skapar en dedikerad systemanvändare `openclaw`:

- **Shell:** `nologin` — ingen interaktiv inloggning; minskar attackytan.

- **Hemkatalog:** t.ex. `/home/openclaw` — innehåller `~/.openclaw` (konfiguration, workspace) och startskriptet `run-openclaw-podman.sh`.

- **Rootless Podman:** Användaren måste ha ett **subuid**- och **subgid**-intervall. Många distributioner tilldelar dessa automatiskt när användaren skapas. Om installationen visar en varning, lägg till rader i `/etc/subuid` och `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Starta sedan gatewayen som den användaren (t.ex. via cron eller systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Konfiguration:** Endast `openclaw` och root kan komma åt `/home/openclaw/.openclaw`. För att redigera konfigurationen: använd Control UI när gatewayen körs, eller `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Miljö och konfiguration

- **Token:** Lagrar i `~openclaw/.openclaw/.env` som `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` och `run-openclaw-podman.sh` genererar den om den saknas (använder `openssl`, `python3` eller `od`).
- **Valfritt:** I den `.env`-filen kan du ange leverantörsnycklar (t.ex. `GROQ_API_KEY`, `OLLAMA_API_KEY`) och andra OpenClaw-miljövariabler.
- **Värdportar:** Som standard mappar skriptet `18789` (gateway) och `18790` (bridge). Åsidosätt **värdens** portmappning med `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` och `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` vid start.
- **Sökvägar:** Värdens konfiguration och workspace är som standard `~openclaw/.openclaw` och `~openclaw/.openclaw/workspace`. Åsidosätt värdsökvägarna som används av startskriptet med `OPENCLAW_CONFIG_DIR` och `OPENCLAW_WORKSPACE_DIR`.

## Användbara kommandon

- **Loggar:** Med quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Med skript: `sudo -u openclaw podman logs -f openclaw`
- **Stoppa:** Med quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Med skript: `sudo -u openclaw podman stop openclaw`
- **Starta igen:** Med quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Med skript: kör startskriptet igen eller `podman start openclaw`
- **Ta bort container:** `sudo -u openclaw podman rm -f openclaw` — konfiguration och workspace på värden behålls

## Felsökning

- **Permission denied (EACCES) för konfiguration eller auth-profiles:** Containern använder som standard `--userns=keep-id` och körs med samma uid/gid som värdanvändaren som kör skriptet. Säkerställ att dina `OPENCLAW_CONFIG_DIR` och `OPENCLAW_WORKSPACE_DIR` på värden ägs av den användaren.
- **Gateway-start blockeras (saknar `gateway.mode=local`):** Säkerställ att `~openclaw/.openclaw/openclaw.json` finns och anger `gateway.mode="local"`. `setup-podman.sh` skapar denna fil om den saknas.
- **Rootless Podman misslyckas för användaren openclaw:** Kontrollera att `/etc/subuid` och `/etc/subgid` innehåller en rad för `openclaw` (t.ex. `openclaw:100000:65536`). Lägg till den om den saknas och starta om.
- **Containernamn används redan:** Startskriptet använder `podman run --replace`, så den befintliga containern ersätts när du startar igen. För att rensa manuellt: `podman rm -f openclaw`.
- **Skript hittas inte när det körs som openclaw:** Säkerställ att `setup-podman.sh` har körts så att `run-openclaw-podman.sh` kopieras till openclaws hemkatalog (t.ex. `/home/openclaw/run-openclaw-podman.sh`).
- **Quadlet-tjänst hittas inte eller misslyckas att starta:** Kör `sudo systemctl --machine openclaw@ --user daemon-reload` efter att du har redigerat `.container`-filen. Quadlet kräver cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` ska visa `2`.

## Valfritt: kör som din egen användare

För att köra gateway som din vanliga användare (ingen dedikerad openclaw-användare): bygg imagen, skapa `~/.openclaw/.env` med `OPENCLAW_GATEWAY_TOKEN` och kör containern med `--userns=keep-id` samt montera din `~/.openclaw`. Startskriptet är utformat för openclaw-user-flödet; för en enanvändarinstallation kan du istället köra kommandot `podman run` från skriptet manuellt och peka konfiguration och workspace till din hemkatalog. Rekommenderas för de flesta användare: använd `setup-podman.sh` och kör som openclaw-användaren så att konfiguration och process är isolerade.

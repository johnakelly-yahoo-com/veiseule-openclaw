---
summary: "OpenClaw in einem rootless Podman-Container ausführen"
read_when:
  - Sie möchten ein containerisiertes Gateway mit Podman statt Docker verwenden
title: "Podman"
---

# Podman

Führen Sie das OpenClaw-Gateway in einem **rootless** Podman-Container aus. Verwendet dasselbe Image wie Docker (Build aus dem Repository-[Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Voraussetzungen

- Podman (rootless)
- Sudo für die einmalige Einrichtung (Benutzer erstellen, Image bauen)

## Schnellstart

**1. Einmalige Einrichtung** (aus dem Repository-Root; erstellt Benutzer, baut das Image, installiert das Startskript):

```bash
./setup-podman.sh
```

Dies erstellt außerdem eine minimale `~openclaw/.openclaw/openclaw.json` (setzt `gateway.mode="local"`), sodass das Gateway ohne Ausführen des Assistenten starten kann.

Standardmäßig wird der Container **nicht** als systemd-Service installiert; Sie starten ihn manuell (siehe unten). Für ein produktionsnahes Setup mit automatischem Start und Neustarts installieren Sie ihn stattdessen als systemd-Quadlet-Benutzerdienst:

```bash
./setup-podman.sh --quadlet
```

(Oder setzen Sie `OPENCLAW_PODMAN_QUADLET=1`; verwenden Sie `--container`, um nur den Container und das Startskript zu installieren.)

**2. Gateway starten** (manuell, für einen schnellen Smoke-Test):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding-Assistent** (z. B. zum Hinzufügen von Kanälen oder Providern):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Öffnen Sie anschließend `http://127.0.0.1:18789/` und verwenden Sie das Token aus `~openclaw/.openclaw/.env` (oder den beim Setup ausgegebenen Wert).

## Systemd (Quadlet, optional)

Wenn Sie `./setup-podman.sh --quadlet` (oder `OPENCLAW_PODMAN_QUADLET=1`) ausgeführt haben, wird eine [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)-Unit installiert, sodass das Gateway als systemd-Benutzerdienst für den Benutzer openclaw läuft. Der Dienst wird am Ende der Einrichtung aktiviert und gestartet.

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stop:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Die Quadlet-Datei befindet sich unter `~openclaw/.config/containers/systemd/openclaw.container`. Um Ports oder Umgebungsvariablen zu ändern, bearbeiten Sie diese Datei (oder die eingebundene `.env`-Datei) und führen Sie anschließend `sudo systemctl --machine openclaw@ --user daemon-reload` aus und starten den Service neu. Beim Booten startet der Service automatisch, wenn für openclaw „lingering“ aktiviert ist (das Setup erledigt dies, wenn loginctl verfügbar ist).

Um Quadlet **nach** einer anfänglichen Einrichtung ohne Quadlet hinzuzufügen, führen Sie erneut aus: `./setup-podman.sh --quadlet`.

## Der openclaw-Benutzer (ohne Login)

`setup-podman.sh` erstellt einen dedizierten Systembenutzer `openclaw`:

- **Shell:** `nologin` — keine interaktive Anmeldung; reduziert die Angriffsfläche.

- **Home:** z. B. `/home/openclaw` — enthält `~/.openclaw` (Konfiguration, Workspace) sowie das Startskript `run-openclaw-podman.sh`.

- **Rootless Podman:** Der Benutzer benötigt einen **subuid**- und **subgid**-Bereich. Viele Distributionen weisen diese automatisch zu, wenn der Benutzer erstellt wird. Wenn das Setup eine Warnung ausgibt, fügen Sie folgende Zeilen zu `/etc/subuid` und `/etc/subgid` hinzu:

  ```text
  openclaw:100000:65536
  ```

  Starten Sie anschließend das Gateway als dieser Benutzer (z. B. über cron oder systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** Nur `openclaw` und root können auf `/home/openclaw/.openclaw` zugreifen. Zum Bearbeiten der Konfiguration: Verwenden Sie die Control UI, sobald das Gateway läuft, oder `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Umgebung und Konfiguration

- **Token:** Gespeichert in `~openclaw/.openclaw/.env` als `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` und `run-openclaw-podman.sh` erzeugen es, falls es fehlt (verwendet `openssl`, `python3` oder `od`).
- **Optional:** In dieser `.env` können Sie Provider-Keys (z. B. `GROQ_API_KEY`, `OLLAMA_API_KEY`) und andere OpenClaw-Umgebungsvariablen festlegen.
- **Host-Ports:** Standardmäßig mappt das Skript `18789` (Gateway) und `18790` (Bridge). Überschreiben Sie das **Host**-Port-Mapping beim Start mit `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` und `OPENCLAW_PODMAN_BRIDGE_HOST_PORT`.
- **Pfade:** Host-Konfiguration und Workspace sind standardmäßig `~openclaw/.openclaw` und `~openclaw/.openclaw/workspace`. Überschreiben Sie die vom Startskript verwendeten Host-Pfade mit `OPENCLAW_CONFIG_DIR` und `OPENCLAW_WORKSPACE_DIR`.

## Nützliche Befehle

- **Logs:** Mit Quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Mit Skript: `sudo -u openclaw podman logs -f openclaw`
- **Stop:** Mit Quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Mit Skript: `sudo -u openclaw podman stop openclaw`
- **Erneut starten:** Mit Quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Mit Skript: Startskript erneut ausführen oder `podman start openclaw`
- **Container entfernen:** `sudo -u openclaw podman rm -f openclaw` — Konfiguration und Workspace auf dem Host bleiben erhalten

## Fehlerbehebung

- **Permission denied (EACCES) bei config oder auth-profiles:** Der Container verwendet standardmäßig `--userns=keep-id` und läuft mit derselben uid/gid wie der Host-Benutzer, der das Skript ausführt. Stellen Sie sicher, dass Ihre Host-Verzeichnisse `OPENCLAW_CONFIG_DIR` und `OPENCLAW_WORKSPACE_DIR` diesem Benutzer gehören.
- **Gateway-Start blockiert (fehlendes `gateway.mode=local`):** Stellen Sie sicher, dass `~openclaw/.openclaw/openclaw.json` existiert und `gateway.mode="local"` setzt. `setup-podman.sh` erstellt diese Datei, falls sie fehlt.
- **Rootless Podman schlägt für Benutzer openclaw fehl:** Überprüfen Sie, ob `/etc/subuid` und `/etc/subgid` eine Zeile für `openclaw` enthalten (z. B. `openclaw:100000:65536`). Fügen Sie sie hinzu, falls sie fehlt, und starten Sie neu.
- **Container-Name bereits in Verwendung:** Das Startskript verwendet `podman run --replace`, sodass der bestehende Container beim erneuten Start ersetzt wird. Zum manuellen Bereinigen: `podman rm -f openclaw`.
- **Skript beim Ausführen als openclaw nicht gefunden:** Stellen Sie sicher, dass `setup-podman.sh` ausgeführt wurde, sodass `run-openclaw-podman.sh` in das Home-Verzeichnis von openclaw kopiert wurde (z. B. `/home/openclaw/run-openclaw-podman.sh`).
- **Quadlet-Service nicht gefunden oder startet nicht:** Führen Sie nach dem Bearbeiten der `.container`-Datei `sudo systemctl --machine openclaw@ --user daemon-reload` aus. Quadlet erfordert cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` sollte `2` anzeigen.

## Optional: als eigener Benutzer ausführen

Um das Gateway als Ihr normaler Benutzer auszuführen (kein dedizierter openclaw-Benutzer): Erstellen Sie das Image, legen Sie `~/.openclaw/.env` mit `OPENCLAW_GATEWAY_TOKEN` an und starten Sie den Container mit `--userns=keep-id` sowie Mounts auf Ihr `~/.openclaw`. Das Startskript ist für den openclaw-Benutzer-Workflow ausgelegt; für ein Einzelbenutzer-Setup können Sie stattdessen den `podman run`-Befehl aus dem Skript manuell ausführen und dabei Konfiguration und Workspace auf Ihr Home-Verzeichnis verweisen lassen. Empfohlen für die meisten Benutzer: Verwenden Sie `setup-podman.sh` und führen Sie es als openclaw-Benutzer aus, sodass Konfiguration und Prozess isoliert sind.

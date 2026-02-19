---
summary: "Uruchom OpenClaw w kontenerze Podman w trybie rootless"
read_when:
  - Chcesz uruchomić gateway w kontenerze z użyciem Podman zamiast Docker
title: "Podman"
---

# Podman

Uruchom gateway OpenClaw w kontenerze Podman w trybie **rootless**. Używa tego samego obrazu co Docker (zbudowanego z repozytorium [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Wymagania

- Podman (rootless)
- Sudo do jednorazowej konfiguracji (utworzenie użytkownika, zbudowanie obrazu)

## Szybki start

**1. Jednorazowa konfiguracja** (z katalogu głównego repozytorium; tworzy użytkownika, buduje obraz, instaluje skrypt uruchamiania):

```bash
./setup-podman.sh
```

Tworzy to również minimalny plik `~openclaw/.openclaw/openclaw.json` (ustawia `gateway.mode="local"`), dzięki czemu gateway może uruchomić się bez przechodzenia przez kreator.

Domyślnie kontener **nie** jest instalowany jako usługa systemd — uruchamiasz go ręcznie (patrz poniżej). W środowisku produkcyjnym z automatycznym uruchamianiem i restartami zainstaluj go zamiast tego jako usługę użytkownika systemd Quadlet:

```bash
./setup-podman.sh --quadlet
```

(Lub ustaw `OPENCLAW_PODMAN_QUADLET=1`; użyj `--container`, aby zainstalować tylko kontener i skrypt uruchamiania.)

**2. Uruchom gateway** (ręcznie, do szybkiego testu smoke):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Kreator wdrożenia** (np. aby dodać kanały lub dostawców):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Następnie otwórz `http://127.0.0.1:18789/` i użyj tokenu z `~openclaw/.openclaw/.env` (lub wartości wyświetlonej przez setup).

## Systemd (Quadlet, opcjonalnie)

Jeśli uruchomiłeś `./setup-podman.sh --quadlet` (lub `OPENCLAW_PODMAN_QUADLET=1`), zostanie zainstalowana jednostka [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html), dzięki czemu gateway działa jako usługa użytkownika systemd dla użytkownika openclaw. Usługa jest włączana i uruchamiana na końcu konfiguracji.

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stop:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logi:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Plik quadlet znajduje się w `~openclaw/.config/containers/systemd/openclaw.container`. Aby zmienić porty lub zmienne środowiskowe, edytuj ten plik (lub plik `.env`, z którego korzysta), następnie wykonaj `sudo systemctl --machine openclaw@ --user daemon-reload` i uruchom ponownie usługę. Przy starcie systemu usługa uruchamia się automatycznie, jeśli dla użytkownika openclaw włączono lingering (setup robi to, gdy dostępne jest loginctl).

Aby dodać quadlet **po** początkowej konfiguracji, która go nie używała, uruchom ponownie: `./setup-podman.sh --quadlet`.

## Użytkownik openclaw (bez możliwości logowania)

`setup-podman.sh` tworzy dedykowanego użytkownika systemowego `openclaw`:

- **Shell:** `nologin` — brak interaktywnego logowania; zmniejsza powierzchnię ataku.

- **Home:** np. `/home/openclaw` — zawiera `~/.openclaw` (konfiguracja, workspace) oraz skrypt uruchamiający `run-openclaw-podman.sh`.

- **Rootless Podman:** Użytkownik musi mieć zakres **subuid** i **subgid**. Wiele dystrybucji przypisuje je automatycznie podczas tworzenia użytkownika. Jeśli setup wyświetli ostrzeżenie, dodaj linie do `/etc/subuid` i `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Następnie uruchom gateway jako ten użytkownik (np. z cron lub systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Konfiguracja:** Tylko `openclaw` i root mają dostęp do `/home/openclaw/.openclaw`. Aby edytować konfigurację: użyj Control UI po uruchomieniu gateway lub `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Środowisko i konfiguracja

- **Token:** Przechowywany w `~openclaw/.openclaw/.env` jako `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` i `run-openclaw-podman.sh` generują go, jeśli nie istnieje (używają `openssl`, `python3` lub `od`).
- **Opcjonalnie:** W tym pliku `.env` możesz ustawić klucze dostawców (np. `GROQ_API_KEY`, `OLLAMA_API_KEY`) oraz inne zmienne środowiskowe OpenClaw.
- **Porty hosta:** Domyślnie skrypt mapuje `18789` (gateway) i `18790` (bridge). Nadpisz mapowanie portów **hosta** za pomocą `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` i `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` podczas uruchamiania.
- **Ścieżki:** Konfiguracja hosta i workspace domyślnie wskazują na `~openclaw/.openclaw` oraz `~openclaw/.openclaw/workspace`. Nadpisz ścieżki hosta używane przez skrypt uruchamiający za pomocą `OPENCLAW_CONFIG_DIR` i `OPENCLAW_WORKSPACE_DIR`.

## Przydatne polecenia

- **Logi:** Z quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Ze skryptem: `sudo -u openclaw podman logs -f openclaw`
- **Zatrzymaj:** Z użyciem quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Z użyciem skryptu: `sudo -u openclaw podman stop openclaw`
- **Uruchom ponownie:** Z użyciem quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Z użyciem skryptu: uruchom ponownie skrypt startowy lub `podman start openclaw`
- **Usuń kontener:** `sudo -u openclaw podman rm -f openclaw` — konfiguracja i workspace na hoście zostaną zachowane

## Rozwiązywanie problemów

- **Permission denied (EACCES) dla config lub auth-profiles:** Kontener domyślnie używa `--userns=keep-id` i działa z tym samym uid/gid co użytkownik hosta uruchamiający skrypt. Upewnij się, że katalogi `OPENCLAW_CONFIG_DIR` i `OPENCLAW_WORKSPACE_DIR` na hoście należą do tego użytkownika.
- **Uruchomienie Gateway zablokowane (brak `gateway.mode=local`):** Upewnij się, że istnieje plik `~openclaw/.openclaw/openclaw.json` i zawiera ustawienie `gateway.mode="local"`. `setup-podman.sh` tworzy ten plik, jeśli go brakuje.
- **Rootless Podman nie działa dla użytkownika openclaw:** Sprawdź, czy `/etc/subuid` i `/etc/subgid` zawierają wpis dla `openclaw` (np. `openclaw:100000:65536`). Dodaj go, jeśli brakuje, i uruchom ponownie.
- **Nazwa kontenera jest już używana:** Skrypt uruchamiający korzysta z `podman run --replace`, więc istniejący kontener zostanie zastąpiony przy ponownym uruchomieniu. Aby wyczyścić ręcznie: `podman rm -f openclaw`.
- **Nie znaleziono skryptu podczas uruchamiania jako openclaw:** Upewnij się, że uruchomiono `setup-podman.sh`, aby `run-openclaw-podman.sh` został skopiowany do katalogu domowego openclaw (np. `/home/openclaw/run-openclaw-podman.sh`).
- **Usługa Quadlet nie została znaleziona lub nie uruchamia się:** Po edycji pliku `.container` uruchom `sudo systemctl --machine openclaw@ --user daemon-reload`. Quadlet wymaga cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` powinno zwrócić `2`.

## Opcjonalnie: uruchom jako własny użytkownik

Aby uruchomić gateway jako zwykły użytkownik (bez dedykowanego użytkownika openclaw): zbuduj obraz, utwórz `~/.openclaw/.env` z `OPENCLAW_GATEWAY_TOKEN`, a następnie uruchom kontener z `--userns=keep-id` i montowaniami do `~/.openclaw`. Skrypt uruchamiający jest przeznaczony dla przepływu z użytkownikiem openclaw; w konfiguracji jednoosobowej możesz zamiast tego ręcznie wykonać polecenie `podman run` ze skryptu, wskazując katalogi config i workspace w swoim katalogu domowym. Rekomendowane dla większości użytkowników: użyj `setup-podman.sh` i uruchamiaj jako użytkownik openclaw, aby odizolować konfigurację i proces.

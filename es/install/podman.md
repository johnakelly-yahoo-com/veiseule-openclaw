---
summary: "Ejecuta OpenClaw en un contenedor rootless de Podman"
read_when:
  - Quieres un gateway en contenedor con Podman en lugar de Docker
title: "Podman"
---

# Podman

Ejecuta el gateway de OpenClaw en un contenedor Podman **rootless**. Usa la misma imagen que Docker (compílala desde el [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) del repositorio).

## Requisitos

- Podman (rootless)
- Sudo para la configuración inicial (crear usuario, compilar imagen)

## Inicio rápido

**1. Configuración inicial** (desde la raíz del repositorio; crea el usuario, compila la imagen, instala el script de inicio):

```bash
./setup-podman.sh
```

Esto también crea un `~openclaw/.openclaw/openclaw.json` mínimo (establece `gateway.mode="local"`) para que el gateway pueda iniciarse sin ejecutar el asistente.

Por defecto, el contenedor **no** se instala como un servicio systemd; debes iniciarlo manualmente (ver abajo). Para una configuración de tipo producción con inicio automático y reinicios, instálalo como un servicio de usuario systemd Quadlet en su lugar:

```bash
./setup-podman.sh --quadlet
```

(O establece `OPENCLAW_PODMAN_QUADLET=1`; usa `--container` para instalar solo el contenedor y el script de inicio).

**2. Iniciar gateway** (manual, para una prueba rápida básica):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Asistente de configuración** (por ejemplo, para añadir canales o proveedores):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Luego abre `http://127.0.0.1:18789/` y usa el token de `~openclaw/.openclaw/.env` (o el valor mostrado por setup).

## Systemd (Quadlet, opcional)

Si ejecutaste `./setup-podman.sh --quadlet` (o `OPENCLAW_PODMAN_QUADLET=1`), se instala una unidad [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) para que el gateway se ejecute como un servicio de usuario systemd para el usuario openclaw. El servicio se habilita e inicia al final de la configuración.

- **Iniciar:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Detener:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Estado:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

El archivo quadlet se encuentra en `~openclaw/.config/containers/systemd/openclaw.container`. Para cambiar los puertos o las variables de entorno, edita ese archivo (o el `.env` que utiliza) y luego ejecuta `sudo systemctl --machine openclaw@ --user daemon-reload` y reinicia el servicio. Al arrancar, el servicio se inicia automáticamente si lingering está habilitado para openclaw (la configuración lo hace cuando loginctl está disponible).

Para añadir quadlet **después** de una configuración inicial que no lo utilizó, vuelve a ejecutar: `./setup-podman.sh --quadlet`.

## El usuario openclaw (sin inicio de sesión)

`setup-podman.sh` crea un usuario de sistema dedicado `openclaw`:

- **Shell:** `nologin` — sin inicio de sesión interactivo; reduce la superficie de ataque.

- **Home:** p. ej. `/home/openclaw` — contiene `~/.openclaw` (configuración, workspace) y el script de arranque `run-openclaw-podman.sh`.

- **Rootless Podman:** El usuario debe tener un rango de **subuid** y **subgid**. Muchas distribuciones los asignan automáticamente cuando se crea el usuario. Si la configuración muestra una advertencia, añade líneas a `/etc/subuid` y `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Luego inicia el gateway como ese usuario (p. ej. desde cron o systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** Solo `openclaw` y root pueden acceder a `/home/openclaw/.openclaw`. Para editar la configuración: usa la Control UI una vez que el gateway esté en ejecución, o `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Entorno y configuración

- **Token:** Se almacena en `~openclaw/.openclaw/.env` como `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` y `run-openclaw-podman.sh` lo generan si no existe (usa `openssl`, `python3` u `od`).
- **Opcional:** En ese archivo `.env` puedes establecer claves de proveedor (p. ej. `GROQ_API_KEY`, `OLLAMA_API_KEY`) y otras variables de entorno de OpenClaw.
- **Puertos del host:** Por defecto, el script asigna `18789` (gateway) y `18790` (bridge). Sobrescribe el mapeo de puertos del **host** con `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` y `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` al iniciar.
- **Rutas:** La configuración y el workspace en el host usan por defecto `~openclaw/.openclaw` y `~openclaw/.openclaw/workspace`. Sobrescribe las rutas del host utilizadas por el script de arranque con `OPENCLAW_CONFIG_DIR` y `OPENCLAW_WORKSPACE_DIR`.

## Comandos útiles

- **Logs:** Con quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Con script: `sudo -u openclaw podman logs -f openclaw`
- **Detener:** Con quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Con script: `sudo -u openclaw podman stop openclaw`
- **Iniciar de nuevo:** Con quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Con script: vuelve a ejecutar el script de arranque o `podman start openclaw`
- **Eliminar contenedor:** `sudo -u openclaw podman rm -f openclaw` — la configuración y el workspace en el host se conservan

## Solución de problemas

- **Permission denied (EACCES) en config o auth-profiles:** El contenedor usa por defecto `--userns=keep-id` y se ejecuta con el mismo uid/gid que el usuario del host que ejecuta el script. Asegúrate de que `OPENCLAW_CONFIG_DIR` y `OPENCLAW_WORKSPACE_DIR` en el host pertenezcan a ese usuario.
- **Inicio del gateway bloqueado (falta `gateway.mode=local`):** Asegúrate de que `~openclaw/.openclaw/openclaw.json` exista y establezca `gateway.mode="local"`. `setup-podman.sh` crea este archivo si no existe.
- **Rootless Podman falla para el usuario openclaw:** Comprueba que `/etc/subuid` y `/etc/subgid` contengan una línea para `openclaw` (p. ej. `openclaw:100000:65536`). Añádela si falta y reinicia.
- **Nombre de contenedor en uso:** El script de arranque usa `podman run --replace`, por lo que el contenedor existente se reemplaza al iniciar de nuevo. Para limpiar manualmente: `podman rm -f openclaw`.
- **Script no encontrado al ejecutarlo como openclaw:** Asegúrate de que se haya ejecutado `setup-podman.sh` para que `run-openclaw-podman.sh` se copie al home de openclaw (p. ej. `/home/openclaw/run-openclaw-podman.sh`).
- **Servicio de Quadlet no encontrado o no inicia:** Ejecuta `sudo systemctl --machine openclaw@ --user daemon-reload` después de editar el archivo `.container`. Quadlet requiere cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` debe mostrar `2`.

## Opcional: ejecutar como tu propio usuario

Para ejecutar el gateway como tu usuario normal (sin un usuario dedicado openclaw): crea la imagen, crea `~/.openclaw/.env` con `OPENCLAW_GATEWAY_TOKEN`, y ejecuta el contenedor con `--userns=keep-id` y montajes a tu `~/.openclaw`. El script de inicio está diseñado para el flujo con el usuario openclaw; para una configuración de un solo usuario puedes, en su lugar, ejecutar manualmente el comando `podman run` del script, apuntando la configuración y el workspace a tu directorio personal. Recomendado para la mayoría de los usuarios: usa `setup-podman.sh` y ejecútalo como el usuario openclaw para que la configuración y el proceso estén aislados.


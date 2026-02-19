---
summary: "Execute o OpenClaw em um contêiner Podman rootless"
read_when:
  - Você quer um gateway em contêiner com Podman em vez de Docker
title: "Podman"
---

# Podman

Execute o gateway OpenClaw em um contêiner Podman **rootless**. Usa a mesma imagem do Docker (build a partir do [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) do repositório).

## Requisitos

- Podman (rootless)
- Sudo para configuração única (criar usuário, gerar imagem)

## Início rápido

**1. Configuração única** (a partir da raiz do repositório; cria o usuário, gera a imagem e instala o script de inicialização):

```bash
./setup-podman.sh
```

Isso também cria um `~openclaw/.openclaw/openclaw.json` mínimo (define `gateway.mode="local"`) para que o gateway possa iniciar sem executar o assistente.

Por padrão, o contêiner **não** é instalado como um serviço systemd; você o inicia manualmente (veja abaixo). Para uma configuração no estilo produção com inicialização automática e reinícios, instale-o como um serviço de usuário systemd Quadlet:

```bash
./setup-podman.sh --quadlet
```

(Ou defina `OPENCLAW_PODMAN_QUADLET=1`; use `--container` para instalar apenas o contêiner e o script de inicialização.)

**2. Iniciar gateway** (manual, para um teste rápido):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Assistente de configuração** (por exemplo, para adicionar canais ou provedores):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Em seguida, abra `http://127.0.0.1:18789/` e use o token de `~openclaw/.openclaw/.env` (ou o valor exibido pela configuração).

## Systemd (Quadlet, opcional)

Se você executou `./setup-podman.sh --quadlet` (ou `OPENCLAW_PODMAN_QUADLET=1`), uma unidade [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) é instalada para que o gateway seja executado como um serviço de usuário systemd para o usuário openclaw. O serviço é habilitado e iniciado ao final da configuração.

- **Iniciar:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Parar:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

O arquivo quadlet está localizado em `~openclaw/.config/containers/systemd/openclaw.container`. Para alterar portas ou variáveis de ambiente, edite esse arquivo (ou o `.env` que ele referencia) e depois execute `sudo systemctl --machine openclaw@ --user daemon-reload` e reinicie o serviço. Na inicialização do sistema, o serviço inicia automaticamente se o lingering estiver habilitado para openclaw (a configuração faz isso quando loginctl está disponível).

Para adicionar o quadlet **após** uma configuração inicial que não o utilizou, execute novamente: `./setup-podman.sh --quadlet`.

## O usuário openclaw (sem login)

`setup-podman.sh` cria um usuário de sistema dedicado `openclaw`:

- **Shell:** `nologin` — sem login interativo; reduz a superfície de ataque.

- **Home:** ex.: `/home/openclaw` — contém `~/.openclaw` (configuração, workspace) e o script de inicialização `run-openclaw-podman.sh`.

- **Rootless Podman:** O usuário deve ter um intervalo de **subuid** e **subgid**. Muitas distros atribuem isso automaticamente quando o usuário é criado. Se a configuração exibir um aviso, adicione linhas a `/etc/subuid` e `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Em seguida, inicie o gateway como esse usuário (ex.: via cron ou systemd):```
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

  ```bash
  **Config:** Apenas `openclaw` e root podem acessar `/home/openclaw/.openclaw`.
  ```

- Para editar a configuração: use a Control UI quando o gateway estiver em execução ou `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`. Ambiente e configuração

## **Token:** Armazenado em `~openclaw/.openclaw/.env` como `OPENCLAW_GATEWAY_TOKEN`.

- `setup-podman.sh` e `run-openclaw-podman.sh` o geram se estiver ausente (usa `openssl`, `python3` ou `od`). **Opcional:** Nesse `.env` você pode definir chaves de provedores (ex.: `GROQ_API_KEY`, `OLLAMA_API_KEY`) e outras variáveis de ambiente do OpenClaw.
- **Portas do host:** Por padrão, o script mapeia `18789` (gateway) e `18790` (bridge).
- Substitua o mapeamento de portas do **host** com `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` e `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` ao iniciar. **Caminhos:** A configuração e o workspace no host usam por padrão `~openclaw/.openclaw` e `~openclaw/.openclaw/workspace`.
- Substitua os caminhos do host usados pelo script de inicialização com `OPENCLAW_CONFIG_DIR` e `OPENCLAW_WORKSPACE_DIR`. Comandos úteis

## **Logs:** Com quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`.

- Com script: `sudo -u openclaw podman logs -f openclaw` **Parar:** Com quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`.
- Com script: `sudo -u openclaw podman stop openclaw` **Iniciar novamente:** Com quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`.
- Com script: execute novamente o script de inicialização ou `podman start openclaw` **Remover contêiner:** `sudo -u openclaw podman rm -f openclaw` — a configuração e o workspace no host são mantidos
- Solução de problemas

## **Permission denied (EACCES) na configuração ou auth-profiles:** O contêiner usa por padrão `--userns=keep-id` e é executado com o mesmo uid/gid do usuário do host que executa o script.

- Certifique-se de que `OPENCLAW_CONFIG_DIR` e `OPENCLAW_WORKSPACE_DIR` no host pertençam a esse usuário. **Inicialização do gateway bloqueada (ausência de `gateway.mode=local`):** Certifique-se de que `~openclaw/.openclaw/openclaw.json` exista e defina `gateway.mode="local"`.
- `setup-podman.sh` cria esse arquivo se ele não existir. **Rootless Podman falha para o usuário openclaw:** Verifique se `/etc/subuid` e `/etc/subgid` contêm uma linha para `openclaw` (ex.: `openclaw:100000:65536`).
- Adicione se estiver ausente e reinicie. **Nome do contêiner já em uso:** O script de inicialização usa `podman run --replace`, então o contêiner existente é substituído ao iniciar novamente.
- Para limpar manualmente: `podman rm -f openclaw`. **Script não encontrado ao executar como openclaw:** Certifique-se de que `setup-podman.sh` foi executado para que `run-openclaw-podman.sh` seja copiado para o diretório home do openclaw (ex.: `/home/openclaw/run-openclaw-podman.sh`).
- **Serviço Quadlet não encontrado ou falha ao iniciar:** Execute `sudo systemctl --machine openclaw@ --user daemon-reload` após editar o arquivo `.container`.
- O Quadlet requer cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` deve mostrar `2`. Opcional: executar com seu próprio usuário

## Para executar o gateway como seu usuário normal (sem um usuário openclaw dedicado): construa a imagem, crie `~/.openclaw/.env` com `OPENCLAW_GATEWAY_TOKEN` e execute o contêiner com `--userns=keep-id` e montagens para seu `~/.openclaw`.

O script de inicialização foi projetado para o fluxo com usuário openclaw; para uma configuração de usuário único, você pode executar manualmente o comando `podman run` do script, apontando a configuração e o workspace para seu diretório home. The launch script is designed for the openclaw-user flow; for a single-user setup you can instead run the `podman run` command from the script manually, pointing config and workspace to your home. Recomendado para a maioria dos usuários: use `setup-podman.sh` e execute como o usuário openclaw para que a configuração e o processo fiquem isolados.

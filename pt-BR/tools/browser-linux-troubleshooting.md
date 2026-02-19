---
summary: "Corrija problemas de inicialização do CDP do Chrome/Brave/Edge/Chromium para o controle de navegador do OpenClaw no Linux"
read_when: "O controle de navegador falha no Linux, especialmente com o Chromium snap"
title: "Solução de problemas do navegador"
---

# Solução de problemas do navegador (Linux)

## Problema: "Failed to start Chrome CDP on port 18800"

O servidor de controle de navegador do OpenClaw não consegue iniciar Chrome/Brave/Edge/Chromium com o erro:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### Causa raiz

No Ubuntu (e em muitas distribuições Linux), a instalação padrão do Chromium é um **pacote snap**. O confinamento AppArmor do snap interfere na forma como o OpenClaw inicia e monitora o processo do navegador.

O comando `apt install chromium` instala um pacote stub que redireciona para o snap:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Este NÃO é um navegador real — é apenas um wrapper.

### Solução 1: Instalar o Google Chrome (Recomendado)

Instale o pacote oficial do Google Chrome `.deb`, que não é isolado por snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # if there are dependency errors
```

Em seguida, atualize sua configuração do OpenClaw (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### Solução 2: Usar o Chromium snap com o modo somente anexar

Se você precisar usar o Chromium snap, configure o OpenClaw para se anexar a um navegador iniciado manualmente:

1. Atualize a configuração:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. Inicie o Chromium manualmente:

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. Opcionalmente, crie um serviço systemd de usuário para iniciar o Chrome automaticamente:

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Habilite com: `systemctl --user enable --now openclaw-browser.service`

### Verificando se o navegador funciona

Verifique o status:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Teste a navegação:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### Referência de configuração

| Opção                    | Descrição                                                                                                | Padrão                                                                                    |
| ------------------------ | -------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `browser.enabled`        | Habilitar controle de navegador                                                                          | `true`                                                                                    |
| `browser.executablePath` | Caminho para um binário de navegador baseado em Chromium (Chrome/Brave/Edge/Chromium) | auto-detectado (prefere o navegador padrão quando baseado em Chromium) |
| `browser.headless`       | Executar sem GUI                                                                                         | `false`                                                                                   |
| `browser.noSandbox`      | Adicionar a flag `--no-sandbox` (necessária para algumas configurações Linux)         | `false`                                                                                   |
| `browser.attachOnly`     | Não iniciar o navegador, apenas anexar a um existente                                                    | `false`                                                                                   |
| `browser.cdpPort`        | Porta do Chrome DevTools Protocol                                                                        | `18800`                                                                                   |

### Problema: "Chrome extension relay is running, but no tab is connected"

Você está usando o perfil `chrome` (relay de extensão). Ele espera que a extensão de navegador do OpenClaw esteja anexada a uma aba ativa.

Opções de correção:

1. **Use o navegador gerenciado:** `openclaw browser start --browser-profile openclaw`
   (ou defina `browser.defaultProfile: "openclaw"`).
2. **Use o relay de extensão:** instale a extensão, abra uma aba e clique no
   ícone da extensão OpenClaw para anexá-la.

Notas:

- O perfil `chrome` usa o **navegador Chromium padrão do sistema** quando possível.
- Perfis locais `openclaw` atribuem automaticamente `cdpPort`/`cdpUrl`; defina-os apenas para CDP remoto.


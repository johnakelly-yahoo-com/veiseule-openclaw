---
summary: "Runbook para o serviço Gateway, ciclo de vida e operações"
read_when:
  - Ao executar ou depurar o processo do gateway
title: "Runbook do Gateway"
---

# Runbook do serviço Gateway

Use esta página para a inicialização do dia 1 e as operações do dia 2 do serviço Gateway.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Diagnósticos orientados por sintomas com sequências exatas de comandos e assinaturas de logs.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Guia de configuração orientado a tarefas + referência completa de configuração.
  
</Card>
</CardGroup>

## Inicialização local em 5 minutos

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Linha de base saudável: `Runtime: running` e `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
O recarregamento de configuração do Gateway monitora o caminho do arquivo de configuração ativo (resolvido a partir dos padrões de perfil/estado ou `OPENCLAW_CONFIG_PATH` quando definido).
Modo padrão: `gateway.reload.mode="hybrid"` (aplica mudanças seguras a quente, reinicia em críticas).
</Note>

## Modelo de runtime

- Um processo sempre ativo para roteamento, plano de controle e conexões de canal.
- Multiplexação em porta única.
  - Controle/RPC via WebSocket
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - Interface de controle e hooks
- Modo de bind padrão: `loopback`.
- A autenticação do Gateway é exigida por padrão: defina `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) ou `gateway.auth.password`.

### Precedência de porta e bind

| Configuração     | Ordem de resolução                                                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Porta do Gateway | Precedência de portas: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > padrão `18789`. |
| Modo de bind     | CLI/override → `gateway.bind` → `loopback`                                                                                   |

### Modos de hot reload

| Desative com `gateway.reload.mode="off"`. | Comportamento de keepalive                                 |
| --------------------------------------------------------- | ---------------------------------------------------------- |
| `off`                                                     | Sem recarregamento de configuração                         |
| `hot`                                                     | Aplicar apenas alterações seguras para hot reload          |
| `restart`                                                 | Reiniciar quando houver alterações que exigem reload       |
| `hybrid` (padrão)                      | Aplicar via hot quando seguro, reiniciar quando necessário |

## Conjunto de comandos do operador

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Acesso remoto

Tailscale/VPN é preferido; caso contrário, túnel SSH:
Fallback: túnel SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Os clientes então se conectam a `ws://127.0.0.1:18789` através do túnel.

<Warning>
Se um token estiver configurado, os clientes devem incluí-lo em `connect.params.auth.token` mesmo através do túnel.
</Warning>

Veja: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Supervisão e ciclo de vida do serviço

Use execuções supervisionadas para confiabilidade semelhante à de produção.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
Para reiniciar, use `openclaw gateway restart` (ou `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
```

Os rótulos do LaunchAgent são `ai.openclaw.gateway` (padrão) ou `ai.openclaw.<profile>` (perfil nomeado). `openclaw doctor` audita e corrige desvios na configuração do serviço.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Habilite lingering (necessário para que o serviço de usuário sobreviva a logout/idle):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Use uma unidade de sistema para hosts multiusuário/sempre ativos.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Múltiplos gateways (mesmo host)

Geralmente desnecessário: um Gateway pode atender a vários canais de mensagens e agentes.
Use múltiplos Gateways apenas para redundância ou isolamento rigoroso (ex: bot de resgate).

Checklist por instância:

- `gateway.port` exclusivo
- `OPENCLAW_CONFIG_PATH` exclusivo
- `OPENCLAW_STATE_DIR` exclusivo
- `agents.defaults.workspace` exclusivo

Exemplo:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Veja [Múltiplos gateways](/gateway/multiple-gateways).

### Perfil Dev (`--dev`)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

Os padrões incluem estado/configuração isolados e porta base do gateway `19001`.

## Protocolo (visão do operador)

- O primeiro frame do cliente deve ser `connect`.
- O Gateway retorna um snapshot `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy).
- Requisições: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Eventos comuns: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

As execuções do agente têm duas etapas:

1. Ack de aceitação imediato (`status:"accepted"`)
2. As respostas `agent` são em duas etapas: primeiro `res` ack `{runId,status:"accepted"}`, depois um `res` `{runId,status:"ok"|"error",summary}` final após o término da execução; a saída em streaming chega como `event:"agent"`.

Documentação completa: [Protocolo do Gateway](/gateway/protocol) e [Protocolo Bridge (legado)](/gateway/bridge-protocol).

## Verificações operacionais

### Vitalidade

- Abra o WS e envie `connect`.
- Espere a resposta `hello-ok` com o snapshot.

### Prontidão

```bash
`openclaw gateway health|status` — solicita saúde/status via WS do Gateway.
```

### Recuperação de lacunas

Os eventos não são reproduzidos. Em caso de lacunas na sequência, atualize o estado (`health`, `system-presence`) antes de continuar.

## Assinaturas comuns de falha

| Assinatura                                                     | Problema provável                                         |
| -------------------------------------------------------------- | --------------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Bind fora de loopback sem token/senha                     |
| `another gateway instance is already listening` / `EADDRINUSE` | Conflito de porta                                         |
| `Gateway start blocked: set gateway.mode=local`                | Configuração definida para modo remoto                    |
| `unauthorized` durante a conexão                               | Incompatibilidade de autenticação entre cliente e gateway |

Para escadas completas de diagnóstico, use [Gateway Troubleshooting](/gateway/troubleshooting).

## Garantias de segurança

- Clientes do protocolo Gateway falham rapidamente quando o Gateway está indisponível (sem fallback implícito para canal direto).
- Frames iniciais não-connect ou JSON malformado são rejeitados e o socket é fechado.
- Desligamento gracioso: emite evento `shutdown` antes de fechar; os clientes devem lidar com fechamento + reconexão.

---

Relacionado:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

---
status: experimental
title: "Grupos de Broadcast"
---

# Grupos de Broadcast

**Status:** Experimental  
**Versão:** Adicionado na 2026.1.9

## Visão geral

Os Grupos de Broadcast permitem que vários agentes processem e respondam à mesma mensagem simultaneamente. Isso permite criar equipes de agentes especializados que trabalham juntos em um único grupo do WhatsApp ou DM — tudo usando um único número de telefone.

Escopo atual: **somente WhatsApp** (canal web).

Os grupos de broadcast são avaliados após as listas de permissões do canal e as regras de ativação de grupo. Em grupos do WhatsApp, isso significa que os broadcasts acontecem quando o OpenClaw normalmente responderia (por exemplo: em menção, dependendo das configurações do seu grupo).

## Casos de uso

### 1. Equipes de agentes especializados

Implante vários agentes com responsabilidades atômicas e focadas:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Cada agente processa a mesma mensagem e fornece sua perspectiva especializada.

### 2. Suporte multilíngue

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. Fluxos de trabalho de garantia de qualidade

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. Automação de tarefas

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## Configuração

### Configuração básica

Adicione uma seção de nível superior `broadcast` (ao lado de `bindings`). As chaves são IDs de pares do WhatsApp:

- chats em grupo: JID do grupo (por exemplo, `120363403215116621@g.us`)
- DMs: número de telefone E.164 (por exemplo, `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Resultado:** Quando o OpenClaw for responder neste chat, ele executará os três agentes.

### Estratégia de processamento

Controle como os agentes processam mensagens:

#### Paralelo (Padrão)

Todos os agentes processam simultaneamente:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Sequencial

Os agentes processam em ordem (um aguarda o término do anterior):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Exemplo completo

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Como funciona

### Fluxo de mensagens

1. **Mensagem recebida** chega em um grupo do WhatsApp
2. **Verificação de broadcast**: o sistema verifica se o ID do par está em `broadcast`
3. **Se estiver na lista de broadcast**:
   - Todos os agentes listados processam a mensagem
   - Cada agente tem sua própria chave de sessão e contexto isolado
   - Os agentes processam em paralelo (padrão) ou sequencialmente
4. **Se não estiver na lista de broadcast**:
   - Aplica-se o roteamento normal (primeiro vínculo correspondente)

Nota: os grupos de broadcast não contornam as listas de permissões do canal nem as regras de ativação de grupo (menções/comandos/etc.). Eles apenas mudam _quais agentes são executados_ quando uma mensagem é elegível para processamento.

### Isolamento de sessão

Cada agente em um grupo de broadcast mantém completamente separado:

- **Chaves de sessão** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
- **Histórico de conversa** (o agente não vê as mensagens de outros agentes)
- **Workspace** (sandboxes separadas, se configurado)
- **Acesso a ferramentas** (listas diferentes de permitir/negar)
- **Memória/contexto** (IDENTITY.md, SOUL.md, etc. separados)
- **Buffer de contexto do grupo** (mensagens recentes do grupo usadas para contexto) é compartilhado por par, então todos os agentes de broadcast veem o mesmo contexto quando acionados

Isso permite que cada agente tenha:

- Personalidades diferentes
- Acesso a ferramentas diferentes (por exemplo, somente leitura vs. leitura e escrita)
- Modelos diferentes (por exemplo, opus vs. sonnet)
- Skills diferentes instaladas

### Exemplo: Sessões isoladas

No grupo `120363403215116621@g.us` com os agentes `["alfred", "baerbel"]`:

**Contexto do Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Contexto da Bärbel:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## Boas práticas

### 1. Mantenha os agentes focados

Projete cada agente com uma única responsabilidade clara:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Bom:** Cada agente tem uma função  
❌ **Ruim:** Um agente genérico "dev-helper"

### 2. Use nomes descritivos

Deixe claro o que cada agente faz:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Configure acessos a ferramentas diferentes

Dê aos agentes apenas as ferramentas de que precisam:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. Monitore o desempenho

Com muitos agentes, considere:

- Usar `"strategy": "parallel"` (padrão) para velocidade
- Limitar grupos de broadcast a 5–10 agentes
- Usar modelos mais rápidos para agentes mais simples

### 5. Trate falhas de forma elegante

Os agentes falham de forma independente. O erro de um agente não bloqueia os outros:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Compatibilidade

### Provedores

Os grupos de broadcast atualmente funcionam com:

- ✅ WhatsApp (implementado)
- 🚧 Telegram (planejado)
- 🚧 Discord (planejado)
- 🚧 Slack (planejado)

### Roteamento

Os grupos de broadcast funcionam junto com o roteamento existente:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: Apenas alfred responde (roteamento normal)
- `GROUP_B`: agent1 E agent2 respondem (broadcast)

**Precedência:** `broadcast` tem prioridade sobre `bindings`.

## Solução de problemas

### Agentes não respondendo

**Verifique:**

1. Os IDs dos agentes existem em `agents.list`
2. O formato do ID do par está correto (por exemplo, `120363403215116621@g.us`)
3. Os agentes não estão em listas de negação

**Depuração:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Apenas um agente respondendo

**Causa:** O ID do par pode estar em `bindings`, mas não em `broadcast`.

**Correção:** Adicione à configuração de broadcast ou remova dos vínculos.

### Problemas de desempenho

**Se estiver lento com muitos agentes:**

- Reduza o número de agentes por grupo
- Use modelos mais leves (sonnet em vez de opus)
- Verifique o tempo de inicialização do sandbox

## Exemplos

### Exemplo 1: Equipe de revisão de código

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Usuário envia:** Trecho de código  
**Respostas:**

- code-formatter: "Corrigi a indentação e adicionei dicas de tipo"
- security-scanner: "⚠️ Vulnerabilidade de injeção de SQL na linha 12"
- test-coverage: "A cobertura é de 45%, faltam testes para casos de erro"
- docs-checker: "Falta docstring para a função `process_data`"

### Exemplo 2: Suporte multilíngue

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## Referência da API

### Esquema de configuração

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Campos

- `strategy` (opcional): Como processar os agentes
  - `"parallel"` (padrão): Todos os agentes processam simultaneamente
  - `"sequential"`: Os agentes processam na ordem do array
- `[peerId]`: JID de grupo do WhatsApp, número E.164 ou outro ID de par
  - Valor: Array de IDs de agentes que devem processar mensagens

## Limitações

1. **Máx. de agentes:** Não há limite rígido, mas 10+ agentes podem ser lentos
2. **Contexto compartilhado:** Os agentes não veem as respostas uns dos outros (por design)
3. **Ordenação de mensagens:** Respostas paralelas podem chegar em qualquer ordem
4. **Limites de taxa:** Todos os agentes contam para os limites de taxa do WhatsApp

## Melhorias futuras

Recursos planejados:

- [ ] Modo de contexto compartilhado (agentes veem as respostas uns dos outros)
- [ ] Coordenação de agentes (agentes podem sinalizar uns aos outros)
- [ ] Seleção dinâmica de agentes (escolher agentes com base no conteúdo da mensagem)
- [ ] Prioridades de agentes (alguns agentes respondem antes de outros)

## Veja também

- [Configuração de múltiplos agentes](/tools/multi-agent-sandbox-tools)
- [Configuração de roteamento](/channels/channel-routing)
- [Gerenciamento de sessões](/concepts/sessions)



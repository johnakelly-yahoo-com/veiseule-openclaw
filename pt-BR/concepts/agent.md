---
summary: "Runtime do agente (pi-mono incorporado), contrato do workspace e bootstrap de sessão"
read_when:
  - Ao alterar o runtime do agente, o bootstrap do workspace ou o comportamento da sessão
title: "Runtime do Agente"
---

# Runtime do Agente 🤖

O OpenClaw executa um único runtime de agente incorporado derivado do **pi-mono**.

## Workspace (obrigatório)

O OpenClaw usa um único diretório de workspace do agente (`agents.defaults.workspace`) como o **único** diretório de trabalho (`cwd`) do agente para ferramentas e contexto.

Recomendado: use `openclaw setup` para criar `~/.openclaw/openclaw.json` se estiver ausente e inicializar os arquivos do workspace.

Layout completo do workspace + guia de backup: [Workspace do agente](/concepts/agent-workspace)

Se `agents.defaults.sandbox` estiver habilitado, sessões não principais podem substituir isso com
workspaces por sessão em `agents.defaults.sandbox.workspaceRoot` (veja
[Configuração do Gateway](/gateway/configuration)).

## Arquivos de bootstrap (injetados)

Dentro de `agents.defaults.workspace`, o OpenClaw espera estes arquivos editáveis pelo usuário:

- `AGENTS.md` — instruções operacionais + “memória”
- `SOUL.md` — persona, limites, tom
- `TOOLS.md` — notas de ferramentas mantidas pelo usuário (ex.: `imsg`, `sag`, convenções)
- `BOOTSTRAP.md` — ritual único da primeira execução (excluído após a conclusão)
- `IDENTITY.md` — nome/vibe/emoji do agente
- `USER.md` — perfil do usuário + forma de tratamento preferida

No primeiro turno de uma nova sessão, o OpenClaw injeta o conteúdo desses arquivos diretamente no contexto do agente.

Arquivos em branco são ignorados. Arquivos grandes são aparados e truncados com um marcador para manter os prompts enxutos (leia o arquivo para o conteúdo completo).

Se um arquivo estiver ausente, o OpenClaw injeta uma única linha de marcador de “arquivo ausente” (e `openclaw setup` criará um template padrão seguro).

`BOOTSTRAP.md` é criado apenas para um **workspace totalmente novo** (nenhum outro arquivo de bootstrap presente). Se você excluí-lo após concluir o ritual, ele não deve ser recriado em reinicializações posteriores.

Para desativar completamente a criação de arquivos de bootstrap (para workspaces pré-semeados), defina:

```json5
{ agent: { skipBootstrap: true } }
```

## Ferramentas integradas

As ferramentas centrais (read/exec/edit/write e ferramentas de sistema relacionadas) estão sempre disponíveis,
sujeitas à política de ferramentas. `apply_patch` é opcional e condicionado por
`tools.exec.applyPatch`. `TOOLS.md` **não** controla quais ferramentas existem; é
orientação sobre como _você_ deseja que elas sejam usadas.

## Habilidades

O OpenClaw carrega skills de três locais (o workspace vence em conflitos de nome):

- Empacotadas (enviadas com a instalação)
- Gerenciadas/locais: `~/.openclaw/skills`
- Espaço de trabalho: `<workspace>/skills`

As skills podem ser condicionadas por configuração/variáveis de ambiente (veja `skills` em [Configuração do Gateway](/gateway/configuration)).

## Integração com pi-mono

O OpenClaw reutiliza partes do código do pi-mono (modelos/ferramentas), mas **o gerenciamento de sessões, a descoberta e a ligação de ferramentas são de propriedade do OpenClaw**.

- Nenhum runtime de agente pi-coding.
- Nenhuma configuração `~/.pi/agent` ou `<workspace>/.pi` é consultada.

## Sessões

As transcrições das sessões são armazenadas como JSONL em:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

O ID da sessão é estável e escolhido pelo OpenClaw.
Pastas de sessão legadas do Pi/Tau **não** são lidas.

## Direcionamento durante o streaming

Quando o modo de fila é `steer`, mensagens de entrada são injetadas na execução atual.
A fila é verificada **após cada chamada de ferramenta**; se houver uma mensagem na fila,
as chamadas de ferramenta restantes da mensagem atual do assistente são ignoradas (resultados de ferramenta com erro "Skipped due to queued user message."), e então a mensagem do usuário em fila é injetada antes da próxima resposta do assistente.

Quando o modo de fila é `followup` ou `collect`, mensagens de entrada são retidas até que o
turno atual termine; então um novo turno do agente começa com as cargas em fila. Veja
[Fila](/concepts/queue) para modo + comportamento de debounce/cap.

O streaming em blocos envia blocos concluídos do assistente assim que terminam; ele vem
**desativado por padrão** (`agents.defaults.blockStreamingDefault: "off"`).
Ajuste o limite via `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; padrão: text_end).
Controle o fracionamento suave de blocos com `agents.defaults.blockStreamingChunk` (padrão
800–1200 caracteres; prefere quebras de parágrafo, depois novas linhas; frases por último).
Agrupe chunks transmitidos com `agents.defaults.blockStreamingCoalesce` para reduzir
spam de linhas únicas (mesclagem baseada em inatividade antes do envio). Canais que não sejam Telegram exigem
`*.blockStreaming: true` explícito para habilitar respostas em bloco.
Resumos verbosos de ferramentas são emitidos no início da ferramenta (sem debounce); a UI de Controle
transmite a saída da ferramenta via eventos do agente quando disponível.
Mais detalhes: [Streaming + chunking](/concepts/streaming).

## Referências de modelo

As referências de modelo na configuração (por exemplo `agents.defaults.model` e `agents.defaults.models`) são analisadas dividindo no **primeiro** `/`.

- Use `provider/model` ao configurar modelos.
- Se o próprio ID do modelo contiver `/` (estilo OpenRouter), inclua o prefixo do provedor (exemplo: `openrouter/moonshotai/kimi-k2`).
- Se você omitir o provedor, o OpenClaw trata a entrada como um alias ou um modelo para o **provedor padrão** (só funciona quando não há `/` no ID do modelo).

## Configuração (mínima)

No mínimo, defina:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (fortemente recomendado)

---

_Próximo: [Conversas em Grupo](/channels/group-messages)_ 🦞

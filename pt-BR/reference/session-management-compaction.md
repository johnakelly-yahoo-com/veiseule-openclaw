---
summary: "Análise aprofundada: armazenamento de sessões + transcrições, ciclo de vida e detalhes internos de (auto)compactação"
read_when:
  - Você precisa depurar ids de sessão, JSONL de transcrições ou campos de sessions.json
  - Você está alterando o comportamento de auto-compactação ou adicionando rotinas de organização “pré-compactação”
  - Você quer implementar despejos de memória ou turnos silenciosos do sistema
title: "Gerenciamento de sessão profunda mergulho"
---

# Gerenciamento de Sessões & Compactação (Análise aprofundada)

Este documento explica como o OpenClaw gerencia sessões de ponta a ponta:

- **Roteamento de sessão** (como mensagens de entrada mapeiam para um `sessionKey`)
- **Loja de sessão** (`sessions.json`) e o que ele controla
- **Persistência de transcrições** (`*.jsonl`) e sua estrutura
- **Higiene de transcrições** (ajustes específicos do provedor antes das execuções)
- **Limites de contexto** (janela de contexto vs. tokens acompanhados)
- **Compactação** (manual + auto-compactação) e onde conectar trabalhos de pré-compactação
- **Organização silenciosa** (ex.: gravações de memória que não devem produzir saída visível ao usuário)

Se você quiser uma visão geral de nível mais alto primeiro, comece por:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## Fonte da verdade: o Gateway

O OpenClaw foi projetado em torno de um único **processo Gateway** que é o dono do estado das sessões.

- UIs (app macOS, UI de Controle web, TUI) devem consultar o Gateway para listas de sessões e contagens de tokens.
- Em modo remoto, os arquivos de sessão ficam no host remoto; “verificar seus arquivos locais do Mac” não reflete o que o Gateway está usando.

---

## Duas camadas de persistência

O OpenClaw persiste sessões em duas camadas:

1. **Armazenamento de sessões (`sessions.json`)**
   - Mapa chave/valor: `sessionKey -> SessionEntry`
   - Pequeno, mutável, seguro para editar (ou excluir entradas)
   - Acompanha metadados da sessão (id da sessão atual, última atividade, alternâncias, contadores de tokens etc.)

2. **Transcrição (`<sessionId>.jsonl`)**
   - Transcrição apenas de acréscimo com estrutura em árvore (entradas têm `id` + `parentId`)
   - Armazena a conversa real + chamadas de ferramentas + resumos de compactação
   - Usada para reconstruir o contexto do modelo em turnos futuros

---

## Localizações em disco

Por agente, no host do Gateway:

- Armazenamento: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transcrições: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Sessões de tópico do Telegram: `.../<sessionId>-topic-<threadId>.jsonl`

O OpenClaw resolve isso via `src/config/sessions.ts`.

---

## Chaves de sessão (`sessionKey`)

Uma `sessionKey` identifica _em qual bucket de conversa_ você está (roteamento + isolamento).

Padrões comuns:

- Chat principal/direto (por agente): `agent:<agentId>:<mainKey>` (padrão `main`)
- Grupo: `agent:<agentId>:<channel>:group:<id>`
- Sala/canal (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` ou `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (a menos que seja substituído)

As regras canônicas estão documentadas em [/concepts/session](/concepts/session).

---

## IDs de sessão (`sessionId`)

Cada `sessionKey` aponta para um `sessionId` atual (o arquivo de transcrição que continua a conversa).

Regras gerais:

- **Reset** (`/new`, `/reset`) cria um novo `sessionId` para essa `sessionKey`.
- **Reset diário** (padrão 4:00 da manhã no horário local do host do gateway) cria um novo `sessionId` na próxima mensagem após o limite do reset.
- **Expiração por inatividade** (`session.reset.idleMinutes` ou legado `session.idleMinutes`) cria um novo `sessionId` quando uma mensagem chega após a janela de inatividade. Quando diário + inatividade estão ambos configurados, o que expirar primeiro prevalece.

Detalhe de implementação: a decisão acontece em `initSessionState()` em `src/auto-reply/reply/session.ts`.

---

## Esquema do armazenamento de sessões (`sessions.json`)

O tipo de valor do armazenamento é `SessionEntry` em `src/config/sessions.ts`.

Campos principais (não exaustivo):

- `sessionId`: id da transcrição atual (o nome do arquivo é derivado disso, a menos que `sessionFile` esteja definido)
- `updatedAt`: timestamp da última atividade
- `sessionFile`: substituição opcional explícita do caminho da transcrição
- `chatType`: `direct | group | room` (ajuda UIs e a política de envio)
- `provider`, `subject`, `room`, `space`, `displayName`: metadados para rotulagem de grupo/canal
- Alternâncias:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (substituição por sessão)
- Seleção de modelo:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Contadores de tokens (melhor esforço / dependente do provedor):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: com que frequência a auto-compactação foi concluída para esta chave de sessão
- `memoryFlushAt`: timestamp do último despejo de memória pré-compactação
- `memoryFlushCompactionCount`: contagem de compactações quando o último despejo foi executado

O armazenamento é seguro para edição, mas o Gateway é a autoridade: ele pode reescrever ou reidratar entradas conforme as sessões são executadas.

---

## Estrutura da transcrição (`*.jsonl`)

As transcrições são gerenciadas pelo `SessionManager` do `@mariozechner/pi-coding-agent`.

O arquivo é JSONL:

- Primeira linha: cabeçalho da sessão (`type: "session"`, inclui `id`, `cwd`, `timestamp`, opcional `parentSession`)
- Em seguida: entradas da sessão com `id` + `parentId` (árvore)

Tipos de entrada notáveis:

- `message`: mensagens de usuário/assistente/toolResult
- `custom_message`: mensagens injetadas por extensões que _entram_ no contexto do modelo (podem ser ocultadas da UI)
- `custom`: estado de extensão que _não_ entra no contexto do modelo
- `compaction`: resumo de compactação persistido com `firstKeptEntryId` e `tokensBefore`
- `branch_summary`: resumo persistido ao navegar por um ramo da árvore

O OpenClaw intencionalmente **não** “corrige” transcrições; o Gateway usa `SessionManager` para lê-las/escrevê-las.

---

## Janelas de contexto vs. tokens acompanhados

Dois conceitos diferentes importam:

1. **Janela de contexto do modelo**: limite rígido por modelo (tokens visíveis ao modelo)
2. **Contadores do armazenamento de sessões**: estatísticas contínuas gravadas em `sessions.json` (usadas para /status e dashboards)

Se você estiver ajustando limites:

- A janela de contexto vem do catálogo de modelos (e pode ser substituída via configuração).
- `contextTokens` no armazenamento é um valor de estimativa/relato em tempo de execução; não o trate como uma garantia estrita.

Para mais detalhes, veja [/token-use](/reference/token-use).

---

## Compactação: o que é

A compactação resume conversas mais antigas em uma entrada `compaction` persistida na transcrição e mantém mensagens recentes intactas.

Após a compactação, turnos futuros veem:

- O resumo de compactação
- Mensagens após `firstKeptEntryId`

A compactação é **persistente** (ao contrário da poda de sessões). Veja [/concepts/session-pruning](/concepts/session-pruning).

---

## Quando a auto-compactação acontece (runtime do Pi)

No agente Pi incorporado, a auto-compactação é acionada em dois casos:

1. **Recuperação de overflow**: o modelo retorna um erro de overflow de contexto → compacta → tenta novamente.
2. **Manutenção por limiar**: após um turno bem-sucedido, quando:

`contextTokens > contextWindow - reserveTokens`

Onde:

- `contextWindow` é a janela de contexto do modelo
- `reserveTokens` é a folga reservada para prompts + a próxima saída do modelo

Essas são semânticas do runtime do Pi (o OpenClaw consome os eventos, mas o Pi decide quando compactar).

---

## Configurações de compactação (`reserveTokens`, `keepRecentTokens`)

As configurações de compactação do Pi ficam nas configurações do Pi:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

O OpenClaw também impõe um piso de segurança para execuções incorporadas:

- Se `compaction.reserveTokens < reserveTokensFloor`, o OpenClaw o aumenta.
- O piso padrão é `20000` tokens.
- Defina `agents.defaults.compaction.reserveTokensFloor: 0` para desativar o piso.
- Se já estiver mais alto, o OpenClaw não altera.

Por quê: deixar folga suficiente para “organização” multi-turno (como gravações de memória) antes que a compactação se torne inevitável.

Implementação: `ensurePiCompactionReserveTokens()` em `src/agents/pi-settings.ts`
(chamado a partir de `src/agents/pi-embedded-runner.ts`).

---

## Superfícies visíveis ao usuário

Você pode observar a compactação e o estado da sessão via:

- `/status` (em qualquer sessão de chat)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Modo verboso: `🧹 Auto-compaction complete` + contagem de compactações

---

## Organização silenciosa (`NO_REPLY`)

O OpenClaw suporta turnos “silenciosos” para tarefas em segundo plano nas quais o usuário não deve ver saída intermediária.

Convenção:

- O assistente inicia sua saída com `NO_REPLY` para indicar “não entregar uma resposta ao usuário”.
- O OpenClaw remove/suprime isso na camada de entrega.

A partir de `2026.1.10`, o OpenClaw também suprime **streaming de rascunho/digitação** quando um trecho parcial começa com `NO_REPLY`, para que operações silenciosas não vazem saída parcial no meio do turno.

---

## “Despejo de memória” pré-compactação (implementado)

Objetivo: antes que a auto-compactação aconteça, executar um turno agentivo silencioso que grave
estado durável em disco (ex.: `memory/YYYY-MM-DD.md` no workspace do agente) para que a compactação não
apague contexto crítico.

O OpenClaw usa a abordagem de **despejo pré-limiar**:

1. Monitorar o uso de contexto da sessão.
2. Quando cruzar um “limiar suave” (abaixo do limiar de compactação do Pi), executar uma diretiva silenciosa
   de “gravar memória agora” para o agente.
3. Usar `NO_REPLY` para que o usuário não veja nada.

Configuração (`agents.defaults.compaction.memoryFlush`):

- `enabled` (padrão: `true`)
- `softThresholdTokens` (padrão: `4000`)
- `prompt` (mensagem do usuário para o turno de despejo)
- `systemPrompt` (prompt de sistema extra anexado para o turno de despejo)

Notas:

- O prompt padrão/prompt de sistema incluem uma dica `NO_REPLY` para suprimir a entrega.
- O despejo é executado uma vez por ciclo de compactação (acompanhado em `sessions.json`).
- O despejo roda apenas para sessões Pi incorporadas (backends de CLI o ignoram).
- O despejo é ignorado quando o workspace da sessão é somente leitura (`workspaceAccess: "ro"` ou `"none"`).
- Veja [Memory](/concepts/memory) para o layout de arquivos do workspace e padrões de escrita.

O Pi também expõe um hook `session_before_compact` na API de extensões, mas a lógica de despejo do OpenClaw
vive hoje no lado do Gateway.

---

## Checklist de solução de problemas

- Chave de sessão errada? Comece por [/concepts/session](/concepts/session) e confirme o `sessionKey` em `/status`.
- Divergência entre store e transcrição? Confirme o host do Gateway e o caminho do store a partir de `openclaw status`.
- Spam de compactação? Verifique:
  - janela de contexto do modelo (muito pequena)
  - configurações de compactação (`reserveTokens` muito alto para a janela do modelo pode causar compactação mais cedo)
  - inchaço de resultados de ferramentas: habilite/ajuste a poda de sessões
- Turnos silenciosos vazando? Confirme que a resposta começa com `NO_REPLY` (token exato) e que você está em um build que inclui a correção de supressão de streaming.

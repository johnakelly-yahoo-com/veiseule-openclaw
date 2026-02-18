---
title: "Compactação"
---

# Janela de Contexto & Compactação

Todo modelo tem uma **janela de contexto** (máximo de tokens que ele consegue ver). Chats de longa duração acumulam mensagens e resultados de ferramentas; quando a janela fica apertada, o OpenClaw **compacta** o histórico mais antigo para permanecer dentro dos limites.

## O que é compactação

A compactação **resume conversas mais antigas** em uma entrada de resumo compacta e mantém as mensagens recentes intactas. O resumo é armazenado no histórico da sessão, de modo que requisições futuras usam:

- O resumo da compactação
- Mensagens recentes após o ponto de compactação

A compactação **persiste** no histórico JSONL da sessão.

## Configuração

Veja [Configuração e modos de compactação](/concepts/compaction) para as configurações `agents.defaults.compaction`.

## Compactação automática (ativada por padrão)

Quando uma sessão se aproxima ou excede a janela de contexto do modelo, o OpenClaw aciona a compactação automática e pode tentar novamente a requisição original usando o contexto compactado.

Você verá:

- `🧹 Auto-compaction complete` no modo verboso
- `/status` mostrando `🧹 Compactions: <count>`

Antes da compactação, o OpenClaw pode executar um turno **silencioso de descarte de memória** para armazenar notas duráveis em disco. Veja [Memory](/concepts/memory) para detalhes e configuração.

## Compactação manual

Use `/compact` (opcionalmente com instruções) para forçar uma passagem de compactação:

```
/compact Focus on decisions and open questions
```

## Origem da janela de contexto

A janela de contexto é específica do modelo. O OpenClaw usa a definição do modelo do catálogo do provedor configurado para determinar os limites.

## Compactação vs poda

- **Compactação**: resume e **persiste** em JSONL.
- **Poda de sessão**: remove apenas **resultados de ferramentas** antigos, **em memória**, por requisição.

Veja [/concepts/session-pruning](/concepts/session-pruning) para detalhes sobre poda.

## Dicas

- Use `/compact` quando as sessões parecerem obsoletas ou o contexto estiver inchado.
- Grandes saídas de ferramentas já são truncadas; a poda pode reduzir ainda mais o acúmulo de resultados de ferramentas.
- Se você precisa de uma página em branco, `/new` ou `/reset` inicia um novo id de sessão.



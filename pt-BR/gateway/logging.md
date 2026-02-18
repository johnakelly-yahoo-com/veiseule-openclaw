---
summary: "Superfícies de logging, logs em arquivo, estilos de log WS e formatação do console"
read_when:
  - Ao alterar a saída ou os formatos de logging
  - Ao depurar a saída da CLI ou do gateway
title: "Registro de logs"
---

# Registro de logs

Para uma visão geral voltada ao usuário (CLI + Control UI + configuração), veja [/logging](/logging).

O OpenClaw tem duas “superfícies” de log:

- **Saída do console** (o que você vê no terminal / Debug UI).
- **Logs em arquivo** (linhas JSON) gravados pelo logger do gateway.

## Logger baseado em arquivo

- O arquivo de log rotativo padrão fica em `/tmp/openclaw/` (um arquivo por dia): `openclaw-YYYY-MM-DD.log`
  - A data usa o fuso horário local do host do Gateway.
- O caminho do arquivo de log e o nível podem ser configurados via `~/.openclaw/openclaw.json`:
  - `logging.file`
  - `logging.level`

O formato do arquivo é um objeto JSON por linha.

A aba Logs da Control UI faz o _tail_ desse arquivo via o gateway (`logs.tail`).
A CLI pode fazer o mesmo:

```bash
openclaw logs --follow
```

**Verbose vs. níveis de log**

- **Logs em arquivo** são controlados exclusivamente por `logging.level`.
- `--verbose` afeta apenas a **verbosidade do console** (e o estilo de log WS); **não**
  aumenta o nível de log em arquivo.
- Para capturar detalhes apenas verbosos nos logs em arquivo, defina `logging.level` como `debug` ou
  `trace`.

## Captura do console

A CLI captura `console.log/info/warn/error/debug/trace` e os grava nos logs em arquivo,
enquanto ainda imprime em stdout/stderr.

Você pode ajustar a verbosidade do console de forma independente via:

- `logging.consoleLevel` (padrão `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)

## Redação de resumo de ferramentas

Resumos verbosos de ferramentas (por exemplo, `🛠️ Exec: ...`) podem mascarar tokens sensíveis antes de chegarem ao
stream do console. Isso é **apenas para ferramentas** e não altera os logs em arquivo.

- `logging.redactSensitive`: `off` | `tools` (padrão: `tools`)
- `logging.redactPatterns`: array de strings regex (substitui os padrões)
  - Use strings regex brutas (auto `gi`), ou `/pattern/flags` se precisar de flags personalizadas.
  - Correspondências são mascaradas mantendo os primeiros 6 + últimos 4 caracteres (comprimento >= 18); caso contrário `***`.
  - Os padrões cobrem atribuições comuns de chaves, flags de CLI, campos JSON, cabeçalhos bearer, blocos PEM e prefixos populares de tokens.

## Logs WebSocket do Gateway

O gateway imprime logs do protocolo WebSocket em dois modos:

- **Modo normal (sem `--verbose`)**: apenas resultados RPC “interessantes” são impressos:
  - erros (`ok=false`)
  - chamadas lentas (limiar padrão: `>= 50ms`)
  - erros de parsing
- **Modo verboso (`--verbose`)**: imprime todo o tráfego de requisição/resposta WS.

### Estilo de log WS

`openclaw gateway` oferece uma alternância de estilo por gateway:

- `--ws-log auto` (padrão): o modo normal é otimizado; o modo verboso usa saída compacta
- `--ws-log compact`: saída compacta (requisição/resposta pareadas) quando verboso
- `--ws-log full`: saída completa por frame quando verboso
- `--compact`: alias para `--ws-log compact`

Exemplos:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# show all WS traffic (full meta)
openclaw gateway --verbose --ws-log full
```

## Formatação do console (logging por subsistema)

O formatador do console é **ciente de TTY** e imprime linhas consistentes com prefixos.
Loggers de subsistema mantêm a saída agrupada e fácil de examinar.

Comportamento:

- **Prefixos de subsistema** em todas as linhas (por exemplo, `[gateway]`, `[canvas]`, `[tailscale]`)
- **Cores de subsistema** (estáveis por subsistema) além de coloração por nível
- **Cores quando a saída é um TTY ou o ambiente parece um terminal rico** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respeita `NO_COLOR`
- **Prefixos de subsistema encurtados**: remove o prefixo inicial `gateway/` + `channels/`, mantém os últimos 2 segmentos (por exemplo, `whatsapp/outbound`)
- **Sub-loggers por subsistema** (prefixo automático + campo estruturado `{ subsystem }`)
- **`logRaw()`** para saída de QR/UX (sem prefixo, sem formatação)
- **Estilos de console** (por exemplo, `pretty | compact | json`)
- **Nível de log do console** separado do nível de log em arquivo (o arquivo mantém o detalhe completo quando `logging.level` está definido como `debug`/`trace`)
- **Corpos de mensagens do WhatsApp** são registrados em `debug` (use `--verbose` para vê-los)

Isso mantém os logs em arquivo existentes estáveis enquanto torna a saída interativa fácil de examinar.

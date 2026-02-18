---
title: "sistema"
---

# `openclaw system`

Auxiliares de nível de sistema para o Gateway: enfileirar eventos do sistema, controlar heartbeats
e visualizar presença.

## Comandos comuns

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## `system event`

Enfileira um evento de sistema na sessão **principal**. O próximo heartbeat irá injetá-lo
como uma linha `System:` no prompt. Use `--mode now` para disparar o heartbeat
imediatamente; `next-heartbeat` aguarda o próximo tick agendado.

Opções:

- `--text <text>`: texto do evento de sistema obrigatório.
- `--mode <mode>`: `now` ou `next-heartbeat` (padrão).
- `--json`: saída legível por máquina.

## `system heartbeat last|enable|disable`

Controles de heartbeat:

- `last`: mostra o último evento de heartbeat.
- `enable`: liga novamente os heartbeats (use isto se eles foram desabilitados).
- `disable`: pausa os heartbeats.

Opções:

- `--json`: saída legível por máquina.

## `system presence`

Lista as entradas atuais de presença do sistema que o Gateway conhece (nós,
instâncias e linhas de status semelhantes).

Opções:

- `--json`: saída legível por máquina.

## Notas

- Requer um Gateway em execução acessível pela sua configuração atual (local ou remota).
- Eventos de sistema são efêmeros e não são persistidos entre reinicializações.



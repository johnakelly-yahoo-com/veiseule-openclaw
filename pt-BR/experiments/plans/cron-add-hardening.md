---
owner: "openclaw"
status: "complete"
last_updated: "2026-01-05"
title: "Reforço do Cron Add"
---

# Reforço do Cron Add e Alinhamento de Esquemas

## Contexto

Logs recentes do gateway mostram falhas repetidas de `cron.add` com parâmetros inválidos (ausência de `sessionTarget`, `wakeMode`, `payload` e `schedule` malformado). Isso indica que pelo menos um cliente (provavelmente o caminho de chamada da ferramenta do agente) está enviando payloads de jobs encapsulados ou parcialmente especificados. Separadamente, há divergência entre enums de provedores de cron no TypeScript, no esquema do gateway, nas flags da CLI e nos tipos de formulário da UI, além de um desalinhamento da UI para `cron.status` (espera `jobCount` enquanto o gateway retorna `jobs`).

## Objetivos

- Interromper o spam de INVALID_REQUEST de `cron.add` normalizando payloads encapsulados comuns e inferindo campos `kind` ausentes.
- Alinhar as listas de provedores de cron entre o esquema do gateway, tipos de cron, docs da CLI e formulários da UI.
- Tornar explícito o esquema da ferramenta de cron do agente para que o LLM produza payloads de jobs corretos.
- Corrigir a exibição da contagem de jobs no status de cron da Control UI.
- Adicionar testes para cobrir normalização e comportamento da ferramenta.

## Não objetivos

- Alterar a semântica de agendamento do cron ou o comportamento de execução de jobs.
- Adicionar novos tipos de agenda ou parsing de expressões cron.
- Reformular a UI/UX do cron além das correções de campos necessárias.

## Constatações (lacunas atuais)

- `CronPayloadSchema` no gateway exclui `signal` + `imessage`, enquanto os tipos TS os incluem.
- O CronStatus da Control UI espera `jobCount`, mas o gateway retorna `jobs`.
- O esquema da ferramenta de cron do agente permite objetos `job` arbitrários, possibilitando entradas malformadas.
- O gateway valida estritamente `cron.add` sem normalização, portanto payloads encapsulados falham.

## O que mudou

- `cron.add` e `cron.update` agora normalizam formatos comuns de encapsulamento e inferem campos `kind` ausentes.
- O esquema da ferramenta de cron do agente corresponde ao esquema do gateway, o que reduz payloads inválidos.
- Enums de provedores foram alinhados entre gateway, CLI, UI e seletor do macOS.
- A Control UI usa o campo de contagem `jobs` do gateway para status.

## Comportamento atual

- **Normalização:** payloads encapsulados `data`/`job` são desembrulhados; `schedule.kind` e `payload.kind` são inferidos quando seguro.
- **Padrões:** padrões seguros são aplicados para `wakeMode` e `sessionTarget` quando ausentes.
- **Provedores:** Discord/Slack/Signal/iMessage agora são exibidos de forma consistente na CLI/UI.

Veja [Cron jobs](/automation/cron-jobs) para o formato normalizado e exemplos.

## Verificação

- Acompanhe os logs do gateway para redução de erros INVALID_REQUEST de `cron.add`.
- Confirme que o status de cron da Control UI mostra a contagem de jobs após a atualização.

## Ações opcionais de acompanhamento

- Smoke test manual da Control UI: adicionar um job de cron por provedor + verificar a contagem de jobs no status.

## Questões em aberto

- `cron.add` deve aceitar `state` explícito dos clientes (atualmente não permitido pelo esquema)?
- Devemos permitir `webchat` como um provedor de entrega explícito (atualmente filtrado na resolução de entrega)?


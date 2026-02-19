---
summary: "Plano: isolar browser act:evaluate da fila do Playwright usando CDP, com prazos ponta a ponta e resolução de referência mais segura"
owner: "openclaw"
status: "rascunho"
last_updated: "2026-02-10"
title: "Refatoração CDP do Browser Evaluate"
---

# Plano de Refatoração CDP do Browser Evaluate

## Contexto

`act:evaluate` executa JavaScript fornecido pelo usuário na página. Atualmente ele é executado via Playwright
(`page.evaluate` ou `locator.evaluate`). O Playwright serializa comandos CDP por página, então um
evaluate travado ou de longa duração pode bloquear a fila de comandos da página e fazer com que cada ação posterior
nessa aba pareça "travada".

O PR #13498 adiciona uma rede de segurança pragmática (evaluate com limite de tempo, propagação de abort e recuperação de melhor esforço). Este documento descreve uma refatoração maior que torna `act:evaluate` inerentemente
isolado do Playwright, para que um evaluate travado não possa bloquear operações normais do Playwright.

## Objetivos

- `act:evaluate` não pode bloquear permanentemente ações posteriores do navegador na mesma aba.
- Os timeouts são uma única fonte de verdade ponta a ponta, para que um chamador possa confiar em um orçamento.
- Abort e timeout são tratados da mesma forma em HTTP e no despacho em processo.
- O direcionamento de elementos para evaluate é suportado sem desligar tudo do Playwright.
- Manter compatibilidade retroativa para chamadores e payloads existentes.

## Não objetivos

- Substituir todas as ações do navegador (click, type, wait, etc.) por implementações CDP.
- Remover a rede de segurança existente introduzida no PR #13498 (ela permanece como um fallback útil).
- Introduzir novas capacidades inseguras além do gate `browser.evaluateEnabled` existente.
- Adicionar isolamento de processo (processo/thread worker) para evaluate. Se ainda observarmos estados travados difíceis de recuperar após esta refatoração, essa é uma ideia para acompanhamento.

## Arquitetura Atual (Por Que Fica Travado)

Em alto nível:

- Os chamadores enviam `act:evaluate` para o serviço de controle do navegador.
- O manipulador de rota chama o Playwright para executar o JavaScript.
- O Playwright serializa os comandos da página, então um evaluate que nunca termina bloqueia a fila.
- Uma fila travada significa que operações posteriores de clique/digitação/espera na aba podem parecer travadas.

## Arquitetura Proposta

### 1. Propagação de Deadline

Introduza um único conceito de orçamento (budget) e derive tudo a partir dele:

- O chamador define `timeoutMs` (ou um deadline no futuro).
- O timeout da requisição externa, a lógica do manipulador de rota e o orçamento de execução dentro da página
  usam o mesmo orçamento, com uma pequena margem onde necessário para overhead de serialização.
- O abort é propagado como um `AbortSignal` em todos os lugares para que o cancelamento seja consistente.

Direção de implementação:

- Adicione um pequeno helper (por exemplo `createBudget({ timeoutMs, signal })`) que retorna:
  - `signal`: o AbortSignal vinculado
  - `deadlineAtMs`: deadline absoluto
  - `remainingMs()`: orçamento restante para operações filhas
- Use este helper em:
  - `src/browser/client-fetch.ts` (HTTP e dispatch em processo)
  - `src/node-host/runner.ts` (caminho de proxy)
  - implementações de ações do navegador (Playwright e CDP)

### 2. Motor de Evaluate Separado (Caminho CDP)

Adicione uma implementação de evaluate baseada em CDP que não compartilhe a fila de comandos por página do Playwright. A propriedade principal é que o transporte de evaluate é uma conexão WebSocket separada
e uma sessão CDP separada anexada ao target.

Direção de implementação:

- Novo módulo, por exemplo `src/browser/cdp-evaluate.ts`, que:
  - Conecta ao endpoint CDP configurado (socket em nível de navegador).
  - Usa `Target.attachToTarget({ targetId, flatten: true })` para obter um `sessionId`.
  - Executa um dos seguintes:
    - `Runtime.evaluate` para evaluate em nível de página, ou
    - `DOM.resolveNode` mais `Runtime.callFunctionOn` para evaluate de elemento.
  - Em caso de timeout ou abort:
    - Envia `Runtime.terminateExecution` como melhor esforço para a sessão.
    - Fecha o WebSocket e retorna um erro claro.

Observações:

- Isso ainda executa JavaScript na página, então a terminação pode ter efeitos colaterais. A vantagem
  é que não bloqueia a fila do Playwright e pode ser cancelado na camada de transporte
  ao encerrar a sessão CDP.

### 3. História de Ref (Direcionamento de Elemento Sem Uma Reescrita Completa)

A parte difícil é o direcionamento de elementos. CDP precisa de um handle de DOM ou `backendDOMNodeId`, enquanto
hoje a maioria das ações no navegador usa locators do Playwright baseados em refs de snapshots.

Abordagem recomendada: manter os refs existentes, mas anexar um id opcional resolvível via CDP.

#### 3.1 Estender as Informações de Ref Armazenadas

Estender os metadados de ref de role armazenados para opcionalmente incluir um id de CDP:

- Hoje: `{ role, name, nth }`
- Proposto: `{ role, name, nth, backendDOMNodeId?: number }`

Isso mantém todas as ações existentes baseadas em Playwright funcionando e permite que o evaluate via CDP aceite
o mesmo valor de `ref` quando o `backendDOMNodeId` estiver disponível.

#### 3.2 Preencher backendDOMNodeId no Momento do Snapshot

Ao gerar um snapshot de role:

1. Gerar o mapa de refs de role existente como hoje (role, name, nth).
2. Buscar a árvore AX via CDP (`Accessibility.getFullAXTree`) e calcular um mapa paralelo de
   `(role, name, nth) -> backendDOMNodeId` usando as mesmas regras de tratamento de duplicados.
3. Mesclar o id de volta nas informações de ref armazenadas para a aba atual.

Se o mapeamento falhar para um ref, deixar `backendDOMNodeId` como undefined. Isso torna o recurso
best-effort e seguro para rollout.

#### 3.3 Comportamento de Evaluate com Ref

Em `act:evaluate`:

- Se `ref` estiver presente e tiver `backendDOMNodeId`, executar o evaluate do elemento via CDP.
- Se `ref` estiver presente, mas não tiver `backendDOMNodeId`, usar o caminho do Playwright como fallback (com
  a rede de segurança).

Escape hatch opcional:

- Estender o formato da requisição para aceitar `backendDOMNodeId` diretamente para usuários avançados (e
  para debugging), mantendo `ref` como a interface principal.

### 4. Manter um Caminho de Recuperação como Último Recurso

Mesmo com evaluate via CDP, há outras formas de travar uma aba ou uma conexão. Manter os
mecanismos de recuperação existentes (encerrar a execução + desconectar o Playwright) como último recurso
para:

- chamadores legados
- ambientes onde o attach via CDP é bloqueado
- casos de borda inesperados do Playwright

## Plano de Implementação (Iteração Única)

### Entregáveis

- Um mecanismo de evaluate baseado em CDP que executa fora da fila de comandos por página do Playwright.
- Um único orçamento de timeout/abort de ponta a ponta usado de forma consistente por chamadores e handlers.
- Metadados de ref que podem opcionalmente carregar `backendDOMNodeId` para evaluate de elemento.
- `act:evaluate` prioriza o mecanismo via CDP quando possível e usa Playwright como fallback quando não.
- Testes que comprovam que um evaluate travado não bloqueia ações posteriores.
- Logs/métricas que tornam falhas e fallbacks visíveis.

### Checklist de Implementação

1. Adicionar um helper compartilhado de "budget" para vincular `timeoutMs` + `AbortSignal` upstream em:
   - um único `AbortSignal`
   - um deadline absoluto
   - um helper `remainingMs()` para operações downstream
2. Atualize todos os caminhos de chamada para usar esse helper para que `timeoutMs` signifique a mesma coisa em todos os lugares:
   - `src/browser/client-fetch.ts` (HTTP e despacho em processo)
   - `src/node-host/runner.ts` (caminho do proxy node)
   - Wrappers de CLI que chamam `/act` (adicionar `--timeout-ms` ao `browser evaluate`)
3. Implementar `src/browser/cdp-evaluate.ts`:
   - conectar ao socket CDP no nível do navegador
   - `Target.attachToTarget` para obter um `sessionId`
   - executar `Runtime.evaluate` para avaliação de página
   - executar `DOM.resolveNode` + `Runtime.callFunctionOn` para avaliação de elemento
   - em caso de timeout/abort: fazer um `Runtime.terminateExecution` como melhor esforço e então fechar o socket
4. Estender os metadados de referência de role armazenados para opcionalmente incluir `backendDOMNodeId`:
   - manter o comportamento existente `{ role, name, nth }` para ações do Playwright
   - adicionar `backendDOMNodeId?: number` para direcionamento de elemento via CDP
5. Preencher `backendDOMNodeId` durante a criação do snapshot (melhor esforço):
   - buscar a árvore AX via CDP (`Accessibility.getFullAXTree`)
   - calcular `(role, name, nth) -> backendDOMNodeId` e mesclar no mapa de referências armazenado
   - se o mapeamento for ambíguo ou estiver ausente, deixar o id como undefined
6. Atualizar o roteamento de `act:evaluate`:
   - se não houver `ref`: sempre usar avaliação via CDP
   - se `ref` resolver para um `backendDOMNodeId`: usar avaliação de elemento via CDP
   - caso contrário: recorrer à avaliação do Playwright (ainda limitada e abortável)
7. Manter o caminho de recuperação existente de "último recurso" como fallback, não como caminho padrão.
8. Adicionar testes:
   - evaluate travado expira dentro do orçamento e o próximo click/type é bem-sucedido
   - abort cancela o evaluate (desconexão do cliente ou timeout) e desbloqueia ações subsequentes
   - falhas de mapeamento recorrem corretamente ao Playwright
9. Adicionar observabilidade:
   - duração do evaluate e contadores de timeout
   - uso de terminateExecution
   - taxa de fallback (CDP -> Playwright) e motivos

### Critérios de Aceitação

- Um `act:evaluate` deliberadamente travado retorna dentro do orçamento do chamador e não trava a
  aba para ações posteriores.
- `timeoutMs` se comporta de forma consistente em CLI, ferramenta do agente, proxy node e chamadas em processo.
- Se `ref` puder ser mapeado para `backendDOMNodeId`, a avaliação de elemento usa CDP; caso contrário, o
  caminho de fallback ainda é limitado e recuperável.

## Plano de Testes

- Testes unitários:
  - lógica de correspondência `(role, name, nth)` entre referências de role e nós da árvore AX.
  - comportamento do helper de orçamento (margem de segurança, cálculo do tempo restante).
- Testes de integração:
  - timeout de avaliação via CDP retorna dentro do orçamento e não bloqueia a próxima ação.
  - Abortar cancela a avaliação e aciona a terminação em modo best-effort.
- Testes de contrato:
  - Garantir que `BrowserActRequest` e `BrowserActResponse` permaneçam compatíveis.

## Riscos e Mitigações

- O mapeamento é imperfeito:
  - Mitigação: mapeamento em modo best-effort, fallback para Playwright evaluate e adição de ferramentas de depuração.
- `Runtime.terminateExecution` tem efeitos colaterais:
  - Mitigação: usar apenas em caso de timeout/abort e documentar o comportamento nos erros.
- Sobrecarga extra:
  - Mitigação: buscar a árvore AX apenas quando snapshots forem solicitados, armazenar em cache por target e manter
    a sessão CDP de curta duração.
- Limitações do relay da extensão:
  - Mitigação: usar APIs de anexação em nível de navegador quando sockets por página não estiverem disponíveis e
    manter o caminho atual do Playwright como fallback.

## Questões em Aberto

- O novo mecanismo deve ser configurável como `playwright`, `cdp` ou `auto`?
- Queremos expor um novo formato "nodeRef" para usuários avançados ou manter apenas `ref`?
- Como snapshots de frames e snapshots com escopo de seletor devem participar do mapeamento AX?

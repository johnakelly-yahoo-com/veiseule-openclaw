---
title: "API OpenResponses"
---

# API OpenResponses (HTTP)

O Gateway do OpenClaw pode servir um endpoint compatível com OpenResponses `POST /v1/responses`.

Este endpoint é **desativado por padrão**. Ative-o primeiro na configuração.

- `POST /v1/responses`
- Mesma porta do Gateway (multiplex WS + HTTP): `http://<gateway-host>:<port>/v1/responses`

Nos bastidores, as requisições são executadas como uma execução normal de agente do Gateway (mesmo caminho de código que
`openclaw agent`), portanto roteamento/permissões/configuração correspondem ao seu Gateway.

## Autenticação

Usa a configuração de autenticação do Gateway. Envie um token bearer:

- `Authorization: Bearer <token>`

Notas:

- Quando `gateway.auth.mode="token"`, use `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
- Quando `gateway.auth.mode="password"`, use `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).

## Escolhendo um agente

Nenhum cabeçalho personalizado é necessário: codifique o id do agente no campo OpenResponses `model`:

- `model: "openclaw:<agentId>"` (exemplo: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (alias)

Ou direcione um agente específico do OpenClaw por cabeçalho:

- `x-openclaw-agent-id: <agentId>` (padrão: `main`)

Avançado:

- `x-openclaw-session-key: <sessionKey>` para controlar totalmente o roteamento de sessão.

## Ativando o endpoint

Defina `gateway.http.endpoints.responses.enabled` como `true`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## Desativando o endpoint

Defina `gateway.http.endpoints.responses.enabled` como `false`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## Comportamento de sessão

Por padrão, o endpoint é **sem estado por requisição** (uma nova chave de sessão é gerada a cada chamada).

Se a requisição incluir uma string OpenResponses `user`, o Gateway deriva uma chave de sessão estável
a partir dela, para que chamadas repetidas possam compartilhar uma sessão do agente.

## Formato da requisição (suportado)

A requisição segue a API OpenResponses com entrada baseada em itens. Suporte atual:

- `input`: string ou array de objetos de item.
- `instructions`: mesclado no prompt do sistema.
- `tools`: definições de ferramentas do cliente (function tools).
- `tool_choice`: filtrar ou exigir ferramentas do cliente.
- `stream`: habilita streaming SSE.
- `max_output_tokens`: limite de saída de melhor esforço (dependente do provedor).
- `user`: roteamento estável de sessão.

Aceitos, mas **atualmente ignorados**:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

## Itens (entrada)

### `message`

Papéis: `system`, `developer`, `user`, `assistant`.

- `system` e `developer` são acrescentados ao prompt do sistema.
- O item mais recente `user` ou `function_call_output` torna-se a “mensagem atual”.
- Mensagens anteriores de usuário/assistente são incluídas como histórico para contexto.

### `function_call_output` (ferramentas baseadas em turnos)

Envie os resultados das ferramentas de volta ao modelo:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### `reasoning` e `item_reference`

Aceitos para compatibilidade de esquema, mas ignorados ao construir o prompt.

## Ferramentas (function tools do lado do cliente)

Forneça ferramentas com `tools: [{ type: "function", function: { name, description?, parameters? } }]`.

Se o agente decidir chamar uma ferramenta, a resposta retorna um item de saída `function_call`.
Em seguida, você envia uma requisição de acompanhamento com `function_call_output` para continuar o turno.

## Imagens (`input_image`)

Suporta fontes base64 ou URL:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Tipos MIME permitidos (atual): `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
Tamanho máximo (atual): 10MB.

## Arquivos (`input_file`)

Suporta fontes base64 ou URL:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Tipos MIME permitidos (atual): `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

Tamanho máximo (atual): 5MB.

Comportamento atual:

- O conteúdo do arquivo é decodificado e adicionado ao **prompt do sistema**, não à mensagem do usuário,
  para que permaneça efêmero (não persistido no histórico da sessão).
- PDFs são analisados para texto. Se pouco texto for encontrado, as primeiras páginas são rasterizadas
  em imagens e passadas ao modelo.

A análise de PDF usa o build legado do `pdfjs-dist` amigável ao Node (sem worker). O build moderno
do PDF.js espera workers de navegador/globais do DOM, portanto não é usado no Gateway.

Padrões de busca por URL:

- `files.allowUrl`: `true`
- `images.allowUrl`: `true`
- As requisições são protegidas (resolução DNS, bloqueio de IP privado, limites de redirecionamento, timeouts).

## Limites de arquivos + imagens (configuração)

Os padrões podem ser ajustados em `gateway.http.endpoints.responses`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          files: {
            allowUrl: true,
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

Padrões quando omitidos:

- `maxBodyBytes`: 20MB
- `files.maxBytes`: 5MB
- `files.maxChars`: 200k
- `files.maxRedirects`: 3
- `files.timeoutMs`: 10s
- `files.pdf.maxPages`: 4
- `files.pdf.maxPixels`: 4.000.000
- `files.pdf.minTextChars`: 200
- `images.maxBytes`: 10MB
- `images.maxRedirects`: 3
- `images.timeoutMs`: 10s

## Streaming (SSE)

Defina `stream: true` para receber Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Cada linha de evento é `event: <type>` e `data: <json>`
- O stream termina com `data: [DONE]`

Tipos de evento atualmente emitidos:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (em erro)

## Uso

`usage` é preenchido quando o provedor subjacente reporta contagens de tokens.

## Erros

Os erros usam um objeto JSON como:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Casos comuns:

- `401` autenticação ausente/inválida
- `400` corpo da requisição inválido
- `405` método incorreto

## Exemplos

Sem streaming:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Com streaming:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```



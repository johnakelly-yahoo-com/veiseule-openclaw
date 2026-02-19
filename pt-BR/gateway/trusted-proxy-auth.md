---
summary: "Delegue a autenticação do gateway a um proxy reverso confiável (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Executando o OpenClaw atrás de um proxy com reconhecimento de identidade
  - Configurando Pomerium, Caddy ou nginx com OAuth na frente do OpenClaw
  - Corrigindo erros WebSocket 1008 unauthorized com configurações de proxy reverso
---

# Autenticação de Proxy Confiável

> ⚠️ **Recurso sensível à segurança.** Este modo delega a autenticação inteiramente ao seu proxy reverso. Uma configuração incorreta pode expor seu Gateway a acesso não autorizado. Leia esta página com atenção antes de ativar.

## Quando usar

Use o modo de autenticação `trusted-proxy` quando:

- Você executa o OpenClaw atrás de um **proxy com reconhecimento de identidade** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Seu proxy gerencia toda a autenticação e envia a identidade do usuário por meio de headers
- Você está em um ambiente Kubernetes ou de contêiner onde o proxy é o único caminho para o Gateway
- Você está recebendo erros WebSocket `1008 unauthorized` porque os navegadores não conseguem enviar tokens em payloads WS

## Quando NÃO usar

- Se o seu proxy não autentica usuários (apenas um terminador TLS ou balanceador de carga)
- Se houver qualquer caminho para o Gateway que contorne o proxy (brechas no firewall, acesso à rede interna)
- Se você não tem certeza se seu proxy remove/substitui corretamente os headers encaminhados
- Se você precisa apenas de acesso pessoal para um único usuário (considere Tailscale Serve + loopback para uma configuração mais simples)

## Como funciona

1. Seu proxy reverso autentica os usuários (OAuth, OIDC, SAML etc.)
2. O proxy adiciona um header com a identidade do usuário autenticado (ex.: `x-forwarded-user: nick@example.com`)
3. O OpenClaw verifica se a requisição veio de um **IP de proxy confiável** (configurado em `gateway.trustedProxies`)
4. O OpenClaw extrai a identidade do usuário a partir do header configurado
5. Se tudo estiver correto, a requisição é autorizada

## Configuração

```json5
{
  gateway: {
    // Deve vincular à interface de rede (não loopback)
    bind: "lan",

    // CRÍTICO: Adicione aqui apenas o(s) IP(s) do seu proxy
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header que contém a identidade do usuário autenticado (obrigatório)
        userHeader: "x-forwarded-user",

        // Opcional: headers que DEVEM estar presentes (verificação do proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Opcional: restringir a usuários específicos (vazio = permitir todos)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Referência de configuração

| Campo                                       | Obrigatório | Descrição                                                                                                                                |
| ------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Sim         | Array de endereços IP de proxies confiáveis. Requisições de outros IPs são rejeitadas.                   |
| `gateway.auth.mode`                         | Sim         | Deve ser `"trusted-proxy"`                                                                                                               |
| `gateway.auth.trustedProxy.userHeader`      | Sim         | Nome do header que contém a identidade do usuário autenticado                                                                            |
| `gateway.auth.trustedProxy.requiredHeaders` | Não         | Cabeçalhos adicionais que devem estar presentes para que a requisição seja confiável                                                     |
| `gateway.auth.trustedProxy.allowUsers`      | Não         | Lista de permissões de identidades de usuários. Vazio significa permitir todos os usuários autenticados. |

## Exemplos de Configuração de Proxy

### Pomerium

O Pomerium passa a identidade em `x-pomerium-claim-email` (ou outros cabeçalhos de claim) e um JWT em `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium's IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Trecho de configuração do Pomerium:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy com OAuth

O Caddy com o plugin `caddy-security` pode autenticar usuários e encaminhar cabeçalhos de identidade.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy's IP (if on same host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Trecho de Caddyfile:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

O oauth2-proxy autentica usuários e passa a identidade em `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

Trecho de configuração do nginx:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik com Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik container IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Checklist de Segurança

Antes de habilitar a autenticação trusted-proxy, verifique:

- [ ] **Proxy é o único caminho**: A porta do Gateway está protegida por firewall contra tudo, exceto seu proxy
- [ ] **trustedProxies é mínimo**: Apenas os IPs reais do seu proxy, não sub-redes inteiras
- [ ] **Proxy remove cabeçalhos**: Seu proxy sobrescreve (não acrescenta) cabeçalhos `x-forwarded-*` vindos dos clientes
- [ ] **Terminação TLS**: Seu proxy gerencia o TLS; os usuários se conectam via HTTPS
- [ ] **allowUsers está definido** (recomendado): Restrinja a usuários conhecidos em vez de permitir qualquer usuário autenticado

## Auditoria de Segurança

`openclaw security audit` sinalizará a autenticação trusted-proxy com severidade **crítica**. Isso é intencional — é um lembrete de que você está delegando a segurança à configuração do seu proxy.

A auditoria verifica:

- Configuração `trustedProxies` ausente
- Configuração `userHeader` ausente
- `allowUsers` vazio (permite qualquer usuário autenticado)

## Solução de Problemas

### "trusted_proxy_untrusted_source"

A requisição não veio de um IP presente em `gateway.trustedProxies`. Verifique:

- O IP do proxy está correto? (IPs de contêineres Docker podem mudar)
- Há um load balancer na frente do seu proxy?
- Use `docker inspect` ou `kubectl get pods -o wide` para encontrar os IPs reais

### "trusted_proxy_user_missing"

O header de usuário estava vazio ou ausente. Verifique:

- Seu proxy está configurado para encaminhar headers de identidade?
- O nome do header está correto? (não diferencia maiúsculas de minúsculas, mas a grafia importa)
- O usuário está realmente autenticado no proxy?

### "trusted_proxy_missing_header_\*"

Um header obrigatório não estava presente. Verifique:

- A configuração do seu proxy para esses headers específicos
- Se os headers estão sendo removidos em algum ponto da cadeia

### "trusted_proxy_user_not_allowed"

O usuário está autenticado, mas não está em `allowUsers`. Adicione-o ou remova a lista de permissões.

### WebSocket ainda falhando

Certifique-se de que seu proxy:

- Suporta upgrades de WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Encaminha os headers de identidade em requisições de upgrade para WebSocket (não apenas HTTP)
- Não possui um caminho de autenticação separado para conexões WebSocket

## Migração de autenticação por token

Se você está migrando de autenticação por token para trusted-proxy:

1. Configure seu proxy para autenticar usuários e encaminhar headers
2. Teste a configuração do proxy de forma independente (curl com headers)
3. Atualize a configuração do OpenClaw com autenticação trusted-proxy
4. Reinicie o Gateway
5. Teste conexões WebSocket a partir da Control UI
6. Execute `openclaw security audit` e revise os resultados

## Relacionado

- [Security](/gateway/security) — guia completo de segurança
- [Configuration](/gateway/configuration) — referência de configuração
- [Remote Access](/gateway/remote) — outros padrões de acesso remoto
- [Tailscale](/gateway/tailscale) — alternativa mais simples para acesso apenas pela tailnet


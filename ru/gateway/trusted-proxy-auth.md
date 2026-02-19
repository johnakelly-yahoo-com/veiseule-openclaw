---
summary: "Делегировать аутентификацию Gateway доверенному reverse proxy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Запуск OpenClaw за identity-aware proxy
  - Настройка Pomerium, Caddy или nginx с OAuth перед OpenClaw
  - Исправление ошибок WebSocket 1008 unauthorized при использовании reverse proxy
---

# Trusted Proxy Auth

> ⚠️ **Функция, критичная с точки зрения безопасности.** В этом режиме аутентификация полностью делегируется вашему reverse proxy. Неправильная конфигурация может открыть доступ к вашему Gateway для неавторизованных пользователей. Внимательно прочитайте эту страницу перед включением.

## Когда использовать

Используйте режим аутентификации `trusted-proxy`, когда:

- Вы запускаете OpenClaw за **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Ваш proxy выполняет всю аутентификацию и передаёт идентификацию пользователя через заголовки
- Вы работаете в среде Kubernetes или контейнеров, где proxy — единственный путь к Gateway
- Вы сталкиваетесь с ошибками WebSocket `1008 unauthorized`, потому что браузеры не могут передавать токены в payload WS

## Когда НЕ использовать

- Если ваш proxy не аутентифицирует пользователей (а только завершает TLS или выступает как балансировщик нагрузки)
- Если существует любой путь к Gateway в обход proxy (дыры в firewall, доступ из внутренней сети)
- Если вы не уверены, что ваш proxy корректно удаляет/перезаписывает forwarded-заголовки
- Если вам нужен только персональный однопользовательский доступ (рассмотрите Tailscale Serve + loopback для более простой настройки)

## Как это работает

1. Ваш reverse proxy аутентифицирует пользователей (OAuth, OIDC, SAML и т. д.)
2. Proxy добавляет заголовок с идентификацией аутентифицированного пользователя (например, `x-forwarded-user: nick@example.com`)
3. OpenClaw проверяет, что запрос поступил от **доверенного IP прокси** (настраивается в `gateway.trustedProxies`)
4. OpenClaw извлекает идентификатор пользователя из настроенного заголовка
5. Если всё в порядке, запрос авторизуется

## Конфигурация

```json5
{
  gateway: {
    // Должен быть привязан к сетевому интерфейсу (не loopback)
    bind: "lan",

    // КРИТИЧНО: Добавляйте сюда только IP вашего прокси
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Заголовок, содержащий идентификатор аутентифицированного пользователя (обязательно)
        userHeader: "x-forwarded-user",

        // Необязательно: заголовки, которые ОБЯЗАТЕЛЬНО должны присутствовать (проверка прокси)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Необязательно: ограничить доступ конкретными пользователями (пусто = разрешить всем)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Справочник по конфигурации

| Поле                                        | Обязательно | Описание                                                                                                                                                     |
| ------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `gateway.trustedProxies`                    | Да          | Массив IP-адресов прокси, которым доверяют. Запросы с других IP-адресов отклоняются.                                         |
| `gateway.auth.mode`                         | Да          | Должно быть равно `"trusted-proxy"`                                                                                                                          |
| `gateway.auth.trustedProxy.userHeader`      | Да          | Имя заголовка, содержащего идентификатор аутентифицированного пользователя                                                                                   |
| `gateway.auth.trustedProxy.requiredHeaders` | Нет         | Дополнительные заголовки, которые должны присутствовать, чтобы запрос считался доверенным                                                                    |
| `gateway.auth.trustedProxy.allowUsers`      | Нет         | Список разрешённых идентификаторов пользователей. Пустое значение означает разрешить всех аутентифицированных пользователей. |

## Примеры настройки прокси

### Pomerium

Pomerium передаёт идентификатор в `x-pomerium-claim-email` (или других claim-заголовках), а JWT — в `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP Pomerium
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

Фрагмент конфигурации Pomerium:

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

### Caddy с OAuth

Caddy с плагином `caddy-security` может аутентифицировать пользователей и передавать заголовки с идентификатором.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP Caddy (если на том же хосте)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Фрагмент Caddyfile:

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

oauth2-proxy аутентифицирует пользователей и передаёт идентификатор в `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

фрагмент конфигурации nginx:

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

### Traefik с Forward Auth

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

## Контрольный список безопасности

Перед включением trusted-proxy auth проверьте:

- [ ] **Proxy — единственный путь доступа**: Порт Gateway закрыт файрволом для всех, кроме вашего proxy
- [ ] **trustedProxies минимален**: Указаны только реальные IP-адреса вашего proxy, а не целые подсети
- [ ] **Proxy очищает заголовки**: Ваш proxy перезаписывает (а не добавляет) заголовки `x-forwarded-*`, полученные от клиентов
- [ ] **Завершение TLS**: Ваш proxy обрабатывает TLS; пользователи подключаются по HTTPS
- [ ] **allowUsers задан** (рекомендуется): Ограничьте доступ известными пользователями вместо разрешения всем аутентифицированным

## Аудит безопасности

`openclaw security audit` пометит trusted-proxy auth как проблему с уровнем серьезности **critical**. Это сделано намеренно — напоминание о том, что вы делегируете безопасность конфигурации вашего proxy.

Аудит проверяет:

- Отсутствие конфигурации `trustedProxies`
- Отсутствие конфигурации `userHeader`
- Пустой `allowUsers` (разрешает доступ любому аутентифицированному пользователю)

## Устранение неполадок

### "trusted_proxy_untrusted_source"

Запрос пришёл с IP-адреса, отсутствующего в `gateway.trustedProxies`. Проверьте:

- Правильный ли IP-адрес у proxy? (IP-адреса Docker-контейнеров могут меняться)
- Есть ли перед вашим proxy балансировщик нагрузки?
- Используйте `docker inspect` или `kubectl get pods -o wide`, чтобы узнать фактические IP-адреса

### "trusted_proxy_user_missing"

Заголовок пользователя пустой или отсутствует. Проверьте:

- Настроен ли proxy на передачу заголовков идентификации?
- Правильно ли указано имя заголовка? (без учёта регистра, но написание имеет значение)
- Аутентифицирован ли пользователь на proxy?

### "trusted_proxy_missing_header_\*"

Отсутствует обязательный заголовок. Проверьте:

- Конфигурацию proxy для этих конкретных заголовков
- Не удаляются ли заголовки где-то в цепочке

### "trusted_proxy_user_not_allowed"

Пользователь аутентифицирован, но отсутствует в `allowUsers`. Либо добавьте их, либо удалите allowlist.

### WebSocket по‑прежнему не работает

Убедитесь, что ваш прокси:

- Поддерживает обновление до WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Передаёт заголовки идентификации при запросах обновления WebSocket (не только для HTTP)
- Не имеет отдельного пути аутентификации для WebSocket‑соединений

## Миграция с аутентификации по токену

Если вы переходите с token auth на trusted-proxy:

1. Настройте прокси для аутентификации пользователей и передачи заголовков
2. Протестируйте настройку прокси отдельно (curl с заголовками)
3. Обновите конфигурацию OpenClaw, включив trusted-proxy auth
4. Перезапустите Gateway
5. Проверьте WebSocket‑соединения из Control UI
6. Выполните `openclaw security audit` и просмотрите результаты

## Связанные материалы

- [Security](/gateway/security) — полное руководство по безопасности
- [Configuration](/gateway/configuration) — справочник по конфигурации
- [Remote Access](/gateway/remote) — другие варианты удалённого доступа
- [Tailscale](/gateway/tailscale) — более простой вариант для доступа только внутри tailnet


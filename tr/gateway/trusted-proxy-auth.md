---
summary: "Gateway kimlik doğrulamasını güvenilir bir reverse proxy’ye devredin (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - OpenClaw’ı kimlik farkındalıklı bir proxy arkasında çalıştırma
  - OpenClaw’ın önünde OAuth ile Pomerium, Caddy veya nginx kurulumu
  - Reverse proxy kurulumlarında WebSocket 1008 yetkisiz hatalarını giderme
---

# Trusted Proxy Auth

> ⚠️ **Güvenlik açısından hassas özellik.** Bu mod, kimlik doğrulamayı tamamen reverse proxy’nize devreder. Yanlış yapılandırma, Gateway’inizi yetkisiz erişime açık hale getirebilir. Etkinleştirmeden önce bu sayfayı dikkatlice okuyun.

## Ne Zaman Kullanılmalı

`trusted-proxy` kimlik doğrulama modunu şu durumlarda kullanın:

- OpenClaw’ı bir **kimlik farkındalıklı proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth) arkasında çalıştırıyorsanız
- Proxy tüm kimlik doğrulama işlemlerini yapıyor ve kullanıcı kimliğini header’lar aracılığıyla iletiyorsa
- Proxy’nin Gateway’e giden tek yol olduğu bir Kubernetes veya container ortamındaysanız
- Tarayıcılar WS payload’larında token iletemediği için WebSocket `1008 unauthorized` hataları alıyorsanız

## Ne Zaman KULLANILMAMALI

- Proxy’niz kullanıcıları kimlik doğrulamıyorsa (yalnızca bir TLS sonlandırıcı veya load balancer ise)
- Gateway’e proxy’yi atlayan herhangi bir yol varsa (güvenlik duvarı açıkları, iç ağ erişimi)
- Proxy’nizin yönlendirilen header’ları doğru şekilde temizlediğinden/üzerine yazdığından emin değilseniz
- Yalnızca kişisel tek kullanıcı erişimine ihtiyacınız varsa (daha basit kurulum için Tailscale Serve + loopback’i değerlendirin)

## Nasıl Çalışır

1. Reverse proxy’niz kullanıcıları kimlik doğrular (OAuth, OIDC, SAML, vb.)
2. Proxy, kimliği doğrulanmış kullanıcı bilgisini içeren bir header ekler (örn. `x-forwarded-user: nick@example.com`)
3. OpenClaw, isteğin **güvenilir bir proxy IP’sinden** geldiğini kontrol eder (`gateway.trustedProxies` içinde yapılandırılmış)
4. OpenClaw, yapılandırılmış header’dan kullanıcı kimliğini çıkarır
5. Her şey doğrulanırsa, istek yetkilendirilir

## Yapılandırma

```json5
{
  gateway: {
    // Ağ arayüzüne bağlanmalıdır (loopback değil)
    bind: "lan",

    // KRİTİK: Buraya yalnızca proxy’nizin IP adres(ler)ini ekleyin
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Kimliği doğrulanmış kullanıcı bilgisini içeren header (zorunlu)
        userHeader: "x-forwarded-user",

        // İsteğe bağlı: MUTLAKA mevcut olması gereken header’lar (proxy doğrulaması)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // İsteğe bağlı: belirli kullanıcılarla sınırla (boş = tümüne izin ver)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Yapılandırma Referansı

| Alan                                        | Zorunlu | Açıklama                                                                                                                                                 |
| ------------------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Evet    | Güvenilecek proxy IP adreslerinin dizisi. Diğer IP’lerden gelen istekler reddedilir.                                     |
| `gateway.auth.mode`                         | Evet    | `"trusted-proxy"` olmalıdır                                                                                                                              |
| `gateway.auth.trustedProxy.userHeader`      | Evet    | Kimliği doğrulanmış kullanıcı bilgisini içeren header adı                                                                                                |
| `gateway.auth.trustedProxy.requiredHeaders` | Hayır   | İsteğin güvenilir sayılması için mevcut olması gereken ek header’lar                                                                                     |
| `gateway.auth.trustedProxy.allowUsers`      | Hayır   | Kullanıcı kimlikleri için izin listesi. Boş olması, kimliği doğrulanmış tüm kullanıcılara izin verildiği anlamına gelir. |

## Proxy Kurulum Örnekleri

### Pomerium

Pomerium, kimliği `x-pomerium-claim-email` (veya diğer claim başlıkları) içinde ve bir JWT’yi `x-pomerium-jwt-assertion` içinde iletir.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium'un IP adresi
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

Pomerium yapılandırma örneği:

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

### OAuth ile Caddy

`caddy-security` eklentisine sahip Caddy, kullanıcıların kimliğini doğrulayabilir ve kimlik başlıklarını iletebilir.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy'nin IP adresi (aynı sunucudaysa)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile örneği:

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

oauth2-proxy, kullanıcıların kimliğini doğrular ve kimliği `x-auth-request-email` içinde iletir.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP adresi
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx yapılandırma örneği:

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

### Forward Auth ile Traefik

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik container IP adresi
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Güvenlik Kontrol Listesi

trusted-proxy auth özelliğini etkinleştirmeden önce doğrulayın:

- [ ] **Proxy tek erişim yolu olmalı**: Gateway portu, proxy’niz dışında her şeye karşı firewall ile korunmuş olmalı
- [ ] **trustedProxies minimal olmalı**: Tüm alt ağlar değil, yalnızca gerçek proxy IP adresleriniz
- [ ] **Proxy başlıkları temizler**: Proxy’niz, istemcilerden gelen `x-forwarded-*` başlıklarını üzerine yazar (eklemez)
- [ ] **TLS sonlandırma**: TLS işlemini proxy’niz yönetir; kullanıcılar HTTPS üzerinden bağlanır
- [ ] **allowUsers ayarlanmış olmalı** (önerilir): Kimliği doğrulanmış herkese izin vermek yerine bilinen kullanıcılarla sınırlandırın

## Güvenlik Denetimi

`openclaw security audit`, trusted-proxy auth kullanımını **critical** seviyesinde bir bulgu olarak işaretler. Bu bilinçli bir tercihtir — güvenliği proxy kurulumunuza devrettiğinizi hatırlatır.

Denetim şunları kontrol eder:

- `trustedProxies` yapılandırmasının eksik olması
- `userHeader` yapılandırmasının eksik olması
- Boş `allowUsers` (kimliği doğrulanmış herhangi bir kullanıcıya izin verir)

## Sorun Giderme

### "trusted_proxy_untrusted_source"

İstek, `gateway.trustedProxies` içinde yer alan bir IP adresinden gelmedi. Kontrol edin:

- Proxy IP adresi doğru mu? (Docker konteyner IP’leri değişebilir)
- Proxy’nizin önünde bir yük dengeleyici var mı?
- Gerçek IP’leri bulmak için `docker inspect` veya `kubectl get pods -o wide` kullanın

### "trusted_proxy_user_missing"

Kullanıcı başlığı boş veya eksikti. Kontrol edin:

- Proxy’niz kimlik başlıklarını iletecek şekilde yapılandırıldı mı?
- Başlık adı doğru mu? (büyük/küçük harfe duyarsızdır, ancak yazım önemlidir)
- Kullanıcı proxy üzerinde gerçekten kimliği doğrulanmış mı?

### "trusted_proxy_missing_header_\*"

Gerekli bir başlık mevcut değildi. Kontrol edin:

- Proxy yapılandırmanızı bu belirli başlıklar için
- Başlıkların zincirin bir yerinde kaldırılıp kaldırılmadığını

### "trusted_proxy_user_not_allowed"

Kullanıcının kimliği doğrulanmış ancak `allowUsers` içinde değil. Ya kullanıcıyı ekleyin ya da izin listesini kaldırın.

### WebSocket Hâlâ Başarısız Oluyor

Proxy’nizin şunları yaptığından emin olun:

- WebSocket yükseltmelerini destekliyor (`Upgrade: websocket`, `Connection: upgrade`)
- WebSocket yükseltme isteklerinde kimlik başlıklarını iletiyor (yalnızca HTTP değil)
- WebSocket bağlantıları için ayrı bir kimlik doğrulama yolu yok

## Token Auth’tan Geçiş

Token auth’tan trusted-proxy’ye geçiyorsanız:

1. Proxy’nizi kullanıcıları doğrulayacak ve başlıkları iletecek şekilde yapılandırın
2. Proxy kurulumunu bağımsız olarak test edin (başlıklarla birlikte curl)
3. OpenClaw yapılandırmasını trusted-proxy auth ile güncelleyin
4. Gateway’i yeniden başlatın
5. Control UI üzerinden WebSocket bağlantılarını test edin
6. `openclaw security audit` çalıştırın ve bulguları inceleyin

## İlgili

- [Security](/gateway/security) — tam güvenlik kılavuzu
- [Configuration](/gateway/configuration) — yapılandırma referansı
- [Remote Access](/gateway/remote) — diğer uzaktan erişim modelleri
- [Tailscale](/gateway/tailscale) — yalnızca tailnet erişimi için daha basit alternatif

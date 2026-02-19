---
summary: "Gateway प्रमाणीकरण को एक विश्वसनीय reverse proxy (Pomerium, Caddy, nginx + OAuth) को सौंपें"
read_when:
  - identity-aware proxy के पीछे OpenClaw चलाना
  - OpenClaw के सामने OAuth के साथ Pomerium, Caddy, या nginx सेट करना
  - reverse proxy सेटअप के साथ WebSocket 1008 unauthorized त्रुटियों को ठीक करना
---

# Trusted Proxy Auth

> ⚠️ **सुरक्षा-संवेदनशील सुविधा।** यह मोड प्रमाणीकरण को पूरी तरह आपके reverse proxy को सौंप देता है। गलत कॉन्फ़िगरेशन आपके Gateway को अनधिकृत पहुँच के लिए उजागर कर सकता है। सक्षम करने से पहले इस पेज को ध्यान से पढ़ें।

## कब उपयोग करें

`trusted-proxy` auth मोड का उपयोग तब करें जब:

- आप OpenClaw को **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth) के पीछे चला रहे हों
- आपका proxy सभी प्रमाणीकरण संभालता है और headers के माध्यम से उपयोगकर्ता पहचान भेजता है
- आप Kubernetes या container वातावरण में हैं जहाँ proxy ही Gateway तक पहुँचने का एकमात्र मार्ग है
- आप WebSocket `1008 unauthorized` त्रुटियों का सामना कर रहे हैं क्योंकि ब्राउज़र WS payload में टोकन नहीं भेज सकते

## कब उपयोग न करें

- यदि आपका proxy उपयोगकर्ताओं का प्रमाणीकरण नहीं करता (सिर्फ एक TLS terminator या load balancer है)
- यदि Gateway तक proxy को बायपास करने वाला कोई भी मार्ग मौजूद है (firewall छिद्र, आंतरिक नेटवर्क पहुँच)
- यदि आपको सुनिश्चित नहीं है कि आपका proxy forwarded headers को सही तरीके से हटाता/ओवरराइट करता है
- यदि आपको केवल व्यक्तिगत single-user पहुँच चाहिए (सरल सेटअप के लिए Tailscale Serve + loopback पर विचार करें)

## यह कैसे काम करता है

1. आपका reverse proxy उपयोगकर्ताओं का प्रमाणीकरण करता है (OAuth, OIDC, SAML, आदि)
2. Proxy प्रमाणित उपयोगकर्ता पहचान के साथ एक header जोड़ता है (उदाहरण: `x-forwarded-user: nick@example.com`)
3. OpenClaw जाँचता है कि अनुरोध एक **trusted proxy IP** से आया है (जिसे `gateway.trustedProxies` में कॉन्फ़िगर किया गया है)
4. OpenClaw कॉन्फ़िगर किए गए header से उपयोगकर्ता पहचान निकालता है
5. यदि सब कुछ सही है, तो अनुरोध अधिकृत किया जाता है

## कॉन्फ़िगरेशन

```json5
{
  gateway: {
    // Must bind to network interface (not loopback)
    bind: "lan",

    // CRITICAL: Only add your proxy's IP(s) here
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header containing authenticated user identity (required)
        userHeader: "x-forwarded-user",

        // Optional: headers that MUST be present (proxy verification)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optional: restrict to specific users (empty = allow all)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### कॉन्फ़िगरेशन संदर्भ

| फ़ील्ड                                      | आवश्यक | विवरण                                                                                             |
| ------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | हाँ    | विश्वसनीय proxy IP पतों की सूची। अन्य IP से आने वाले अनुरोध अस्वीकार कर दिए जाते हैं।             |
| `gateway.auth.mode`                         | हाँ    | यह "trusted-proxy" होना चाहिए                                                                     |
| `gateway.auth.trustedProxy.userHeader`      | हाँ    | प्रमाणित उपयोगकर्ता पहचान वाला हेडर नाम                                                           |
| `gateway.auth.trustedProxy.requiredHeaders` | नहीं   | अतिरिक्त हेडर जो अनुरोध को विश्वसनीय माने जाने के लिए मौजूद होने चाहिए                            |
| `gateway.auth.trustedProxy.allowUsers`      | नहीं   | उपयोगकर्ता पहचानों की अनुमति सूची। खाली होने का अर्थ है सभी प्रमाणित उपयोगकर्ताओं को अनुमति देना। |

## Proxy सेटअप उदाहरण

### Pomerium

Pomerium `x-pomerium-claim-email` (या अन्य claim हेडर) में पहचान पास करता है और `x-pomerium-jwt-assertion` में एक JWT भेजता है।

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium का IP
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

Pomerium कॉन्फ़िग स्निपेट:

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

### OAuth के साथ Caddy

`caddy-security` प्लगइन के साथ Caddy उपयोगकर्ताओं को प्रमाणित कर सकता है और पहचान हेडर पास कर सकता है।

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy का IP (यदि उसी होस्ट पर हो)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile स्निपेट:

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

oauth2-proxy उपयोगकर्ताओं को प्रमाणित करता है और पहचान `x-auth-request-email` में पास करता है।

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy का IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx कॉन्फ़िग स्निपेट:

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

### Forward Auth के साथ Traefik

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik कंटेनर IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## सुरक्षा चेकलिस्ट

trusted-proxy auth सक्षम करने से पहले, सत्यापित करें:

- [ ] **Proxy ही एकमात्र मार्ग है**: Gateway पोर्ट आपके proxy को छोड़कर सभी से फ़ायरवॉल द्वारा सुरक्षित है
- [ ] **trustedProxies न्यूनतम है**: केवल आपके वास्तविक proxy IPs, पूरे सबनेट नहीं
- [ ] **Proxy हेडर हटाता है**: आपका proxy क्लाइंट से आने वाले `x-forwarded-*` हेडर को ओवरराइट करता है (जोड़ता नहीं है)
- [ ] **TLS termination**: आपका proxy TLS संभालता है; उपयोगकर्ता HTTPS के माध्यम से कनेक्ट करते हैं
- [ ] **allowUsers सेट है** (अनुशंसित): किसी भी प्रमाणित उपयोगकर्ता को अनुमति देने के बजाय ज्ञात उपयोगकर्ताओं तक सीमित करें

## सुरक्षा ऑडिट

`openclaw security audit` trusted-proxy auth को **critical** गंभीरता के निष्कर्ष के साथ चिह्नित करेगा। यह जानबूझकर किया गया है — यह याद दिलाने के लिए कि आप सुरक्षा को अपने proxy सेटअप को सौंप रहे हैं।

ऑडिट निम्नलिखित की जाँच करता है:

- `trustedProxies` कॉन्फ़िगरेशन का अभाव
- `userHeader` कॉन्फ़िगरेशन का अभाव
- खाली `allowUsers` (किसी भी प्रमाणित उपयोगकर्ता को अनुमति देता है)

## समस्या निवारण

### "trusted_proxy_untrusted_source"

अनुरोध `gateway.trustedProxies` में शामिल किसी IP से नहीं आया। जाँच करें:

- क्या प्रॉक्सी IP सही है? (Docker कंटेनर IP बदल सकते हैं)
- क्या आपके प्रॉक्सी के सामने कोई लोड बैलेंसर है?
- वास्तविक IP पता करने के लिए `docker inspect` या `kubectl get pods -o wide` का उपयोग करें

### "trusted_proxy_user_missing"

यूज़र हेडर खाली था या अनुपस्थित था। जाँच करें:

- क्या आपका प्रॉक्सी पहचान हेडर पास करने के लिए कॉन्फ़िगर किया गया है?
- क्या हेडर का नाम सही है? (केस-सेंसिटिव नहीं है, लेकिन वर्तनी सही होनी चाहिए)
- क्या उपयोगकर्ता वास्तव में प्रॉक्सी पर प्रमाणित है?

### "trusted_proxy_missing_header_\*"

एक आवश्यक हेडर मौजूद नहीं था। जाँच करें:

- उन विशिष्ट हेडरों के लिए अपने प्रॉक्सी कॉन्फ़िगरेशन की जाँच करें
- क्या कहीं श्रृंखला में हेडर हटाए जा रहे हैं

### "trusted_proxy_user_not_allowed"

उपयोगकर्ता प्रमाणित है लेकिन `allowUsers` में शामिल नहीं है। या तो उन्हें जोड़ें या अनुमति सूची हटाएँ।

### WebSocket अभी भी विफल हो रहा है

सुनिश्चित करें कि आपका प्रॉक्सी:

- WebSocket अपग्रेड का समर्थन करता है (`Upgrade: websocket`, `Connection: upgrade`)
- WebSocket अपग्रेड अनुरोधों पर पहचान हेडर पास करता है (सिर्फ HTTP पर नहीं)
- WebSocket कनेक्शनों के लिए अलग प्रमाणीकरण पथ नहीं रखता

## Token Auth से माइग्रेशन

यदि आप token auth से trusted-proxy पर जा रहे हैं:

1. अपने प्रॉक्सी को उपयोगकर्ताओं को प्रमाणित करने और हेडर पास करने के लिए कॉन्फ़िगर करें
2. प्रॉक्सी सेटअप को स्वतंत्र रूप से परीक्षण करें (हेडरों के साथ curl)
3. trusted-proxy auth के साथ OpenClaw कॉन्फ़िग अपडेट करें
4. Gateway को पुनः प्रारंभ करें
5. Control UI से WebSocket कनेक्शन का परीक्षण करें
6. `openclaw security audit` चलाएँ और निष्कर्षों की समीक्षा करें

## संबंधित

- [Security](/gateway/security) — पूर्ण सुरक्षा मार्गदर्शिका
- [Configuration](/gateway/configuration) — कॉन्फ़िगरेशन संदर्भ
- [Remote Access](/gateway/remote) — अन्य रिमोट एक्सेस पैटर्न
- [Tailscale](/gateway/tailscale) — केवल tailnet एक्सेस के लिए सरल विकल्प


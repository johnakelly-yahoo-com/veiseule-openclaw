---
summary: "Uyandırma ve izole ajan çalıştırmaları için webhook girişi"
read_when:
  - Webhook uç noktaları eklerken veya değiştirirken
  - Harici sistemleri OpenClaw’a bağlarken
title: "Webhook'lar"
---

# Webhook'lar

Gateway, harici tetikleyiciler için küçük bir HTTP webhook uç noktası sunabilir.

## Etkinleştirme

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

Notlar:

- `hooks.token`, `hooks.enabled=true` olduğunda gereklidir.
- `hooks.path` varsayılan olarak `/hooks` değerini alır.

## Kimlik doğrulama

Her isteğin hook belirtecini içermesi gerekir. Tercihen başlıkları kullanın:

- `Authorization: Bearer <token>` (önerilir)
- `x-openclaw-token: <token>`
- Sorgu dizesi belirteçleri reddedilir (`?token=...` `400` döndürür).

## Uç Noktalar

### `POST /hooks/wake`

Yük (payload):

```json
{ "text": "System line", "mode": "now" }
```

- `text` **gerekli** (string): Olayın açıklaması (örn. "Yeni e-posta alındı").
- `mode` isteğe bağlı (`now` | `next-heartbeat`): Anında bir heartbeat tetiklenip tetiklenmeyeceği (varsayılan `now`) ya da bir sonraki periyodik kontrolün beklenmesi.

Etkisi:

- **Ana** oturum için bir sistem olayı kuyruğa alınır
- `mode=now` ise, anında bir heartbeat tetiklenir

### `POST /hooks/agent`

Yük (payload):

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **gerekli** (string): Ajanın işlemesi için istem veya mesaj.
- `name` isteğe bağlı (string): Hook için insan tarafından okunabilir ad (örn. "GitHub"); oturum özetlerinde önek olarak kullanılır.
- `agentId` isteğe bağlıdır (string): Bu hook’u belirli bir agente yönlendirir. Bilinmeyen kimlikler varsayılan agente geri döner. Ayarlandığında, hook çözümlenen agent’in workspace’i ve yapılandırması kullanılarak çalışır.
- `sessionKey` isteğe bağlı (string): Ajanın oturumunu tanımlamak için kullanılan anahtar. Varsayılan olarak bu alan, `hooks.allowRequestSessionKey=true` olmadıkça reddedilir.
- `wakeMode` isteğe bağlı (`now` | `next-heartbeat`): Anında bir heartbeat tetiklenip tetiklenmeyeceği (varsayılan `now`) ya da bir sonraki periyodik kontrolün beklenmesi.
- `deliver` isteğe bağlı (boolean): `true` ise, ajanın yanıtı mesajlaşma kanalına gönderilir. Varsayılan `true`. Yalnızca heartbeat onayları olan yanıtlar otomatik olarak atlanır.
- `channel` isteğe bağlı (string): Teslimat için mesajlaşma kanalı. Şunlardan biri: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (eklenti), `signal`, `imessage`, `msteams`. Varsayılan `last`.
- `to` isteğe bağlı (string): Kanal için alıcı tanımlayıcısı (örn. WhatsApp/Signal için telefon numarası, Telegram için sohbet kimliği, Discord/Slack/Mattermost (eklenti) için kanal kimliği, MS Teams için konuşma kimliği). Varsayılan olarak ana oturumdaki son alıcı.
- `model` isteğe bağlı (string): Model geçersiz kılma (örn. `anthropic/claude-3-5-sonnet` veya bir takma ad). Kısıtlıysa izin verilen model listesinde olmalıdır.
- `thinking` isteğe bağlı (string): Düşünme düzeyi geçersiz kılma (örn. `low`, `medium`, `high`).
- `timeoutSeconds` isteğe bağlı (number): Ajan çalıştırması için saniye cinsinden azami süre.

Etkisi:

- **İzole** bir ajan turu çalıştırır (kendi oturum anahtarı)
- Her zaman **ana** oturuma bir özet gönderir
- `wakeMode=now` ise, anında bir heartbeat tetiklenir

## Oturum anahtarı politikası (geriye dönük uyumsuz değişiklik)

`/hooks/agent` yükündeki `sessionKey` geçersiz kılmaları varsayılan olarak devre dışıdır.

- Önerilen: sabit bir `hooks.defaultSessionKey` ayarlayın ve istek bazlı geçersiz kılmaları kapalı tutun.
- İsteğe bağlı: yalnızca gerektiğinde istek geçersiz kılmalarına izin verin ve önekleri kısıtlayın.

Önerilen yapılandırma:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Uyumluluk yapılandırması (eski davranış):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // şiddetle tavsiye edilir
  },
}
```

### `POST /hooks/<name>` (eşlenmiş)

Özel hook adları `hooks.mappings` üzerinden çözülür (yapılandırmaya bakın). Bir eşleme,
isteğe bağlı şablonlar veya kod dönüştürmeleriyle keyfi yükleri `wake` ya da `agent` eylemlerine dönüştürebilir.

Eşleme seçenekleri (özet):

- `hooks.presets: ["gmail"]` yerleşik Gmail eşlemesini etkinleştirir.
- `hooks.mappings`, yapılandırmada `match`, `action` ve şablonları tanımlamanıza olanak tanır.
- `hooks.transformsDir` + `transform.module`, özel mantık için bir JS/TS modülü yükler.
  - `hooks.transformsDir` (ayarlanmışsa) OpenClaw yapılandırma dizininiz altındaki transforms kök dizini içinde kalmalıdır (genellikle `~/.openclaw/hooks/transforms`).
  - `transform.module`, etkin transforms dizini içinde çözümlenmelidir (dizin geçişi/kaçış yolları reddedilir).
- Genel bir ingest uç noktasını (yük güdümlü yönlendirme) korumak için `match.source` kullanın.
- TS dönüşümleri, çalışma zamanında bir TS yükleyici (örn. `bun` veya `tsx`) ya da önceden derlenmiş `.js` gerektirir.
- Yanıtları bir sohbet yüzeyine yönlendirmek için eşlemelerde `deliver: true` + `channel`/`to` ayarlayın
  (`channel` varsayılan olarak `last`’dir ve WhatsApp’a geri düşer).
- `agentId`, hook’u belirli bir agente yönlendirir; bilinmeyen kimlikler varsayılan agente geri döner.
- `hooks.allowedAgentIds`, açık `agentId` yönlendirmesini kısıtlar. Herhangi bir agente izin vermek için bunu boş bırakın (veya `*` ekleyin). Açık `agentId` yönlendirmesini reddetmek için `[]` ayarlayın.
- `hooks.defaultSessionKey`, açık bir anahtar sağlanmadığında hook agent çalıştırmaları için varsayılan oturumu belirler.
- `hooks.allowRequestSessionKey`, `/hooks/agent` yüklerinin `sessionKey` ayarlayıp ayarlayamayacağını kontrol eder (varsayılan: `false`).
- `hooks.allowedSessionKeyPrefixes`, istek yüklerinden ve eşlemelerden gelen açık `sessionKey` değerlerini isteğe bağlı olarak kısıtlar.
- `allowUnsafeExternalContent: true`, bu hook için harici içerik güvenliği sarmalayıcısını devre dışı bırakır
  (tehlikelidir; yalnızca güvenilir dahili kaynaklar için).
- `openclaw webhooks gmail setup`, `openclaw webhooks gmail run` için `hooks.gmail` yapılandırmasını yazar.
  Tam Gmail izleme akışı için [Gmail Pub/Sub](/automation/gmail-pubsub) bölümüne bakın.

## Yanıtlar

- `/hooks/wake` için `200`
- `/hooks/agent` için `202` (eşzamansız çalıştırma başlatıldı)
- Kimlik doğrulama hatasında `401`
- Aynı istemciden tekrarlanan kimlik doğrulama hatalarından sonra `429` (bkz. `Retry-After`)
- Geçersiz yükte `400`
- Aşırı büyük yüklerde `413`

## Örnekler

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Farklı bir model kullanma

Bu çalıştırma için modeli geçersiz kılmak üzere ajan yüküne (veya eşlemeye) `model` ekleyin:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

`agents.defaults.models` uyguluyorsanız, geçersiz kılınan modelin oraya dahil edildiğinden emin olun.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Güvenlik

- Hook uç noktalarını local loopback, tailnet veya güvenilir bir ters proxy arkasında tutun.
- Ayrı bir hook belirteci kullanın; gateway kimlik doğrulama belirteçlerini yeniden kullanmayın.
- Tekrarlanan kimlik doğrulama hataları, kaba kuvvet denemelerini yavaşlatmak için istemci adresi başına hız sınırlamasına tabi tutulur.
- Çoklu agent yönlendirmesi kullanıyorsanız, açık `agentId` seçimini sınırlamak için `hooks.allowedAgentIds` ayarlayın.
- Çağıran tarafından seçilen oturumlara ihtiyaç duymadığınız sürece `hooks.allowRequestSessionKey=false` olarak bırakın.
- İstek `sessionKey` özelliğini etkinleştirirseniz, `hooks.allowedSessionKeyPrefixes` değerini kısıtlayın (örneğin, `["hook:"]`).
- Webhook günlüklerine hassas ham yükleri dahil etmekten kaçının.
- Hook yükleri varsayılan olarak güvenilmeyen kabul edilir ve güvenlik sınırlarıyla sarılır.
  Belirli bir hook için bunu devre dışı bırakmanız gerekiyorsa, o hook’un eşlemesinde `allowUnsafeExternalContent: true` ayarlayın
  (tehlikelidir).

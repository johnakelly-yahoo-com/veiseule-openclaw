---
summary: "Gateway hizmeti, yaşam döngüsü ve operasyonları için çalıştırma kılavuzu"
read_when:
  - Gateway sürecini çalıştırırken veya hata ayıklarken
title: "Gateway Çalıştırma Kılavuzu"
---

# Gateway hizmeti çalıştırma kılavuzu

Gateway hizmetinin ilk gün başlatma ve ikinci gün operasyonları için bu sayfayı kullanın.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Belirti odaklı tanılama; tam komut adımları ve günlük (log) imzaları ile.
   
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Görev odaklı kurulum kılavuzu + tam yapılandırma referansı.
  
</Card>
</CardGroup>

## 5 dakikalık yerel başlatma

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Sağlıklı temel durum: `Runtime: running` ve `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway yapılandırma yeniden yükleme, etkin yapılandırma dosyası yolunu izler (profil/durum varsayılanlarından çözülür veya ayarlandığında `OPENCLAW_CONFIG_PATH`).
Varsayılan mod: `gateway.reload.mode="hybrid"` (güvenli değişiklikleri anında uygular, kritiklerde yeniden başlatır).
</Note>

## Runtime modeli

- Yönlendirme, kontrol düzlemi ve kanal bağlantıları için her zaman açık tek bir süreç.
- Tek port çoklama.
  - WebSocket kontrol/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - Kontrol UI ve kancalar
- Varsayılan bağlama modu: `loopback`.
- Gateway kimlik doğrulaması varsayılan olarak gereklidir: `gateway.auth.token` (veya `OPENCLAW_GATEWAY_TOKEN`) ya da `gateway.auth.password` ayarlayın.

### Port ve bağlama önceliği

| Ayar          | Çözümleme sırası                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Gateway portu | Port önceliği: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > varsayılan `18789`. |
| Bağlama modu  | CLI/override → `gateway.bind` → `loopback`                                                                               |

### Hot reload modları

| `gateway.reload.mode="off"` ile devre dışı bırakın. | Keepalive davranışı                                       |
| ------------------------------------------------------------------- | --------------------------------------------------------- |
| `off`                                                               | Yapılandırma yeniden yükleme yok                          |
| `hot`                                                               | Yalnızca hot-safe değişiklikleri uygula                   |
| `restart`                                                           | Yeniden yükleme gerektiren değişikliklerde yeniden başlat |
| `hybrid` (varsayılan)                            | Güvenliyse hot uygula, gerekliyse yeniden başlat          |

## Operatör komut seti

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Uzaktan erişim

Tailscale/VPN tercih edilir; aksi halde SSH tüneli:
Yedek seçenek: SSH tüneli.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Ardından istemcileri yerel olarak `ws://127.0.0.1:18789` adresine bağlayın.

<Warning>
Gateway kimlik doğrulaması yapılandırılmışsa, istemciler SSH tünelleri üzerinden bile kimlik doğrulama (`token`/`password`) göndermelidir.
</Warning>

Bkz.: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Denetim ve servis yaşam döngüsü

Üretim benzeri güvenilirlik için denetimli çalıştırmaları kullanın.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
Yeniden başlatmak için `openclaw gateway restart` (veya `launchctl kickstart -k gui/$UID/bot.molt.gateway`) kullanın.
```

OpenClaw.app, Node tabanlı bir gateway rölesi paketleyebilir ve kullanıcı başına bir LaunchAgent kurar; etiketi
`bot.molt.gateway` (veya `bot.molt.<profile>` (adlandırılmış profil). `openclaw doctor` servis yapılandırma sapmalarını denetler ve onarır.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Çıkış yaptıktan sonra kalıcılık için lingering’i etkinleştirin:

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Çok kullanıcılı/her zaman açık ana makineler için bir sistem birimi kullanın.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Birden fazla gateway (aynı ana makine)

Genellikle gereksizdir: tek bir Gateway birden fazla mesajlaşma kanalını ve ajanı sunabilir.
Birden fazla Gateway’i yalnızca yedeklilik veya katı yalıtım için kullanın (örn: kurtarma botu).

Örnek başına kontrol listesi:

- benzersiz `gateway.port`
- benzersiz `OPENCLAW_CONFIG_PATH`
- benzersiz `OPENCLAW_STATE_DIR`
- benzersiz `agents.defaults.workspace`

Örnek:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

[Birden fazla gateway](/gateway/multiple-gateways).

### Dev profili hızlı yol

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

Varsayılanlar, izole durum/yapılandırma ve temel gateway portu `19001` içerir.

## Protokol (operatör görünümü)

- İlk istemci çerçevesi `connect` olmalıdır.
- Gateway, `hello-ok` anlık görüntüsünü (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy) döndürür.
- İstekler: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Yaygın olaylar: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Agent çalıştırmaları iki aşamalıdır:

1. Anında kabul onayı (`status:"accepted"`)
2. `agent` yanıtları iki aşamalıdır: önce `res` onayı `{runId,status:"accepted"}`, ardından çalışma bittikten sonra nihai `res` `{runId,status:"ok"|"error",summary}`; akışlanan çıktı `event:"agent"` olarak gelir.

Tam belgeler: [Gateway protokolü](/gateway/protocol) ve [Köprü protokolü (eski)](/gateway/bridge-protocol).

## Operasyonel kontroller

### Canlılık

- WS’yi açın ve `connect` gönderin.
- Anlık görüntü ile birlikte `hello-ok` yanıtını bekleyin.

### Hazır olma durumu

```bash
`openclaw gateway health|status` — Gateway WS üzerinden sağlık/durum iste.
```

### Boşluk kurtarma

Olaylar tekrar oynatılmaz. İstemciler sıra boşluklarını algılar ve devam etmeden önce yenilemelidir (`health` + `system-presence`).

## Yaygın hata imzaları

| İmza                                                           | Olası sorun                                               |
| -------------------------------------------------------------- | --------------------------------------------------------- |
| `refusing to bind gateway ... `without auth\`                  | Token/parola olmadan loopback dışı bağlama                |
| `another gateway instance is already listening` / `EADDRINUSE` | Port çakışması                                            |
| `Gateway start blocked: set gateway.mode=local`                | Yapılandırma remote moduna ayarlanmış                     |
| Bağlantı sırasında `unauthorized`                              | İstemci ile Gateway arasında kimlik doğrulama uyumsuzluğu |

Tam teşhis adımları için [Gateway Troubleshooting](/gateway/troubleshooting) bölümünü kullanın.

## Güvenlik garantileri

- Doğrudan Baileys bağlantılarına geri dönüş yoktur; Gateway kapalıysa gönderimler hızlıca başarısız olur.
- Bağlanma dışı ilk çerçeveler veya hatalı JSON reddedilir ve soket kapatılır.
- Zarif kapatma, soket kapanmadan önce `shutdown` olayı yayar.

---

İlgili:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)


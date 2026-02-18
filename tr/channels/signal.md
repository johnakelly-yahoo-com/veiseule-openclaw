---
title: "Signal"
---

# Signal (signal-cli)

Durum: harici CLI entegrasyonu. Gateway, HTTP JSON-RPC + SSE üzerinden `signal-cli` ile konuşur.

## Quick setup (beginner)

1. Bot için **ayrı bir Signal numarası** kullanın (önerilir).
2. `signal-cli`’i kurun (Java gereklidir).
3. Bot cihazını bağlayın ve daemon’u başlatın:
   - `signal-cli link -n "OpenClaw"`
4. OpenClaw’ı yapılandırın ve gateway’i başlatın.

Minimal yapılandırma:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

## What it is

- `signal-cli` üzerinden Signal kanalı (gömülü libsignal değildir).
- Deterministik yönlendirme: yanıtlar her zaman Signal’e geri gider.
- DM’ler ajanın ana oturumunu paylaşır; gruplar yalıtılmıştır (`agent:<agentId>:signal:group:<groupId>`).

## Config writes

Varsayılan olarak Signal, `/config set|unset` tarafından tetiklenen yapılandırma güncellemelerini yazmaya izinlidir (`commands.config: true` gerektirir).

Şununla devre dışı bırakın:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## The number model (important)

- Gateway bir **Signal cihazına** ( `signal-cli` hesabı) bağlanır.
- Botu **kişisel Signal hesabınız** üzerinde çalıştırırsanız, kendi mesajlarınızı yok sayar (döngü koruması).
- “Bota yazarım ve o yanıtlar” senaryosu için **ayrı bir bot numarası** kullanın.

## Setup (fast path)

1. `signal-cli`’i kurun (Java gereklidir).
2. Bir bot hesabını bağlayın:
   - `signal-cli link -n "OpenClaw"`, ardından Signal’de QR’ı tarayın.
3. Signal’i yapılandırın ve gateway’i başlatın.

Örnek:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Çoklu hesap desteği: hesap başına yapılandırma ve isteğe bağlı `name` ile `channels.signal.accounts` kullanın. Ortak desen için bkz. [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts).

## External daemon mode (httpUrl)

`signal-cli`’i kendiniz yönetmek istiyorsanız (yavaş JVM soğuk başlatmaları, konteyner başlatma veya paylaşılan CPU’lar), daemon’u ayrı çalıştırın ve OpenClaw’ı ona yönlendirin:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Bu, OpenClaw içindeki otomatik başlatmayı ve başlangıç beklemesini atlar. Otomatik başlatmada yavaş başlangıçlar için `channels.signal.startupTimeoutMs`’i ayarlayın.

## Access control (DMs + groups)

DM’ler:

- Varsayılan: `channels.signal.dmPolicy = "pairing"`.
- Bilinmeyen gönderenler bir eşleştirme kodu alır; onaylanana kadar mesajlar yok sayılır (kodlar 1 saat sonra dolar).
- Onaylama:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Eşleştirme, Signal DM’leri için varsayılan belirteç değişimidir. Ayrıntılar: [Pairing](/channels/pairing)
- UUID‑yalnız gönderenler (`sourceUuid`’ten) `channels.signal.allowFrom` içinde `uuid:<id>` olarak saklanır.

Gruplar:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `allowlist` ayarlandığında, gruplarda kimin tetikleyebileceğini `channels.signal.groupAllowFrom` kontrol eder.

## How it works (behavior)

- `signal-cli` bir daemon olarak çalışır; gateway olayları SSE üzerinden okur.
- Gelen mesajlar paylaşılan kanal zarfına normalize edilir.
- Yanıtlar her zaman aynı numaraya veya gruba yönlendirilir.

## Media + limits

- Giden metin `channels.signal.textChunkLimit`’ye bölünür (varsayılan 4000).
- İsteğe bağlı yeni satır bölme: uzunluk bölmeden önce boş satırlarda (paragraf sınırları) bölmek için `channels.signal.chunkMode="newline"`’ü ayarlayın.
- Ekler desteklenir (`signal-cli`’ten alınan base64).
- Varsayılan medya üst sınırı: `channels.signal.mediaMaxMb` (varsayılan 8).
- Medya indirmeyi atlamak için `channels.signal.ignoreAttachments`’yı kullanın.
- Grup geçmişi bağlamı `channels.signal.historyLimit` (veya `channels.signal.accounts.*.historyLimit`) kullanır, `messages.groupChat.historyLimit`’a geri düşer. Devre dışı bırakmak için `0`’ı ayarlayın (varsayılan 50).

## Typing + read receipts

- **Yazıyor göstergeleri**: OpenClaw, `signal-cli sendTyping` üzerinden yazıyor sinyalleri gönderir ve bir yanıt çalışırken bunları yeniler.
- **Okundu bilgileri**: `channels.signal.sendReadReceipts` true olduğunda, OpenClaw izin verilen DM’ler için okundu bilgilerini iletir.
- Signal-cli gruplar için okundu bilgilerini sunmaz.

## Reactions (message tool)

- `channel=signal` ile birlikte `message action=react`’ü kullanın.
- Hedefler: gönderen E.164 veya UUID (eşleştirme çıktısından `uuid:<id>`’i kullanın; yalın UUID de çalışır).
- `messageId`, tepki verdiğiniz mesajın Signal zaman damgasıdır.
- Grup tepkileri için `targetAuthor` veya `targetAuthorUuid` gerekir.

Örnekler:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Yapılandırma:

- `channels.signal.actions.reactions`: tepki eylemlerini etkinleştir/devre dışı bırak (varsayılan true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack`, ajan tepkilerini devre dışı bırakır (mesaj aracı `react` hata verir).
  - `minimal`/`extensive`, ajan tepkilerini etkinleştirir ve rehberlik seviyesini ayarlar.
- Hesap başına geçersiz kılmalar: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Delivery targets (CLI/cron)

- DM’ler: `signal:+15551234567` (veya yalın E.164).
- UUID DM’ler: `uuid:<id>` (veya yalın UUID).
- Gruplar: `signal:group:<groupId>`.
- Kullanıcı adları: `username:<name>` (Signal hesabınız destekliyorsa).

## Troubleshooting

Önce şu merdiveni çalıştırın:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Gerekirse DM eşleştirme durumunu doğrulayın:

```bash
openclaw pairing list signal
```

Yaygın hatalar:

- Daemon erişilebilir ama yanıt yok: hesap/daemon ayarlarını (`httpUrl`, `account`) ve alma modunu doğrulayın.
- DM’ler yok sayılıyor: gönderen eşleştirme onayı bekliyor.
- Grup mesajları yok sayılıyor: grup göndereni/mention kapılaması teslimi engelliyor.

Triaj akışı için: [/channels/troubleshooting](/channels/troubleshooting).

## Configuration reference (Signal)

Tam yapılandırma: [Configuration](/gateway/configuration)

Sağlayıcı seçenekleri:

- `channels.signal.enabled`: kanal başlatmayı etkinleştir/devre dışı bırak.
- `channels.signal.account`: bot hesabı için E.164.
- `channels.signal.cliPath`: `signal-cli` yolu.
- `channels.signal.httpUrl`: tam daemon URL’si (host/port’u geçersiz kılar).
- `channels.signal.httpHost`, `channels.signal.httpPort`: daemon bağlama (varsayılan 127.0.0.1:8080).
- `channels.signal.autoStart`: daemon’u otomatik başlat ( `httpUrl` ayarlı değilse varsayılan true).
- `channels.signal.startupTimeoutMs`: başlangıç bekleme zaman aşımı (ms) (üst sınır 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: ek indirmelerini atla.
- `channels.signal.ignoreStories`: daemon’dan gelen hikâyeleri yok say.
- `channels.signal.sendReadReceipts`: okundu bilgilerini ilet.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (varsayılan: eşleştirme).
- `channels.signal.allowFrom`: DM izin listesi (E.164 veya `uuid:<id>`). `open`, `"*"` gerektirir. Signal’de kullanıcı adları yoktur; telefon/UUID kimliklerini kullanın.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (varsayılan: izin listesi).
- `channels.signal.groupAllowFrom`: grup gönderen izin listesi.
- `channels.signal.historyLimit`: bağlam olarak eklenecek maksimum grup mesajı (0 devre dışı bırakır).
- `channels.signal.dmHistoryLimit`: kullanıcı dönüşleri cinsinden DM geçmişi sınırı. Kullanıcı bazlı geçersiz kılmalar: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: giden parça boyutu (karakter).
- `channels.signal.chunkMode`: uzunluk bölmeden önce boş satırlarda (paragraf sınırları) bölmek için `length` (varsayılan) veya `newline`.
- `channels.signal.mediaMaxMb`: gelen/giden medya üst sınırı (MB).

İlgili küresel seçenekler:

- `agents.list[].groupChat.mentionPatterns` (Signal yerel mention’ları desteklemez).
- `messages.groupChat.mentionPatterns` (küresel geri dönüş).
- `messages.responsePrefix`.



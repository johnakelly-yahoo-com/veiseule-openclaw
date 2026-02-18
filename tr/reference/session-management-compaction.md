---
summary: "Derinlemesine inceleme: oturum deposu + transkriptler, yaşam döngüsü ve (otomatik) sıkıştırma iç işleyişi"
read_when:
  - Oturum kimliklerini, transkript JSONL dosyalarını veya sessions.json alanlarını hata ayıklamanız gerektiğinde
  - Otomatik sıkıştırma davranışını değiştiriyor ya da “ön sıkıştırma” bakım işleri ekliyorsanız
  - Bellek boşaltmaları veya sessiz sistem turları uygulamak istiyorsanız
title: "Oturum Yönetimine Derinlemesine Bakış"
---

# Oturum Yönetimi ve Sıkıştırma (Derinlemesine İnceleme)

Bu belge, OpenClaw’ın oturumları uçtan uca nasıl yönettiğini açıklar:

- **Oturum yönlendirme** (gelen mesajların bir `sessionKey` ile nasıl eşlendiği)
- **Oturum deposu** (`sessions.json`) ve neyi izlediği
- **Transkript kalıcılığı** (`*.jsonl`) ve yapısı
- **Transkript hijyeni** (çalıştırmalar öncesinde sağlayıcıya özgü düzeltmeler)
- **Bağlam sınırları** (bağlam penceresi ile izlenen token’lar)
- **Sıkıştırma** (manuel + otomatik sıkıştırma) ve ön sıkıştırma çalışmalarının nereye bağlanacağı
- **Sessiz bakım işleri** (örn. kullanıcıya görünür çıktı üretmemesi gereken bellek yazımları)

Önce daha üst düzey bir genel bakış istiyorsanız, şuradan başlayın:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## Tek doğruluk kaynağı: Gateway

OpenClaw, oturum durumunun sahibi olan tek bir **Gateway süreci** etrafında tasarlanmıştır.

- UI’ler (macOS uygulaması, web Control UI, TUI) oturum listeleri ve token sayıları için Gateway’i sorgulamalıdır.
- Uzak modda, oturum dosyaları uzak ana makinededir; “yerel Mac dosyalarınızı kontrol etmek” Gateway’in kullandıklarını yansıtmaz.

---

## İki kalıcılık katmanı

OpenClaw, oturumları iki katmanda kalıcı hale getirir:

1. **Oturum deposu (`sessions.json`)**
   - Anahtar/değer haritası: `sessionKey -> SessionEntry`
   - Küçük, değiştirilebilir, düzenlemesi (veya girdileri silmesi) güvenlidir
   - Oturum meta verilerini izler (geçerli oturum kimliği, son etkinlik, anahtarlar, token sayaçları vb.)

2. **Transkript (`<sessionId>.jsonl`)**
   - Ağaç yapılı, yalnızca eklemeli transkript (girdiler `id` + `parentId` içerir)
   - Gerçek konuşmayı + araç çağrılarını + sıkıştırma özetlerini depolar
   - Gelecek turlar için model bağlamını yeniden oluşturmakta kullanılır

---

## Disk üzerindeki konumlar

Gateway ana makinesinde, ajan başına:

- Depo: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transkriptler: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Telegram konu oturumları: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw bunları `src/config/sessions.ts` üzerinden çözümler.

---

## Oturum anahtarları (`sessionKey`)

Bir `sessionKey`, _hangi konuşma kovasında_ olduğunuzu (yönlendirme + yalıtım) tanımlar.

Yaygın kalıplar:

- Ana/doğrudan sohbet (ajan başına): `agent:<agentId>:<mainKey>` (varsayılan `main`)
- Grup: `agent:<agentId>:<channel>:group:<id>`
- Oda/kanal (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` veya `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (aksi belirtilmedikçe)

Kanonik kurallar [/concepts/session](/concepts/session) adresinde belgelenmiştir.

---

## Oturum kimlikleri (`sessionId`)

Her `sessionKey`, geçerli bir `sessionId`’e (konuşmayı sürdüren transkript dosyası) işaret eder.

Pratik kurallar:

- **Sıfırlama** (`/new`, `/reset`), bu `sessionKey` için yeni bir `sessionId` oluşturur.
- **Günlük sıfırlama** (Gateway ana makinesinde varsayılan yerel saatle 04:00), sıfırlama sınırından sonraki ilk mesajda yeni bir `sessionId` oluşturur.
- **Boşta kalma süresi dolumu** (`session.reset.idleMinutes` veya eski `session.idleMinutes`), boşta penceresinden sonra bir mesaj geldiğinde yeni bir `sessionId` oluşturur. Günlük + boşta birlikte yapılandırıldığında, önce süresi dolan kazanır.

Uygulama detayı: karar, `src/auto-reply/reply/session.ts` içindeki `initSessionState()`’te verilir.

---

## Oturum deposu şeması (`sessions.json`)

Deponun değer türü, `src/config/sessions.ts` içindeki `SessionEntry`’dır.

Temel alanlar (kapsamlı değildir):

- `sessionId`: geçerli transkript kimliği (dosya adı, `sessionFile` ayarlanmadıkça buradan türetilir)
- `updatedAt`: son etkinlik zaman damgası
- `sessionFile`: isteğe bağlı açık transkript yol geçersiz kılma
- `chatType`: `direct | group | room` (UI’lere ve gönderim politikasına yardımcı olur)
- `provider`, `subject`, `room`, `space`, `displayName`: grup/kanal etiketleme için meta veriler
- Anahtarlar:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (oturum başına geçersiz kılma)
- Model seçimi:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Token sayaçları (en iyi çaba / sağlayıcıya bağlı):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: bu oturum anahtarı için otomatik sıkıştırmanın kaç kez tamamlandığı
- `memoryFlushAt`: son ön sıkıştırma bellek boşaltmasının zaman damgası
- `memoryFlushCompactionCount`: son boşaltmanın çalıştığı sıradaki sıkıştırma sayısı

Depo düzenlenmesi güvenlidir; ancak otorite Gateway’dir: oturumlar çalıştıkça girdileri yeniden yazabilir veya yeniden doldurabilir.

---

## Transkript yapısı (`*.jsonl`)

Transkriptler, `@mariozechner/pi-coding-agent`’in `SessionManager`’i tarafından yönetilir.

Dosya JSONL’dir:

- İlk satır: oturum başlığı (`type: "session"`; `id`, `cwd`, `timestamp`, isteğe bağlı `parentSession` içerir)
- Sonrasında: `id` + `parentId` (ağaç) ile oturum girdileri

Öne çıkan girdi türleri:

- `message`: kullanıcı/asistan/toolResult mesajları
- `custom_message`: modele bağlama _giren_ uzantı tarafından enjekte edilen mesajlar (UI’den gizlenebilir)
- `custom`: modele bağlama _girmeyen_ uzantı durumu
- `compaction`: `firstKeptEntryId` ve `tokensBefore` içeren kalıcı sıkıştırma özeti
- `branch_summary`: ağaç dalında gezinirken kalıcı özet

OpenClaw, transkriptleri kasıtlı olarak “düzeltmez”; Gateway, onları okumak/yazmak için `SessionManager`’i kullanır.

---

## Bağlam pencereleri ve izlenen token’lar

İki farklı kavram önemlidir:

1. **Model bağlam penceresi**: model başına katı üst sınır (modelin görebildiği token’lar)
2. **Oturum deposu sayaçları**: `sessions.json` içine yazılan döngüsel istatistikler (/status ve panolar için kullanılır)

Sınırları ayarlarken:

- Bağlam penceresi, model kataloğundan gelir (ve yapılandırma ile geçersiz kılınabilir).
- Depodaki `contextTokens`, çalışma zamanı tahmini/raporlama değeridir; katı bir garanti olarak ele almayın.

Daha fazlası için bkz. [/token-use](/reference/token-use).

---

## Compaction: what it is

Sıkıştırma, daha eski konuşmayı transkriptte kalıcı bir `compaction` girdisi olarak özetler ve yakın mesajları olduğu gibi bırakır.

Sıkıştırmadan sonra, gelecek turlar şunları görür:

- Sıkıştırma özeti
- `firstKeptEntryId`’ten sonraki mesajlar

Compaction is **persistent** (unlike session pruning). [/concepts/session-pruning](/concepts/session-pruning).

---

## Otomatik sıkıştırma ne zaman olur (Pi çalışma zamanı)

Gömülü Pi ajanında, otomatik sıkıştırma iki durumda tetiklenir:

1. **Taşma kurtarma**: model bağlam taşması hatası döndürür → sıkıştır → yeniden dene.
2. **Eşik bakımı**: başarılı bir turdan sonra, şu koşulda:

`contextTokens > contextWindow - reserveTokens`

Burada:

- `contextWindow`, modelin bağlam penceresidir
- `reserveTokens`, istemler + bir sonraki model çıktısı için ayrılan baş boşluktur

Bunlar Pi çalışma zamanı semantikleridir (OpenClaw olayları tüketir, ancak ne zaman sıkıştırılacağına Pi karar verir).

---

## Sıkıştırma ayarları (`reserveTokens`, `keepRecentTokens`)

Pi’nin sıkıştırma ayarları Pi ayarlarında yer alır:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw ayrıca gömülü çalıştırmalar için bir güvenlik tabanı uygular:

- `compaction.reserveTokens < reserveTokensFloor` ise, OpenClaw yükseltir.
- Varsayılan taban `20000` token’dır.
- Tabanı devre dışı bırakmak için `agents.defaults.compaction.reserveTokensFloor: 0` ayarlayın.
- Zaten daha yüksekse, OpenClaw dokunmaz.

Neden: sıkıştırma kaçınılmaz hale gelmeden önce çok turlu “bakım işleri” (bellek yazımları gibi) için yeterli baş boşluğu bırakmak.

Uygulama: `src/agents/pi-settings.ts` içindeki `ensurePiCompactionReserveTokens()`
(`src/agents/pi-embedded-runner.ts`’dan çağrılır).

---

## Kullanıcıya görünen yüzeyler

Sıkıştırmayı ve oturum durumunu şuradan gözlemleyebilirsiniz:

- `/status` (herhangi bir sohbet oturumunda)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Ayrıntılı mod: `🧹 Auto-compaction complete` + sıkıştırma sayısı

---

## Sessiz bakım işleri (`NO_REPLY`)

OpenClaw, kullanıcının ara çıktıları görmemesi gereken arka plan görevleri için “sessiz” turları destekler.

Convention:

- Asistan çıktısına “kullanıcıya yanıt teslim etme” anlamına gelen `NO_REPLY` ile başlar.
- OpenClaw, teslim katmanında bunu ayıklar/bastırır.

`2026.1.10` itibarıyla, OpenClaw ayrıca bir kısmi parça `NO_REPLY` ile başladığında **taslak/yazıyor akışını** da bastırır; böylece sessiz işlemler tur ortasında kısmi çıktı sızdırmaz.

---

## Ön sıkıştırma “bellek boşaltma” (uygulandı)

Amaç: otomatik sıkıştırma gerçekleşmeden önce, kalıcı durumu diske yazan sessiz, ajan temelli bir tur çalıştırmak (örn. ajan çalışma alanındaki `memory/YYYY-MM-DD.md`) ki sıkıştırma kritik bağlamı silemesin.

OpenClaw **eşik öncesi boşaltma** yaklaşımını kullanır:

1. Monitor session context usage.
2. “Yumuşak eşik” (Pi’nin sıkıştırma eşiğinin altında) aşıldığında, ajana sessiz bir
   “şimdi belleği yaz” yönergesi çalıştırır.
3. Kullanıcının hiçbir şey görmemesi için `NO_REPLY` kullanılır.

Yapılandırma (`agents.defaults.compaction.memoryFlush`):

- `enabled` (varsayılan: `true`)
- `softThresholdTokens` (varsayılan: `4000`)
- `prompt` (boşaltma turu için kullanıcı mesajı)
- `systemPrompt` (boşaltma turu için eklenen ekstra sistem istemi)

Notlar:

- Varsayılan istem/sistem istemi, teslimi bastırmak için bir `NO_REPLY` ipucu içerir.
- Boşaltma, her sıkıştırma döngüsünde bir kez çalışır (`sessions.json`’da izlenir).
- Boşaltma yalnızca gömülü Pi oturumları için çalışır (CLI arka uçları atlar).
- Oturum çalışma alanı salt okunur olduğunda boşaltma atlanır (`workspaceAccess: "ro"` veya `"none"`).
- Çalışma alanı dosya düzeni ve yazma kalıpları için bkz. [Memory](/concepts/memory).

Pi, uzantı API’sinde bir `session_before_compact` kancası da sunar; ancak OpenClaw’ın
boşaltma mantığı bugün Gateway tarafında yer alır.

---

## Sorun giderme kontrol listesi

- Oturum anahtarı yanlış mı? [/concepts/session](/concepts/session) ile başlayın ve `/status` içindeki `sessionKey`’ı doğrulayın.
- Depo ile transkript uyuşmazlığı mı? Gateway ana makinesini ve `openclaw status`’den depo yolunu doğrulayın.
- Sıkıştırma spam’i mi? Şunları kontrol edin:
  - model bağlam penceresi (çok küçük)
  - sıkıştırma ayarları (model penceresi için `reserveTokens` çok yüksekse daha erken sıkıştırmaya neden olabilir)
  - tool-result şişmesi: oturum budamayı etkinleştirin/ayarlayın
- Sessiz turlar sızdırıyor mu? Yanıtın `NO_REPLY` (tam belirteç) ile başladığını ve akış bastırma düzeltmesini içeren bir derlemede olduğunuzu doğrulayın.

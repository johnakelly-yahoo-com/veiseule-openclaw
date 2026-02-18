---
title: "Menü Çubuğu"
---

# Menü Çubuğu Durum Mantığı

## Neler gösterilir

- Mevcut ajan çalışma durumu, menü çubuğu simgesinde ve menünün ilk durum satırında gösterilir.
- Çalışma aktifken sağlık durumu gizlenir; tüm oturumlar boşta olduğunda yeniden görünür.
- Menüdeki “Nodes” bloğu, istemci/varlık girdileri değil, yalnızca **cihazları** ( `node.list` aracılığıyla eşleştirilmiş düğümler) listeler.
- Sağlayıcı kullanım anlık görüntüleri mevcut olduğunda Context altında bir “Usage” bölümü görünür.

## Durum modeli

- Oturumlar: olaylar `runId` (çalışma başına) ile birlikte yükte `sessionKey` içerir. “Ana” oturumun anahtarı `main`’tür; yoksa en son güncellenen oturuma geri dönülür.
- Öncelik: ana oturum her zaman kazanır. Ana aktifse durumu hemen gösterilir. Ana boşta ise en son aktif olan ana‑olmayan oturum gösterilir. Etkinlik sırasında ileri‑geri geçiş yapılmaz; yalnızca mevcut oturum boşta olduğunda veya ana aktif hale geldiğinde geçiş yapılır.
- Etkinlik türleri:
  - `job`: üst düzey komut yürütme (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` ve `toolName` ile `meta/args`.

## IconState enum'u (Swift)

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (debug geçersiz kılma)

### ActivityKind → glif

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- varsayılan → 🛠️

### Görsel eşleme

- `idle`: normal yaratık.
- `workingMain`: glifli rozet, tam renk tonu, “çalışıyor” bacak animasyonu.
- `workingOther`: glifli rozet, kısık renk tonu, koşuşturma yok.
- `overridden`: etkinlikten bağımsız olarak seçilen glif/renk tonu kullanılır.

## Durum satırı metni (menü)

- Çalışma aktifken: `<Session role> · <activity label>`
  - Örnekler: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Boştayken: sağlık özetine geri döner.

## Olay alımı

- Kaynak: control‑channel `agent` olayları (`ControlChannel.handleAgentEvent`).
- Ayrıştırılan alanlar:
  - Başlatma/durdurma için `data.state` ile birlikte `stream: "job"`.
  - `data.phase`, `name` ve isteğe bağlı `meta`/`args` ile `stream: "tool"`.
- Etiketler:
  - `exec`: `args.command`’nın ilk satırı.
  - `read`/`write`: kısaltılmış yol.
  - `edit`: `meta`/diff sayımlarından çıkarılan değişiklik türüyle birlikte yol.
  - yedek: araç adı.

## Debug geçersiz kılma

- Ayarlar ▸ Debug ▸ “Icon override” seçici:
  - `System (auto)` (varsayılan)
  - `Working: main` (araç türüne göre)
  - `Working: other` (araç türüne göre)
  - `Idle`
- `@AppStorage("iconOverride")` ile saklanır; `IconState.overridden`’ya eşlenir.

## Test kontrol listesi

- Ana oturum işini tetikleyin: simgenin hemen değiştiğini ve durum satırının ana etiketi gösterdiğini doğrulayın.
- Ana boşta iken ana‑olmayan oturum işini tetikleyin: simge/durum ana‑olmayanı gösterir; bitene kadar sabit kalır.
- Diğeri aktifken anayı başlatın: simge anında ana’ya döner.
- Hızlı araç patlamaları: rozetin titremediğinden emin olun (araç sonuçlarında TTL toleransı).
- Tüm oturumlar boşta olduğunda sağlık satırı yeniden görünür.


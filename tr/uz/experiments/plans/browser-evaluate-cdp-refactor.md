---
summary: "Plan: uçtan uca zaman sınırları ve daha güvenli ref çözümlemesi ile CDP kullanarak browser act:evaluate işlemini Playwright kuyruğundan izole etmek"
owner: "openclaw"
status: "taslak"
last_updated: "2026-02-10"
title: "Tarayıcı Evaluate CDP Yeniden Düzenleme"
---

# Tarayıcı Evaluate CDP Yeniden Düzenleme Planı

## Bağlam

`act:evaluate`, kullanıcı tarafından sağlanan JavaScript kodunu sayfa içinde çalıştırır. Bugün bu işlem Playwright üzerinden çalıştırılıyor
(`page.evaluate` veya `locator.evaluate`). Playwright, CDP komutlarını sayfa başına seri hale getirir; bu nedenle takılan veya uzun süren bir evaluate işlemi sayfa komut kuyruğunu engelleyebilir ve bu sekmedeki sonraki tüm işlemlerin "takılmış" gibi görünmesine neden olabilir.

PR #13498, pragmatik bir güvenlik ağı ekler (sınırlı evaluate, abort yayılımı ve mümkün olan en iyi kurtarma). Bu belge, `act:evaluate` işlemini Playwright'tan doğal olarak izole eden daha kapsamlı bir yeniden düzenlemeyi açıklar; böylece takılan bir evaluate, normal Playwright işlemlerini kilitleyemez.

## Hedefler

- `act:evaluate`, aynı sekmedeki sonraki tarayıcı işlemlerini kalıcı olarak engelleyemez.
- Zaman aşımı, uçtan uca tek bir doğruluk kaynağıdır; böylece çağıran taraf belirlenen bütçeye güvenebilir.
- Abort ve zaman aşımı, HTTP ve süreç içi dispatch genelinde aynı şekilde ele alınır.
- Evaluate için öğe hedefleme, her şeyi Playwright'tan çıkarmadan desteklenir.
- Mevcut çağıranlar ve payload’lar için geriye dönük uyumluluğu korumak.

## Hedef Dışı

- Tüm tarayıcı işlemlerini (click, type, wait vb.) CDP implementasyonlarıyla değiştirmek.
- PR #13498 ile sunulan mevcut güvenlik ağını kaldırmak (kullanışlı bir geri dönüş seçeneği olarak kalır).
- Mevcut `browser.evaluateEnabled` korumasının ötesinde yeni güvensiz yetenekler eklemek.
- Evaluate için süreç izolasyonu (worker process/thread) eklemek. Bu yeniden düzenlemeden sonra hâlâ kurtarması zor takılma durumları görürsek,
  bu bir sonraki adım fikri olacaktır.

## Mevcut Mimari (Neden Takılıyor)

Yüksek seviyede:

- Çağıranlar, `act:evaluate` isteğini tarayıcı kontrol servisine gönderir.
- Route handler, JavaScript’i çalıştırmak için Playwright’ı çağırır.
- Playwright sayfa komutlarını seri hale getirir; bu nedenle hiç tamamlanmayan bir evaluate işlemi kuyruğu engeller.
- Takılan bir kuyruk, sekmedeki sonraki click/type/wait işlemlerinin askıda kalmış gibi görünmesine neden olabilir.

## Önerilen Mimari

### 1. Deadline Yayılımı

Tek bir bütçe kavramı tanıtılır ve her şey bundan türetilir:

- Çağıran taraf `timeoutMs` (veya gelecekte bir deadline) belirler.
- Dış istek zaman aşımı, route handler mantığı ve sayfa içindeki yürütme bütçesi
  aynı bütçeyi kullanır; gerekli yerlerde serileştirme ek yükü için küçük bir pay bırakılır.
- Abort, iptalin tutarlı olması için her yerde bir `AbortSignal` olarak yayılır.

Uygulama yönü:

- Küçük bir yardımcı (örneğin `createBudget({ timeoutMs, signal })`) ekleyin; bu yardımcı şunları döndürür:
  - `signal`: bağlı AbortSignal
  - `deadlineAtMs`: mutlak son tarih
  - `remainingMs()`: alt işlemler için kalan bütçe
- Şunu şu dosyalarda kullanın:
  - `src/browser/client-fetch.ts` (HTTP ve süreç içi yönlendirme)
  - `src/node-host/runner.ts` (proxy yolu)
  - tarayıcı eylemi uygulamaları (Playwright ve CDP)

### 2. Ayrı Evaluate Motoru (CDP Yolu)

Playwright’ın sayfa başına komut kuyruğunu paylaşmayan CDP tabanlı bir evaluate uygulaması ekleyin. Temel özellik, evaluate taşımasının hedefe bağlı ayrı bir WebSocket bağlantısı ve ayrı bir CDP oturumu olmasıdır.

Uygulama yönü:

- Örneğin `src/browser/cdp-evaluate.ts` gibi yeni bir modül:
  - Yapılandırılmış CDP uç noktasına bağlanır (tarayıcı düzeyi soket).
  - Bir `sessionId` almak için `Target.attachToTarget({ targetId, flatten: true })` kullanır.
  - Şunlardan birini çalıştırır:
    - Sayfa düzeyi evaluate için `Runtime.evaluate`, veya
    - Öğe düzeyi evaluate için `DOM.resolveNode` artı `Runtime.callFunctionOn`.
  - Zaman aşımı veya iptal durumunda:
    - Oturum için mümkün olan en iyi şekilde `Runtime.terminateExecution` gönderir.
    - WebSocket bağlantısını kapatır ve net bir hata döndürür.

Notlar:

- Bu işlem hâlâ sayfa içinde JavaScript çalıştırır, bu nedenle sonlandırma yan etkilere neden olabilir. Avantajı
  Playwright kuyruğunu kilitlememesi ve CDP oturumunu sonlandırarak taşıma katmanında iptal edilebilir olmasıdır.

### 3. Ref Hikâyesi (Tam Bir Yeniden Yazım Olmadan Öğe Hedefleme)

Zor kısım öğe hedeflemedir. CDP bir DOM tanıtıcısına veya `backendDOMNodeId` değerine ihtiyaç duyar, oysa
bugün çoğu tarayıcı eylemi snapshot’lardaki ref’lere dayalı Playwright locator’larını kullanır.

Önerilen yaklaşım: mevcut ref’leri koruyun, ancak isteğe bağlı bir CDP çözümlenebilir kimliği ekleyin.

#### 3.1 Saklanan Ref Bilgisini Genişletin

Saklanan role ref meta verisini isteğe bağlı olarak bir CDP kimliği içerecek şekilde genişletin:

- Bugün: `{ role, name, nth }`
- Önerilen: `{ role, name, nth, backendDOMNodeId?: number }`

Bu, mevcut Playwright tabanlı tüm eylemlerin çalışmaya devam etmesini sağlar ve `backendDOMNodeId` mevcut olduğunda CDP evaluate’in aynı `ref` değerini kabul etmesine olanak tanır.

#### 3.2 Snapshot Anında backendDOMNodeId Değerini Doldurun

Bir role snapshot’ı oluştururken:

1. Mevcut role ref haritasını bugün olduğu gibi üretin (role, name, nth).
2. CDP aracılığıyla AX ağacını alın (`Accessibility.getFullAXTree`) ve aynı yinelenen öğe işleme kurallarını kullanarak paralel bir `(role, name, nth) -> backendDOMNodeId` haritası hesaplayın.
3. Kimliği mevcut sekme için saklanan ref bilgisine geri birleştirin.

Bir ref için eşleme başarısız olursa, `backendDOMNodeId` değerini undefined bırakın. Bu, özelliği
en iyi çaba yaklaşımıyla ve güvenli bir şekilde devreye almayı mümkün kılar.

#### 3.3 Ref ile Evaluate Davranışı

`act:evaluate` içinde:

- `ref` mevcutsa ve `backendDOMNodeId` içeriyorsa, CDP üzerinden element evaluate çalıştırın.
- `ref` mevcut ancak `backendDOMNodeId` içermiyorsa, Playwright yoluna geri dönün (güvenlik ağı ile birlikte).

İsteğe bağlı kaçış yolu:

- İstek yapısını, birincil arayüz olarak `ref`’i korurken, ileri seviye çağıranlar (ve hata ayıklama) için doğrudan `backendDOMNodeId` kabul edecek şekilde genişletin.

### 4. Son Çare Kurtarma Yolunu Korumaya Devam Edin

CDP evaluate ile bile, bir sekmeyi veya bağlantıyı kilitlemenin başka yolları vardır. Aşağıdaki durumlar için mevcut kurtarma mekanizmalarını (çalıştırmayı sonlandırma + Playwright bağlantısını kesme) son çare olarak koruyun:

- eski çağıranlar
- CDP bağlantısının engellendiği ortamlar
- beklenmeyen Playwright edge case’leri

## Uygulama Planı (Tek İterasyon)

### Teslimatlar

- Playwright sayfa başına komut kuyruğunun dışında çalışan CDP tabanlı bir evaluate motoru.
- Çağıranlar ve işleyiciler tarafından tutarlı şekilde kullanılan tek bir uçtan uca timeout/abort bütçesi.
- Element evaluate için isteğe bağlı olarak `backendDOMNodeId` taşıyabilen ref meta verisi.
- `act:evaluate` mümkün olduğunda CDP motorunu tercih eder, mümkün olmadığında Playwright’a geri döner.
- Takılı kalan bir evaluate işleminin sonraki aksiyonları kilitlemediğini kanıtlayan testler.
- Hataları ve geri dönüşleri görünür kılan loglar/metrikler.

### Uygulama Kontrol Listesi

1. `timeoutMs` + upstream `AbortSignal` değerlerini aşağıdakilere bağlamak için paylaşılan bir "budget" yardımcı fonksiyonu ekleyin:
   - tek bir `AbortSignal`
   - mutlak bir son teslim zamanı
   - alt işlemler için bir `remainingMs()` yardımcı fonksiyonu
2. Tüm çağıran yollarını bu yardımcıyı kullanacak şekilde güncelleyin, böylece `timeoutMs` her yerde aynı anlama gelsin:
   - `src/browser/client-fetch.ts` (HTTP ve in-process dispatch)
   - `src/node-host/runner.ts` (node proxy yolu)
   - `/act` çağıran CLI sarmalayıcıları (`browser evaluate` için `--timeout-ms` ekleyin)
3. `src/browser/cdp-evaluate.ts` dosyasını uygulayın:
   - tarayıcı seviyesindeki CDP soketine bağlanın
   - bir `sessionId` almak için `Target.attachToTarget`
   - sayfa evaluate için `Runtime.evaluate` çalıştırın
   - element evaluate için `DOM.resolveNode` + `Runtime.callFunctionOn` çalıştırın
   - timeout/abort durumunda: en iyi çabayla `Runtime.terminateExecution`, ardından soketi kapatın
4. Saklanan rol ref meta verisini isteğe bağlı olarak `backendDOMNodeId` içerecek şekilde genişletin:
   - Playwright aksiyonları için mevcut `{ role, name, nth }` davranışını koruyun
   - CDP element hedefleme için `backendDOMNodeId?: number` ekleyin
5. Snapshot oluşturma sırasında `backendDOMNodeId` değerini doldurun (en iyi çaba):
   - CDP üzerinden AX ağacını alın (`Accessibility.getFullAXTree`)
   - `(role, name, nth) -> backendDOMNodeId` eşlemesini hesaplayın ve saklanan ref haritasıyla birleştirin
   - eşleme belirsiz veya eksikse, id değerini undefined bırakın
6. `act:evaluate` yönlendirmesini güncelleyin:
   - `ref` yoksa: her zaman CDP evaluate kullanın
   - `ref` bir `backendDOMNodeId` değerine çözümleniyorsa: CDP element evaluate kullanın
   - aksi takdirde: Playwright evaluate’e geri dönün (yine de sınırlı ve iptal edilebilir)
7. Mevcut "son çare" kurtarma yolunu varsayılan yol değil, yedek olarak tutun.
8. Testler ekleyin:
   - takılı kalan evaluate, ayrılan süre içinde zaman aşımına uğrar ve sonraki click/type işlemi başarılı olur
   - abort, evaluate’i iptal eder (istemci bağlantı kesilmesi veya zaman aşımı) ve sonraki işlemlerin önünü açar
   - eşleme hataları temiz bir şekilde Playwright’a geri döner
9. Gözlemlenebilirlik ekleyin:
   - evaluate süresi ve zaman aşımı sayaçları
   - terminateExecution kullanımı
   - geri dönüş oranı (CDP -> Playwright) ve nedenleri

### Kabul Kriterleri

- Bilinçli olarak askıda bırakılan bir `act:evaluate`, çağıranın ayırdığı süre içinde döner ve
  sekmeyi sonraki işlemler için kilitlemez.
- `timeoutMs`, CLI, agent aracı, node proxy ve süreç içi çağrılar arasında tutarlı davranır.
- `ref`, `backendDOMNodeId` değerine eşlenebiliyorsa element evaluate CDP kullanır; aksi takdirde
  geri dönüş yolu yine sınırlı ve kurtarılabilir olur.

## Test Planı

- Birim testleri:
  - rol referansları ile AX tree düğümleri arasındaki `(role, name, nth)` eşleştirme mantığı.
  - Bütçe yardımcı fonksiyon davranışı (headroom, kalan süre hesaplaması).
- Entegrasyon testleri:
  - CDP evaluate zaman aşımı, ayrılan süre içinde döner ve bir sonraki işlemi engellemez.
  - Abort, evaluate’i iptal eder ve en iyi çaba ile termination tetikler.
- Sözleşme testleri:
  - `BrowserActRequest` ve `BrowserActResponse` uyumluluğunun korunduğundan emin olun.

## Riskler ve Azaltma Yöntemleri

- Eşleme kusurlu olabilir:
  - Azaltma: en iyi çaba eşleme, Playwright evaluate’e geri dönüş ve hata ayıklama araçları ekleme.
- `Runtime.terminateExecution` yan etkilere sahiptir:
  - Azaltma: yalnızca zaman aşımı/abort durumunda kullanın ve davranışı hata mesajlarında belgelendirin.
- Ek yük:
  - Azaltma: AX tree’yi yalnızca snapshot istendiğinde alın, hedef başına önbelleğe alın ve
    CDP oturumunu kısa ömürlü tutun.
- Extension relay sınırlamaları:
  - Azaltma: sayfa başına soketler mevcut olmadığında tarayıcı düzeyinde attach API’lerini kullanın ve
    mevcut Playwright yolunu geri dönüş olarak koruyun.

## Açık Sorular

- Yeni motor `playwright`, `cdp` veya `auto` olarak yapılandırılabilir olmalı mı?
- İleri düzey kullanıcılar için yeni bir "nodeRef" formatı sunmak istiyor muyuz, yoksa yalnızca `ref` mi kalsın?
- Frame snapshot’ları ve selector kapsamlı snapshot’lar AX eşlemesine nasıl dahil olmalı?

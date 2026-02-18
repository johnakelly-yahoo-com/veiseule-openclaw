---
summary: "Sağlayıcı + CLI geri dönüşleriyle gelen görüntü/ses/video anlama (isteğe bağlı)"
read_when:
  - Medya anlama tasarlarken veya yeniden düzenlerken
  - Gelen ses/video/görüntü ön işleme ayarlaması yaparken
title: "Medya Anlama"
---

# Medya Anlama (Gelen) — 2026-01-17

OpenClaw, yanıt hattı çalışmadan önce **gelen medyayı** (görüntü/ses/video) **özetleyebilir**. Yerel araçların veya sağlayıcı anahtarlarının mevcut olup olmadığını otomatik algılar ve devre dışı bırakılabilir ya da özelleştirilebilir. Anlama kapalıysa, modeller her zamanki gibi özgün dosyaları/URL’leri almaya devam eder.

## Hedefler

- İsteğe bağlı: daha hızlı yönlendirme + daha iyi komut ayrıştırma için gelen medyayı kısa metne önceden sindirmek.
- Özgün medya teslimini modele her zaman korumak.
- **Sağlayıcı API’leri** ve **CLI geri dönüşlerini** desteklemek.
- Sıralı geri dönüşle (hata/boyut/zaman aşımı) birden fazla modeli desteklemek.

## Yüksek düzey davranış

1. Gelen ekleri topla (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Etkin her yetenek için (görüntü/ses/video), ekleri politikaya göre seç (varsayılan: **ilk**).
3. Uygun ilk model girdisini seç (boyut + yetenek + yetkilendirme).
4. Bir model başarısız olursa veya medya çok büyükse, **bir sonraki girdiye geri dön**.
5. Başarı durumunda:
   - `Body`, `[Image]`, `[Audio]` veya `[Video]` bloğu olur.
   - Ses, `{{Transcript}}` ayarlar; komut ayrıştırma mevcutsa altyazı metnini,
     aksi halde dökümü kullanır.
   - Altyazılar blok içinde `User text:` olarak korunur.

Anlama başarısız olursa veya devre dışıysa, **yanıt akışı** özgün gövde + eklerle devam eder.

## Yapılandırmaya genel bakış

`tools.media`, **paylaşılan modelleri** ve yetenek başına geçersiz kılmaları destekler:

- `tools.media.models`: paylaşılan model listesi (`capabilities` ile kapıla).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - varsayılanlar (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - sağlayıcı geçersiz kılmaları (`baseUrl`, `headers`, `providerOptions`)
  - `tools.media.audio.providerOptions.deepgram` üzerinden Deepgram ses seçenekleri
  - isteğe bağlı **yetenek başına `models` listesi** (paylaşılan modellerden önce tercih edilir)
  - `attachments` politikası (`mode`, `maxAttachments`, `prefer`)
  - `scope` (kanal/chatType/oturum anahtarına göre isteğe bağlı kapılama)
- `tools.media.concurrency`: eşzamanlı yetenek çalıştırmalarının azami sayısı (varsayılan **2**).

```json5
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### Model girdileri

Her `models[]` girdisi **sağlayıcı** veya **CLI** olabilir:

```json5
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

CLI şablonları ayrıca şunları kullanabilir:

- `{{MediaDir}}` (medya dosyasını içeren dizin)
- `{{OutputDir}}` (bu çalışma için oluşturulan geçici dizin)
- `{{OutputBase}}` (uzantısız geçici dosya temel yolu)

## Varsayılanlar ve sınırlar

Önerilen varsayılanlar:

- `maxChars`: görüntü/video için **500** (kısa, komut dostu)
- `maxChars`: ses için **ayarsız** (bir sınır belirlemediğiniz sürece tam döküm)
- `maxBytes`:
  - görüntü: **10MB**
  - ses: **20MB**
  - video: **50MB**

Kurallar:

- Medya `maxBytes`’yi aşarsa, o model atlanır ve **bir sonraki model denenir**.
- Model `maxChars`’den fazla döndürürse, çıktı kırpılır.
- `prompt` varsayılan olarak basit “{media}’yı tanımla.” ve `maxChars` yönlendirmesini (yalnızca görüntü/video) kullanır.
- `<capability>.enabled: true` ancak model yapılandırılmamışsa, OpenClaw yeteneği desteklediğinde
  **etkin yanıt modelini** dener.

### Medya anlamayı otomatik algılama (varsayılan)

`tools.media.<capability>.enabled`, `false` olarak ayarlanmadıysa ve
modelleri yapılandırmadıysanız, OpenClaw aşağıdaki sırayla otomatik algılar ve **ilk
çalışan seçenekte durur**:

1. **Yerel CLI’lar** (yalnızca ses; yüklüyse)
   - `sherpa-onnx-offline` (`SHERPA_ONNX_MODEL_DIR` gerektirir: encoder/decoder/joiner/tokens)
   - `whisper-cli` (`whisper-cpp`; `WHISPER_CPP_MODEL` veya paketli tiny modeli kullanır)
   - `whisper` (Python CLI; modelleri otomatik indirir)
2. **Gemini CLI** (`gemini`) — `read_many_files` kullanır
3. **Sağlayıcı anahtarları**
   - Ses: OpenAI → Groq → Deepgram → Google
   - Görüntü: OpenAI → Anthropic → Google → MiniMax
   - Video: Google

Otomatik algılamayı devre dışı bırakmak için şunu ayarlayın:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

Not: İkili algılama macOS/Linux/Windows genelinde en iyi çaba esaslıdır; CLI’nin `PATH` üzerinde olduğundan emin olun ( `~`’ü genişletiriz ) veya tam komut yolu ile açık bir CLI modeli ayarlayın.

## Yetenekler (isteğe bağlı)

`capabilities` ayarlarsanız, giriş yalnızca bu medya türleri için çalışır. Paylaşılan
listelerde OpenClaw varsayılanları çıkarım yapabilir:

- `openai`, `anthropic`, `minimax`: **görüntü**
- `google` (Gemini API): **görüntü + ses + video**
- `groq`: **ses**
- `deepgram`: **ses**

CLI girdileri için, **beklenmedik eşleşmeleri önlemek adına `capabilities`’i açıkça ayarlayın**.
`capabilities`’yi atlaranız, giriş göründüğü liste için uygundur.

## Sağlayıcı destek matrisi (OpenClaw entegrasyonları)

| Yetenek | Sağlayıcı entegrasyonu                                    | Notlar                                                                         |
| ------- | --------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Görüntü | OpenAI / Anthropic / Google / `pi-ai` üzerinden diğerleri | Kayıttaki görüntü yetenekli herhangi bir model çalışır.        |
| Ses     | OpenAI, Groq, Deepgram, Google                            | Sağlayıcı dökümü (Whisper/Deepgram/Gemini). |
| Video   | Google (Gemini API)                    | Sağlayıcı video anlama.                                        |

## Önerilen sağlayıcılar

**Görüntü**

- Destekliyorsa etkin modelinizi tercih edin.
- İyi varsayılanlar: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Ses**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` veya `deepgram/nova-3`.
- CLI geri dönüşü: `whisper-cli` (whisper-cpp) veya `whisper`.
- Deepgram kurulumu: [Deepgram (ses dökümü)](/providers/deepgram).

**Video**

- `google/gemini-3-flash-preview` (hızlı), `google/gemini-3-pro-preview` (daha zengin).
- CLI geri dönüşü: `gemini` CLI (`read_file`’i video/ses üzerinde destekler).

## Ek politikası

Yetenek başına `attachments`, hangi eklerin işlendiğini denetler:

- `mode`: `first` (varsayılan) veya `all`
- `maxAttachments`: işlenen sayıyı sınırla (varsayılan **1**)
- `prefer`: `first`, `last`, `path`, `url`

`mode: "all"` olduğunda, çıktılar `[Image 1/2]`, `[Audio 2/2]` vb. olarak etiketlenir.

## Yapılandırma örnekleri

### 1. Paylaşılan modeller listesi + geçersiz kılmalar

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2. Yalnızca Ses + Video (görüntü kapalı)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3. İsteğe bağlı görüntü anlama

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4. Çok kipli tek giriş (açık yetenekler)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## Durum çıktısı

Medya anlama çalıştığında, `/status` kısa bir özet satırı içerir:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Bu, yetenek başına sonuçları ve uygulanabilirse seçilen sağlayıcı/modeli gösterir.

## Notlar

- Anlama **en iyi çaba** esaslıdır. Hatalar yanıtları engellemez.
- Anlama devre dışı olsa bile ekler modellere iletilmeye devam eder.
- Anlamanın nerede çalışacağını sınırlamak için `scope` kullanın (ör. yalnızca DM’ler).

## İlgili belgeler

- [Yapılandırma](/gateway/configuration)
- [Görüntü ve Medya Desteği](/nodes/images)

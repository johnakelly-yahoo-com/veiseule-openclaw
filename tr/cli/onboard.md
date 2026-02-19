---
summary: "`openclaw onboard` için CLI başvuru belgesi (etkileşimli katılım sihirbazı)"
read_when:
  - gateway, çalışma alanı, kimlik doğrulama, kanallar ve Skills için rehberli kurulum istediğinizde
title: "onboard"
---

# `openclaw onboard`

Etkileşimli katılım sihirbazı (yerel veya uzak Gateway kurulumu).

## İlgili kılavuzlar

- CLI katılım merkezi: [Katılım Sihirbazı (CLI)](/start/wizard)
- Onboarding genel bakış: [Onboarding Overview](/start/onboarding-overview)
- CLI otomasyonu: [CLI Otomasyonu](/start/wizard-cli-automation)
- CLI katılım başvurusu: [CLI Katılım Başvurusu](/start/wizard-cli-reference)
- macOS katılımı: [Katılım (macOS Uygulaması)](/start/onboarding)

## Örnekler

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Etkileşimsiz özel sağlayıcı:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` etkileşimsiz modda isteğe bağlıdır. Belirtilmezse, onboarding `CUSTOM_API_KEY` değişkenini kontrol eder.

Etkileşimsiz Z.AI uç nokta seçenekleri:

Not: `--auth-choice zai-api-key` artık anahtarınız için en iyi Z.AI uç noktasını otomatik olarak algılar (`zai/glm-5` ile genel API’yi tercih eder).
Özellikle GLM Coding Plan uç noktalarını istiyorsanız, `zai-coding-global` veya `zai-coding-cn` seçin.

```bash
# İstem olmadan uç nokta seçimi
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Diğer Z.AI uç nokta seçenekleri:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Akış notları:

- `quickstart`: minimum istemler, bir gateway belirteci otomatik oluşturulur.
- `manual`: bağlantı noktası/bağlama/kimlik doğrulama için tam istemler (`advanced` takma adı).
- En hızlı ilk sohbet: `openclaw dashboard` (Kontrol Arayüzü, kanal kurulumu yok).
- Özel Sağlayıcı: Listelenmemiş barındırılan sağlayıcılar dahil, OpenAI veya Anthropic uyumlu herhangi bir uç noktaya bağlanın. Otomatik algılama için Unknown kullanın.

## Yaygın takip komutları

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` etkileşimsiz modu ifade etmez. Betikler için `--non-interactive` kullanın.
</Note>

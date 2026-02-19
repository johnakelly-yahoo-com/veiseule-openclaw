---
summary: "OpenClaw başlangıç seçenekleri ve akışlarına genel bakış"
read_when:
  - Bir başlangıç yolu seçme
  - Yeni bir ortam kurma
title: "Başlangıca Genel Bakış"
sidebarTitle: "Başlangıca Genel Bakış"
---

# Başlangıca Genel Bakış

OpenClaw, Gateway’in nerede çalıştığına ve sağlayıcıları nasıl yapılandırmayı tercih ettiğinize bağlı olarak birden fazla başlangıç yolu sunar.

## Başlangıç yolunuzu seçin

- **CLI sihirbazı** macOS, Linux ve Windows (WSL2 aracılığıyla) için.
- **macOS uygulaması** Apple silicon veya Intel Mac’lerde yönlendirmeli bir ilk kurulum için.

## CLI başlangıç sihirbazı

Sihirbazı bir terminalde çalıştırın:

```bash
openclaw onboard
```

Gateway, çalışma alanı, kanallar ve skills üzerinde tam kontrol istediğinizde CLI sihirbazını kullanın. Belgeler:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS uygulamasıyla başlangıç

macOS üzerinde tamamen yönlendirmeli bir kurulum istediğinizde OpenClaw uygulamasını kullanın. Belgeler:

- [Onboarding (macOS App)](/start/onboarding)

## Özel Sağlayıcı

Listede yer almayan bir uç noktaya ihtiyacınız varsa, standart OpenAI veya Anthropic API’lerini sunan barındırılan sağlayıcılar dahil, CLI sihirbazında **Custom Provider** seçeneğini seçin. Şunları yapmanız istenecek:

- OpenAI-compatible, Anthropic-compatible veya **Unknown** (otomatik algıla) seçin.
- Bir temel URL ve API anahtarı girin (sağlayıcı gerektiriyorsa).
- Bir model ID ve isteğe bağlı bir takma ad sağlayın.
- Bir Endpoint ID seçin; böylece birden fazla özel endpoint birlikte var olabilir.

Ayrıntılı adımlar için yukarıdaki CLI onboarding belgelerini izleyin.

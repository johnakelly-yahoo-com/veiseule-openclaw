---
summary: "OpenClaw’ı rootless bir Podman container içinde çalıştırın"
read_when:
  - Docker yerine Podman ile konteynerleştirilmiş bir gateway istiyorsunuz
title: "Podman"
---

# Podman

OpenClaw gateway’i **rootless** bir Podman konteynerinde çalıştırın. Docker ile aynı imajı kullanır (repo içindeki [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) üzerinden derleyin).

## Gereksinimler

- Podman (rootless)
- Tek seferlik kurulum için Sudo (kullanıcı oluşturma, imaj derleme)

## Hızlı başlangıç

**1. Tek seferlik kurulum** (repo kök dizininden; kullanıcı oluşturur, imajı derler, başlatma betiğini yükler):

```bash
./setup-podman.sh
```

Bu işlem ayrıca minimal bir `~openclaw/.openclaw/openclaw.json` dosyası oluşturur (`gateway.mode="local"` olarak ayarlanır), böylece sihirbazı çalıştırmadan gateway başlatılabilir.

Varsayılan olarak konteyner bir systemd servisi olarak kurulmaz, manuel olarak başlatırsınız (aşağıya bakın). Otomatik başlatma ve yeniden başlatmalar içeren üretim tarzı bir kurulum için bunun yerine systemd Quadlet kullanıcı servisi olarak kurun:

```bash
./setup-podman.sh --quadlet
```

(Veya `OPENCLAW_PODMAN_QUADLET=1` ayarlayın; yalnızca konteyneri ve başlatma betiğini kurmak için `--container` kullanın.)

**2. Gateway’i başlatın** (manuel, hızlı bir smoke testi için):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding sihirbazı** (örneğin kanal veya sağlayıcı eklemek için):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Ardından `http://127.0.0.1:18789/` adresini açın ve `~openclaw/.openclaw/.env` içindeki token’ı (veya kurulum sırasında yazdırılan değeri) kullanın.

## Systemd (Quadlet, isteğe bağlı)

`./setup-podman.sh --quadlet` (veya `OPENCLAW_PODMAN_QUADLET=1`) çalıştırdıysanız, gateway’in openclaw kullanıcısı için bir systemd kullanıcı servisi olarak çalışması amacıyla bir [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) birimi kurulur. Servis, kurulumun sonunda etkinleştirilir ve başlatılır.

- **Başlat:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Durdur:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Durum:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Loglar:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Quadlet dosyası `~openclaw/.config/containers/systemd/openclaw.container` konumundadır. Portları veya ortam değişkenlerini değiştirmek için bu dosyayı (veya kaynak aldığı `.env` dosyasını) düzenleyin, ardından `sudo systemctl --machine openclaw@ --user daemon-reload` çalıştırın ve servisi yeniden başlatın. Sistem açılışında, openclaw için lingering etkinleştirilmişse servis otomatik olarak başlar (loginctl mevcutsa kurulum bunu yapar).

Quadlet’i, başlangıçta kullanılmamış bir kurulumdan **sonra** eklemek için şu komutu yeniden çalıştırın: `./setup-podman.sh --quadlet`.

## openclaw kullanıcısı (girişsiz)

`setup-podman.sh`, `openclaw` adında özel bir sistem kullanıcısı oluşturur:

- **Kabuk:** `nologin` — etkileşimli giriş yoktur; saldırı yüzeyini azaltır.

- **Ana dizin:** örn. `/home/openclaw` — `~/.openclaw` (yapılandırma, çalışma alanı) ve `run-openclaw-podman.sh` başlatma betiğini barındırır.

- **Rootless Podman:** Kullanıcının bir **subuid** ve **subgid** aralığına sahip olması gerekir. Birçok distro, kullanıcı oluşturulduğunda bunları otomatik olarak atar. Kurulum bir uyarı yazdırırsa, `/etc/subuid` ve `/etc/subgid` dosyalarına satırlar ekleyin:

  ```text
  openclaw:100000:65536
  ```

  Ardından gateway’i o kullanıcı olarak başlatın (örneğin cron veya systemd üzerinden):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Yapılandırma:** `/home/openclaw/.openclaw` dizinine yalnızca `openclaw` ve root erişebilir. Yapılandırmayı düzenlemek için: gateway çalıştıktan sonra Control UI’ı kullanın veya `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json` komutunu çalıştırın.

## Ortam ve yapılandırma

- **Token:** `~openclaw/.openclaw/.env` içinde `OPENCLAW_GATEWAY_TOKEN` olarak saklanır. `setup-podman.sh` ve `run-openclaw-podman.sh` eksikse bunu oluşturur (`openssl`, `python3` veya `od` kullanır).
- **İsteğe bağlı:** Bu `.env` dosyasında sağlayıcı anahtarlarını (ör. `GROQ_API_KEY`, `OLLAMA_API_KEY`) ve diğer OpenClaw ortam değişkenlerini ayarlayabilirsiniz.
- **Host portları:** Varsayılan olarak script `18789` (gateway) ve `18790` (bridge) portlarını eşler. Başlatma sırasında **host** port eşlemesini `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` ve `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` ile geçersiz kılabilirsiniz.
- **Yollar:** Host yapılandırma ve çalışma alanı varsayılan olarak `~openclaw/.openclaw` ve `~openclaw/.openclaw/workspace` dizinleridir. Başlatma scriptinin kullandığı host yollarını `OPENCLAW_CONFIG_DIR` ve `OPENCLAW_WORKSPACE_DIR` ile geçersiz kılabilirsiniz.

## Kullanışlı komutlar

- **Loglar:** Quadlet ile: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Script ile: `sudo -u openclaw podman logs -f openclaw`
- **Durdur:** Quadlet ile: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Script ile: `sudo -u openclaw podman stop openclaw`
- **Yeniden başlat:** Quadlet ile: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Script ile: başlatma scriptini yeniden çalıştırın veya `podman start openclaw` komutunu kullanın
- **Container’ı kaldır:** `sudo -u openclaw podman rm -f openclaw` — host üzerindeki yapılandırma ve çalışma alanı korunur

## Sorun Giderme

- **Yapılandırma veya auth-profiles üzerinde Permission denied (EACCES):** Container varsayılan olarak `--userns=keep-id` kullanır ve scripti çalıştıran host kullanıcısıyla aynı uid/gid ile çalışır. Host üzerindeki `OPENCLAW_CONFIG_DIR` ve `OPENCLAW_WORKSPACE_DIR` dizinlerinin bu kullanıcıya ait olduğundan emin olun.
- **Gateway başlatma engellendi (eksik `gateway.mode=local`):** `~openclaw/.openclaw/openclaw.json` dosyasının mevcut olduğundan ve `gateway.mode="local"` ayarını içerdiğinden emin olun. `setup-podman.sh` bu dosya eksikse oluşturur.
- **Rootless Podman openclaw kullanıcısı için başarısız oluyor:** `/etc/subuid` ve `/etc/subgid` dosyalarında `openclaw` için bir satır bulunduğunu kontrol edin (ör. `openclaw:100000:65536`). Eksikse ekleyin ve yeniden başlatın.
- **Container adı kullanımda:** Başlatma scripti `podman run --replace` kullanır, bu nedenle yeniden başlattığınızda mevcut container değiştirilir. Manuel olarak temizlemek için: `podman rm -f openclaw`.
- **openclaw kullanıcısı olarak çalıştırırken script bulunamadı:** `run-openclaw-podman.sh` dosyasının openclaw kullanıcısının home dizinine (ör. `/home/openclaw/run-openclaw-podman.sh`) kopyalanması için `setup-podman.sh` komutunun çalıştırıldığından emin olun.
- **Quadlet servisi bulunamadı veya başlatılamıyor:** `.container` dosyasını düzenledikten sonra `sudo systemctl --machine openclaw@ --user daemon-reload` komutunu çalıştırın. Quadlet cgroups v2 gerektirir: `podman info --format '{{.Host.CgroupsVersion}}'` çıktısı `2` göstermelidir.

## İsteğe bağlı: kendi kullanıcınız olarak çalıştırma

Gateway’i normal kullanıcınız olarak (ayrı bir openclaw kullanıcısı olmadan) çalıştırmak için: imajı oluşturun, `~/.openclaw/.env` içinde `OPENCLAW_GATEWAY_TOKEN` oluşturun ve container’ı `--userns=keep-id` ve `~/.openclaw` dizininize mount ederek çalıştırın. Başlatma scripti openclaw-kullanıcı akışı için tasarlanmıştır; tek kullanıcılı bir kurulumda bunun yerine scriptteki `podman run` komutunu manuel olarak çalıştırabilir, yapılandırma ve çalışma alanını kendi home dizininize yönlendirebilirsiniz. Çoğu kullanıcı için önerilen: `setup-podman.sh` kullanın ve yapılandırma ile sürecin izole edilmesi için openclaw kullanıcısı olarak çalıştırın.

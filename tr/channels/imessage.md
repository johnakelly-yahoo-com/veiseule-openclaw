---
summary: "imsg üzerinden eski iMessage desteği (stdio üzerinde JSON-RPC). Yeni kurulumlar BlueBubbles kullanmalıdır."
read_when:
  - iMessage desteğini kurma
  - iMessage gönderme/alma hata ayıklama
title: "iMessage"
---

# iMessage (eski: imsg)

<Warning>
**Önerilen:** Yeni iMessage kurulumları için [BlueBubbles](/channels/bluebubbles) kullanın.

`imsg` kanalı, eski bir harici CLI entegrasyonudur ve gelecekteki bir sürümde kaldırılabilir. 
</Warning>

Durum: eski harici CLI entegrasyonu. Gateway, `imsg rpc`’ü (stdio üzerinde JSON-RPC) başlatır.

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Yeni kurulumlar için tercih edilen iMessage yolu.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">Eşleştirme, iMessage DM’leri için varsayılan belirteç değişimidir.
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    iMessage alanları için tam referans.
  
</Card>
</CardGroup>

## Hızlı kurulum (başlangıç)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="OpenClaw Yapılandır">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Gateway’i başlat">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="İlk DM eşleştirmesini onayla (varsayılan dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            Eşleştirme istekleri 1 saat sonra sona erer.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">iMessage’ı başka bir Mac’te istiyorsanız, `channels.imessage.cliPath`’yi uzak macOS ana makinesinde SSH üzerinden `imsg`’i çalıştıran bir sarmalayıcıya ayarlayın. OpenClaw yalnızca stdio’ya ihtiyaç duyar.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Eklerin etkin olduğu durumlar için önerilen yapılandırma:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    `remoteHost` ayarlanmazsa, OpenClaw sarmalayıcı betiğinizdeki SSH komutunu ayrıştırarak otomatik algılamaya çalışır.
    ```

  
</Tab>
</Tabs>

## İlgili macOS klasör izinleri (Masaüstü/Belgeler/İndirilenler): [/platforms/mac/permissions](/platforms/mac/permissions).

- Messages’ta oturum açılmış macOS.
- OpenClaw + `imsg` için Tam Disk Erişimi (Messages DB erişimi).
- Messages.app üzerinden mesaj göndermek için Automation izni gereklidir.

<Tip>
İzinler her işlem bağlamı için ayrı ayrı verilir. Gateway başsız (LaunchAgent/SSH) çalışıyorsa, istemleri tetiklemek için aynı bağlamda tek seferlik etkileşimli bir komut çalıştırın:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Erişim kontrolü ve yönlendirme

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` doğrudan mesajları kontrol eder:


    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (varsayılan: izin listesi).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` grup işlemlerini kontrol eder:


    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DM’ler doğrudan yönlendirme kullanır; gruplar grup yönlendirmesi kullanır.
    - Varsayılan `session.dmScope=main` ile iMessage DM’leri ajan ana oturumunda birleştirilir.
    - Grup oturumları izole edilir (`agent:<agentId>Gruplar:<chat_id>`).
    - Yanıtlar, kaynak kanal/hedef meta verileri kullanılarak iMessage’a geri yönlendirilir.

    ```
    Bazı iMessage iş parçacıkları birden fazla katılımcıya sahip olabilir ancak Messages’ın sohbet tanımlayıcıyı nasıl sakladığına bağlı olarak yine de `is_group=false` ile gelebilir.
    ```

  
</Tab>
</Tabs>

## Dağıtım desenleri

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Botun **ayrı bir iMessage kimliğinden** göndermesini (ve kişisel Messages’ınızı temiz tutmayı) istiyorsanız, özel bir Apple ID + özel bir macOS kullanıcısı kullanın.

    ```
    Bu macOS kullanıcısında Messages’ı açın ve bot Apple ID’siyle iMessage’a giriş yapın.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Yaygın topoloji:


    ```
    Gateway bir Linux ana makinesinde/VM’de çalışıyor ancak iMessage’ın bir Mac’te çalışması gerekiyorsa, Tailscale en basit köprüdür: Gateway, tailnet üzerinden Mac ile konuşur, SSH üzerinden `imsg`’i çalıştırır ve ekleri SCP ile geri alır.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    `ssh bot@mac-mini.tailnet-1234.ts.net`’nin istemler olmadan çalışması için SSH anahtarlarını kullanın.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">macOS’te `imsg` tarafından desteklenen iMessage kanalı.

    ```
    Her hesap `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` ve geçmiş ayarları gibi alanları geçersiz kılabilir.
    ```

  
</Accordion>
</AccordionGroup>

## Medya, parçalama ve teslim hedefleri

<AccordionGroup>
  <Accordion title="Attachments and media">Medya yüklemeleri `channels.imessage.mediaMaxMb` ile sınırlandırılır (varsayılan 16).
</Accordion>

  <Accordion title="Outbound chunking">`channels.imessage.chunkMode`: uzunluk parçalamadan önce boş satırlarda (paragraf sınırları) bölmek için `length` (varsayılan) veya `newline`.
</Accordion>

  <Accordion title="Addressing formats">
    Tercih edilen açık hedefler:


    ```
    - `chat_id:123` (kararlı yönlendirme için önerilir)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle hedefleri de desteklenir:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Yapılandırma yazımları

Varsayılan olarak iMessage, `/config set|unset` tarafından tetiklenen yapılandırma güncellemelerini yazmaya izinlidir ( `commands.config: true` gerektirir).

Devre dışı bırak:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Sorun Giderme

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    İkili dosyayı ve RPC desteğini doğrulayın:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Probe RPC’nin desteklenmediğini bildirirse, `imsg`’yi güncelleyin.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Notlar:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (varsayılan: eşleştirme).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">Kontrol listesi:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Onaylama:

    ```
    `channels.imessage.accounts.bot.cliPath`’ı, bot kullanıcısı olarak `imsg`’i çalıştıran bir SSH sarmalayıcısına yönlendirin.
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">İstemi zorlamak için bir GUI terminalinde tek seferlik etkileşimli bir komut çalıştırın, ardından yeniden deneyin:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Gateway’i başlatın ve macOS istemlerini (Otomasyon + Tam Disk Erişimi) onaylayın.
    ```

  
</Accordion>
</AccordionGroup>

## Yapılandırma referans noktaları

- iMessage’ı yapılandırın ve gateway’i başlatın.
- Tam yapılandırma: [Yapılandırma](/gateway/configuration)
- Ayrıntılar: [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)


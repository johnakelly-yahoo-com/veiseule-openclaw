---
summary: "Hooks: komutlar ve yaşam döngüsü olayları için olay güdümlü otomasyon"
read_when:
  - /new, /reset, /stop ve ajan yaşam döngüsü olayları için olay güdümlü otomasyon istiyorsunuz
  - Hook’ları oluşturmak, yüklemek veya hata ayıklamak istiyorsunuz
title: "Kancalar"
---

# Kancalar

Hooks, ajan komutları ve olaylarına yanıt olarak eylemleri otomatikleştirmek için genişletilebilir, olay güdümlü bir sistem sağlar. Hooks dizinlerden otomatik olarak keşfedilir ve Skills’in OpenClaw’da çalışmasına benzer şekilde CLI komutlarıyla yönetilebilir.

## Yön Bulma

Hooks, bir şey olduğunda çalışan küçük betiklerdir. İki tür vardır:

- **Hooks** (bu sayfa): `/new`, `/reset`, `/stop` gibi ajan olayları veya yaşam döngüsü olayları tetiklendiğinde Gateway içinde çalışır.
- **Webhooks**: Diğer sistemlerin OpenClaw’da iş tetiklemesine olanak tanıyan harici HTTP webhook’larıdır. [Webhook Hooks](/automation/webhook) bölümüne bakın veya Gmail yardımcı komutları için `openclaw webhooks` kullanın.

Hooks, eklentilerin (plugins) içine de paketlenebilir; bkz. [Plugins](/tools/plugin#plugin-hooks).

Yaygın kullanımlar:

- Bir oturumu sıfırladığınızda bellek anlık görüntüsünü kaydetmek
- Sorun giderme veya uyumluluk için komutların denetim kaydını tutmak
- Bir oturum başladığında veya bittiğinde takip otomasyonunu tetikleme
- Olaylar tetiklendiğinde ajan çalışma alanına dosya yazmak veya harici API’leri çağırmak

Küçük bir TypeScript fonksiyonu yazabiliyorsanız, bir hook yazabilirsiniz. Hooks otomatik olarak keşfedilir ve CLI üzerinden etkinleştirip devre dışı bırakabilirsiniz.

## Genel Bakış

Hooks sistemi şunları yapmanıza olanak tanır:

- `/new` verildiğinde oturum bağlamını belleğe kaydetmek
- Denetim için tüm komutları kaydetmek
- Ajan yaşam döngüsü olaylarında özel otomasyonları tetikleme
- Çekirdek kodu değiştirmeden OpenClaw davranışını genişletmek

## Başlarken

### Paketli Hooks

OpenClaw, otomatik olarak keşfedilen dört paketli hook ile gelir:

- **💾 session-memory**: `/new` verdiğinizde, oturum bağlamını ajan çalışma alanınıza (varsayılan `~/.openclaw/workspace/memory/`) kaydeder
- **😈 soul-evil**: Bir temizleme penceresinde veya rastgele bir olasılıkla enjekte edilen `SOUL.md` içeriğini `SOUL_EVIL.md` ile değiştirir
- **📝 command-logger**: Tüm komut olaylarını `~/.openclaw/logs/commands.log` dosyasına kaydeder
- **🚀 boot-md**: Gateway başladığında `BOOT.md` çalıştırır (dahili hooks etkin olmalıdır)

Mevcut hook’ları listeleme:

```bash
openclaw hooks list
```

Bir hook’u etkinleştirme:

```bash
openclaw hooks enable session-memory
```

Hook durumunu kontrol etme:

```bash
openclaw hooks check
```

Ayrıntılı bilgi alma:

```bash
openclaw hooks info session-memory
```

### Onboarding

Onboarding sırasında (`openclaw onboard`), önerilen hook’ları etkinleştirmeniz istenir. Sihirbaz, uygun hook’ları otomatik olarak keşfeder ve seçim için sunar.

## Hook Keşfi

Hooks, öncelik sırasına göre üç dizinden otomatik olarak keşfedilir:

1. **Çalışma alanı hook’ları**: `<workspace>/hooks/` (ajan başına, en yüksek öncelik)
2. **Yönetilen hook’lar**: `~/.openclaw/hooks/` (kullanıcı tarafından yüklenen, çalışma alanları arasında paylaşılan)
3. **Paketli hook’lar**: `<openclaw>/dist/hooks/bundled/` (OpenClaw ile birlikte gelir)

Yönetilen hook dizinleri **tek bir hook** veya **hook paketi** (paket dizini) olabilir.

Her hook, aşağıdakileri içeren bir dizindir:

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## Hook Paketleri (npm/arşivler)

Hook paketleri, `package.json` içindeki
`openclaw.hooks` aracılığıyla bir veya daha fazla hook dışa aktaran standart npm paketleridir. Şu şekilde yükleyin:

```bash
openclaw hooks install <path-or-spec>
```

Npm tanımları yalnızca registry tabanlıdır (paket adı + isteğe bağlı sürüm/etiket). Git/URL/dosya tanımları reddedilir.

Örnek `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Her giriş, `HOOK.md` ve `handler.ts` (veya `index.ts`) içeren bir hook dizinine işaret eder.
Hook paketleri bağımlılıklarıyla birlikte gelebilir; bunlar `~/.openclaw/hooks/<id>` altında yüklenecektir.

Güvenlik notu: `openclaw hooks install`, bağımlılıkları `npm install --ignore-scripts` ile kurar
(yaşam döngüsü betikleri olmadan). Hook paket bağımlılık ağaçlarını "pure JS/TS" olarak tutun ve `postinstall` derlemelerine dayanan paketlerden kaçının.

## Hook Yapısı

### HOOK.md Biçimi

`HOOK.md` dosyası, YAML frontmatter içindeki meta veriler ile Markdown dokümantasyonu içerir:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### Meta Veri Alanları

`metadata.openclaw` nesnesi şunları destekler:

- **`emoji`**: CLI için görüntü emojisi (örn. `"💾"`)
- **`events`**: Dinlenecek olaylar dizisi (örn. `["command:new", "command:reset"]`)
- **`export`**: Kullanılacak adlandırılmış dışa aktarım (varsayılan `"default"`)
- **`homepage`**: Dokümantasyon URL’si
- **`requires`**: İsteğe bağlı gereksinimler
  - **`bins`**: PATH üzerinde gerekli ikililer (örn. `["git", "node"]`)
  - **`anyBins`**: Bu ikililerden en az biri mevcut olmalıdır
  - **`env`**: Gerekli ortam değişkenleri
  - **`config`**: Gerekli yapılandırma yolları (örn. `["workspace.dir"]`)
  - **`os`**: Gerekli platformlar (örn. `["darwin", "linux"]`)
- **`always`**: Uygunluk denetimlerini atla (boolean)
- **`install`**: Yükleme yöntemleri (paketli hook’lar için: `[{"id":"bundled","kind":"bundled"}]`)

### İşleyici (Handler) Uygulaması

`handler.ts` dosyası bir `HookHandler` fonksiyonu dışa aktarır:

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const myHandler: HookHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### Olay Bağlamı

Her olay şunları içerir:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

## Olay Türleri

### Komut Olayları

Ajan komutları verildiğinde tetiklenir:

- **`command`**: Tüm komut olayları (genel dinleyici)
- **`command:new`**: `/new` komutu verildiğinde
- **`command:reset`**: `/reset` komutu verildiğinde
- **`command:stop`**: `/stop` komutu verildiğinde

### Ajan Olayları

- **`agent:bootstrap`**: Çalışma alanı bootstrap dosyaları enjekte edilmeden önce (hook’lar `context.bootstrapFiles`’i değiştirebilir)

### Gateway Olayları

Gateway başladığında tetiklenir:

- **`gateway:startup`**: Kanallar başladıktan ve hook’lar yüklendikten sonra

### Araç Sonucu Hook’ları (Plugin API)

Bu hook’lar olay akışı dinleyicileri değildir; eklentilerin, OpenClaw bunları kalıcı hale getirmeden önce araç sonuçlarını eşzamanlı olarak ayarlamasına olanak tanır.

- **`tool_result_persist`**: Oturum dökümüne yazılmadan önce araç sonuçlarını dönüştürür. Eşzamanlı olmalıdır; güncellenmiş araç sonucu yükünü döndürün veya olduğu gibi bırakmak için `undefined` döndürün. [Agent Loop](/concepts/agent-loop).

### Gelecek Olaylar

Planlanan olay türleri:

- **`session:start`**: Yeni bir oturum başladığında
- **`session:end`**: Bir oturum sona erdiğinde
- **`agent:error`**: Bir ajan hata ile karşılaştığında
- **`message:sent`**: Bir mesaj gönderildiğinde
- **`message:received`**: Bir mesaj alındığında

## Özel Hook’lar Oluşturma

### 1. Konum Seçin

- **Çalışma alanı hook’ları** (`<workspace>/hooks/`): Ajan başına, en yüksek öncelik
- **Yönetilen hook’lar** (`~/.openclaw/hooks/`): Çalışma alanları arasında paylaşılan

### 2. Dizin Yapısını Oluşturun

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. HOOK.md Oluşturun

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. handler.ts Oluşturun

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const handler: HookHandler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // Your logic here
};

export default handler;
```

### 5. Etkinleştirin ve Test Edin

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```

## Yapılandırma

### Yeni Yapılandırma Biçimi (Önerilen)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Hook Başına Yapılandırma

Hook’ların özel yapılandırması olabilir:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Ek Dizinler

Ek dizinlerden hook yükleyin:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Eski Yapılandırma Biçimi (Hâlâ Desteklenir)

Geriye dönük uyumluluk için eski yapılandırma biçimi hâlâ çalışır:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

Not: `module`, workspace’e göreli bir yol olmalıdır. Mutlak yollar ve workspace dışına geçişler reddedilir.

**Geçiş**: Yeni hook’lar için keşfe dayalı yeni sistemi kullanın. Eski işleyiciler, dizin tabanlı hook’lardan sonra yüklenir.

## CLI Komutları

### Hook’ları Listeleme

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Verbose output (show missing requirements)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

### Hook Bilgisi

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### Uygunluğu Kontrol Etme

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### Etkinleştir/Devre Dışı Bırak

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```

## Paketlenmiş kanca referansı

### session-memory

`/new` verdiğinizde oturum bağlamını belleğe kaydeder.

**Olaylar**: `command:new`

**Gereksinimler**: `workspace.dir` yapılandırılmış olmalıdır

**Çıktı**: `<workspace>/memory/YYYY-MM-DD-slug.md` (varsayılan `~/.openclaw/workspace`)

**Ne yapar**:

1. Doğru dökümü bulmak için sıfırlama öncesi oturum girdisini kullanır
2. Konuşmanın son 15 satırını çıkarır
3. Tanımlayıcı bir dosya adı slug’ı üretmek için LLM kullanır
4. Oturum meta verilerini tarihli bir bellek dosyasına kaydeder

**Örnek çıktı**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Dosya adı örnekleri**:

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (slug üretimi başarısız olursa yedek zaman damgası)

**Etkinleştir**:

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

Bir temizleme penceresinde veya rastgele bir olasılıkla enjekte edilen `SOUL.md` içeriğini `SOUL_EVIL.md` ile değiştirir.

**Olaylar**: `agent:bootstrap`

Hata ayıklama

**Çıktı**: Dosya yazılmaz; değişimler yalnızca bellek içinde gerçekleşir.

**Yapılandırma**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

**Dokümanlar**: [SOUL Evil Hook](/hooks/soul-evil)

- Yollar workspace’e göreli olarak çözülür.
- Dosyalar workspace içinde kalmalıdır (realpath kontrolü yapılır).
- Yalnızca tanınan bootstrap temel adları yüklenir.
- Subagent allowlist korunur (yalnızca `AGENTS.md` ve `TOOLS.md`).

**Etkinleştir**:

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

Tüm komut olaylarını merkezi bir denetim dosyasına kaydeder.

**Olaylar**: `command`

**Gereksinimler**: Yok

**Çıktı**: `~/.openclaw/logs/commands.log`

**Ne yapar**:

1. Olay ayrıntılarını yakalar (komut eylemi, zaman damgası, oturum anahtarı, gönderen kimliği, kaynak)
2. Günlük dosyasına JSONL biçiminde ekler
3. Arka planda sessizce çalışır

**Örnek günlük girdileri**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Günlükleri görüntüle**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Etkinleştir**:

```bash
openclaw hooks enable command-logger
```

### boot-md

Gateway başladığında (kanallar başladıktan sonra) `BOOT.md` çalıştırır.
Bunun çalışması için dahili hook’lar etkin olmalıdır.

**Olaylar**: `gateway:startup`

**Gereksinimler**: `workspace.dir` yapılandırılmış olmalıdır

**Ne yapar**:

1. Çalışma alanınızdan `BOOT.md` okur
2. Talimatları ajan çalıştırıcısı aracılığıyla yürütür
3. İstenen giden mesajları mesaj aracıyla gönderir

**Etkinleştir**:

```bash
openclaw hooks enable boot-md
```

## En İyi Uygulamalar

### İşleyicileri Hızlı Tutun

Hook’lar komut işleme sırasında çalışır. Hafif tutun:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Hataları Zarifçe Ele Alın

Riskli işlemleri her zaman sarmalayın:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### Olayları Erken Filtreleyin

Olay ilgili değilse erken dönün:

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### Belirli Olay Anahtarlarını Kullanın

Mümkün olduğunda meta verilerde tam olayları belirtin:

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

Şunun yerine:

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## Hata ayıklama

### Hook Günlüğünü Etkinleştirin

Gateway, başlangıçta hook yüklemeyi günlüğe alır:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Keşfi Kontrol Et

Keşfedilen tüm hook’ları listeleyin:

```bash
openclaw hooks list --verbose
```

### Kayıt (Registration) Kontrolü

İşleyicinizde, çağrıldığında günlüğe kaydedin:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Uygunluğu Doğrulayın

Bir hook’un neden uygun olmadığını kontrol edin:

```bash
openclaw hooks info my-hook
```

Çıktıda eksik gereksinimleri arayın.

## Test

### Gateway Günlükleri

Hook yürütümünü görmek için gateway günlüklerini izleyin:

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### Hook’ları Doğrudan Test Edin

İşleyicilerinizi yalıtılmış olarak test edin:

```typescript
import { test } from "vitest";
import { createHookEvent } from "./src/hooks/hooks.js";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = createHookEvent("command", "new", "test-session", {
    foo: "bar",
  });

  await myHandler(event);

  // Assert side effects
});
```

## Mimari

### Çekirdek Bileşenler

- **`src/hooks/types.ts`**: Tür tanımları
- **`src/hooks/workspace.ts`**: Dizin tarama ve yükleme
- **`src/hooks/frontmatter.ts`**: HOOK.md meta veri ayrıştırma
- **`src/hooks/config.ts`**: Uygunluk denetimi
- **`src/hooks/hooks-status.ts`**: Durum raporlama
- **`src/hooks/loader.ts`**: Dinamik modül yükleyici
- **`src/cli/hooks-cli.ts`**: CLI komutları
- **`src/gateway/server-startup.ts`**: Gateway başlangıcında hook’ları yükler
- **`src/auto-reply/reply/commands-core.ts`**: Komut olaylarını tetikler

### Keşif Akışı

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

### Olay Akışı

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

## Sorun Giderme

### Hook Keşfedilmiyor

1. Dizin yapısını kontrol edin:

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. HOOK.md biçimini doğrulayın:

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. Keşfedilen tüm hook’ları listeleyin:

   ```bash
   openclaw hooks list
   ```

### Hook Çalışmıyor

Gereksinimleri kontrol edin:

```bash
openclaw hooks info my-hook
```

Eksik olanları arayın:

- İkililer (PATH’i kontrol edin)
- Ortam değişkenleri
- Yapılandırma değerleri
- İşletim sistemi uyumluluğu

### Hook Uygun Değil

1. Hook’un etkin olduğunu doğrulayın:

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. Hook’ların yeniden yüklenmesi için gateway sürecinizi yeniden başlatın.

3. Hatalar için gateway günlüklerini kontrol edin:

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### İşleyici Hataları

TypeScript/ithalat hatalarını kontrol edin:

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Geçiş Kılavuzu

### Eski Yapılandırmadan Keşfe

**Önce**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Sonra**:

1. Hook dizini oluşturun:

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. HOOK.md oluşturun:

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. Yapılandırmayı güncelleyin:

   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Doğrulayın ve gateway sürecinizi yeniden başlatın:

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Geçişin faydaları**:

- Otomatik keşif
- CLI ile yönetim
- Uygunluk denetimi
- Daha iyi dokümantasyon
- Tutarlı yapı

## Ayrıca Bakınız

- [CLI Reference: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Configuration](/gateway/configuration#hooks)

---
title: "Voice Call Plagini"
---

# Voice Call (plagin)

OpenClaw uchun plagin orqali ovozli qo‘ng‘iroqlar. Chiqish bildirishnomalari va
kirish siyosatlari bilan ko‘p bosqichli (multi-turn) suhbatlarni qo‘llab-quvvatlaydi.

Hozirgi provayderlar:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML transfer + GetInput speech)
- `mock` (dev/tarmoqsiz)

Qisqa tushuncha:

- Plaginni o‘rnating
- Gateway’ni qayta ishga tushiring
- `plugins.entries.voice-call.config` ostida sozlang
- `openclaw voicecall ...` yoki `voice_call` vositasidan foydalaning

## Qayerda ishlaydi (lokal yoki masofaviy)

Voice Call plagini **Gateway jarayoni ichida** ishlaydi.

Agar siz masofaviy Gateway’dan foydalansangiz, plaginni **Gateway ishlayotgan mashinaga** o‘rnating/sozlang, so‘ng uni yuklash uchun Gateway’ni qayta ishga tushiring.

## O‘rnatish

### Variant A: npm’dan o‘rnatish (tavsiya etiladi)

```bash
openclaw plugins install @openclaw/voice-call
```

Shundan so‘ng Gateway’ni qayta ishga tushiring.

### Variant B: lokal papkadan o‘rnatish (dev, nusxalashsiz)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Shundan so‘ng Gateway’ni qayta ishga tushiring.

## Sozlash

Sozlamalarni `plugins.entries.voice-call.config` ostida belgilang:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // yoki "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Telnyx Mission Control Portal’dan olingan Telnyx webhook public key
            // (Base64 satr; TELNYX_PUBLIC_KEY orqali ham berish mumkin).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Webhook serveri
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Webhook xavfsizligi (tunnel/proxy uchun tavsiya etiladi)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // Ommaviy ochish (bittasini tanlang)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
          },
        },
      },
    },
  },
}
```

Eslatmalar:

- Twilio/Telnyx uchun **ommaviy kirish mumkin bo‘lgan** webhook URL talab qilinadi.
- Plivo uchun ham **ommaviy kirish mumkin bo‘lgan** webhook URL talab qilinadi.
- `mock` — lokal dev provayder (tarmoq chaqiruvlarisiz).
- Telnyx uchun `telnyx.publicKey` (yoki `TELNYX_PUBLIC_KEY`) talab qilinadi, agar `skipSignatureVerification` true bo‘lmasa.
- `skipSignatureVerification` faqat lokal test uchun.
- Agar ngrok’ning bepul tarifidan foydalansangiz, `publicUrl` ni aniq ngrok URL’ga o‘rnating; imzo tekshiruvi har doim majburiy.
- `tunnel.allowNgrokFreeTierLoopbackBypass: true` Twilio webhook’lariga noto‘g‘ri imzo bilan **faqat** `tunnel.provider="ngrok"` va `serve.bind` loopback (ngrok lokal agenti) bo‘lganda ruxsat beradi. Faqat lokal dev uchun.
- Ngrok bepul URL’lari o‘zgarishi yoki qo‘shimcha oraliq sahifa (interstitial) qo‘shishi mumkin; agar `publicUrl` o‘zgarsa, Twilio imzolari muvaffaqiyatsiz bo‘ladi. Production uchun barqaror domen yoki Tailscale funnel tavsiya etiladi.

## Webhook xavfsizligi

Gateway oldida proxy yoki tunnel turganda, plagin imzo tekshiruvi uchun
ommaviy URL’ni qayta tiklaydi. Quyidagi opsiyalar qaysi forwarded
sarlavhalarga ishonilishini boshqaradi.

`webhookSecurity.allowedHosts` forwarded sarlavhalardagi host’larni allowlist qiladi.

`webhookSecurity.trustForwardingHeaders` allowlist’siz forwarded sarlavhalarga ishonadi.

`webhookSecurity.trustedProxyIPs` faqat so‘rovning remote IP manzili ro‘yxatga mos kelsa forwarded sarlavhalarga ishonadi.

Barqaror ommaviy host bilan misol:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## Qo‘ng‘iroqlar uchun TTS

Voice Call qo‘ng‘iroqlarda oqimli nutq (streaming speech) uchun asosiy `messages.tts`
sozlamasidan (OpenAI yoki ElevenLabs) foydalanadi. Uni plagin sozlamasi ostida
**xuddi shu shaklda** qayta belgilashingiz mumkin — u `messages.tts` bilan deep‑merge qilinadi.

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

Eslatmalar:

- **Edge TTS ovozli qo‘ng‘iroqlar uchun e’tiborga olinmaydi** (telefon audio PCM talab qiladi; Edge chiqishi ishonchsiz).
- Agar Twilio media streaming yoqilgan bo‘lsa, asosiy TTS ishlatiladi; aks holda qo‘ng‘iroqlar provayderning o‘z ovozlariga o‘tadi.

### Qo‘shimcha misollar

Faqat asosiy TTS’dan foydalanish (override’siz):

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

Faqat qo‘ng‘iroqlar uchun ElevenLabs’ga override qilish (boshqa joyda asosiy sozlama saqlanadi):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

Faqat qo‘ng‘iroqlar uchun OpenAI modelini override qilish (deep‑merge misoli):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## Kirish qo‘ng‘iroqlari

Kirish siyosati sukut bo‘yicha `disabled`. Kirish qo‘ng‘iroqlarini yoqish uchun:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Salom! Qanday yordam bera olaman?",
}
```

Avtomatik javoblar agent tizimi orqali ishlaydi. Quyidagilar bilan sozlang:

- `responseModel`
- `responseSystemPrompt`
- `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## Agent vositasi

Vositа nomi: `voice_call`

Harakatlar:

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

Ushbu repo’da mos skill hujjati mavjud: `skills/voice-call/SKILL.md`.

## Gateway RPC

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)
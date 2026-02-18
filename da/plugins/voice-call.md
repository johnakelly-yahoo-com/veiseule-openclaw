---
summary: "Voice Call-plugin: udgående + indgående opkald via Twilio/Telnyx/Plivo (plugin-installation + konfiguration + CLI)"
read_when:
  - Du vil foretage et udgående taleopkald fra OpenClaw
  - Du konfigurerer eller udvikler voice-call-pluginet
title: "Voice Call-plugin"
---

# Taleopkald (plugin)

Stemmeopkald til OpenClaw via et plugin. Understøtter udgående meddelelser og
multi-turn samtaler med indgående politikker.

Nuværende udbydere:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML-overførsel + GetInput-tale)
- `mock` (dev/ingen netværk)

Hurtig mental model:

- Installér plugin
- Genstart Gateway
- Konfigurer under `plugins.entries.voice-call.config`
- Brug `openclaw voicecall ...` eller værktøjet `voice_call`

## Hvor det kører (lokalt vs. remote)

Voice Call-pluginet kører **inde i Gateway-processen**.

Hvis du bruger en remote Gateway, skal du installere/konfigurere pluginet på
**maskinen, der kører Gateway**, og derefter genstarte Gateway for at indlæse det.

## Installér

### Mulighed A: installér fra npm (anbefalet)

```bash
openclaw plugins install @openclaw/voice-call
```

Genstart Gateway bagefter.

### Mulighed B: installér fra en lokal mappe (dev, ingen kopiering)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Genstart Gateway bagefter.

## Konfiguration

Sæt konfiguration under `plugins.entries.voice-call.config`:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // or "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Webhook security (recommended for tunnels/proxies)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // Public exposure (pick one)
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

Noter:

- Twilio/Telnyx kræver en **offentligt tilgængelig** webhook-URL.
- Plivo kræver en **offentligt tilgængelig** webhook-URL.
- `mock` er en lokal dev-udbyder (ingen netværkskald).
- `skipSignatureVerification` er kun til lokal test.
- Hvis du bruger ngrok free tier, skal du sætte `publicUrl` til den præcise ngrok-URL; signaturverifikation håndhæves altid.
- `tunnel.allowNgrokFreeTierLoopbackBypass: true` tillader Twilio webhooks med ugyldige signaturer **kun** når `tunnel.provider="ngrok"` og `serve.bind` er loopback (ngrok local agent). Må kun anvendes til lokal dev.
- Ngrok free tier URLs can change or add interstitial behavior; if `publicUrl` drifts, Twilio signaturer will fail. Til produktion foretrækker et stabilt domæne eller Tailscale tragt.

## Webhook-sikkerhed

Når en proxy eller tunnel sidder foran Gateway, rekonstruerer plugin den offentlige URL
for signaturverifikation. Disse indstillinger styrer som videresendte
overskrifter er betroede.

`webhookSecurity.allowedHosts` tilladelseslister værter fra videresendte headers.

`webhookSecurity.trustForwardingHeaders` stoler på videresendte headers uden en tilladelsesliste.

`webhookSecurity.trustedProxyIPs` stoler kun på videresendte headers, når anmodningens
remote IP matcher listen.

Eksempel med en stabil offentlig vært:

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

## TTS til opkald

Stemmeopkald bruger kernen `messages.tts` konfiguration (OpenAI eller ElevenLabs) til
streaming af tale ved opkald. Du kan tilsidesætte det under plugin config med
**samme form** — det dybe-fusionerer med `messages.tts`.

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

Noter:

- **Edge TTS ignoreres for taleopkald** (telefoni-lyd kræver PCM; Edge-output er upålideligt).
- Kerne-TTS bruges, når Twilio media streaming er aktiveret; ellers falder opkald tilbage til udbyderens native stemmer.

### Flere eksempler

Brug kun kerne-TTS (ingen tilsidesættelse):

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

Tilsidesæt til ElevenLabs kun for opkald (bevar kernestandard andre steder):

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

Tilsidesæt kun OpenAI-modellen for opkald (deep-merge-eksempel):

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

## Indgående opkald

Indgående politik er standard til `deaktiveret`. For at aktivere indgående opkald, indstillet:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hello! How can I help?",
}
```

Auto-svar bruge agent-systemet. Tune med:

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

## Agent-værktøj

Værktøjsnavn: `voice_call`

Handlinger:

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

Dette repo leverer en matchende Skills-dokumentation på `skills/voice-call/SKILL.md`.

## Gateway RPC

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)

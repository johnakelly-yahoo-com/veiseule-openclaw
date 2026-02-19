---
summary: "Indgående billed-/lyd-/videoforståelse (valgfri) med udbyder + CLI-fallbacks"
read_when:
  - Design eller refaktorering af medieforståelse
  - Finjustering af indgående lyd-/video-/billedforbehandling
title: "Medieforståelse"
---

# Medieforståelse (Indgående) — 2026-01-17

OpenClaw kan **opsummere indgående medier** (billede/audio/video) før svarrørledningen kører. Den autodetekterer når lokale værktøjer eller leverandørnøgler er tilgængelige, og kan deaktiveres eller tilpasses. Hvis forståelse er slået fra, modeller stadig modtage de originale filer/URL'er som sædvanligt.

## Mål

- Valgfrit: forfordøj indgående medier til kort tekst for hurtigere routing + bedre kommandofortolkning.
- Bevar altid levering af originalmedier til modellen.
- Understøt **udbyder-API’er** og **CLI-fallbacks**.
- Tillad flere modeller med ordnet fallback (fejl/størrelse/timeout).

## Overordnet adfærd

1. Indsaml indgående vedhæftninger (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. For hver aktiveret kapabilitet (billede/lyd/video) vælges vedhæftninger efter politik (standard: **første**).
3. Vælg den første egnede modelpost (størrelse + kapabilitet + godkendelse).
4. Hvis en model fejler, eller mediet er for stort, **faldes der tilbage til næste post**.
5. Ved succes:
   - `Body` bliver til en `[Image]`-, `[Audio]`- eller `[Video]`-blok.
   - Lyd sætter `{{Transcript}}`; kommandofortolkning bruger billedtekst, når den findes,
     ellers transskriptionen.
   - Billedtekster bevares som `User text:` inde i blokken.

Hvis forståelse fejler eller er deaktiveret, **fortsætter svarflowet** med den oprindelige brødtekst + vedhæftninger.

## Konfigurationsoverblik

`tools.media` understøtter **delte modeller** plus tilsidesættelser pr. kapabilitet:

- `tools.media.models`: delt modelliste (brug `capabilities` til gating).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - standarder (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - udbyder-tilsidesættelser (`baseUrl`, `headers`, `providerOptions`)
  - Deepgram-lydindstillinger via `tools.media.audio.providerOptions.deepgram`
  - valgfri **pr.-kapabilitet `models`-liste** (foretrækkes før delte modeller)
  - `attachments`-politik (`mode`, `maxAttachments`, `prefer`)
  - `scope` (valgfri gating efter kanal/chatType/session-nøgle)
- `tools.media.concurrency`: max samtidige funktioner kører (standard **2**).

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

### Modelposter

Hver `models[]`-post kan være **udbyder** eller **CLI**:

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

CLI-skabeloner kan også bruge:

- `{{MediaDir}}` (mappe, der indeholder mediefilen)
- `{{OutputDir}}` (scratch-mappe oprettet til denne kørsel)
- `{{OutputBase}}` (scratch-filens basissti uden filendelse)

## Standarder og grænser

Anbefalede standarder:

- `maxChars`: **500** for billede/video (kort, kommando-venlig)
- `maxChars`: **ikke sat** for lyd (fuld transskription, medmindre du sætter en grænse)
- `maxBytes`:
  - billede: **10MB**
  - lyd: **20MB**
  - video: **50MB**

Regler:

- Hvis mediet overstiger `maxBytes`, springes modellen over, og **næste model prøves**.
- Hvis modellen returnerer mere end `maxChars`, trimmes outputtet.
- `prompt` er som standard den simple “Describe the {media}.” plus `maxChars`-vejledning (kun billede/video).
- Hvis `<capability>.enabled: true`, men der ikke er konfigureret modeller, forsøger OpenClaw den
  **aktive svarmodel**, når dens udbyder understøtter kapabiliteten.

### Automatisk registrering af medieforståelse (standard)

Hvis `tools.media.<capability>.enabled` er **ikke** sat til \`false', og du har ikke
konfigurerede modeller, OpenClaw auto-registrerer i denne rækkefølge og **stopper ved den første
arbejdstilvalg**:

1. **Lokale CLI’er** (kun lyd; hvis installeret)
   - `sherpa-onnx-offline` (kræver `SHERPA_ONNX_MODEL_DIR` med encoder/decoder/joiner/tokens)
   - `whisper-cli` (`whisper-cpp`; bruger `WHISPER_CPP_MODEL` eller den medfølgende tiny-model)
   - `whisper` (Python-CLI; downloader modeller automatisk)
2. **Gemini CLI** (`gemini`) ved brug af `read_many_files`
3. **Udbydernøgler**
   - Lyd: OpenAI → Groq → Deepgram → Google
   - Billede: OpenAI → Anthropic → Google → MiniMax
   - Video: Google

For at deaktivere automatisk registrering, sæt:

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

Bemærk: Binær registrering er best-effort på tværs af macOS/Linux/Windows; sørg for, at CLI’en er på `PATH` (vi udvider `~`), eller angiv en eksplicit CLI-model med fuld kommandosti.

## Kapabiliteter (valgfrit)

Hvis du sætter `capabilities `, posten kører kun for disse medietyper. For delte
-lister kan OpenClaw udlede standard:

- `openai`, `anthropic`, `minimax`: **billede**
- `google` (Gemini API): **billede + lyd + video**
- `groq`: **lyd**
- `deepgram`: **lyd**

For CLI poster, **sæt `kapaciteter` eksplicly** for at undgå overraskende kampe.
Hvis du udelader 'kapaciteter', er posten berettiget til den liste, den vises i.

## Understøttelsesmatrix for udbydere (OpenClaw-integrationer)

| Kapabilitet | Udbyderintegration                              | Noter                                                                               |
| ----------- | ----------------------------------------------- | ----------------------------------------------------------------------------------- |
| Billede     | OpenAI / Anthropic / Google / andre via `pi-ai` | Enhver billed-kompatibel model i registreret virker.                |
| Lyd         | OpenAI, Groq, Deepgram, Google                  | Udbydertransskription (Whisper/Deepgram/Gemini). |
| Video       | Google (Gemini API)          | Udbyderbaseret videoforståelse.                                     |

## Anbefalede udbydere

**Billede**

- Foretræk din aktive model, hvis den understøtter billeder.
- Gode standarder: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Lyd**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` eller `deepgram/nova-3`.
- CLI-fallback: `whisper-cli` (whisper-cpp) eller `whisper`.
- Deepgram-opsætning: [Deepgram (lydtransskription)](/providers/deepgram).

**Video**

- `google/gemini-3-flash-preview` (hurtig), `google/gemini-3-pro-preview` (rigere).
- CLI-fallback: `gemini` CLI (understøtter `read_file` på video/lyd).

## Politik for vedhæftninger

Pr.-kapabilitet `attachments` styrer, hvilke vedhæftninger der behandles:

- `mode`: `first` (standard) eller `all`
- `maxAttachments`: begræns antallet, der behandles (standard **1**)
- `prefer`: `first`, `last`, `path`, `url`

Når `mode: "all"`, mærkes output som `[Image 1/2]`, `[Audio 2/2]` osv.

## Konfigurationseksempler

### 1. Delt modelliste + tilsidesættelser

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

### 2. Kun lyd + video (billede slået fra)

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

### 3. Valgfri billedforståelse

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

### 4. Multimodal enkeltpost (eksplicitte kapabiliteter)

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

## Statusoutput

Når medieforståelse kører, inkluderer `/status` en kort opsummeringslinje:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Dette viser udfald pr. kapabilitet og den valgte udbyder/model, når relevant.

## Noter

- Forståelse er **best effort**. Fejl blokerer ikke svar.
- Vedhæftninger sendes stadig til modeller, selv når forståelse er deaktiveret.
- Brug `scope` til at begrænse, hvor forståelse kører (fx kun DM’er).

## Relaterede dokumenter

- [Konfiguration](/gateway/configuration)
- [Billede- og mediesupport](/nodes/images)


---
summary: "Kameraoptagelse (iOS-node + macOS-app) til agentbrug: fotos (jpg) og korte videoklip (mp4)"
read_when:
  - Tilføjelse eller ændring af kameraoptagelse på iOS-noder eller macOS
  - Udvidelse af agenttilgængelige MEDIA-tempfil-workflows
title: "Kameraoptagelse"
---

# Kameraoptagelse (agent)

OpenClaw understøtter **kameraoptagelse** til agent-workflows:

- **iOS-node** (parret via Gateway): optag et **foto** (`jpg`) eller et **kort videoklip** (`mp4`, med valgfri lyd) via `node.invoke`.
- **Android-node** (parret via Gateway): optag et **foto** (`jpg`) eller et **kort videoklip** (`mp4`, med valgfri lyd) via `node.invoke`.
- **macOS-app** (node via Gateway): optag et **foto** (`jpg`) eller et **kort videoklip** (`mp4`, med valgfri lyd) via `node.invoke`.

Al kameraadgang er låst bag **brugerstyrede indstillinger**.

## iOS-node

### Brugerindstilling (standard til)

- iOS Indstillinger-fane → **Kamera** → **Tillad kamera** (`camera.enabled`)
  - Standard: **til** (manglende nøgle behandles som aktiveret).
  - Når slået fra: `camera.*`-kommandoer returnerer `CAMERA_DISABLED`.

### Kommandoer (via Gateway `node.invoke`)

- `camera.list`
  - Svar-payload:
    - `devices`: array af `{ id, name, position, deviceType }`

- `camera.snap`
  - Parametre:
    - `facing`: `front|back` (standard: `front`)
    - `maxWidth`: number (valgfri; standard `1600` på iOS-noden)
    - `quality`: `0..1` (valgfri; standard `0.9`)
    - `format`: i øjeblikket `jpg`
    - `delayMs`: number (valgfri; standard `0`)
    - `deviceId`: string (valgfri; fra `camera.list`)
  - Svar-payload:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Payload-værn: fotos recomprimeres for at holde base64-payload under 5 MB.

- `camera.clip`
  - Parametre:
    - `facing`: `front|back` (standard: `front`)
    - `durationMs`: number (standard `3000`, begrænset til maks `60000`)
    - `includeAudio`: boolean (standard `true`)
    - `format`: i øjeblikket `mp4`
    - `deviceId`: string (valgfri; fra `camera.list`)
  - Svar-payload:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

### Krav om forgrund

Ligesom `canvas.*`, tillader iOS-noden kun `kamera.*` kommandoer i **forgrunden**. Baggrundsangivelser returnerer `NODE_BACKGROUND_UNAVAILABLE`.

### CLI-hjælper (tempfiler + MEDIA)

Den nemmeste måde at få vedhæftninger på er via CLI-hjælperen, som skriver dekodet medie til en tempfil og udskriver `MEDIA:<path>`.

Eksempler:

```bash
openclaw nodes camera snap --node <id>               # default: both front + back (2 MEDIA lines)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Noter:

- `nodes camera snap` er som standard **begge** kameraretninger for at give agenten begge visninger.
- Outputfiler er midlertidige (i OS’ tempmappe), medmindre du bygger din egen wrapper.

## Android-node

### Android-brugerindstilling (standard til)

- Android Indstillinger-ark → **Kamera** → **Tillad kamera** (`camera.enabled`)
  - Standard: **til** (manglende nøgle behandles som aktiveret).
  - Når slået fra: `camera.*`-kommandoer returnerer `CAMERA_DISABLED`.

### Tilladelser

- Android kræver runtime-tilladelser:
  - `CAMERA` for både `camera.snap` og `camera.clip`.
  - `RECORD_AUDIO` for `camera.clip`, når `includeAudio=true`.

Hvis tilladelser mangler, vil appen spørge, når det er muligt; hvis de nægtes, mislykkes `camera.*`-anmodninger med en
`*_PERMISSION_REQUIRED`-fejl.

### Android-krav om forgrund

Ligesom `canvas.*`, tillader Android-knuden kun `kamera.*` kommandoer i **forgrunden**. Baggrundsangivelser returnerer `NODE_BACKGROUND_UNAVAILABLE`.

### Payload-værn

Fotos recomprimeres for at holde base64-payload under 5 MB.

## macOS-app

### Brugerindstilling (standard fra)

macOS Companion-appen eksponerer et afkrydsningsfelt:

- **Indstillinger → Generelt → Tillad kamera** (`openclaw.cameraEnabled`)
  - Standard: **fra**
  - Når slået fra: kameraanmodninger returnerer “Camera disabled by user”.

### CLI-hjælper (node invoke)

Brug den primære `openclaw`-CLI til at kalde kamerakommandoer på macOS-noden.

Eksempler:

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # prints MEDIA:<path> (legacy flag)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Noter:

- `openclaw nodes camera snap` er som standard `maxWidth=1600`, medmindre den tilsidesættes.
- På macOS venter `camera.snap` `delayMs` (standard 2000 ms) efter opvarmning/eksponeringsstabilisering, før der optages.
- Foto-payloads recomprimeres for at holde base64 under 5 MB.

## Sikkerhed + praktiske grænser

- Kamera- og mikrofonadgang udløser de sædvanlige OS-tilladelsesprompter (og kræver usage-strings i Info.plist).
- Videoklip er begrænset (aktuelt `<= 60s`) for at undgå for store node-payloads (base64-overhead + beskedgrænser).

## macOS skærmvideo (OS-niveau)

Til _skærm_-video (ikke kamera) skal du bruge macOS Companion:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # prints MEDIA:<path>
```

Noter:

- Kræver macOS **Skærmoptagelse**-tilladelse (TCC).


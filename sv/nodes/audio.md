---
summary: "Hur inkommande ljud/röstmeddelanden laddas ned, transkriberas och injiceras i svar"
read_when:
  - Ändring av ljudtranskribering eller mediehantering
title: "Ljud och röstmeddelanden"
---

# Ljud / röstmeddelanden — 2026-01-17

## Vad som fungerar

- **Medieförståelse (ljud)**: Om ljudförståelse är aktiverad (eller autodetekteras) gör OpenClaw:
  1. Lokaliserar den första ljudbilagan (lokal sökväg eller URL) och laddar ned den vid behov.
  2. Tillämpa `maxBytes` innan sändning till varje modellpost.
  3. Kör den första kvalificerade modellposten i ordning (leverantör eller CLI).
  4. Om den misslyckas eller hoppas över (storlek/timeout) prövas nästa post.
  5. Vid framgång ersätts `Body` med ett `[Audio]`-block och `{{Transcript}}` sätts.
- **Kommandotolkning**: När transkriberingen lyckas sätts `CommandBody`/`RawBody` till transkriptet så att snedstreckskommandon fortfarande fungerar.
- **Utförlig loggning**: I `--verbose` loggar vi när transkriberingen körs och när den ersätter brödtexten.

## Autodetektering (standard)

Om du **inte konfigurerar modeller** och `tools.media.audio.enabled` **inte** är satt till `false`,
autodetekterar OpenClaw i denna ordning och stannar vid första fungerande alternativ:

1. **Lokala CLI:er** (om installerade)
   - `sherpa-onnx-offline` (kräver `SHERPA_ONNX_MODEL_DIR` med encoder/decoder/joiner/tokens)
   - `whisper-cli` (från `whisper-cpp`; använder `WHISPER_CPP_MODEL` eller den medföljande tiny-modellen)
   - `whisper` (Python-CLI; laddar ned modeller automatiskt)
2. **Gemini CLI** (`gemini`) med `read_many_files`
3. **Leverantörsnycklar** (OpenAI → Groq → Deepgram → Google)

För att inaktivera automatisk identifiering, ange `tools.media.audio.enabled: false`.
För att anpassa, ange `tools.media.audio.models`.
Obs: Binär identifiering är best‑effort över macOS/Linux/Windows; säkerställ att CLI:n finns på `PATH` (vi expanderar `~`), eller ange en explicit CLI‑modell med fullständig kommandosökväg.

## Konfigexempel

### Leverantör + CLI-fallback (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### Endast leverantör med scope-begränsning

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### Endast leverantör (Deepgram)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Noteringar & begränsningar

- Leverantörsautentisering följer standardordningen för modellautentisering (autentiseringsprofiler, miljövariabler, `models.providers.*.apiKey`).
- Deepgram plockar upp `DEEPGRAM_API_KEY` när `provider: "deepgram"` används.
- Detaljer för Deepgram-konfigurering: [Deepgram (ljudtranskribering)](/providers/deepgram).
- Ljudleverantörer kan åsidosätta `baseUrl`, `headers` och `providerOptions` via `tools.media.audio`.
- Standardstorlek cap är 20 MB (`tools.media.audio.maxBytes`). Oversize ljud hoppas över för den modellen och nästa inlägg provas.
- Standard `maxChars` för ljud är **unset** (full transkript). Ange `tools.media.audio.maxChars` eller per-entry `maxChars` för att trimma utgången.
- OpenAI:s autostandard är `gpt-4o-mini-transcribe`; sätt `model: "gpt-4o-transcribe"` för högre noggrannhet.
- Använd `tools.media.audio.attachments` för att bearbeta flera röstmeddelanden (`mode: "all"` + `maxAttachments`).
- Transkriptet är tillgängligt för mallar som `{{Transcript}}`.
- CLI-stdout är begränsad (5 MB); håll CLI-utdata koncis.

## Fallgropar

När `requireMention: true` är inställt för en gruppchatt transkriberar OpenClaw nu ljud **innan** den kontrollerar omnämnanden. Detta gör att röstmeddelanden kan bearbetas även när de innehåller omnämnanden.

**Så fungerar det:**

1. Om ett röstmeddelande saknar textinnehåll och gruppen kräver omnämnanden utför OpenClaw en "preflight"-transkribering.
2. Transkriberingen kontrolleras för mönster av omnämnanden (t.ex. `@BotName`, emoji-triggers).
3. Om ett omnämnande hittas går meddelandet vidare genom hela svarspipelinen.
4. Transkriptet används för omnämningsdetektering så att röstmeddelanden kan passera omnämningsgrinden.

**Fallback-beteende:**

- Om transkriberingen misslyckas under preflight (timeout, API-fel osv.) behandlas meddelandet baserat på endast textbaserad omnämningsdetektering.
- Detta säkerställer att blandade meddelanden (text + ljud) aldrig felaktigt ignoreras.

**Exempel:** En användare skickar ett röstmeddelande som säger "Hey @Claude, what's the weather?" i en Telegram-grupp med `requireMention: true`. Röstmeddelandet transkriberas, omnämningen identifieras och agenten svarar.

## Fallgropar

- Omfattningsregler använder första match vinster. `chatType` är normaliserad till `direct`, `group`, eller `room`.
- Säkerställ att ditt CLI avslutar med 0 och skriver ren text; JSON behöver bearbetas via `jq -r .text`.
- Håll timeouts rimliga (`timeoutSeconds`, standard 60 s) för att undvika att blockera svarskön.
- Preflight-transkribering bearbetar endast den **första** ljudbilagan för omnämningsdetektering. Ytterligare ljud bearbetas under den huvudsakliga fasen för medieförståelse.

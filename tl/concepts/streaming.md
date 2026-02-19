---
summary: "Pag-uugali ng streaming + chunking (block replies, draft streaming, mga limitasyon)"
read_when:
  - Ipinapaliwanag kung paano gumagana ang streaming o chunking sa mga channel
  - Binabago ang block streaming o channel chunking behavior
  - Pag-debug ng duplicate/maagang block replies o draft streaming
title: "Streaming at Chunking"
---

# Streaming + pag-chunk

May dalawang magkahiwalay na “streaming” layer ang OpenClaw:

- **Block streaming (mga channel):** maglabas ng mga natapos na **block** habang sumusulat ang assistant. These are normal channel messages (not token deltas).
- **Token-ish streaming (Telegram lamang):** ina-update ang isang **draft bubble** gamit ang bahagyang teksto habang nagge-generate; ang final na mensahe ay ipinapadala sa dulo.

Wala pang **tunay na token-delta streaming** sa mga mensahe ng channel sa kasalukuyan. Ang Telegram preview streaming ang tanging partial-stream surface.

## Block streaming (mga mensahe ng channel)

Ang block streaming ay nagpapadala ng output ng assistant sa malalaking tipak habang nagiging available ito.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

Legend:

- `text_delta/events`: mga event ng model stream (maaaring bihira para sa mga non-streaming na model).
- `chunker`: `EmbeddedBlockChunker` na nag-a-apply ng min/max bounds + break preference.
- `channel send`: aktuwal na outbound messages (block replies).

**Mga kontrol:**

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (default ay off).
- Mga channel override: `*.blockStreaming` (at mga per-account na variant) para pilitin ang `"on"`/`"off"` kada channel.
- `agents.defaults.blockStreamingBreak`: `"text_end"` o `"message_end"`.
- `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`.
- `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (pagsamahin ang mga na-stream na block bago ipadala).
- Channel hard cap: `*.textChunkLimit` (hal., `channels.whatsapp.textChunkLimit`).
- Channel chunk mode: `*.chunkMode` (`length` default, `newline` naghahati sa mga blank line (hangganan ng talata) bago ang length chunking).
- Discord soft cap: `channels.discord.maxLinesPerMessage` (default 17) naghahati ng mahahabang reply para maiwasan ang UI clipping.

**Semantika ng hangganan:**

- `text_end`: i-stream ang mga block sa sandaling maglabas ang chunker; mag-flush sa bawat `text_end`.
- `message_end`: maghintay hanggang matapos ang mensahe ng assistant, saka i-flush ang naka-buffer na output.

`message_end` ay gumagamit pa rin ng chunker kung ang naka-buffer na teksto ay lumampas sa `maxChars`, kaya maaari itong maglabas ng maraming chunk sa dulo.

## Algorithm ng pag-chunk (mababang/mataas na hangganan)

Ang block chunking ay ipinapatupad ng `EmbeddedBlockChunker`:

- **Low bound:** huwag maglabas hangga’t ang buffer ay >= `minChars` (maliban kung pinilit).
- **High bound:** mas gustong maghati bago ang `maxChars`; kung pinilit, maghati sa `maxChars`.
- **Prayoridad ng paghahati:** `paragraph` → `newline` → `sentence` → `whitespace` → sapilitang paghinto.
- **Code fences:** huwag kailanman maghati sa loob ng mga fence; kapag pinilit sa `maxChars`, isara at buksan muli ang fence para manatiling valid ang Markdown.

Ang `maxChars` ay naka-clamp sa channel `textChunkLimit`, kaya hindi ka maaaring lumampas sa mga per-channel cap.

## Coalescing (pagsasanib ng mga streamed block)

When block streaming is enabled, OpenClaw can **merge consecutive block chunks**
before sending them out. Binabawasan nito ang “single-line spam” habang nagbibigay pa rin ng
progressive output.

- Naghihintay ang coalescing ng mga **idle gap** (`idleMs`) bago mag-flush.
- Ang mga buffer ay may cap na `maxChars` at magfa-flush kung lalampas dito.
- Pinipigilan ng `minChars` ang pagpapadala ng maliliit na fragment hangga’t hindi sapat ang naipong teksto
  (ang final flush ay laging nagpapadala ng natitirang teksto).
- Ang joiner ay hinango mula sa `blockStreamingChunk.breakPreference`
  (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → espasyo).
- May mga channel override sa pamamagitan ng `*.blockStreamingCoalesce` (kasama ang mga per-account config).
- Ang default na coalesce `minChars` ay itinataas sa 1500 para sa Signal/Slack/Discord maliban kung overridden.

## Human-like na pacing sa pagitan ng mga block

When block streaming is enabled, you can add a **randomized pause** between
block replies (after the first block). This makes multi-bubble responses feel
more natural.

- Config: `agents.defaults.humanDelay` (override kada agent sa pamamagitan ng `agents.list[].humanDelay`).
- Mga mode: `off` (default), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
- Nalalapat lamang sa **block replies**, hindi sa final replies o mga tool summary.

## “I-stream ang mga chunk o lahat”

Ito ay tumutugma sa:

- **Stream chunks:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (maglabas habang nagpapatuloy). Non-Telegram channels also need `*.blockStreaming: true`.
- **I-stream ang lahat sa dulo:** `blockStreamingBreak: "message_end"` (isang flush, posibleng maraming chunk kung napakahaba).
- **Walang block streaming:** `blockStreamingDefault: "off"` (final reply lamang).

**Channel note:** For non-Telegram channels, block streaming is **off unless**
`*.blockStreaming` is explicitly set to `true`. Maaaring mag-stream ang Telegram ng live preview
(`channels.telegram.streamMode`) nang walang block replies.

Paalala sa lokasyon ng config: ang mga default ng `blockStreaming*` ay nasa ilalim ng
`agents.defaults`, hindi sa root config.

## Telegram draft streaming (token-ish)

Ang Telegram lang ang channel na may draft streaming:

- Gumagamit ng Bot API `sendMessageDraft` sa **private chats na may topics**.
- `channels.telegram.streamMode: "partial" | "block" | "off"`.
  - `partial`: mga update ng draft gamit ang pinakabagong stream text.
  - `block`: mga update ng draft sa mga chunked block (parehong mga patakaran ng chunker).
  - `off`: walang draft streaming.
- Config ng draft chunk (para lamang sa `streamMode: "block"`): `channels.telegram.draftChunk` (mga default: `minChars: 200`, `maxChars: 800`).
- Hiwalay ang draft streaming sa block streaming; naka-off ang mga block reply bilang default at pinapagana lamang ng `*.blockStreaming: true` sa mga non-Telegram channel.
- Ang final reply ay isang normal na mensahe pa rin.
- Isinusulat ng `/reasoning stream` ang reasoning sa loob ng draft bubble (Telegram lamang).
- Ang mga non-text/complex na final ay babalik sa normal na paghahatid ng final message.
- Isinusulat ng `/reasoning stream` ang reasoning sa live preview (Telegram lamang).

```
Telegram
  └─ sendMessage (pansamantalang preview message)
       ├─ streamMode=partial → i-edit ang pinakabagong teksto
       └─ streamMode=block   → chunker + mga update sa pag-edit
  └─ final text-only reply → final edit sa parehong mensahe
  └─ fallback: linisin ang preview + normal na final delivery (media/complex)
```

Legend:

- `preview message`: pansamantalang mensahe sa Telegram na ina-update habang nagge-generate.
- `final edit`: in-place na pag-edit sa parehong preview message (text-only).


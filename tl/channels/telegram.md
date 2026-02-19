---
summary: "Katayuan ng suporta ng Telegram bot, mga kakayahan, at konpigurasyon"
read_when:
  - Nagtatrabaho sa mga feature o webhook ng Telegram
title: "Telegram"
---

# Telegram (Bot API)

Status: handa para sa produksyon para sa mga bot DM + group sa pamamagitan ng grammY. Long polling ang default na mode; opsyonal ang webhook mode.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Ang default na DM policy para sa Telegram ay pairing.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Cross-channel na diagnostics at mga playbook para sa pag-aayos.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Buong mga pattern at halimbawa ng channel config.
  
</Card>
</CardGroup>

## Mabilisang setup

<Steps>
  <Step title="Create the bot token in BotFather">
    Buksan ang Telegram at makipag-chat kay **@BotFather** (tiyaking eksaktong `@BotFather` ang handle).
  

    ```
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",
        },
      },
    }
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Env fallback: `TELEGRAM_BOT_TOKEN=...` (default account lamang).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Ang mga pairing code ay nag-e-expire pagkatapos ng 1 oras.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Idagdag ang bot sa iyong grupo, pagkatapos ay itakda ang `channels.telegram.groups` at `groupPolicy` upang tumugma sa iyong access model.
  
</Step>
</Steps>

<Note>
Ang pagkakasunod-sunod ng pagresolba ng token ay account-aware. Sa praktika, mas nangingibabaw ang mga config value kaysa sa env fallback, at ang `TELEGRAM_BOT_TOKEN` ay nalalapat lamang sa default account.
</Note>

## Mga setting sa panig ng Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">
    Ang mga Telegram bot ay naka-default sa **Privacy Mode**, na naglilimita sa mga group message na kanilang natatanggap.


    ```
    Kung kailangang makita ng bot ang lahat ng group message, alinman sa:
    
    - i-disable ang privacy mode sa pamamagitan ng `/setprivacy`, o
    - gawing group admin ang bot.
    
    Kapag binabago ang privacy mode, alisin at muling idagdag ang bot sa bawat grupo upang maipatupad ng Telegram ang pagbabago.
    
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    Ang admin status ay kinokontrol sa mga setting ng Telegram group.


    ```
    Ang mga admin bot ay tumatanggap ng lahat ng group message, na kapaki-pakinabang para sa palaging aktibong pag-uugali sa grupo.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - `/setjoingroups` upang payagan/tanggihan ang pagdagdag sa grupo
    - `/setprivacy` para sa behavior ng visibility sa grupo
    ```

  
</Accordion>
</AccordionGroup>

## Kontrol sa access at pag-activate

<Tabs>
  <Tab title="DM policy">
    Kinokontrol ng `channels.telegram.dmPolicy` ang access sa direct message:


    ```
    - `pairing` (default)
    - `allowlist`
    - `open` (nangangailangan na isama ng `allowFrom` ang `"*"`)
    - `disabled`
    
    Ang `channels.telegram.allowFrom` ay tumatanggap ng numeric Telegram user IDs. Ang mga prefix na `telegram:` / `tg:` ay tinatanggap at ino-normalize.
    Ang onboarding wizard ay tumatanggap ng `@username` na input at nireresolba ito sa numeric IDs.
    Kung nag-upgrade ka at ang iyong config ay naglalaman ng `@username` allowlist entries, patakbuhin ang `openclaw doctor --fix` upang maresolba ang mga ito (best-effort; nangangailangan ng Telegram bot token).
    
    ### Paghahanap ng iyong Telegram user ID
    
    Mas ligtas (walang third-party bot):
    
    1. Mag-DM sa iyong bot.
    2. Patakbuhin ang `openclaw logs --follow`.
    3. Basahin ang `from.id`.
    
    Opisyal na paraan gamit ang Bot API:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Paraan gamit ang third-party (mas hindi pribado): `@userinfobot` o `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    May dalawang magkahiwalay na kontrol:


    ```
    1. **Aling mga grupo ang pinapayagan** (`channels.telegram.groups`)
       - walang `groups` config: lahat ng grupo ay pinapayagan
       - may `groups` config: nagsisilbing allowlist (tahasan na mga ID o `"*"`)
    
    2. **Aling mga sender ang pinapayagan sa mga grupo** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (default)
       - `disabled`
    
    Ang `groupAllowFrom` ay ginagamit para sa pag-filter ng group sender. Kung hindi nakatakda, babalik ang Telegram sa `allowFrom`.
    Ang mga entry ng `groupAllowFrom` ay dapat numeric Telegram user IDs.
    
    Halimbawa: payagan ang sinumang miyembro sa isang partikular na grupo:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">
    Ang mga group reply ay nangangailangan ng mention bilang default.


    ```
    Maaaring manggaling ang mention mula sa:
    
    - native `@botusername` mention, o
    - mga mention pattern sa:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Mga command toggle sa antas ng session:
    
    - `/activation always`
    - `/activation mention`
    
    Ina-update lamang nito ang session state. Gamitin ang config para sa persistence.
    
    Halimbawa ng persistent config:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    ```
    Pagkuha ng group chat ID:
    
    - i-forward ang group message sa `@userinfobot` / `@getidsbot`
    - o basahin ang `chat.id` mula sa `openclaw logs --follow`
    - o siyasatin ang Bot API `getUpdates`
    ```

  
</Tab>
</Tabs>

## Pag-uugali sa runtime

- Ang Telegram ay pagmamay-ari ng gateway process.
- Deterministic ang routing: ang mga papasok na mensahe sa Telegram ay sasagutin din sa Telegram (hindi pumipili ng channel ang model).
- Ang mga papasok na mensahe ay ino-normalize sa shared channel envelope na may reply metadata at mga media placeholder.
- Ang mga group session ay nakahiwalay batay sa group ID. Ang mga forum topic ay nagdadagdag ng `:topic:<threadId>` upang mapanatiling hiwalay ang mga topic.
- Ang mga DM message ay maaaring maglaman ng `message_thread_id`; niruruta ng OpenClaw ang mga ito gamit ang thread-aware session keys at pinananatili ang thread ID para sa mga reply.
- Gumagamit ang long polling ng grammY runner na may sequencing bawat chat/bawat thread. Ang kabuuang runner sink concurrency ay gumagamit ng `agents.defaults.maxConcurrent`.
- Walang suporta ang Telegram Bot API para sa read-receipt (`sendReadReceipts` ay hindi nalalapat).

## Sanggunian ng mga feature

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">
    Maaaring mag-stream ang OpenClaw ng mga partial na sagot sa pamamagitan ng pagpapadala ng pansamantalang Telegram message at pag-edit nito habang dumarating ang teksto.


    ```
    Kinakailangan:
    
    - ang `channels.telegram.streamMode` ay hindi `"off"` (default: `"partial"`)
    
    Mga mode:
    
    - `off`: walang live preview
    - `partial`: madalas na preview update mula sa partial text
    - `block`: naka-chunk na preview update gamit ang `channels.telegram.draftChunk`
    
    Mga default ng `draftChunk` para sa `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    Ang `maxChars` ay nililimitahan ng `channels.telegram.textChunkLimit`.
    
    Gumagana ito sa direct chat at mga group/topic.
    
    Para sa text-only na sagot, pinananatili ng OpenClaw ang parehong preview message at nagsasagawa ng huling pag-edit sa parehong mensahe (walang pangalawang mensahe).
    
    Para sa mas kumplikadong sagot (halimbawa, may media payload), babalik ang OpenClaw sa normal na final delivery at pagkatapos ay lilinisin ang preview message.
    
    Ang `streamMode` ay hiwalay sa block streaming. Kapag tahasang pinagana ang block streaming para sa Telegram, nilalaktawan ng OpenClaw ang preview stream upang maiwasan ang dobleng streaming.
    
    Telegram-only reasoning stream:
    
    - `/reasoning stream` ay nagpapadala ng reasoning sa live preview habang nagge-generate
    - ang final na sagot ay ipinapadala nang walang reasoning text
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">
    Ang outbound text ay gumagamit ng Telegram `parse_mode: "HTML"`.


    ```
    - Ang tekstong parang Markdown ay nire-render sa Telegram-safe HTML.
    - Ang raw model HTML ay ini-escape upang mabawasan ang Telegram parse failures.
    - Kung tanggihan ng Telegram ang parsed HTML, muling susubukan ng OpenClaw bilang plain text.
    
    Ang link preview ay naka-enable bilang default at maaaring i-disable gamit ang `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Ang pagpaparehistro ng Telegram command menu ay pinangangasiwaan sa startup gamit ang `setMyCommands`.


    ```
    Mga default na native command:
    
    - `commands.native: "auto"` ay nag-e-enable ng native commands para sa Telegram
    
    Magdagdag ng custom na command menu entries:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    {
      channels: {
        telegram: {
          groups: {
            "-1001234567890": { requireMention: false }, // always respond in this group
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    I-configure ang inline keyboard scope:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Override kada account:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    Mga Scope:
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (default)
    
    Ang legacy na `capabilities: ["inlineButtons"]` ay tumutugma sa `inlineButtons: "all"`.
    
    Halimbawa ng message action:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Ang mga callback click ay ipinapasa sa agent bilang text:
    `callback_data: <value>`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Kasama sa mga Telegram tool action ang:

    ```
    - `sendMessage` (`to`, `content`, opsyonal `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Ang mga channel message action ay may ergonomic na mga alias (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Mga gating control:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (default: disabled)
    
    Semantika ng pagtanggal ng reaction: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">
    Sinusuportahan ng Telegram ang tahasang reply threading tags sa generated output:

    ```
    - `[[reply_to_current]]` ay nagre-reply sa nag-trigger na mensahe
    - `[[reply_to:<id>]]` ay nagre-reply sa partikular na Telegram message ID
    
    Kinokontrol ng `channels.telegram.replyToMode` ang paghawak:
    
    - `off` (default)
    - `first`
    - `all`
    
    Tandaan: Ang `off` ay nagdi-disable ng implicit reply threading. Ang tahasang `[[reply_to_*]]` tags ay patuloy na sinusunod.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    Mga forum supergroup:

    ```
    - ang topic session keys ay dinaragdagan ng `:topic:<threadId>`
    - ang mga reply at typing ay nakatuon sa topic thread
    - path ng topic config:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    Espesyal na kaso para sa general topic (`threadId=1`):
    
    - ang pagpapadala ng mensahe ay hindi isinasama ang `message_thread_id` (tinatanggihan ng Telegram ang `sendMessage(...thread_id=1)`)
    - ang typing actions ay patuloy na may kasamang `message_thread_id`
    
    Topic inheritance: ang mga topic entry ay nagmamana ng group settings maliban kung may override (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).
    
    Kasama sa template context ang:
    
    - `MessageThreadId`
    - `IsForum`
    
    Behavior ng DM thread:
    
    - ang mga private chat na may `message_thread_id` ay nananatili sa DM routing ngunit gumagamit ng thread-aware na session keys/reply targets.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Mga audio message

    ```
    Ipinagkaiba ng Telegram ang voice notes at audio files.
    
    - default: behavior ng audio file
    - tag na `[[audio_as_voice]]` sa reply ng agent upang pilitin ang pagpapadala bilang voice note
    
    Halimbawa ng message action:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### Mga video message
    
    Ipinagkaiba ng Telegram ang video files at video notes.
    
    Halimbawa ng message action:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Hindi sinusuportahan ng video notes ang captions; ang ibinigay na text ng mensahe ay ipinapadala nang hiwalay.
    
    ### Mga Sticker
    
    Paghawak sa inbound sticker:
    
    - static WEBP: dina-download at pinoproseso (placeholder `<media:sticker>`)
    - animated TGS: nilalaktawan
    - video WEBM: nilalaktawan
    
    Mga field ng sticker context:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Sticker cache file:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Ang mga sticker ay inilalarawan nang isang beses (kung maaari) at ini-cache upang mabawasan ang paulit-ulit na vision calls.
    
    I-enable ang mga sticker action:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Action para magpadala ng sticker:
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Maghanap ng mga naka-cache na sticker:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">
    Ang mga Telegram reaction ay dumarating bilang `message_reaction` updates (hiwalay sa mga message payload).

    ```
    Kapag naka-enable, nag-e-enqueue ang OpenClaw ng mga system event tulad ng:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Config:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (default: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (default: `minimal`)
    
    Mga Tala:
    
    - Ang `own` ay nangangahulugang mga reaction ng user sa mga mensaheng ipinadala ng bot lamang (best-effort gamit ang sent-message cache).
    - Hindi nagbibigay ang Telegram ng thread IDs sa reaction updates.
      - ang mga non-forum group ay niruruta sa group chat session
      - ang mga forum group ay niruruta sa group general-topic session (`:topic:1`), hindi sa eksaktong pinagmulan na topic
    
    Ang `allowed_updates` para sa polling/webhook ay awtomatikong kasama ang `message_reaction`.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    Ang `ackReaction` ay nagpapadala ng acknowledgement emoji habang pinoproseso ng OpenClaw ang isang inbound message.

    ```
    Order ng resolution:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - fallback na emoji ng agent identity (`agents.list[].identity.emoji`, kung wala ay "👀")
    
    Mga Tala:
    
    - Inaasahan ng Telegram ang unicode emoji (halimbawa "👀").
    - Gamitin ang `""` upang i-disable ang reaction para sa isang channel o account.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Ang pagsusulat ng channel config ay naka-enable bilang default (`configWrites !== false`).

    ```
    Kasama sa mga write na na-trigger ng Telegram ang:
    
    - mga group migration event (`migrate_to_chat_id`) upang i-update ang `channels.telegram.groups`
    - `/config set` at `/config unset` (nangangailangan ng command enablement)
    
    I-disable:
    ```

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    Default: long polling.

    ```
    Webhook mode:
    
    - itakda ang `channels.telegram.webhookUrl`
    - itakda ang `channels.telegram.webhookSecret` (kinakailangan kapag nakatakda ang webhook URL)
    - opsyonal `channels.telegram.webhookPath` (default `/telegram-webhook`)
    - opsyonal `channels.telegram.webhookHost` (default `127.0.0.1`)
    
    Ang default na local listener para sa webhook mode ay naka-bind sa `127.0.0.1:8787`.
    
    Kung iba ang iyong public endpoint, maglagay ng reverse proxy sa harap at ituro ang `webhookUrl` sa public URL.
    Itakda ang `webhookHost` (halimbawa `0.0.0.0`) kapag sadyang kailangan mo ng external ingress.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - ang `channels.telegram.textChunkLimit` ay may default na 4000.
    - ang `channels.telegram.chunkMode="newline"` ay inuuna ang paragraph boundaries (mga blank line) bago hatiin ayon sa haba.
    - ang `channels.telegram.mediaMaxMb` (default 5) ay naglilimita sa laki ng inbound Telegram media na dina-download/pinoproseso.
    - ang `channels.telegram.timeoutSeconds` ay nag-o-override sa timeout ng Telegram API client (kung hindi nakatakda, ginagamit ang grammY default).
    - ang group context history ay gumagamit ng `channels.telegram.historyLimit` o `messages.groupChat.historyLimit` (default 50); ang `0` ay nagdi-disable.
    - Mga kontrol sa DM history:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - ang outbound Telegram API retries ay maaaring i-configure sa pamamagitan ng `channels.telegram.retry`.

    ```
    Ang CLI send target ay maaaring numeric chat ID o username:
    ```

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  
</Accordion>
</AccordionGroup>

## Pag-troubleshoot

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - Kung `requireMention=false`, dapat payagan ng Telegram privacy mode ang buong visibility.
      - BotFather: `/setprivacy` -> Disable
      - pagkatapos ay alisin + idagdag muli ang bot sa group
    - Nagbibigay-babala ang `openclaw channels status` kapag inaasahan ng config ang mga group message na walang mention.
    - Maaaring suriin ng `openclaw channels status --probe` ang tahasang numeric group IDs; ang wildcard `"*"` ay hindi maaaring ma-probe para sa membership.
    - mabilisang session test: `/activation always`.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - kapag umiiral ang `channels.telegram.groups`, dapat nakalista ang group (o may kasamang `"*"`)
    - tiyaking miyembro ang bot sa group
    - suriin ang logs: `openclaw logs --follow` para sa mga dahilan ng pag-skip
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - i-authorize ang iyong sender identity (pairing at/o numeric `allowFrom`)
    - nananatiling umiiral ang command authorization kahit ang group policy ay `open`
    - ang `setMyCommands failed` ay karaniwang nagpapahiwatig ng mga isyu sa DNS/HTTPS reachability papuntang `api.telegram.org`
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Ang Node 22+ + custom fetch/proxy ay maaaring mag-trigger ng agarang abort behavior kung hindi tugma ang mga uri ng AbortSignal.
    - May ilang host na nireresolba muna ang `api.telegram.org` sa IPv6; ang sirang IPv6 egress ay maaaring magdulot ng pasulput-sulpot na Telegram API failures.
    - I-validate ang mga DNS answer:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Sinusuportahan ng Telegram ang opsyonal na threaded replies sa pamamagitan ng mga tag:

## Mga reference pointer ng config ng Telegram

Kinokontrol ng `channels.telegram.replyToMode`:

- `first` (default), `all`, `off`.

- `channels.telegram.botToken`: bot token (BotFather).

- `channels.telegram.tokenFile`: basahin ang token mula sa file path.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing).

- `channels.telegram.allowFrom`: allowlist ng DM (numeric Telegram user IDs). `open` ay nangangailangan ng `"*"`. Maaaring i-resolve ng `openclaw doctor --fix` ang mga legacy `@username` entry papunta sa mga ID.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (default: allowlist).

- `channels.telegram.groupAllowFrom`: allowlist ng sender sa group (numeric Telegram user IDs). Maaaring i-resolve ng `openclaw doctor --fix` ang mga legacy `@username` entry papunta sa mga ID.

- `channels.telegram.groups`: per-group na mga default + allowlist (gamitin ang `"*"` para sa global na mga default).
  - `channels.telegram.groups.<id>`.groupPolicy`: per-group override para sa groupPolicy (`open | allowlist | disabled\`).
  - `channels.telegram.groups.<id>`.requireMention\`: default na mention gating.
  - `channels.telegram.groups.<id>`.skills\`: skill filter (omit = lahat ng skills, empty = wala).
  - `channels.telegram.groups.<id>`.allowFrom\`: per-group override ng sender allowlist.
  - `channels.telegram.groups.<id>`.systemPrompt\`: karagdagang system prompt para sa grupo.
  - `channels.telegram.groups.<id>`.enabled`: i-disable ang grupo kapag `false\`.
  - `channels.telegram.groups.<id>`.topics.<threadId>.\*\`: per-topic overrides (kaparehong mga field gaya ng group).
  - `channels.telegram.groups.<id>`.topics.<threadId>`.groupPolicy`: per-topic override para sa groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>`.topics.<threadId>`.requireMention`: per-topic override ng mention gating.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (default: allowlist).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account override.

- `channels.telegram.replyToMode`: `off | first | all` (default: `off`).

- `channels.telegram.textChunkLimit`: outbound chunk size (chars).

- `channels.telegram.chunkMode`: `length` (default) o `newline` upang maghiwa sa mga blangkong linya (mga hangganan ng talata) bago ang paghiwa batay sa haba.

- `channels.telegram.linkPreview`: i-toggle ang link previews para sa mga palabas na mensahe (default: true).

- `channels.telegram.streamMode`: `off | partial | block` (live stream preview).

- `channels.telegram.mediaMaxMb`: inbound/outbound na limitasyon ng media (MB).

- `channels.telegram.retry`: retry policy para sa mga palabas na Telegram API call (attempts, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: override ng Node autoSelectFamily (true=enable, false=disable). Naka-disable bilang default sa Node 22 upang maiwasan ang Happy Eyeballs timeouts.

- `channels.telegram.proxy`: proxy URL para sa mga Bot API call (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: i-enable ang webhook mode (nangangailangan ng `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: webhook secret (kinakailangan kapag naka-set ang webhookUrl).

- `channels.telegram.webhookPath`: lokal na webhook path (default `/telegram-webhook`).

- `channels.telegram.webhookHost`: local webhook bind host (default `127.0.0.1`).

- `channels.telegram.actions.reactions`: i-gate ang mga reaksyon ng Telegram tool.

- `channels.telegram.actions.sendMessage`: i-gate ang pagpapadala ng mensahe ng Telegram tool.

- `channels.telegram.actions.deleteMessage`: i-gate ang pagtanggal ng mensahe ng Telegram tool.

- `channels.telegram.actions.sticker`: i-gate ang mga aksyon ng Telegram sticker — send at search (default: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — kontrolin kung aling mga reaksyon ang nagti-trigger ng mga system event (default: `own` kapag hindi naka-set).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — kontrolin ang kakayahan ng agent sa reaksyon (default: `minimal` kapag hindi naka-set).

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

Mga high-signal na field na partikular sa Telegram:

- startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- access control: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- command/menu: `commands.native`, `customCommands`
- threading/replies: `replyToMode`
- streaming: `streamMode` (preview), `draftChunk`, `blockStreaming`
- formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- actions/capabilities: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- reactions: `reactionNotifications`, `reactionLevel`
- writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Kaugnay

- `[[audio_as_voice]]` — ipadala ang audio bilang voice note sa halip na file.
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
